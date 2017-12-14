---
title: Platformy ASP.NET Core MVC podstawowych EF - dziedziczenia - 9, 10
author: tdykstra
description: "Ten samouczek przedstawia sposób implementacji dziedziczenia w modelu danych, przy użyciu programu Entity Framework Core w aplikacji platformy ASP.NET Core."
keywords: Dziedziczenie platformy ASP.NET Core Entity Framework Core,
ms.author: tdykstra
manager: wpickett
ms.date: 03/15/2017
ms.topic: get-started-article
ms.assetid: 41dc0db7-6f17-453e-aba6-633430609c74
ms.technology: aspnet
ms.prod: asp.net-core
uid: data/ef-mvc/inheritance
ms.openlocfilehash: 10bde121dac3bdbbf0e55f2d146d91dea0f0210f
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 11/10/2017
---
# <a name="inheritance---ef-core-with-aspnet-core-mvc-tutorial-9-of-10"></a><span data-ttu-id="25d85-104">Dziedziczenie - Core EF z samouczek platformy ASP.NET Core MVC (9, 10)</span><span class="sxs-lookup"><span data-stu-id="25d85-104">Inheritance - EF Core with ASP.NET Core MVC tutorial (9 of 10)</span></span>

<span data-ttu-id="25d85-105">Przez [Dykstra Tomasz](https://github.com/tdykstra) i [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="25d85-105">By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="25d85-106">Przykładową aplikację sieci web firmy Contoso University przedstawia sposób tworzenia aplikacji sieci web platformy ASP.NET Core MVC przy użyciu programu Entity Framework Core i Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="25d85-106">The Contoso University sample web application demonstrates how to create ASP.NET Core MVC web applications using Entity Framework Core and Visual Studio.</span></span> <span data-ttu-id="25d85-107">Informacje o samouczek serii, zobacz [pierwszy samouczek z tej serii](intro.md).</span><span class="sxs-lookup"><span data-stu-id="25d85-107">For information about the tutorial series, see [the first tutorial in the series](intro.md).</span></span>

<span data-ttu-id="25d85-108">Samouczek poprzedniej służy do obsługi wyjątków współbieżności.</span><span class="sxs-lookup"><span data-stu-id="25d85-108">In the previous tutorial, you handled concurrency exceptions.</span></span> <span data-ttu-id="25d85-109">Ten samouczek przedstawia sposób wykonania dziedziczenia w modelu danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-109">This tutorial will show you how to implement inheritance in the data model.</span></span>

<span data-ttu-id="25d85-110">W programowanie zorientowane obiektowo można użyć dziedziczenia, aby ułatwić ponowne użycie kodu.</span><span class="sxs-lookup"><span data-stu-id="25d85-110">In object-oriented programming, you can use inheritance to facilitate code reuse.</span></span> <span data-ttu-id="25d85-111">W tym samouczku zostanie zmieniona `Instructor` i `Student` klasy tak, że pochodzą one od `Person` podstawowa klasa, która zawiera właściwości, takie jak `LastName` które są wspólne dla uczniów lub studentów i instruktorów.</span><span class="sxs-lookup"><span data-stu-id="25d85-111">In this tutorial, you'll change the `Instructor` and `Student` classes so that they derive from a `Person` base class which contains properties such as `LastName` that are common to both instructors and students.</span></span> <span data-ttu-id="25d85-112">Nie dodać lub zmienić żadnych stron sieci web, ale zostanie zmieniona części kodu i te zmiany są automatycznie odzwierciedlane w bazie danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-112">You won't add or change any web pages, but you'll change some of the code and those changes will be automatically reflected in the database.</span></span>

## <a name="options-for-mapping-inheritance-to-database-tables"></a><span data-ttu-id="25d85-113">Opcje mapowania dziedziczenia w tabelach bazy danych</span><span class="sxs-lookup"><span data-stu-id="25d85-113">Options for mapping inheritance to database tables</span></span>

<span data-ttu-id="25d85-114">`Instructor` i `Student` klasy w szkole modelu danych mają kilka właściwości, które są identyczne:</span><span class="sxs-lookup"><span data-stu-id="25d85-114">The `Instructor` and `Student` classes in the School data model have several properties that are identical:</span></span>

![Dla użytkowników domowych i instruktora klas](inheritance/_static/no-inheritance.png)

<span data-ttu-id="25d85-116">Załóżmy, że chcesz wyeliminować nadmiarowy kod dla właściwości, które są udostępniane przez `Instructor` i `Student` jednostek.</span><span class="sxs-lookup"><span data-stu-id="25d85-116">Suppose you want to eliminate the redundant code for the properties that are shared by the `Instructor` and `Student` entities.</span></span> <span data-ttu-id="25d85-117">Lub chcesz zapisać to usługa, która może sformatować nazwy bez opiekowanie, czy nazwa pochodzi od instruktora lub studenta.</span><span class="sxs-lookup"><span data-stu-id="25d85-117">Or you want to write a service that can format names without caring whether the name came from an instructor or a student.</span></span> <span data-ttu-id="25d85-118">Można utworzyć `Person` podstawowa klasa, która zawiera tylko te właściwości udostępnionego, a następnie wprowadź `Instructor` i `Student` klasy dziedziczą z tej klasy podstawowej, jak pokazano na poniższej ilustracji:</span><span class="sxs-lookup"><span data-stu-id="25d85-118">You could create a `Person` base class that contains only those shared properties, then make the `Instructor` and `Student` classes inherit from that base class, as shown in the following illustration:</span></span>

![Dla użytkowników domowych i instruktora klasy wywodzące się z osobą klasy](inheritance/_static/inheritance.png)

<span data-ttu-id="25d85-120">Istnieje kilka metod, które ta struktura dziedziczenia może być reprezentowane w bazie danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-120">There are several ways this inheritance structure could be represented in the database.</span></span> <span data-ttu-id="25d85-121">Może mieć tabeli osoby, która zawiera informacje o studentów i instruktorów w jednej tabeli.</span><span class="sxs-lookup"><span data-stu-id="25d85-121">You could have a Person table that includes information about both students and instructors in a single table.</span></span> <span data-ttu-id="25d85-122">Niektóre kolumny można zastosować tylko do instruktorów (DataZatrudnienia), niektórych tylko dla uczniów lub studentów (EnrollmentDate), niektórych zarówno (LastName, FirstName).</span><span class="sxs-lookup"><span data-stu-id="25d85-122">Some of the columns could apply only to instructors (HireDate), some only to students (EnrollmentDate), some to both (LastName, FirstName).</span></span> <span data-ttu-id="25d85-123">Zazwyczaj trzeba kolumna dyskryminatora, aby wskazać, który typ reprezentuje każdego wiersza.</span><span class="sxs-lookup"><span data-stu-id="25d85-123">Typically, you'd have a discriminator column to indicate which type each row represents.</span></span> <span data-ttu-id="25d85-124">Na przykład kolumny rozróżniacza mogą mieć "Instruktora" dla szkolenia i "Uczniów" dla uczniów lub studentów.</span><span class="sxs-lookup"><span data-stu-id="25d85-124">For example, the discriminator column might have "Instructor" for instructors and "Student" for students.</span></span>

![Przykład tabeli na hierarchii](inheritance/_static/tph.png)

<span data-ttu-id="25d85-126">Ten wzorzec generowania strukturę jednostek dziedziczenia z tabeli pojedynczej bazy danych jest nazywana tabeli na hierarchii dziedziczenia (TPH).</span><span class="sxs-lookup"><span data-stu-id="25d85-126">This pattern of generating an entity inheritance structure from a single database table is called table-per-hierarchy (TPH) inheritance.</span></span>

<span data-ttu-id="25d85-127">Alternatywą jest wyglądu struktury dziedziczenia bazę danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-127">An alternative is to make the database look more like the inheritance structure.</span></span> <span data-ttu-id="25d85-128">Na przykład może zawierać tylko pola Nazwa tabeli osób i mieć osobne instruktora i uczniów tabele z polami daty.</span><span class="sxs-lookup"><span data-stu-id="25d85-128">For example, you could have only the name fields in the Person table and have separate Instructor and Student tables with the date fields.</span></span>

![Dziedziczenie tabeli według typu](inheritance/_static/tpt.png)

<span data-ttu-id="25d85-130">Ten wzorzec dokonywania tabeli bazy danych dla każdej klasy jednostki nosi nazwę tabeli na dziedziczenia typu (TPT).</span><span class="sxs-lookup"><span data-stu-id="25d85-130">This pattern of making a database table for each entity class is called table per type (TPT) inheritance.</span></span>

<span data-ttu-id="25d85-131">Jeszcze inną możliwością jest wykonanie mapowania wszystkich typów nieabstrakcyjnej poszczególnych tabel.</span><span class="sxs-lookup"><span data-stu-id="25d85-131">Yet another option is to map all non-abstract types to individual tables.</span></span> <span data-ttu-id="25d85-132">Wszystkie właściwości klasy, w tym właściwości dziedziczone są mapowane na kolumny tej tabeli.</span><span class="sxs-lookup"><span data-stu-id="25d85-132">All properties of a class, including inherited properties, map to columns of the corresponding table.</span></span> <span data-ttu-id="25d85-133">Ten wzorzec jest nazywany dziedziczenia tabeli na konkretnych klas (TPC).</span><span class="sxs-lookup"><span data-stu-id="25d85-133">This pattern is called Table-per-Concrete Class (TPC) inheritance.</span></span> <span data-ttu-id="25d85-134">Jeśli zaimplementowano TPC dziedziczenia klas osoby, dla użytkowników domowych i instruktora jak pokazano wcześniej, tabele dla użytkowników domowych i instruktora będzie wyglądać nie różni się po wdrożeniu dziedziczenia, niż zajmowały przed.</span><span class="sxs-lookup"><span data-stu-id="25d85-134">If you implemented TPC inheritance for the Person, Student, and Instructor classes as shown earlier, the Student and Instructor tables would look no different after implementing inheritance than they did before.</span></span>

<span data-ttu-id="25d85-135">Wzorce dziedziczenia TPC i TPH zazwyczaj zapewniają lepszą wydajność niż w przypadku wzorców dziedziczenia TPT, wzorce TPT może powodować sprzężenia złożonych zapytań.</span><span class="sxs-lookup"><span data-stu-id="25d85-135">TPC and TPH inheritance patterns generally deliver better performance than TPT inheritance patterns, because TPT patterns can result in complex join queries.</span></span>

<span data-ttu-id="25d85-136">Ten samouczek pokazuje, jak zaimplementować dziedziczenia TPH.</span><span class="sxs-lookup"><span data-stu-id="25d85-136">This tutorial demonstrates how to implement TPH inheritance.</span></span> <span data-ttu-id="25d85-137">TPH to wzorzec tylko dziedziczenia, który obsługuje programu Entity Framework Core.</span><span class="sxs-lookup"><span data-stu-id="25d85-137">TPH is the only inheritance pattern that the Entity Framework Core supports.</span></span>  <span data-ttu-id="25d85-138">Co należy zrobić to utworzyć `Person` klasy, zmień `Instructor` i `Student` pochodzi od klasy `Person`, Dodaj nową klasę do `DbContext`i Utwórz migracji.</span><span class="sxs-lookup"><span data-stu-id="25d85-138">What you'll do is create a `Person` class, change the `Instructor` and `Student` classes to derive from `Person`, add the new class to the `DbContext`, and create a migration.</span></span>

> [!TIP] 
> <span data-ttu-id="25d85-139">Warto zapisać kopię projektu przed wprowadzeniem następujące zmiany.</span><span class="sxs-lookup"><span data-stu-id="25d85-139">Consider saving a copy of the project before making the following changes.</span></span>  <span data-ttu-id="25d85-140">Następnie w razie problemów i należy zacząć od początku, będzie łatwiejsze do uruchamiania z zapisanym projektu zamiast cofania czynności wykonywane w ramach tego samouczka lub będzie powrót do początku całej serii.</span><span class="sxs-lookup"><span data-stu-id="25d85-140">Then if you run into problems and need to start over, it will be easier to start from the saved project instead of reversing steps done for this tutorial or going back to the beginning of the whole series.</span></span>

## <a name="create-the-person-class"></a><span data-ttu-id="25d85-141">Utwórz klasę osoby</span><span class="sxs-lookup"><span data-stu-id="25d85-141">Create the Person class</span></span>

<span data-ttu-id="25d85-142">W folderze modeli Utwórz Person.cs i Zastąp kod szablonu z następującym kodem:</span><span class="sxs-lookup"><span data-stu-id="25d85-142">In the Models folder, create Person.cs and replace the template code with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu/Models/Person.cs)]

