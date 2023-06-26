---
id: 410
guid: "https://overcoded.pl/?p=410"
permalink: /repozytorium-ddd/
title: "Repozytorium &#8211; DDD"
date: "2022-04-03T15:11:28+02:00"
author: Carq
excerpt: "Repozytorium jest powszechnie znanym wzorcem projektowym do separacji technologii zapisu od kodu biznesowego. Używany jest również w DDD, trzeba jednak pamiętać o kilku zasadach."
categories: [DDD]
tags: [ddd, "c#", .net]
---

Repozytorium ([Repository](https://martinfowler.com/eaaCatalog/repository.html)) jest ogólnie znanym wzorcem, którego rolą jest zapisywanie i pobieranie obiektów z bazy danych lub innego miejsca gdzie możemy utrwalić dane. Uzupełniając ten wzorzec o kilka dodatkowych właściwości dostaniemy **_Building Blocks_** z Domain Driven Design (DDD).

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości Repozytorium

- **Operują na [Agregatach](/agregat-ddd/)** – jest to najważniejsza właściwość repozytorium – pobiera lub zapisuje wyłącznie agregaty wraz z połączonymi encjami (aczkolwiek są pewne wyjątki od tego 😉, o tym poniżej). Powinniśmy dążyć do tego, aby wyciągać całe agregaty, a nie tylko jego części.

- **Repozytoria mogą być używane przez Agregaty** – do pobierania dzieci (np. Lazy Loading w przypadku dużych agregatów) lub do pobierania potrzebnych danych z innych agregatów – są to wyjątki do powyższej zasady.

- **Jedno Repozytorium na jeden agregat** – Każdy agregat powinien mieć swoje repozytorium, za pomocą którego można go zapisać lub pobrać.

- **Posiada metody dodające i usuwające agregaty** – czyli po prostu metody do dodawania nowych agregatów – **Add**, usuwania już istniejących – **Remove**. Ewentualnie metody pomocnicze np. **Exists** – zwracająca informacje czy agregat istnieje.

- **Posiada metody pozwalające na wyszukiwanie Agregatu** – można to zrealizować na dwa sposoby: albo tworząc po prostu osobne metody dla każdego kryterium wyszukiwania, albo tworzyć osobny obiekt, zwany Specyfikacją (więcej o Specyfikacjach będzie w osobnym wpisie) i przekazywać go do pojedynczej metody w repozytorium, która go przyjmuje, przykłady:

```csharp
public interface ICalendarRepository
{
    // 1. sposób: Jedna metoda na jedno kryterium wyszukiwania
    Calendar FindGetById(int calendarId);
    Calendar FindByUserId(int userId);

    // 2. sposób: użycie Specyfikacji
    Calendar Find(ISpecification<Calendar> specification);

}

// użycie Specyfikacji
calendarRepository.Find(CalendarSpecifications.ForUser(1));
calendarRepository.Find(CalendarSpecifications.EmptyCalendar);

```

**Complex Query** – Repozytorium może zebrać dane z wielu agregatów (np. dla jakiegoś raportu). Wykorzystujemy do tego **Use Case Optimal Query** czyli złożonego query na warstwie persystencji bazy, dynamicznie zamieniające wynik query na [ValueObject ](/value-objects/)– specjalnie stworzony dla takich danych.

### Podsumowanie

Repozytorium jest powszechnie znanym wzorcem projektowym do separacji technologii zapisu od kodu biznesowego. Używany jest również w DDD, trzeba jednak pamiętać o powyższych zasadach ⤴️
