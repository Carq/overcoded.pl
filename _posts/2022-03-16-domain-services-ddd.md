---
id: 369
title: "Domain Service &#8211; DDD"
date: "2022-03-16T18:44:04+01:00"
author: Carq
excerpt: "Serwis domenowy (domain service) to biznesowa operacja która nie wpasowuje się do odpowiedzialności Agregatu lub Value Objectu. "
categories: [DDD]
tags: [ddd, "C#", .net]
---

Serwis domenowy (domain service) to biznesowa operacja, która nie wpasowuje się w odpowiedzialności [Agregatów](/posts/agregat-ddd/) lub [Value Objectów](posts/value-objects/).

<!-- prettier-ignore-start  -->
> Wpis jest częścią serii o [DDD](/ddd/).
{: .prompt-tip }
<!-- prettier-ignore-end  -->

### Właściwości Serwisu Domenowego

- **Bezstanowy (stateless)** – nie przechowuje żadnych danych.

- **Należy do warstwy domenowej** – nazwy metod serwisów domenowych odzwierciedlają operacje biznesowe.

- **Specjalizuje się w robieniu tylko jednej rzeczy** – jeden serwis domenowy jest odpowiedzialny za robienie tylko jednej rzeczy.

- **Może publikować zdarzenia domenowe (Domain Events)** – może powiadamiać resztę systemu, że dana operacja domenowa została zakończona.

- **Może wchodzić w interakcje z innymi serwisami domenowymi lub repozytoriami** – czyli mogą wykorzystywać inne serwisy domenowe i do pewnego stopnia repozytoria.

### Przykładowe operacje Serwisu Domenowego

- Transformacja jednego obiektu domenowego w drugi
- Wyliczanie wartości na podstawie więcej niż jednego obiektu domenowego
- Sprawdzanie czy podana wartość istnieje (np. nazwa waluty, kod pocztowy miasta itp.)

### Przykładowa implementacja

Poniżej przykładowa implementacja serwisu domenowego który konwertuje temperature (która w tym wypadku jest ValueObjectem) z stopni Celsjusza na Fahrenheita i w drugą stronę. Serwis ma dwie metody ale obie robią tylko jedną i tą samą rzecz – konwertuje jeden obiekt domenowy na drugi.

```csharp
public class TemperatureConverter
{
    public FahrenheitTemperature CelsiusToFahrenheit(CelsiusTemperature temperature)
    {
        return new FahrenheitTemperature(temperature.Value * 9 / 5 + 32);
    }

    public CelsiusTemperature CelsiusToFahrenheit(FahrenheitTemperature temperature)
    {
        return new CelsiusTemperature((temperature.Value + 32) * 5 / 9);
    }

}
```

Oczywiście ValueObject **CelsiusTemperature** mógłby korzystać (lub nie) z **TemperatureConverter** i sam posiadać metodę **ToFahrenheitTemperature**, ale obiekty domenowe nie powinny posiadać wiedzy jak się konwertować w inny obiekt domenowy, to już nie jest ich odpowiedzialność tylko „czegoś” innego.

### Podsumowanie

Krótko: Domain service – „Sometimes, it just isn’t thing…”
