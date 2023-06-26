---
id: 450
guid: "https://overcoded.pl/?p=450"
permalink: /factory-ddd/
title: "Factory, metody wytwórcza &#8211; DDD"
date: "2022-06-11T14:22:21+02:00"
author: Carq
image: /assets/posts/factory-ddd-header.png
categories: [DDD]
tags: [ddd, "c#", .net]
---

Factory (fabryka) jest dobrze znanym wzorcem projektowym, który występuje też w DDD jako część Tactical Design. Głównym celem fabryk, albo mówiąc inaczej metod wytwórczych jest uproszczenie tworzenia skomplikowanych obiektów.

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości fabryki

- **Tworzenie instancji złożonych obiektów** – [Agreg](/agregat-ddd/)[at](/agregat-ddd/), [Encja](/encje-podstawy-ddd/) lub [Value Objects](/value-objects/) może posiadać np. statyczną metodę wytwórczą, która upraszcza tworzenie instancji obiektów domenowych posiadających np. konstruktory z wieloma parametrami. Przykład będzie poniżej ⤵️

- **Nazwy metod odwzorowują domenowy język** – konstruktory mówią zazwyczaj jakie dane są wymagane do utworzenia obiektu, a nie co konkretnie powstanie np. mając klasę **Car**, która przyjmuje jako parametry ilość koni mechanicznych i prędkość maksymalną, to przy użyciu samego konstruktora nie dowiemy się jaki samochód tworzymy, ale jeżeli mielibyśmy metodę **_CreateFerrari_**, która by tworzyła obiekt Car z odpowiednimi parametrami, to kod byłby dużo bardziej czytelny. Z czasem ilość parametrów w konstruktorze może urosnąć lub też ilość samych konstruktorów się może zwiększyć przez co kod może być cięższy do zrozumienia. Używając dobrze nazwanych metod wytwórczych możemy zwiększyć czytelność naszego kodu. Przykład będzie poniżej ⤵️

- **Fabryka może być osobną klasą, lub częścią istniejącego już obiektu domenowego** – metody fabryk mogą być umieszczone w kompletnie nowych klasach, które są odpowiedzialne tylko za tworzenie instancji lub też po prostu być częścią [Agregatów](/agregat-ddd/), [Encji ](/encje-podstawy-ddd/)lub [Value Objectów](/value-objects/).

- **Metoda wytwórcza na agregacie może tworzyć inny agregat** – przy tworzeniu agregatów czasem są potrzebne dane z innego obiektu. Aby uprościć ten proces agregat może mieć metodę wytwórczą, która tworzy na podstawie własnych właściwości agregat innego typu.

- **Pomagają unikać błędów przy tworzeniu obiektów** – metoda wytwórcza na [Agregacie](/agregat-ddd/) tworzy obiekt, który od razu ma przypisanego poprawnego użytkownika, klienta (jest to dosyć istotne jeżeli nasza aplikacja wspiera [multitenancy](https://en.wikipedia.org/wiki/Multitenancy)) lub cokolwiek innego.

## Przykładowa implementacja

##### Statyczne metody wytwórcze

Mamy agregat **Invoice**, który w naszej uproszczonej domenie reprezentuje fakturę. Faktura zawiera informacje o kwocie netto do zapłaty (**_NetAmount_**), informacje o podatku VAT (pole **_VAT_**) i numer faktury. Załóżmy, że w naszej domenie wystawiamy tylko faktury z 3 rodzajami opodatkowania VAT, faktury od dóbr luksusowych z podatkiem VAT 23%, faktury od usług z podatkiem 12% i w pewnych przypadkach faktury, gdzie podatek VAT się nie należy:

```csharp
public class Invoice
{
    private Invoice(InvoiceNumber number, decimal netAmount, decimal? vat)
    {
        Number = number ?? throw new ArgumentNullException(nameof(number));
        NetAmount = netAmount;
        Vat = vat;
    }

    public InvoiceNumber Number { get; private set; }

    public decimal NetAmount { get; private set; }

    public decimal? Vat { get; set; }

    public static Invoice LuxuryGoods(decimal netAmount)
    {
        return new Invoice(InvoiceNumber.Generate(), netAmount, netAmount * 0.23m);
    }

    public static Invoice Services(decimal netAmount)
    {
        return new Invoice(InvoiceNumber.Generate(), netAmount, netAmount * 0.12m);
    }

    public static Invoice WithoutTaxes(decimal netAmount)
    {
        return new Invoice(InvoiceNumber.Generate(), netAmount, null);
    }

}
```

W powyższym przypadku użyliśmy statycznych metod wytwórczych, które zawierają nazwy domenowe i mają zahardcodowane wartości podatków VAT. Oznaczyliśmy konstruktor jako **_private_**, dzięki takiemu zabiegowi nikt nie użyje konstruktora i nie będzie musiał się domyślać jakich wartości powinien użyć. Zamiast tego mamy do dyspozycji metody wytwórcze, które nazwą oddają intencje, czyli np. stworzenie faktury od dóbr luksusowych. W ten sposób minimalizujemy szanse na wystąpienie błędu z użyciem klasy, którą zamodelowaliśmy.

##### Agregat tworzący inny obiekt domenowy

Powiedzmy, że mamy klasę **PaymentReminder**, która jest odpowiedzialna za wysyłanie przypomnień odnośnie niezapłaconej faktury. Klasa ta wymagałaby informacji o numerze faktury i kwocie do spłaty. W takim przypadku agregat **Invoice** mógłby posiadać metodę wytwórczą, która tworzy właśnie instancje **PaymentReminder** przy użyciu własnych danych. Dzięki temu tworzenie obiektu upraszcza się i minimalizujemy szanse na pomyłki.

```csharp
public class Invoice
{
    // Other stuff...

    public InvoiceNumber Number { get; private set; }

    public decimal NetAmount { get; private set; }

    public decimal? Vat { get; set; }

    public PaymentReminder PaymentReminder(DateTime remindedOn)
    {
        return new PaymentReminder(Number, NetAmount + Vat.GetValueOrDefault(), remindedOn);
    }

}

```

## Podsumowanie

Fabryki to kolejny popularny wzorzec projektowy używany w DDD. Dzięki niemu możemy uprościć tworzenie złożonych obiektów domenowych, zminimalizować szansę występowania błędu i jednocześnie nadać domenowe nazwy procesowi tworzenia obiektów.