## <a name="make-student-and-instructor-classes-inherit-from-person"></a><span data-ttu-id="25d85-143">Utworzyć klasy dla użytkowników domowych i instruktora dziedziczyć osoby</span><span class="sxs-lookup"><span data-stu-id="25d85-143">Make Student and Instructor classes inherit from Person</span></span>

<span data-ttu-id="25d85-144">W *Instructor.cs*, pochodzi z klasy instruktora od klasy osoby i usuń pola klucza i nazwę.</span><span class="sxs-lookup"><span data-stu-id="25d85-144">In *Instructor.cs*, derive the Instructor class from the Person class and remove the key and name fields.</span></span> <span data-ttu-id="25d85-145">Ten kod będzie wyglądać następująco:</span><span class="sxs-lookup"><span data-stu-id="25d85-145">The code will look like the following example:</span></span>

[!code-csharp[Main](intro/samples/cu/Models/Instructor.cs?name=snippet_AfterInheritance&highlight=8)]

<span data-ttu-id="25d85-146">Wprowadzenie identycznych zmian w *Student.cs*.</span><span class="sxs-lookup"><span data-stu-id="25d85-146">Make the same changes in *Student.cs*.</span></span>

[!code-csharp[Main](intro/samples/cu/Models/Student.cs?name=snippet_AfterInheritance&highlight=8)]

