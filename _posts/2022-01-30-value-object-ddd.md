---
id: 205
guid: "https://overcoded.pl/?p=205"
permalink: /value-object-ddd/
title: "Value Objects"
date: "2022-01-30T16:32:51+01:00"
author: Carq
image: /assets/posts/value-objects-header.png
categories: [DDD]
tags: [ddd, "c#"]
---

Value Object jest najmniejszym z **_Building Blocks_** w Domain Driven Design (DDD). Value Objects pomagają pisać znacznie prostszy kod, który łatwo się testuje, modeluje, utrzymuje i jednocześnie jest niezwykle zrozumiały. W szczególności Value Objects upraszczają walidacje typów prostych (np. `int`, `string`) które wpuszczamy do systemu. Kolejną dużą zaletą jest prostota implementacji, która pozwala używać Value Objects w każdym projekcie bez konieczności wprowadzenia całego podejścia jakim jest DDD.

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości Value Object

- **Modeluje pojedynczą rzecz w domenie, którą da się zmierzyć, albo ją opisuje** – obiektami, które da się zmierzyć są np. temperatura (13°C) albo dystans (1337 metrów). Opisem czegoś w domenie może być imię osoby albo nazwa miasta.

- **Niezmienny (Immutable)** – po utworzeniu takiego obiektu nie można go modyfikować (prywatne metody lub właściwości do ustawiania wartości są używane wyłącznie przez konstruktor). Jeżeli potrzebujemy „ustawić stan” to możemy pokusić o stworzenie metody, która zwraca nowo utworzony Value Object ze zmienioną wartością.

- **Stanowi integralną całość** (**whole object**) – Value Object może składać się z kilku wartości, które razem są spójne, np. Temperatura składa się z dwóch właściwości 36.6 i jednostki Celsjusz, razem są spójne i dostarczają nam konkretną informacje o temperaturze. Osobno te wartości mogą opisywać cokolwiek i nie mówią o kontekście ich użycia.

- **Zastępowalny** – Jeżeli chcemy zmienić wartość lub opis czegoś, zastępujemy ValueObject innym ValueObject o tym samym typie i nie powoduje to efektów ubocznych (Side-Effects). **_Zastępujemy_**, a nie zmieniamy ValueObject.

```csharp
Temperature currentTemperature = new Temperature(36.6, "°C");
// currentTemperature.ChangeTemperature(35.1) - Źle! Nie modyfikujemy ValueObjectów.
currentTemperature = new Temperature(37.2, "°C") // Zastępujemy

```

- **Porównywalność** – ValueObjecty są porównywalne do innych instancji o tym samym typie:

```csharp
new Temperature(10, "°C") == new Temparature(10, "°C") // true
new Temperature(10, "°C") == new Temparature(10, "°F") // false
```

- **Brak efektów ubocznych (Side-effects free)** – metody ValueObjecta mogą zwrócić wynik, ale nie modyfikują stanu ValueObjectu, nie robi nic poza zwróceniem wyniku, czyli nie ma efektów ubocznych.

```csharp
Temperature currentTemperature = new Temperature(36.6, "°C");
currentTemperature = currentTemperature.Increase(2.1) // Metoda zwraca wynik bez modyfikowania stanu pierwotnej instacji
```

### Co może być ValueObjectem

Używamy ValueObjectów kiedy interesują nas wyłącznie właściwości (properties) danego obiektu, a nie jego tożsamość (identity). Czyli ValueObjectem nie zrobiliśmy obiektu **_Person_**, bo pomimo tego, że dwie osoby mogą mieć to samo imię i nazwisko, to i tak dla nas jest ważniejsze to, że to są dwie różne osoby.

### Jak używać

- 🎭Nadawać wymowne nazwy dla samego ValueObjecta jak i dla jego właściwości
- 🛡️ValueObject posiada metody, które walidują samego siebie w konstruktorze, przez co nie jest możliwe stworzenie ValueObject w niepoprawnym stanie, np. obiekt dystans nie może mieć ujemnej wartości
- ValueObject może się składać z kilku typów prostych i innych ValueObjectów
- Pojedynczy, prosty typ może być opakowany ValueObjectem, np. dystans albo Id jest pojedynczą wartością, która nie może by ujemna. Jeżeli zrobimy z niej ValueObjecta i zaczniemy używać go w całym systemie to nie musimy za każdym razem sprawdzać czy wartość jest poprawna, upraszcza to sporą część naszego kodu. Tutaj jednak należy uważać, aby nie przesadzić z ilością ValueObjectów dla pojedynczych wartości, bo może skończyć się to tym, że w wielu miejscach będziemy mieli rzutowanie z typu prostego na ValueObject i w drugą stronę
- 🤝🏼ValueObject może być współdzielony przez wiele bounded contextów (projektów 😉)
- 🍝Unikać wewnątrz ValueObjectu skomplikowanej logiki, która jest potrzebna do jego utrzymywania

### Implementacja

Użyjemy wzorca [**SuperType**](https://www.martinfowler.com/eaaCatalog/layerSupertype.html), aby każdy ValueObject nie musiał powielać kodu związanego z operatorami porównania.

```csharp
public abstract class ValueObject
{
    public static bool operator ==(ValueObject a, ValueObject b)
    {
        if (ReferenceEquals(a, null) && ReferenceEquals(b, null))
        {
            return true;
        }

        if (ReferenceEquals(a, null) || ReferenceEquals(b, null))
        {
            return false;
        }

        return a.Equals(b);
    }

    public static bool operator !=(ValueObject a, ValueObject b)
    {
        return !(a == b);
    }

    public override bool Equals(object obj)
    {
        if (obj == null || GetType() != obj.GetType())
        {
            return false;
        }

        var valueObject = (ValueObject)obj;
        return GetAtomicValues().SequenceEqual(valueObject.GetAtomicValues());
    }

    public override int GetHashCode()
    {
        return GetAtomicValues().Aggregate(1, HashCode.Combine);
    }

    protected abstract IEnumerable<object> GetAtomicValues();

}
```

Metoda **_GetAtomicValues()_** wymusza, aby klasy pochodne podawały wartości, po których ValueObject ma być porównywalny z innymi instancjami ValueObjecta.

Poniżej przykładowa implementacja ValueObject **_Temperature_**:

```csharp
public class Temperature : ValueObject
{
    public Temperature(decimal value, string unit)
    {
        Value = value;
        SetUnit(unit);
    }

    public decimal Value { get; private set; }

    public string Unit { get; private set; }

    private void SetUnit(string unit)
    {
        if (unit != "°C" && unit != "°F")
        {
            throw new ArgumentException("Given unit is not supported");
        }

        Unit = unit;
    }

    protected override IEnumerable<object> GetAtomicValues()
    {
        yield return Value;
        yield return Unit;
    }
}
```

Powyższy przykład przedstawia obiekt Temperature. Obiekt ten jest Immutable, posiada wewnętrzną walidację wartości, przez co nie można go stworzyć w niepoprawnym stanie i przez nadpisanie metody **_ValueObject_**.**_GetAtomicValues()_** można porównywać różne instancje pomiędzy sobą. Jeżeli różne instancje mają te same wartości to operator porównania (==) zwróci **_true_**.

#### Podsumowanie

ValueObject to mały, ale potężny building block z DDD, który małym kosztem pozwala na uproszczenie naszej aplikacji, oraz zmniejsza szansę powstania błędu. Można go z powodzeniem stosować bez wdrażania czy też rozumienia reszty DDD. Value Objecty obrazują też najważniejsze cechy programowania obiektowego, takie jak enkapsulacja (inaczej hermetyzacja).
