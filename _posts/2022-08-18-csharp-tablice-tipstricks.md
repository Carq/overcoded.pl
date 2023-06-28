---
id: 638
title: "Tablice w C# &#8211; Tips&#038;Tricks"
date: "2022-08-18T07:00:00+02:00"
guid: "https://overcoded.pl/?p=638"
permalink: /csharp-tablice-tipstricks/
author: Carq
excerpt: "Oto kilka pożytecznych wskazówek i trików, które mogą się przydać przy pracy z tablicami w C# i .NET."
categories: ["C#"]
image: /assets/posts/csharp-tablice-tipstricks-header.png
tags: [.net, "c#", performance, tips, tricks]
---

Oto kilka pożytecznych wskazówek i trików, które mogą się przydać przy pracy z tablicami w C# i .NET😉

#### Inicjalizacja tablic

```csharp
// Standardowy sposób
var intArray1 = new int[1] { 1337 };

// Bez podawania rozmiaru tablicy
var intArray2 = new int[] { 1337 };

// Bez podawania typu przy przypisywaniu 😎
int[] intArray3 = { 2, 3 };
```

#### Tworzenie pustej tablicy a wydajność?

```csharp
// Standardowy sposób
var badEmptyArray = new int[0];

// Z użyciem klasy Array - tworzy pustą tablicę o około 33 razy szybciej niż poprzedni sposób!
var niceEmptyArray = Array.Empty<int>();
```

Użycie `Array.Empty<T>` jest szybsze od zwykłej inicjalizacji tablicy o około 33 razy! 😯 Dlaczego tak jest? `Array.Empty<T>` nie powoduje rezerwacji pamięci. Poniżej zrzut ekranu z pomiaru wykonanego przy pomocy [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet), kodzik [tutaj](https://github.com/Carq/PerformanceLab/blob/master/EmptyArray/EmptyArrayBenchmark.cs). Zwróćcie uwagę na kolumny **Mean** i **Allocated** — ostatnia mówi, ile pamięci zostało zarezerwowane:

![](/assets/posts/arrays_01.png)

#### Tworzenie tablic z predefiniowanymi elementami

```csharp
// Tworzenie tablicy z określanym rozmiarem i tymi samymi elementami
var threeFives = Enumerable.Repeat(element: 5, count: 3).ToArray();
// Wynik: [5, 5, 5]

// Tworzenie tablicy z określanym rozmiarem i różnymi elementami
var threeDevelopers = Enumerable.Range(start: 0, count: 3).Select(i => $"Dev number {i + 1}").ToArray();
// Wynik: ["Dev number 1", "Dev number 2", "Dev number 3"]
```

Taki sposób inicjalizacji tablic/list szczególnie przydaje się w testach jednostkowych, gdzie często musimy przygotować odpowiednie dane testowe, np. w formie listy obiektów.

Polecam zapoznać się z innymi metodami klasy [Enumerable](https://docs.microsoft.com/pl-pl/dotnet/api/system.linq.enumerable?view=net-6.0#methods), takimi jak:

- **Chunk** – dzielenie kolekcji na równe kawałki
- **Prepend** – dodawanie elementów na początek kolekcji

#### Operacje na tablicach przy pomocy [Index ](https://docs.microsoft.com/pl-pl/dotnet/api/system.index?view=net-6.0)i [Range](https://docs.microsoft.com/en-us/dotnet/api/system.range?view=net-6.0)

```csharp
int[] array = { 1, 2, 3, 4, 5 };

// Zwrócenie pierwszego elementu — standardowo
int firstElement = array[0];
// 1

// Zwrócenie elementów z tablicy przy użyciu Index [1^]
// Ostatni element — ekwiwalent to array[array.Length - 1]
int lastElement = array[^1];
// 5

// Czwarty od końca
int fourthFromLast = array[^4];
// 2

// Wycięcie kawałka tablicy przy pomocy Range [..]
// Od 3 elementu do ostatniego
int[] slice1 = array[2..];
// [3, 4, 5]

// Od początku do 2 elementu
int[] slice2 = array[..2];
// [1, 2]

// Od 2 do 3 elementu tablicy
int[] slice3 = array[1..3]
// [2, 3]

// Od 3 elementu od końca do końca tablicy 🤯
int[] slice4 = array[^3..];
// [3, 4, 5]
```

Index (`[^]`) i Range (`[..]`) są strukturami wprowadzonymi razem z C# 8. Posiadają ciekawą składnię i ułatwiają pracę na tablicach.

#### Wydajna praca z dużymi tablicami

Gdy nasz kod wymaga częstych operacji na dużych tablicach, to wydajność naszego rozwiązania może okazać się kluczowa. Może nam w tym pomóc specjalnie do tego stworzona klasa **[ArrayPool&lt;T&gt;](https://docs.microsoft.com/pl-pl/dotnet/api/system.buffers.arraypool-1?view=net-6.0)** — nie wgłębiając się w szczegóły, pomaga ona lepiej zarządzać alokacją pamięci tablic, przez co znacząco wpływa na wydajność. Może napiszę osobny, szczegółowy wpis na ten temat, bylibyście zainteresowani? Tymczasem krótki przykład jak używać `ArrayPool`:

```csharp
// Stworzenie klasy do zarządzania pulami tablic
ArrayPool<int> pool = ArrayPool<int>.Create();

// Wyciągnięcie z pamięci tablicy, która już wcześniej mogła zostać stworzona
int[] renterArray = pool.Rent(minimumLength: 10);
for (int i = 0; i < 10; i++)
{
Console.WriteLine(renterArray[i]);
}

// Zwrócenie tablicy do puli, aby mogła zostać wykorzystana ponownie
pool.Return(renterArray);
```

Mam nadzieję, że tych kilka wskazówek okaże się przydatne w Waszych codziennych programistycznych zadaniach 🙂

**A czy Wy znacie jakieś przydatne wskazówki przy pracy na tablicach? 🤔**