## <a name="add-the-person-entity-type-to-the-data-model"></a><span data-ttu-id="25d85-147">Dodaj typ jednostki osoby do modelu danych</span><span class="sxs-lookup"><span data-stu-id="25d85-147">Add the Person entity type to the data model</span></span>

<span data-ttu-id="25d85-148">Dodaj typ jednostki osoby do *SchoolContext.cs*.</span><span class="sxs-lookup"><span data-stu-id="25d85-148">Add the Person entity type to *SchoolContext.cs*.</span></span> <span data-ttu-id="25d85-149">Nowe wiersze są wyróżnione.</span><span class="sxs-lookup"><span data-stu-id="25d85-149">The new lines are highlighted.</span></span>

[!code-csharp[Main](intro/samples/cu/Data/SchoolContext.cs?name=snippet_AfterInheritance&highlight=19,30)]

<span data-ttu-id="25d85-150">To wszystko, który programu Entity Framework wymaga, aby skonfigurować tabelę dla hierarchii dziedziczenia.</span><span class="sxs-lookup"><span data-stu-id="25d85-150">This is all that the Entity Framework needs in order to configure table-per-hierarchy inheritance.</span></span> <span data-ttu-id="25d85-151">Jak można zauważyć, gdy baza danych jest aktualizowana, ma zamiast tabel dla użytkowników domowych i instruktora tabeli osoby.</span><span class="sxs-lookup"><span data-stu-id="25d85-151">As you'll see, when the database is updated, it will have a Person table in place of the Student and Instructor tables.</span></span>

