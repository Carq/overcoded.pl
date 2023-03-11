---
title: "Agregat - DDD"
date: "2022-03-05T12:46:54+01:00"
author: Carq
excerpt: "Agregat (Aggregate) jest głównym Building Blocks\_w Domain Driven Design (DDD). Grupuje on powiązane biznesowo ze sobą Encje i Value Objects, które można traktować jako pojedynczy obiekt. Agregat zapewnia trakcyjność operacji biznesowych w systemie."
categories: [DDD]
tags: [.net, "c#", ddd]
---

Agregat (Aggregate) jest głównym **_Building Blocks_** w Domain Driven Design (DDD). Grupuje on powiązane biznesowo ze sobą [Encje](/posts/encje-podstawy-ddd/) i [Value Objects](/posts/value-objects/), które można traktować jako pojedynczy obiekt. Agregat zapewnia transakcyjności operacji biznesowych w systemie.

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości Agregatu

- **Posiada Encje i Value Objecty** – posiada powiązane biznesowo Encje i Value Objecty.

- **Posiada Encję, która jest korzeniem agregatu (Aggregate root)** – inaczej mówiąc wszystkie powiązane Encje i ValueObjecty mają jednego rodzica – Aggregate Root. Jeżeli usuniemy rodzica to usuwamy również wszystkie powiązane obiekty. Bez Aggregate Root ich istnienie nie ma sensu. Np. możemy mieć agregat **Calendar**, która posiada kolekcję **CalendarEvents**, jeżeli usuniemy instancję **Calendar** to instancje **CalendarEvents** już nie mają sensu, ponieważ były powiązane z konkretnym kalendarzem. Jest to bardzo ważna zasada, która pomaga nam poprawnie modelować agregaty.

- **Modyfikacje wewnętrznych obiektów są możliwe tylko przez Aggregate Roota** – dostęp do child obiektów nie jest możliwy poza Aggregate Root. Czyli nie możemy osobno pobrać **CalendarEvents** i go zmienić. Tylko obiekt **Calendar** udostępnia metody pozwalające na to.

- **Aggregate dba o spójność biznesową** – np. obiekt **Calendar** ma kolekcję **CalendarEvent** i przy dodawaniu nowego „spotkania” sprawdza czy nie nachodzi się z innym wydarzeniem w kalendarzu. Dodatkowo nie pozwala na równoczesną modyfikację swoich dzieci (np. poprzez użycie Concurrency Token).

- **Należy modyfikować tylko jeden Aggregate w transakcji bazodanowej** – dzięki temu łatwiej jest osiągnąć skalowalność i spójność transakcji dla operacji biznesowych. Jeżeli jedna akcja biznesowa wymaga zmian w wielu agregatach to należy zastanowić się czy tych agregatów nie zastąpić jednym albo czy możliwe jest **_„eventual consistency and acceptable update delay”_**.

- **Aggregate może trzymać reference przez „identity”** – czyli może posiadać reference do innych agregatów (najlepiej jako Id do tego korzenia agregatu) i pobiera go tylko wtedy, kiedy go potrzebuje **(Disconnected Domaion Model)**. Obiekt **Calendar** może mieć reference na użytkownika (UserId lub UserGlobalId), ale nie może go modyfikować np. nie może zablokować użytkownika. Patrz punkt wyżej ⤴️

- **Publikuje zdarzenia domenowe (Domain Events)** – więcej w osobnym wpisie o [zdarzeniach domenowych](/posts/zdarzenia-domenowe-ddd/).

- **Może korzystać z [Domain Service](/posts/domain-services-ddd/)** – w swoich metodach może korzystać z Domain Service, aby zapewnić spójność biznesową. **Calendar** korzystałby z Domain Serwisu, który zwracałby informacje czy dany dzień jest dniem wolnym od pracy.

### Dobre praktyki przy modelowaniu Agregatów

- **Aggregate powinny robić jak najmniej się da** – całą logikę powinny „spychać” na Encje i ValueObjecty. Przykład: przy dodawaniu nowego **CaledarEvent** do **Calendar** chcemy sprawdzić czy wydarzenia z kalendarza nie nachodzą się na siebie. Więc encja **CaledarEvent** posiadałaby metodę **IsOverlap**, która zwracałyby informację czy **CalendarEventy** się nakładają, a **Calendar** byłby tylko odpowiedzialny za odpowiednie wywołanie tych metody i dodanie nowego **CalendarEvent** do kolekcji.

- **Aggregate powinny być małe** – im większy mamy aggregate tym większy problem będziemy mieli z zapewnieniem transakcyjności, czytelności i wydajności.

- **Aggregate powinny ładować wszystkie swoje encje-dzieci** – dodatkowo, może używać innych Repozytoriów (lub Lazy Loading), aby pobrać inne agregaty, których referencje trzyma jako Id. Pamiętamy, że nie należny modyfikować tych referencyjnych agregatów.

- **Repozytoria zwracają tylko Aggregate** – nie zwracają Encji czy ValueObjectów, zwracają wyłącznie agregaty (choć są wyjątki od tej reguły). Tutaj osobny wpis o [Repozytorium – DDD](/posts/repozytorium-ddd/).

### Przykładowa Implementacja

```csharp
public class Calendar
{
    public int Id { get; private set; }

    public IList<CalendarEvent> CalendarEvents { get; private set; }

    public User User { get; private set; }

    public void ScheduleEvent(CalendarEvent eventToSchedule, IBankHolidayService bankHolidayService)
    {
        if (bankHolidayService.IsHoliday(eventToSchedule.Day))
        {
            throw new InvalidOperationException("Cannot schedule event on bank holiday!");
        }

        if (CalendarEvents.Any(x => x.IsOverlap(eventToSchedule)))
        {
            throw new InvalidOperationException("Calendar Events cannot overlap!");
        }

        CalendarEvents.Add(eventToSchedule);
    }

}

```

### Podsumowanie

Agregat jest ogniwem, który spaja logikę biznesową zawartą w Encjach i ValueObjectach, jednocześnie dba o spójność systemu poprzez zapewnienie transakcyjności operacji biznesowych.
