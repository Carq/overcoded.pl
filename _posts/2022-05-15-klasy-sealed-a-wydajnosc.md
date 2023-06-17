---
id: 450
guid: "https://overcoded.pl/?p=407"
permalink: "/klasy-sealed-a-wydajnosc/"
title: "Klasy sealed a wydajność"
date: "2022-05-15T17:56:44+01:00"
author: Carq
excerpt: "W .NET można oznaczyć klasy słowem kluczowym sealed co powoduje, że poprawia się wydajność wywołań metod wirtualnych 😲."
categories: ["C#"]
tags: [.net, "c#", notka, peformance]
---

W .NET można oznaczyć klasy słowem kluczowym sealed co powoduje, że po klasie nie można dziedziczyć. Poza tą właściwością, sealed ma jeszcze jedną ważna zaletę - poprawia wydajność wywołań metod wirtualnych 😲.

### sealed vs non-sealed - Wydajność

Spójrzmy na prosty kodzik:

```csharp
public class BaseClass
{
    public virtual int Method() => 5;
}

public class NonSealedClass : BaseClass
{
    public override int Method() => 22;
}

public sealed class SealedClass : BaseClass
{
    public override int Method() => 11;
}
```

Dwie klasy dziedziczą po BaseClass i jedyna różnica pomiędzy nimi to słówko kluczowe sealed. Sprawdźmy teraz różnice w wydajności przy pomocy [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) i następującego kodu:

```csharp
public class SealedBenchmark
{
    SealedClass sealedClass = new();

    NonSealedClass nonSealedClass = new();

    [Benchmark]
    public int SealedClass()
    {
        return sealedClass.Method() + 1337;
    }

    [Benchmark]
    public int NonSealedClass()
    {
        return nonSealedClass.Method() + 1337;
    }
}
```

Wynik:  
![](/assets/posts/sealed_01.png)

Wywołanie metody wirtualnej z SealedClass jest w tym przypadku około **162 razy szybsze** ⚡ niż dla klasy która nie jest sealed.

### Dlaczego tak się dzieje

Za tym wszystkim stoi tzw. [Virtual method table](https://en.wikipedia.org/wiki/Virtual_method_table) - upraszczając: podczas tworzenia instancji klasy, która ma metody wirtualne, kompilator dodaje tablice z metodami wirtualnymi. Tablice te są używana podczas runtime, aby określić która metoda ma być faktycznie użyta (z klasy bazowej czy z klas pochodnych), bo w czasie kompilacji kompilator może jeszcze tego nie widzieć. Natomiast jeżeli klasa jest zapieczętowana (`sealed`) to runtime dokładnie wie którą metodę użyć i nie korzysta już z `Virtual method table`.

Oczywiście są sytuacje w których kompilator wie, że dana instancja klasy jest konkretnego typu i wtedy już na poziomie kompilacji potrafi określić która metoda powinna być użyta, wtedy nie ma różnicy w wydajności pomiędzy `sealed` a `nonSealed`, przykład:

```csharp
public class SealedBenchmark
{
    [Benchmark]
    public int SealedClassDirectUse()
    {
        SealedClass sealedClass = new();
        return sealedClass.Method() + 1337;
    }

    [Benchmark]
    public int NonSealedClassDirectUse()
    {
        NonSealedClass nonSealedClass = new();
        // Kompilator widzi linijke wyżej inicjalizacje klasa NonSealedClass i dlatego wie, że
        // ma wywołać metode wirtualną dokładnie z tej klasy.
        return nonSealedClass.Method() + 1337;
    }
}
```

Wynik:  
![](/assets/posts/sealed_02.png)

### Rzutowanie też jest szybsze

Tym razem bez rozpisywania się. Kodzik i wynik:

```csharp
public class SealedCastBenchmark
{
    BaseClass baseClass = new();

    [Benchmark]
    public bool SealedClass()
    {
        return baseClass is SealedClass;
    }

    [Benchmark]
    public bool NonSealedClass()
    {
        return baseClass is NonSealedClass;
    }
}
```

![](/assets/posts/sealed_03.png)

### Podsumowanie

Oznaczenie klas słówkiem sealed jest bardzo tanim sposobem na optymalizacje naszego kodu. Oczywiście, różnice są w nanosekundach i można powiedzieć, że ostatecznie to nie zrobi większej różnicy na ogólnej wydajności naszej aplikacji, ale chyba lepiej żeby działała trochę szybciej niż trochę wolniej? 😉Tym bardziej, że w samym kodzie dotneta zachodzą takie zmiany: https://github.com/dotnet/runtime/pull/50225 - setki klas zostało oznaczone jako sealed.

#### Pomocne linki

- [Kod z powyższych benchamarków - Carq/PerformanceLab/tree/master/SealedClasses](https://github.com/Carq/PerformanceLab/tree/master/SealedClasses)
- [Dotnet - performance improvements in NET 6](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-6/#peanut-butter)
- [Wiki - Virtual method table](https://en.wikipedia.org/wiki/Virtual_method_table)