## <a name="create-and-customize-migration-code"></a><span data-ttu-id="25d85-152">Tworzyć i dostosowywać kod migracji</span><span class="sxs-lookup"><span data-stu-id="25d85-152">Create and customize migration code</span></span>

<span data-ttu-id="25d85-153">Zapisz zmiany i skompilować projekt.</span><span class="sxs-lookup"><span data-stu-id="25d85-153">Save your changes and build the project.</span></span> <span data-ttu-id="25d85-154">Następnie otwórz okno polecenia w folderze projektu i wprowadź następujące polecenie:</span><span class="sxs-lookup"><span data-stu-id="25d85-154">Then open the command window in the project folder and enter the following command:</span></span>

```console
dotnet ef migrations add Inheritance
```

<span data-ttu-id="25d85-155">Nie uruchamiaj `database update` jeszcze polecenia.</span><span class="sxs-lookup"><span data-stu-id="25d85-155">Don't run the `database update` command yet.</span></span> <span data-ttu-id="25d85-156">Tego polecenia zostanie spowodować utratę danych, ponieważ będzie porzucić tę tabelę instruktora i nazwy tabeli uczniów do osoby.</span><span class="sxs-lookup"><span data-stu-id="25d85-156">That command will result in lost data because it will drop the Instructor table and rename the Student table to Person.</span></span> <span data-ttu-id="25d85-157">Musisz podać kodu niestandardowego, aby zachować istniejące dane.</span><span class="sxs-lookup"><span data-stu-id="25d85-157">You need to provide custom code to preserve existing data.</span></span>

