---
id: 623
title: "Jak urozmaicić aplikacje konsolowe? Figgle!"
date: "2022-08-04T07:00:00+02:00"
guid: "https://overcoded.pl/?p=623"
permalink: /jak-urozmaicic-aplikacje-konsolowe/
author: Carq
excerpt: "Programista czasem musi napisać narzędzie wspomagające pracę zespołu (lub swoją) w formie aplikacji konsolowej. Można napisać kolejną nudną aplikację albo..."
categories: [Narzędzia]
tags: [biblioteka, "c#", notka]
---

Programista czasem musi napisać narzędzie wspomagające pracę zespołu (lub swoją) w formie aplikacji konsolowej. Można napisać kolejną nudną aplikację albo… Albo sprawić, aby nasza aplikacja konsolowa trochę się wyróżniała i zrobiła efekt „wow” wśród jej użytkowników 😉. Żeby zrobić to małym kosztem, wystarczy użyć biblioteki [**Figgle**](https://github.com/drewnoakes/figgle). Biblioteka ta umożliwia rysowanie nietuzinkowych „nagłówków” w aplikacjach konsolowych z użyciem ponad 250 czcionek. Poniżej przykładowy kod w .NET 6 wraz z rezultatem.

#### Kodzik (.NET 6)

```csharp
using Figgle;

Console.ForegroundColor = ConsoleColor.Magenta;
Console.WriteLine(FiggleFonts.Standard.Render("Lubie kodzic : )"));

Console.ForegroundColor = ConsoleColor.Blue;
Console.WriteLine(FiggleFonts.Ogre.Render("Lubie kodzic : )"));

Console.ForegroundColor = ConsoleColor.Green;
Console.WriteLine(FiggleFonts.Big.Render("Lubie kodzic : )"));

Console.ForegroundColor = ConsoleColor.Yellow;
Console.WriteLine(FiggleFonts.Basic.Render("Lubie kodzic : )"));

Console.ReadKey();

```

#### Rezultat

![Napis „Lubie kodzic” w czterech odsłonach 🎨](/assets/posts/figgle_01.png)
_Napis „Lubie kodzic” w czterech odsłonach 🎨_

Fajne? 😎

### Pomocne linki

- [github.com/drewnoakes/figgle](https://github.com/drewnoakes/figgle)
