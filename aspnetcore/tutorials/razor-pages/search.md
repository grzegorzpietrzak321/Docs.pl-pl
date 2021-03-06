---
title: Dodaj wyszukiwanie do platformy ASP.NET Core Razor strony
author: rick-anderson
description: Przedstawiono sposób dodawania wyszukiwania do platformy ASP.NET Core Razor stron
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 05/30/2018
uid: tutorials/razor-pages/search
ms.openlocfilehash: 14b3b501915a22aedbc10bd7bc8e5eef408f185c
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/20/2018
ms.locfileid: "36278151"
---
# <a name="add-search-to-aspnet-core-razor-pages"></a>Dodaj wyszukiwanie do platformy ASP.NET Core Razor strony

przez [Rick Anderson](https://twitter.com/RickAndMSFT)

W tym dokumencie możliwości wyszukiwania jest dodawany do strony indeksu, która umożliwia wyszukiwanie filmów przez *genre* lub *nazwa*.

Zaktualizuj strony indeksu `OnGetAsync` metodę z następującym kodem:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

W pierwszym wierszu `OnGetAsync` metoda tworzy [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) zapytanie, aby wybrać filmy:

```csharp
var movies = from m in _context.Movie
             select m;
```

Zapytanie jest *tylko* zdefiniowane w tym momencie, ma **nie** zostały uruchomione w bazie danych.

Jeśli `searchString` parametru zawiera ciąg, filmy zapytania są modyfikowane w celu filtrowania ciąg wyszukiwania:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

`s => s.Title.Contains()` Kod [wyrażenia Lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). Wyrażenia lambda są używane w oparte na metodzie [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) wysyła zapytanie jako argumenty do metod operator standardowej kwerendy, takich jak [gdzie](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) metody lub `Contains` (używane w powyższym kodzie). Podczas definiowania lub są one zmienione przez wywołanie metody nie zostaną wykonane zapytań LINQ (takich jak `Where`, `Contains` lub `OrderBy`). Zamiast wykonywania zapytania została odroczona. Oznacza to, że obliczania wyrażenia jest opóźnione aż do jej wartość rzeczywista jest iterowane za pośrednictwem lub `ToListAsync` metoda jest wywoływana. Zobacz [wykonywania zapytania](/dotnet/framework/data/adonet/ef/language-reference/query-execution) Aby uzyskać więcej informacji.

**Uwaga:** [zawiera](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) metody jest uruchamiane na bazie danych, nie znajduje się w kodzie języka C#. Uwzględniana wielkość liter w zapytaniu jest zależna od bazy danych i sortowanie. W programie SQL Server `Contains` mapuje [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql), która jest uwzględniana wielkość liter. W SQLite z sortowania domyślnego, jest uwzględniana wielkość liter.

Przejdź do strony filmy i Dołącz ciąg zapytania, takie jak `?searchString=Ghost` do adresu URL (na przykład `http://localhost:5000/Movies?searchString=Ghost`). Filtrowane filmów są wyświetlane.

![Widok indeksu](search/_static/ghost.png)

Jeśli następujący szablon trasy zostanie dodany do strony indeksu, ciąg wyszukiwania mogą być przekazywane jako segment adresu URL (na przykład `http://localhost:5000/Movies/Ghost`).

```cshtml
@page "{searchString?}"
```

Poprzednie ograniczenie trasy umożliwia wyszukiwanie tytuł jako dane trasy (segment adresu URL) zamiast jako wartość ciągu zapytania.  `?` w `"{searchString?}"` oznacza, że jest to parametr opcjonalny trasy.

![Widok indeksu z widma word dodane do adresu Url i filmu zwrócona lista filmów Ghostbusters i Ghostbusters 2](search/_static/g2.png)

Nie można jednak spodziewać się użytkowników, aby zmodyfikować adres URL, aby wyszukać filmu. W tym kroku interfejsu użytkownika jest dodawany do filtrowania filmów. Jeśli dodano ograniczenia trasy `"{searchString?}"`, usuń go.

Otwórz *Pages/Movies/Index.cshtml* pliku, a następnie dodaj `<form>` wyróżnione następujący kod znaczników:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

Kod HTML `<form>` tagów używa [pomocnika Tag formularza](xref:mvc/views/working-with-forms#the-form-tag-helper). Po przesłaniu formularza ciąg filtru jest wysyłana do *stron/filmów/indeksu* strony. Zapisz zmiany i przetestować filtr.

![Widok indeksu z widma word wpisane w polu tekstowym filtru tytułu](search/_static/filter.png)

## <a name="search-by-genre"></a>Wyszukiwanie według rodzaju

Dodaj następujące właściwości wyróżnione *Pages/Movies/Index.cshtml.cs*:

::: moniker range="= aspnetcore-2.0"
[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]
::: moniker-end

::: moniker range=">= aspnetcore-2.1"
[!code-csharp[](razor-pages-start/sample/RazorPagesMovie21/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]
::: moniker-end


`SelectList Genres` Zawiera listę gatunkami muzyki. Dzięki temu użytkownikowi na wybranie określonego rodzaju z listy.

`MovieGenre` Właściwość zawiera określonego rodzaju wybiera użytkownika (na przykład "zachodni").

Aktualizacja `OnGetAsync` metodę z następującym kodem:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

Następujący kod jest zapytania LINQ, która pobiera wszystkie genres z bazy danych.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

`SelectList` Gatunkami muzyki jest tworzony przez różne genres projekcji.

<!-- BUG in OPS
Tag snippet_selectlist's start line '75' should be less than end line '29' when resolving "[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]"

There's no start line.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]
-->

```csharp
Genres = new SelectList(await genreQuery.Distinct().ToListAsync());
```

### <a name="adding-search-by-genre"></a>Dodawanie wyszukiwania według rodzaju

Aktualizacja *Index.cshtml* w następujący sposób:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

Aby przetestować aplikację, wyszukiwanie według rodzaju, tytuł filmu i obu.

> [!div class="step-by-step"]
> [Poprzedni: Aktualizowanie stron](xref:tutorials/razor-pages/da1)
> [dalej: dodanie nowego pola](xref:tutorials/razor-pages/new-field)