<span data-ttu-id="25d85-158">Otwórz *migracje /\<sygnatury czasowej > _Inheritance.cs* i Zastąp `Up` metodę z następującym kodem:</span><span class="sxs-lookup"><span data-stu-id="25d85-158">Open *Migrations/\<timestamp>_Inheritance.cs* and replace the `Up` method with the following code:</span></span>

[!code-csharp[Main](intro/samples/cu/Migrations/20170216215525_Inheritance.cs?name=snippet_Up)]

<span data-ttu-id="25d85-159">Ten kod zajmuje się następujące zadania aktualizacji bazy danych:</span><span class="sxs-lookup"><span data-stu-id="25d85-159">This code takes care of the following database update tasks:</span></span>

* <span data-ttu-id="25d85-160">Usuwa ograniczeń klucza obcego i indeksów, które wskazują tabeli uczniów.</span><span class="sxs-lookup"><span data-stu-id="25d85-160">Removes foreign key constraints and indexes that point to the Student table.</span></span>

* <span data-ttu-id="25d85-161">Zmienia nazwę tabeli instruktora osoby i wprowadza zmiany w przypadku przechowywania danych dla użytkowników domowych:</span><span class="sxs-lookup"><span data-stu-id="25d85-161">Renames the Instructor table as Person and makes changes needed for it to store Student data:</span></span>

* <span data-ttu-id="25d85-162">Dodaje EnrollmentDate wartości null dla uczniów lub studentów.</span><span class="sxs-lookup"><span data-stu-id="25d85-162">Adds nullable EnrollmentDate for students.</span></span>

* <span data-ttu-id="25d85-163">Dodaje kolumnę dyskryminator wskazująca, czy wiersz jest dla uczniów lub instruktora.</span><span class="sxs-lookup"><span data-stu-id="25d85-163">Adds Discriminator column to indicate whether a row is for a student or an instructor.</span></span>

* <span data-ttu-id="25d85-164">Sprawia, że DataZatrudnienia nullable ponieważ wierszy uczniów nie będziesz mieć dat.</span><span class="sxs-lookup"><span data-stu-id="25d85-164">Makes HireDate nullable since student rows won't have hire dates.</span></span>

