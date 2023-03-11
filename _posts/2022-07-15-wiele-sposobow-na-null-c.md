---
id: 487
title: "Wiele sposobów na null C#"
date: "2022-07-15T22:47:10+02:00"
author: Carq
excerpt: "W .NET jest kilka sposobów aby sprawdzić czy obiekt lub parametr jest null."
categories: ["C#"]
tags: [.net, "c#"]
---

Sprawdzanie, czy obiekt lub przekazywany parametr nie jest null, jest najczęściej powtarzanym kawałkiem kodu. C# oferuje kilka sposobów, aby to osiągnąć. Ten wpis ma na celu Wam je przybliżyć.

### 6 sposobów na sprawdzenie, czy obiekt jest null

Często w naszych aplikacjach chcemy wykonać jakiś kod tylko wtedy, kiedy jakiś obiekt nie jest null, np. wywołać metodę ze sprawdzanego obiektu. Poniżej przedstawiam 6 różnych sposobów na osiągnięcie tego.

```csharp
Overcoded overcoded = GetOvercoded();

// 1. Operator porównania
if (overcoded != null) overcoded.Code("1.");

// 2. Null propagator - dostępne od wersji C# 6
overcoded?.Code("2.");

// 3. Z użyciem keyword 'is' - dostępne od wersji C# 7
if (!(overcoded is null)) overcoded.Code("3.");

// 4. Można też sprawdzić czy obiekt jest obiektem ;)
if (overcoded is object) overcoded.Code("4.");

// 5. Czy obiekt jest obiektem w inny sposób - trochę haxorski 8). Od C# 8
if (overcoded is { }) overcoded.Code("5.");

// 6. Pattern matching 'is not'. Od C# 9
if (overcoded is not null) overcoded.Code("6.");

```

Jak widzimy, prawie każda nowa wersja C# dodawała kolejny wariant sprawdzania, czy coś jest null. Który sposób jest Waszym ulubiony?

### Rzucanie wyjątku, gdy przekazany parament jest null

Kolejną częstą sytuacją jest sprawdzanie, czy podczas tworzenia obiektu przekazywane parametry nie są null. W przeciwnym przypadku chcemy nie dopuścić do stworzenia takiego obiektu i rzucić wyjątek (np. [ArgumentNullException](https://docs.microsoft.com/pl-pl/dotnet/api/system.argumentnullexception?view=net-6.0)).

```csharp
public class Overcoded
{
    public string Name { get; private set; }

    // 1. Standardowe sprawdzenie
    public void UpdateName1(string name)
    {
        if (name == null) throw new ArgumentNullException("name");
        Name = name;
    }

    // 2. null-coalescing - C# 7.3
    public void UpdateName2(string name)
    {
        Name = name ?? throw new ArgumentNullException("name");
    }

    // 3. C# 10 i .NET 6.0
    public void UpdateName3(string name)
    {
        ArgumentNullException.ThrowIfNull(name);
        Name = name;
    }

    // 4. Metoda rzuci ArgumentNullException, jeżeli name będzie null. C# 11 - preview
    public void UpdateName4(string name!!)
    {
        Name = name;
    }

}
```

Ciekawie wygląda propozycja numer 4 z użyciem operatora `!!` (dwa wykrzykniki). **_Na razie operator ten jest dostępny w wersji C# 11 preview i nie wiadomo, czy znajdzie się w ostatecznej wersji_**, ponieważ wielu osobom ze społeczności taki zapis się nie podoba. 🤔

### Podsumowanie

Język C# w ostatnich latach dynamicznie się rozwija i zaczyna oferować wiele nowych możliwości, o których warto widzieć. Wliczając w to trywialne sprawdzenie, czy obiekt jest null. 😉
