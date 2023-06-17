---
id: 66
guid: "https://overcoded.pl/?p=66"
permalink: /hello-world-%f0%9f%a4%93/
title: "Hello World 🤓"
author: "Carq"
date: "2022-01-03T21:25:49+01:00"
image:
  path: /assets/posts/hello-world-header.png
  alt: Hello World in Visual Studio
categories: [inne]
---

Jak inaczej mógłby się nazywać pierwszy post na blogu programisty – Hello World 🤓

## Dlaczego założyłem bloga i o czym będę pisał?

Po 10 latach programowania pomyślałem, że poprowadzę bloga, gdzie będę mógł utrwalać wszelką wiedzę, którą zdobywam w codziennej pracy jako programista/architekt .NET. Mam nadzieje, że sam proces „utrwalania” wiedzy w formie blogowych postów zmotywuje mnie do jeszcze głębszego badania konkretnych tematów 💪🏼

Dodatkowo mam zamiar tworzyć:

- posty/tutoriale dla początkujących lub przyszłych (🤞🏼) programistów
- opisy narzędzi przydatnych dla programistów
- opis działania usług Azure Cloud (<https://azure.microsoft.com/pl-pl/>)
- streszczenia/mini-recenzje książek o programowaniu
- opisy moich prywatnych projektów i wniosków, jakie z nich wyciągnąłem

Zapraszam do komentowania i dyskutowania. Enjoy! 😊

## Na koniec program Hello World i .NET 6

Oto **CAŁY** kod aplikacji `Hello World` w wersji **Minimal Web APi** i **.NET 6**:

```csharp
var app = WebApplication.CreateBuilder(args).Build();
app.MapGet("/hello-world", () => "Hello World ^\_^");
app.Run();
```

Jeszcze wynik odpalenia powyższego programu:

![img-description](/assets/posts/hello-world-01.png)