* <span data-ttu-id="25d85-165">Dodaje tymczasowego pola, które będą służyć do aktualizowania kluczy obcych, które wskazują studentów.</span><span class="sxs-lookup"><span data-stu-id="25d85-165">Adds a temporary field that will be used to update foreign keys that point to students.</span></span> <span data-ttu-id="25d85-166">Podczas kopiowania studentów w tabeli osób otrzymują nowe wartości klucza podstawowego.</span><span class="sxs-lookup"><span data-stu-id="25d85-166">When you copy students into the Person table they'll get new primary key values.</span></span>

* <span data-ttu-id="25d85-167">Kopiuje dane z tabeli uczniów do tabeli osoby.</span><span class="sxs-lookup"><span data-stu-id="25d85-167">Copies data from the Student table into the Person table.</span></span> <span data-ttu-id="25d85-168">Powoduje to, że można zostaną przypisane nowe wartości klucza podstawowego.</span><span class="sxs-lookup"><span data-stu-id="25d85-168">This causes students to get assigned new primary key values.</span></span>

* <span data-ttu-id="25d85-169">Rozwiązuje wartości kluczy obcych, które wskazują studentów.</span><span class="sxs-lookup"><span data-stu-id="25d85-169">Fixes foreign key values that point to students.</span></span>

* <span data-ttu-id="25d85-170">Ponownie tworzy ograniczeń klucza obcego i indeksów, teraz wskazaniem tabeli osób.</span><span class="sxs-lookup"><span data-stu-id="25d85-170">Re-creates foreign key constraints and indexes, now pointing them to the Person table.</span></span>

