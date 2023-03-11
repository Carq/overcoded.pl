---
id: 534
title: "Zdarzenia domenowe &#8211; DDD"
date: "2022-07-21T08:00:00+02:00"
author: Carq
excerpt: "W DDD zdarzenia domenowe (Domain Events) są odpowiedzialne za powiadamianie całej naszej domeny o tym, że coś interesującego wydarzyło się. Inna część aplikacji może nasłuchiwać na konkretne wydarzenie i zareagować na nie."
categories: [DDD]
tags: [ddd, "C#", .net, "domain events", "zdarzenia domenowe"]
---

W DDD zdarzenia domenowe (Domain Events) są odpowiedzialne za powiadamianie całej naszej domeny o tym, że coś interesującego **się wydarzyło**. Inna część aplikacji może nasłuchiwać na konkretne wydarzenie i zareagować na nie.

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości zdarzeń domenowych

- **[Agregaty](/posts/agregat-ddd/) lub [Encje](/posts/encje-podstawy-ddd/) publikują zdarzenia** — każda metoda, która zmienia stan systemu, może publikować zdarzenia domenowe.

- **Najpierw wydarzenie, następnie publikacja** — system dostaje informację, że coś **już się wydarzyło**, a nie że dopiero może się wydarzyć. Załóżmy, że mamy encję `User`, która posiada metodę aktywującą użytkownika `Activate`. Metoda `Activate` opublikuje zdarzenie, na które zareaguje reszta systemu (wysłany zostanie email powitalny, zostaną pobrane pieniądze z konta użytkownika itd.). Może się natomiast zdarzyć, że przy zapisie zmian w obiekcie `User` do bazy coś się posypało, np. jakaś walidacja pola w tabeli nie przeszła i zmiany nie zostały zapisane. Aktywacja użytkownika nie doszła do skutku, jednak zdarzenie świadczące o tym zostało już rozpropagowane w systemie. Dlatego publikacja zdarzeń i ich obsługa powinna następować wtedy, gdy akcja powodująca publikację zdarzenia faktycznie zakończyła się powodzeniem.

- **Nazwy obiektów reprezentujących zdarzenia zapisujemy w czasie przeszłym** — wiąże się to z powyższym punktem. Zdarzenie mówi o tym, że coś się wydarzyło w przeszłości, więc nazwy zdarzeń powinny to odwzorowywać. Np. w przypadku metody aktywującej użytkownika, metoda ta powinna opublikować zdarzenie o nazwie `UserActivated`.

- **Zdarzenia są widoczne w całym systemie** — inne części systemu powinny mieć dostęp do klasy reprezentującej zdarzenie, bo mogą chcieć na nie zareagować. Najlepiej zdarzenia trzymać jako osobny projekt, który może być współdzielony pomiędzy różnymi częściami aplikacji.

- **Zdarzenie powinno zawierać informację, kiedy się wydarzyło** — nie mamy gwarancji, w jakiej kolejności zdarzenia dotrą do odbiorców. W świecie systemów rozproszonych kolejność docierania zdarzeń nie zawsze jest taka sama jak kolejność ich wysyłania. Dlatego informacja o czasie zdarzenia jest bardzo przydatna.

### Co nam dają zdarzenia domenowe?

- **Rozdzielają różne procesy biznesowe na osobne klasy** (separation of concerns) – dzięki temu kod jest czytelniejszy, prościej go testować i utrzymywać.

- **Skalowalność i wydajność systemu** — wyobraźmy sobie, że nagle w naszym systemie zostało aktywowanych milion użytkowników. Każda aktywacja użytkownika wymaga np. wysłania emaila powitalnego. Istnieje duża szansa, że system się zatka, jeżeli wysłanie emaila miałoby nastąpić w momencie aktywacji użytkownika. Natomiast, jeżeli wysyłanie emaila jest reakcją na zdarzenie `UserActivated`, które jest obsługiwane w osobnym procesie (transakcji bazodanowej), np. jako część background jobów, to nasz system może sobie zakolejkować obsługę tych zdarzeń i rozłożyć wysyłanie emaila w czasie.

### Obsługa zdarzeń domenowych

Obsługę zdarzeń domenowych można zrealizować na wiele sposób, ale dokładniej opiszę to w osobnym poście. Tutaj, bez wchodzenia w szczegóły pokrótce opiszę, jak może to wyglądać:

