# <a name="work-with-sqlite-in-an-aspnet-core-mvc-app"></a>Praca z bazy danych SQLite w platformy ASP.NET Core aplikacji MVC

Przez [Rick Anderson](https://twitter.com/RickAndMSFT)

`MvcMovieContext` Obiektu obsługuje zadanie łączenia z bazą danych i mapowanie `Movie` obiektów do rekordów bazy danych. Kontekst bazy danych został zarejestrowany za pomocą [iniekcji zależności](xref:fundamentals/dependency-injection) kontenera w `ConfigureServices` metody w *Startup.cs* pliku:

[!code-csharp[](~/tutorials/first-mvc-app-xplat/start-mvc/sample/MvcMovie/Startup.cs?name=snippet2&highlight=6-8)]

## <a name="sqlite"></a>SQLite

[SQLite](https://www.sqlite.org/) stanów witryny sieci Web:

> SQLite jest niezależna, wysokiej niezawodności, osadzone, oferujący wszystkie funkcje, domeny publicznej, aparatu bazy danych SQL. SQLite jest najczęściej używanych aparatu bazy danych na całym świecie.

Istnieje wiele narzędzi innych firm, które można pobrać do zarządzania i wyświetlić bazy danych SQLite. Na poniższym obrazie pochodzi z [przeglądarki bazy danych dla bazy danych SQLite](http://sqlitebrowser.org/). Jeśli masz ulubionego narzędzia SQLite, zostaw komentarz w takich jak informacji na ten temat.

![Przeglądarka bazy danych dla bazy danych Film przedstawiający SQLite](~/tutorials/first-mvc-app-xplat/working-with-sql/_static/dbb.png)

## <a name="seed-the-database"></a>Inicjatora bazy danych

Utwórz nową klasę o nazwie `SeedData` w *modele* folderu. Zastąp wygenerowany kod poniżej:

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Models/SeedData.cs?name=snippet_1)]

Jeśli w bazie danych są wszystkie filmy, zwraca inicjatora inicjatora.

```csharp
if (context.Movie.Any())
{
    return;   // DB has been seeded.
}
```

<a name="si"></a>
### <a name="add-the-seed-initializer"></a>Dodaj inicjatora inicjatora

Inicjator inicjatora, aby dodać `Main` metody w *Program.cs* pliku:

::: moniker range=">= aspnetcore-2.1"
[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie21/Program.cs)]
::: moniker-end
::: moniker range="<= aspnetcore-2.0"
[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie/Program.cs?highlight=6,16-32)]
::: moniker-end

### <a name="test-the-app"></a>Testowanie aplikacji

Usuń wszystkie rekordy w bazie danych (co seed — metoda będzie uruchamiane). Zatrzymać i uruchomić aplikację w celu umieszczenia bazy danych.
   
Aplikacja zawiera wprowadzonych danych.

![Przeglądarka Otwórz aplikacji MVC Movie danych Film przedstawiający](~/tutorials/first-mvc-app/working-with-sql/_static/m55.png)
