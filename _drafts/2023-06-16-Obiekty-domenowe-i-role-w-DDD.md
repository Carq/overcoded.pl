---
id: 264
title: "Obiekty domenowe i role w DDD"
date: "2022-08-26T19:34:07+02:00"
author: Carq
tags: [ddd, "c#", .net]
---

Zazwyczaj, gdy tworzymy nowy obiekt domenowy, to jest on prosty. Jednak z czasem dochodzą nowe wymagania biznesowe i musimy rozbudowywać encję o nowe zachowania czy też odpowiedzialności (responsibilities). Ostatecznie może to doprowadzić do tego, że nasza encja rozrośnie się do dużych rozmiarów. W pewnym momencie uznamy, że najlepiej będzie ją rozbić na mniejsze encje, pomiędzy, które zostaną rozdzielone odpowiedzialności z oryginalnej przerośniętej encji. Aby nasz kod był bardziej elastyczny i dodatkowo pozwalał wprowadzać takie zmiany minimalnym kosztem, możemy Encjom nadawać **Role (Roles)**.

## Role i odpowiedzialności

**Role (Roles)** w postaci interfejsów (interface), które posiadają konkretne odpowiedzialności.

Ułatwianie zmian przy refaktorach

mokowanie w testach

Czystrzy kod, w metodanie ustawiajacej test nie mamy dostepu do publikowania artykułu

- Optymalizacja pobierania

```csharp
IMakeCustomerPreferred customer = session.Get<IMakeCustomerPreferred>(customerId);
customer.MakePreferred();
...
IAddOrdersToCustomer customer = session.Get<IAddOrdersToCustomer>(customerId);
customer.AddOrder(order);
```

## Przykład z artykułem

**Article** ma dwie ważne akcje biznesowe **SetText** i **Publish**. Jednak po pewnym czasie mogą pojawić się nowe wymagania biznesowe, typu: artykuł musi zostać zaakceptowana przez kogoś innego, ma być możliwość zgłaszania komentarzy i redagowania tekstu itd. W takim wypadku encja mocno urośnie i uznamy, że pora podzielić ją na mniejsze encje, np. na **ArticleDraft** i **PublishedArticle**. Taka zmiana wymagać będzie zmian w wielu miejscach w kodzie i może się okazać czasochłonna. Jednak nadając role (poprzez interface) rozrastającej się Encji, minimalizujemy ten koszt.

### Pomocne linki

- [Making Roles Explicit prezentacja z QCon London 2008](https://www.infoq.com/presentations/Making-Roles-Explicit-Udi-Dahan/)