- Ten sam proces (np. żądania HTTP, transakcji bazodanowej itd), który publikuje dane zdarzenie, może być odpowiedzialny za obsługę zdarzenia. Zaletą takiego podejścia jest prostota implementacji i niska złożoność. Ma za to jedną dużą wadę; jeżeli obsługa zdarzenia się nie powiedzie, to cały proces zakończy się niepowodzeniem. Np. wysyłanie emaila powitalnego się nie powidło, bo serwer emaila nie odpowiada, wtedy aktywacja użytkownika nie dojdzie do skutku. 😟
- Obsługę zdarzenia najlepiej realizować w osobnym procesie, który nie jest powiązany z procesem publikującym zdarzenie, np. po aktywacji użytkownika zdarzenie domenowe `UserActivated` trafia do tabeli w bazie, a osobny Background Job pobierze to zdarzenie i obsłuży je w osobnym procesie. Zdarzenia też można obsługiwać za pomocą jakiegoś Service Busa, jak [Azure Service Bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview) lub [Kafka](https://kafka.apache.org/). Minusem tego podejścia jest duża złożoność i czas potrzebny na implementację tego podejścia. Należy też pamiętać o **_„eventual consistency”_** — reakcja na zdarzenie `UserActivated` nie następuje natychmiast. Obsługa zdarzenia może nastąpić po kilku minutach czy godzinach np. przez duże obciążenie systemu, czy niedostępność zewnętrznych serwisów, z których korzystamy.

### Przykładowa implementacja

Poniżej mamy dwa zdarzenia i wspólny interface `IDomainEvent` dla każdego zdarzenia domenowego w systemie. Posiadają one dwie właściwości `UserId` — czyli id użytkownika, którego dotyczy wydarzenie i `OccurredOn`, które mówi o tym, kiedy konkretnie zdarzenie miało miejsce.

```csharp
public interface IDomainEvent
{
    DateTime OccurredOn { get; }
}

public class UserActivatedEvent : IDomainEvent
{
    public UserActivatedEvent(int userId, DateTime occurredOn)
    {
        UserId = userId;
        OccurredOn = occurredOn;
    }

    public int UserId { get; }

    public DateTime OccurredOn { get; }
}

public class UserDeactivatedEvent : IDomainEvent
{
    public UserDeactivatedEvent(int userId, DateTime occurredOn)
    {
    UserId = userId;
    OccurredOn = occurredOn;
    }

    public int UserId { get; }

    public DateTime OccurredOn { get; }
}
```

W poniższym kodzie mamy publikację zdarzeń domenowych przez encję `User` oraz obsługę zdarzenia `UserDeactivatedEvent` przez klasę `SendWelcomeEmailHandler`. Klasa ta implementuje interface `IDomainEventHandler`, który jest używany, aby wywołać odpowiednią klasę obsługującą zdarzenie domenowe.

```csharp
public class User
{
    public User(int id)
    {
        Id = id;
    }

    public int Id { get; }

    public bool IsActive { get; private set; }

    public IList<IDomainEvent> DomainEvents = new List<IDomainEvent>();

    public void Activate()
    {
        IsActive = true;
        Publish(new UserActivatedEvent(Id, DateTime.Now));
    }

    public void Deactivate()
    {
        IsActive = false;
        Publish(new UserDeactivatedEvent(Id, DateTime.Now));
    }

    private void Publish(IDomainEvent domainEvent)
    {
        DomainEvents.Add(domainEvent);
    }

}

// Gdzieś indziej w aplikacji
public class SendWelcomeEmailHandler : IDomainEventHandler<UserActivatedEvent>
{
public Task Handle(UserActivatedEvent domainEvent)
{
// Pobierz email użytkownika na podstawie UserId
// Wyślij email...
}
}
```

Encja `User` posiada listę `IDomainEvent`, do której są dodawane domenowe zdarzania w metodach zmieniających stan systemu (`Activate` i `Deactivate`). Zdarzenia z listy `DomainEvents` powinny zostać użyte do odpowiedniej obsługi zdarzeń.

### Podsumowanie

Zdarzenia domenowe (domain events) są kolejnym ważnym **_Building Blockiem_** w DDD. Za ich pomocą możemy przejrzyściej modelować złożone procesy biznesowe. A przy odpowiedniej obsłudze zapewnić skalowalność dla dużych systemów.