<span data-ttu-id="25d85-171">(W przypadku identyfikatora GUID zamiast liczby całkowitej jako typu klucza podstawowego, nie trzeba zmieniać wartości klucza podstawowego dla użytkowników domowych i niektóre z tych kroków można została pominięta.)</span><span class="sxs-lookup"><span data-stu-id="25d85-171">(If you had used GUID instead of integer as the primary key type, the student primary key values wouldn't have to change, and several of these steps could have been omitted.)</span></span>

<span data-ttu-id="25d85-172">Uruchom `database update` polecenia:</span><span class="sxs-lookup"><span data-stu-id="25d85-172">Run the `database update` command:</span></span>

```console
dotnet ef database update
```

<span data-ttu-id="25d85-173">(W systemie produkcji może wprowadzić odpowiednie zmiany do `Down` metody w przypadku kiedykolwiek musiał używać, aby wrócić do poprzedniej wersji bazy danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-173">(In a production system you would make corresponding changes to the `Down` method in case you ever had to use that to go back to the previous database version.</span></span> <span data-ttu-id="25d85-174">W tym samouczku nie będzie używany `Down` metody.)</span><span class="sxs-lookup"><span data-stu-id="25d85-174">For this tutorial you won't be using the `Down` method.)</span></span>

> [!NOTE] 
> <span data-ttu-id="25d85-175">Istnieje możliwość pobrania inne błędy podczas wprowadzania zmian schematu w bazie danych istniejących danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-175">It's possible to get other errors when making schema changes in a database that has existing data.</span></span> <span data-ttu-id="25d85-176">Jeśli występują błędy migracji, których nie można rozwiązać, można zmienić nazwę bazy danych w parametrach połączenia lub Usuń bazę danych.</span><span class="sxs-lookup"><span data-stu-id="25d85-176">If you get migration errors that you can't resolve, you can either change the database name in the connection string or delete the database.</span></span> <span data-ttu-id="25d85-177">Z nową bazę danych nie ma żadnych danych do migracji, a polecenia update-database jest bardziej prawdopodobne zakończyć bez błędów.</span><span class="sxs-lookup"><span data-stu-id="25d85-177">With a new database, there is no data to migrate, and the update-database command is more likely to complete without errors.</span></span> <span data-ttu-id="25d85-178">Aby usunąć bazę danych, użyj SSOX lub uruchom `database drop` polecenia interfejsu wiersza polecenia.</span><span class="sxs-lookup"><span data-stu-id="25d85-178">To delete the database, use SSOX or run the `database drop` CLI command.</span></span>

## <a name="test-with-inheritance-implemented"></a><span data-ttu-id="25d85-179">Test z dziedziczenia zaimplementowany</span><span class="sxs-lookup"><span data-stu-id="25d85-179">Test with inheritance implemented</span></span>

<span data-ttu-id="25d85-180">Uruchom aplikację i spróbuj różnych stron.</span><span class="sxs-lookup"><span data-stu-id="25d85-180">Run the app and try various pages.</span></span> <span data-ttu-id="25d85-181">Wszystko, co działa tak samo, tak jak poprzednio.</span><span class="sxs-lookup"><span data-stu-id="25d85-181">Everything works the same as it did before.</span></span>

<span data-ttu-id="25d85-182">W **Eksplorator obiektów SQL Server**, rozwiń węzeł **danych połączenia/SchoolContext** , a następnie **tabel**, i zastępuje tabel dla użytkowników domowych i instruktora Tabela osoby.</span><span class="sxs-lookup"><span data-stu-id="25d85-182">In **SQL Server Object Explorer**, expand **Data Connections/SchoolContext** and then **Tables**, and you see that the Student and Instructor tables have been replaced by a Person table.</span></span> <span data-ttu-id="25d85-183">Otwórz projektanta tabel osoby i zobacz, czy zawiera wszystkie kolumny, które są używane w tabelach dla użytkowników domowych i instruktora.</span><span class="sxs-lookup"><span data-stu-id="25d85-183">Open the Person table designer and you see that it has all of the columns that used to be in the Student and Instructor tables.</span></span>

![Osoba tabeli w SSOX](inheritance/_static/ssox-person-table.png)

<span data-ttu-id="25d85-185">Kliknij prawym przyciskiem myszy tabeli osób, a następnie kliknij przycisk **Pokaż dane tabeli** wyświetlić kolumny rozróżniacza.</span><span class="sxs-lookup"><span data-stu-id="25d85-185">Right-click the Person table, and then click **Show Table Data** to see the discriminator column.</span></span>

![Tabela osoby w SSOX - tabeli danych](inheritance/_static/ssox-person-data.png)

## <a name="summary"></a><span data-ttu-id="25d85-187">Podsumowanie</span><span class="sxs-lookup"><span data-stu-id="25d85-187">Summary</span></span>

<span data-ttu-id="25d85-188">Zostały zaimplementowane tabeli na hierarchii dziedziczenia dla `Person`, `Student`, i `Instructor` klasy.</span><span class="sxs-lookup"><span data-stu-id="25d85-188">You've implemented table-per-hierarchy inheritance for the `Person`, `Student`, and `Instructor` classes.</span></span> <span data-ttu-id="25d85-189">Aby uzyskać więcej informacji na temat dziedziczenia w Entity Framework Core, zobacz [dziedziczenia](https://docs.microsoft.com/ef/core/modeling/inheritance).</span><span class="sxs-lookup"><span data-stu-id="25d85-189">For more information about inheritance in Entity Framework Core, see [Inheritance](https://docs.microsoft.com/ef/core/modeling/inheritance).</span></span> <span data-ttu-id="25d85-190">W następnym samouczku zobaczysz sposób obsługi różnych stosunkowo zaawansowanych scenariuszy programu Entity Framework.</span><span class="sxs-lookup"><span data-stu-id="25d85-190">In the next tutorial you'll see how to handle a variety of relatively advanced Entity Framework scenarios.</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="25d85-191">[Poprzednie](concurrency.md)
[dalej](advanced.md)</span><span class="sxs-lookup"><span data-stu-id="25d85-191">[Previous](concurrency.md)
[Next](advanced.md)</span></span>  