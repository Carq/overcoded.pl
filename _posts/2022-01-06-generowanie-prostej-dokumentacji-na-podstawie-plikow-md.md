---
id: 141
guid: "https://overcoded.pl/?p=141"
permalink: /generowanie-prostej-dokumentacji-na-podstawie-plikow-md/
title: "Generowanie prostej dokumentacji na podstawie plików .md"
date: "2022-01-06T16:52:34+01:00"
author: Carq
image: /assets/posts/generowanie-prostej-dokumentacji-na-podstawie-plikow-md-header.png
categories: [Narzędzia]
tags: [biblioteka, "java script"]
---

Często w projektach zdarza się, że jest potrzeba udokumentowania pewnych aspektów naszej aplikacji np. opis pliku konfiguracyjnego, opisanie pewnych nietrywialnych procesów biznesowych, architektura aplikacji itd.. Najlepiej to zrobić poprzez dodanie opisu jako pliku .md ([markdown](https://www.markdownguide.org/getting-started/#what-is-markdown)) i trzymanie go wraz z kodem źródłowym. Dzięki temu dokumentacja zawsze będzie trzymana z wersją kodu, której dotyczy. Pomaga to w utrzymywaniu aktualnej dokumentacji przez członków zespołu (inaczej jest gdy dokumentacje trzymamy na stronach, o których nie każdy członek zespołu może wiedzieć lub mieć dostęp). Niestety, minusem takiego podejścia jest to, że osoby bez uprawnień do danego repozytorium kodu nie będą w stanie przeczytać takiej dokumentacji, co wtedy?

Można wygenerować dokumentację na podstawie plików .md jako stronę HTML. Jest wiele narzędzi, aby to osiągnąć np.

- <https://www.gitbook.com/> (może kiedyś opiszę to narzędzie)
- <https://jekyllrb.com/>
- <https://www.gatsbyjs.com/>
- i wiele innych

Jednak te narzędzia wymagają trochę czasu, aby się z nimi zapoznać. Dodatkowo każde z nich ma swoje wymagania, jak np. odpowiednia struktura folderów, albo specyficzne wymagania odnośnie składni w plikach .md. Członkowie zespołu musieliby się z nimi zapoznać, co mogłoby ich zniechęcić do utrzymywania dokumentacji 😈.

Jeżeli potrzebujemy prostego rozwiązania, które po prostu działa i nie wymaga więcej niż **15 minut** naszego czasu, to odpowiedź jest w kolejnym akapicie.

## Generowanie strony HTML

Użyjemy prostej JSowej biblioteki **[Single Page Markdown Website](https://yuanqing.github.io/single-page-markdown-website/)** (wymagany jest [NodeJS](https://nodejs.org/)). Dzięki niej wystarczy jedna komenda, aby wygenerować stronę HTML.

![single-page markdown website](/assets/posts/generowanie_dokumentacji_01.png)

Poniższą komendę odpalamy w folderze, gdzie znajdują się pliki .md, na podstawie których chcemy wygenerować stronę:

```
npx --yes -- single-page-markdown-website '*.md' --output page
```

Wynikiem będzie folder `page`, w którym będą się znajdowały pliki wygenerowanej strony. Oczywiście pliki będzie trzeba umieścić na jakimś serwerze, aby każdy miał do nich dostęp np. [GitHub Pages](https://pages.github.com/) (miejsce gdzie można za darmo hostować statyczne strony).

Narzędzie posiada także kilka dodatkowych opcji, takich jak:

- generowanie strony z kilku plików .md
- ustawienie tytułu i opisu strony
- wygenerowanie spisu treści
- dołożenie dodatkowych linków, które będą umieszczone na stronie

Opcje te ustawiamy poprzez plik konfiguracyjny `package.json` (odsyłam do dokumentacji u źródła [single-page-markdown-website/#configuration](https://yuanqing.github.io/single-page-markdown-website/#configuration)). U mnie konfiguracja wygląda tak:

```json
{
  "single-page-markdown-website": {
    "baseUrl": "https://carq.github.io/reading-list/",
    "title": "Carq's Reading List",
    "description": "A collection of books that I have readed",
    "toc": true,
    "sections": true,
    "links": [
      {
        "text": "Carq GitHub",
        "url": "https://github.com/Carq"
      }
    ],
    "faviconImage": "images/icon.png"
  }
}
```

## Rezultat działania

Rezultat działania zademonstruję na przykładzie mojego repozytorium GITa, gdzie zapisuję w postaci pliku .md przeczytane książki:

<https://github.com/Carq/reading-list> – tutaj repozytorium z plikiem README.md, na podstawie którego wygenerowałem stronę

<https://carq.github.io/reading-list/> – wygenerowana strona, a poniżej zrzut ekranu.

![single-page markdown website](/assets/posts/generowanie_dokumentacji_02.png)

## Podsumowanie

[Single Page Markdown Website](https://yuanqing.github.io/single-page-markdown-website/) – jest prostą biblioteką do szybkiego tworzenia statycznych stron. Znajomość takiego narzędzia może nam się przydać zarówno w pracy (np. do tworzenie prostej dokumentacji) jak i w prywatnych projektach (np. generowaniu listy przeczytanych książek 🤓) .

Jakie jest Wasza zadanie, przyda się Wam? 🤔
