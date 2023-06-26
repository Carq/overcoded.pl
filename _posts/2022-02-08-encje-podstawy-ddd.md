---
id: 253
guid: "https://overcoded.pl/?p=253"
permalink: /encje-podstawy-ddd/
title: "Encje - podstawy DDD"
date: "2022-02-08T20:17:56+01:00"
author: Carq
categories: [DDD]
tags: [ddd, "c#", .net]
---

Encja (entity) lub też obiekt domenowy jest kolejnym **_Building Blocks_** w Domain Driven Design (DDD). Encja opisuje znaczący kawałek domeny, który posiada **unikalną tożsamość** wraz z zestawem zachowań i danych (które mogą się zmieniać z czasem). Mówiąc prościej, większość logiki biznesowej i danych naszej aplikacji będzie w Encji. 😉 Przykładowymi encjami mogą być: **User**, **Article** lub **Smartphone** – Jest wiele takich encji, ale każda jest unikalna.

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości Encji

- **Identyfikowalne, ma tożsamość** – czyli posiada coś, co pozwala ją jednoznacznie zidentyfikować – jest to główna cecha encji. Obecnie najczęściej nadaje się tożsamość obiektowi domenowemu przez wygenerowaniu mu **Id**, za co odpowiedzialny jest silnik bazodanowy. Innymi przykładami z życia to numer konta bankowego albo numer seryjny urządzenia nadawany przez producentów. Encja będzie się zmieniać z czasem i może wyglądać zupełnie inaczej niż na samym początku, gdy ją tworzyliśmy, ale dalej będzie miała tą samą tożsamość (np. **Id**).

- **Opisuje coś znaczącego w domenie –** encja powinna modelować konkretną logikę domenową aplikacji, np. jeżeli mielibyśmy obiekt **User**, to mógłby on zawierać logikę związaną z rejestracją użytkownika lub też jego zablokowaniem. Jeżeli mielibyśmy obiekt, który trzyma tylko typy proste, jedynie dba o ich poprawną wartością i nie posiada żadnej innej logiki domenowej to prawdopodobnie nie jest to Encja (może [Value Objects](/value-objects/)?), raczej będzie to anemiczny model, który jest antywzorcem.

- **Posiada domenowe zachowania i dane** – encja posiada metody, które odpowiadają biznesowym akcjom lub procesom. Jeżeli akcją biznesową jest „Publikowanie Artykułu” to możliwe, że nasza domena będzie zawierać encję **Article** z metodą **Publish()**.

- **Zmiana danych Encji odbywa się wyłącznie przez metody** – nie ustawiamy właściwości przez bezpośrednie przypisanie wartości do nich, tylko używamy odpowiednich metod, które jednocześnie dbają o poprawność danych.

```csharp
article.Title = "Podstawy DDD"; // ŹLE!
article.SetTitle("Podstawy DDD"); // DOBRZE
```

- **Encja posiada walidację, która zapobiega wprowadzeniu jej w niepoprawny stan** – każda metoda, która zmienia stan Encji, powinna weryfikować parametry wejściowe oraz stan całego obiektu. Możemy też użyć wzorca Specyfikacji (Specification) lub Strategi (Strategy), oraz opóźnić walidacje stanu całej encji (Deferred Validation) – powstanie osobny wpis na ten temat.

- **Publikuje zdarzenia domenowe (Domain Events)** – wracając do przykładu z metodą **Publish()** – to na jej końcu moglibyśmy wygenerować event **ArticleHasBeenPublishedEvent**, który by powiadamiał cały system o tym wydarzeniu. Tutaj osobny wpis o [zdarzeniach domenowych](/zdarzenia-domenowe-ddd/).

- **Prawa Demeter** – Nie odwołujemy się bezpośrednio do zachowań lub danych wewnętrznych obiektów Encji. Zamiast tego tworzymy odpowiednią metodę, która robi to, co potrzebujemy. Dzięki temu, jeżeli wewnętrzna implementacja się zmieni, to nie wpłynie to na resztę aplikacji, w której dany kod jest używany. W poniższym przykładzie pobieramy imię autora, ale w przyszłości może się okazać, że imię autora będziemy pobierać z obiektu, który w momencie implementacji naszej głównej Encji jeszcze nie istnieje, np. z **Author** zamiast obiektu **User**. Zachowanie prawa Demeter zaoszczędza dużo czasu przy refaktoryzacjach kodu 😉

```csharp
article.User.Name; // ŹLE
article.GetAuthorName(); // DOBRZE
```

- **Staramy się mieć Encje czysto biznesowe** – encja powinna reprezentować głównie logikę domenową. Wszystko co nie jest bezpośrednio związane z domeną tylko z technicznym aspektem naszej aplikacji możemy przenieść do klasy bazowej.

### Przykładowa implementacja

Oto przykładowa implementacja Encji **Article**, która posiada pola **_Title_**, **_Text_** oraz informację czy artykuł został opublikowany (**_IsPublished_**):

```csharp
public class Article
{
    public Article(string title, string text)
    {
        SetTitle(title);
        SetText(text);
    }

    public int Id { get; private set; }

    public string Title { get; private set; }

    public string Text { get; private set; }

    public bool IsPublished { get; private set; }

    public void SetTitle(string title)
    {
        if (string.IsNullOrEmpty(title))
        {
            throw new ArgumentNullException("Title cannot be empty");
        }

        if (title.Length > 30)
        {
            throw new ArgumentException("Title cannot be longer than 30 chars.");
        }

        Title = title;
    }

    public void SetText(string text)
    {
        Text = text;
    }

    public void Publish()
    {
        if (IsPublished)
        {
            throw new InvalidOperationException("The article has already been published");
        }

        // Dodatkowo walidacja tytułu i tekstu

        IsPublished = true;
        // Rzucenie Domain Event ArticleHasBeenPublishedEvent
    }

}

```

Właściwości **_Title_** i **_Text_** mogłyby być [Value Objects](/value-objects/), które same by się walidowały, wtedy kod powyższy Encji by się uprościł.

## Podsumowanie

Encja jest **_Building Blockiem_**, który skupia się bardziej na tożsamości, zachowaniu i cyklu życia obiektu niż na jego właściwościach/atrybutach. Duża część logiki domenowej będzie w Encjach, dlatego też powinniśmy rozważnie je projektować, bazując na regułach biznesowych z realnego świata.
