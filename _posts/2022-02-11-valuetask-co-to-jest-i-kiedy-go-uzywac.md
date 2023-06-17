---
id: 295
guid: "https://overcoded.pl/?p=295"
permalink: /valuetask-co-to-jest-i-kiedy-go-uzywac/
title: "ValueTask - co to jest i kiedy go używać"
date: "2022-02-11T17:50:26+01:00"
author: Carq
categories: ["C#"]
tags: [.net, "c#", performance, task, ValueTask]
---

Z C# 7 został wprowadzony nowy typ a konkretniej struktura [**ValueTask**](https://docs.microsoft.com/pl-pl/dotnet/api/system.threading.tasks.valuetask-1?view=net-6.0) która jest odpowiednikiem klasy [**Task**](https://docs.microsoft.com/pl-pl/dotnet/api/system.threading.tasks.task-1?view=net-6.0). Głównym celem tej struktury jest poprawa wydajności metod asynchronicznych w których ścieżka synchroniczna jest dużo częściej wykonywana niż asynchroniczna 🤔 Spójrzmy na poniższy kod:

```csharp
public async Task<IList<HolidayDetailsDto>> GetHolidaysTask()
{
    if (_cachedTaskData == null)
    {
        _cachedTaskData = await _httpClient.GetFromJsonAsync<IList<HolidayDetailsDto>>("https://date.nager.at/api/v2/publicholidays/2022/PL");
    }

    return _cachedTaskData;

}
```

Kod jest prosty, jeżeli nie mamy danych w cachu to pobieramy je, jeżeli są to od razu je zwracamy. Czyli kod synchroniczny – zwracanie danych z cacha, będzie dużo częściej wykonywany niż kod asynchroniczny – pobieranie danych z jakiegoś serwera. Poniżej kod który używa już ValueTask:

```csharp
public async ValueTask<IList<HolidayDetailsDto>> GetHolidaysValueTask()
{
    if (_cachedValueTaskData == null)
    {
        _cachedValueTaskData = await _httpClient.GetFromJsonAsync<IList<HolidayDetailsDto>>("https://date.nager.at/api/v2/publicholidays/2022/PL");
    }

    return _cachedValueTaskData;

}
```

Jedyna różnica to tylko zwracany typ. Jak to ma się do wydajności? ⤵️

### ValueTask vs Task – wydajność

Za pomocą poniższego kodu porównałem wydajność Task i ValueTask. Wydajność mierzyłem za pomocą [](https://github.com/dotnet/BenchmarkDotNet)[BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet).

```csharp
[Benchmark]
public async ValueTask ValueTask()
{
    for (int i = 0; i < 5000; i++)
    {
        var result = await GetHolidaysValueTask();
    }
}

[Benchmark]
public async Task Task()
{
    for (int i = 0; i < 5000; i++)
    {
        var result = await GetHolidaysTask();
    }
}
```

**Wynik:**
![](assets/posts/value_task_01.png)
Jak widzimy, szybkość działania metod jest bardzo podobna jednak użycie pamięci (kolumna Allocated) jest dużo mniejsze gdy używamy ValueTask.

Kod powyższego benchmarku znajdziecie tutaj: <https://github.com/Carq/PerformanceLab/tree/master/ValueTaskVsTask>

### Podsumowanie – kiedy używać

**_ValueTask_** należy używać gdy w aplikacji korzystamy z metod asynchronicznych w których ścieżka synchroniczna jest dużo częściej wykonywana niż asynchroniczna. Używając ValueTask możemy zaoszczędzić prawie połowę pamięci.

#### Polecane linki

- [Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/)
- [C# Language Highlights ValueTask](https://www.youtube.com/watch?v=IN4dRdKlISI)
