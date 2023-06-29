---
id: 313
guid: "https://overcoded.pl/?p=313"
permalink: /null-wybacz/
title: "null! &#8211; Operator wybaczający"
date: "2022-02-15T17:56:44+01:00"
author: Carq
excerpt: "Bardzo przydatny operator w sytuacjach kiedy wiemy o tym, że zmienna nie może być w danym miejscu nullem bo np.  wyżej jest to sprawdzane wewnątrz innej metody a statyczna analiza kodu zgłasza nam irytujące ostrzeżenie.  Operator ten nie wpływa na runtime kodu, jedynie na statyczną analizę kodu."
categories: ["C#"]
image: /assets/posts/null-wybacz-header.png
tags: [.net, "c#", notka]
---

[Operator null-forgiving](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/null-forgiving) (!) – operator wprowadzany z C#8, mówi narzędziom do statycznej analizy kody, że dana zmienna referencyjna nie jest `null`. Bardzo przydatny operator w sytuacjach kiedy wiemy o tym, że zmienna nie może być w danym miejscu nullem bo np. wyżej jest to sprawdzane wewnątrz innej metody a statyczna analiza kodu zgłasza nam irytujące ostrzeżenie. Operator ten nie wpływa na runtime kodu, jedynie na statyczną analizę kodu.

![](/assets/posts/null_01.png)

Poniższa wersja kodu z użyciem operatora **!** (wykrzyknik) już nie będzie nam zgłaszać ostrzeżenia:

```csharp
IList<SomeDev> devs = new List<SomeDev>() { new SomeDev(1), new SomeDev(3), new SomeDev(4) };
var dev = devs.SingleOrDefault(x => x.SkillLevel == 1);

Console.WriteLine(dev!.Name);
```
