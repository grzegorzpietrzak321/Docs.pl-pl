---
title: Uruchamianie aplikacji w programie ASP.NET Core
author: ardalis
description: Dowiedz się, jak klasa startowa. w programie ASP.NET Core umożliwia skonfigurowanie usług i potok żądań aplikacji.
ms.author: tdykstra
ms.custom: mvc
ms.date: 4/13/2018
uid: fundamentals/startup
ms.openlocfilehash: 465d33cc1f19428d5189b3a6fa7088ac402a9751
ms.sourcegitcommit: 25150f4398de83132965a89f12d3a030f6cce48d
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/25/2018
ms.locfileid: "42927974"
---
# <a name="application-startup-in-aspnet-core"></a>Uruchamianie aplikacji w programie ASP.NET Core

Przez [Steve Smith](https://ardalis.com), [Tom Dykstra](https://github.com/tdykstra), i [Luke Latham](https://github.com/guardrex)

`Startup` Klasa służy do konfigurowania usług i potok żądań aplikacji.

## <a name="the-startup-class"></a>Klasa początkowa

Użyj aplikacji platformy ASP.NET Core `Startup` klasy, która nosi nazwę `Startup` przez Konwencję. `Startup` Klasy:

* Można opcjonalnie dołączyć [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configureservices) metodę, aby skonfigurować usługi aplikacji.
* Musi zawierać [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configure) metodę, aby utworzyć potok przetwarzania żądania przez aplikację.

`ConfigureServices` i `Configure` są wywoływane przez środowisko uruchomieniowe, po uruchomieniu aplikacji:

[!code-csharp[](startup/snapshot_sample/Startup1.cs)]

Określ `Startup` klasy [WebHostBuilderExtensions](/dotnet/api/Microsoft.AspNetCore.Hosting.WebHostBuilderExtensions) [UseStartup&lt;TStartup&gt; ](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.usestartup#Microsoft_AspNetCore_Hosting_WebHostBuilderExtensions_UseStartup__1_Microsoft_AspNetCore_Hosting_IWebHostBuilder_) metody:

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main&highlight=10)]

Host sieci web udostępnia kilka usług, które są dostępne do `Startup` konstruktora klasy. Aplikacja dodaje dodatkowe usługi za pośrednictwem `ConfigureServices`. Usługi aplikacji i hostów będą dostępne w `Configure` i w całej aplikacji.

Typowym zastosowaniem [wstrzykiwanie zależności](xref:fundamentals/dependency-injection) do `Startup` klasa jest do dodania:

* [IHostingEnvironment](/dotnet/api/Microsoft.AspNetCore.Hosting.IHostingEnvironment) do skonfigurowania usług przez środowisko.
* [Wartości IConfiguration](/dotnet/api/microsoft.extensions.configuration.iconfiguration) można odczytać konfiguracji.
* [Element ILoggerFactory](/dotnet/api/microsoft.extensions.logging.iloggerfactory) można utworzyć rejestratora w `Startup.ConfigureServices`.

[!code-csharp[](startup/snapshot_sample/Startup2.cs)]

Alternatywa wprowadza `IHostingEnvironment` jest wykorzystanie podejścia opartego na Konwencji. Aplikację można zdefiniować w oddzielnym `Startup` klas w różnych środowiskach (na przykład `StartupDevelopment`), a odpowiedni `Startup` klasy jest zaznaczona w czasie wykonywania. Klasy, w których sufiks nazwy pasuje do bieżącego środowiska jest podzielony na priorytety. Jeśli aplikacja jest uruchamiana w środowisku deweloperskim i obejmuje zarówno `Startup` klasy i `StartupDevelopment` klasy `StartupDevelopment` klasa jest używana. Aby uzyskać więcej informacji, zobacz [używanie wielu środowisk](xref:fundamentals/environments#environment-based-startup-class-and-methods).

Aby dowiedzieć się więcej na temat `WebHostBuilder`, zobacz [hostingu](xref:fundamentals/host/index) tematu. Aby uzyskać informacji na temat obsługi błędów podczas uruchamiania, zobacz [uruchamiania obsługi wyjątków](xref:fundamentals/error-handling#startup-exception-handling).

## <a name="the-configureservices-method"></a>Metoda ConfigureServices

[ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configureservices) metodą jest:

* Optional
* Wywołanie przez hosta sieci web przed `Configure` metodę, aby skonfigurować usługi aplikacji.
* Gdzie [opcje konfiguracji](xref:fundamentals/configuration/index) są ustawiane przez Konwencję.

Dodawanie usług do kontenera usługi udostępnia je w aplikacji, a w `Configure` metody. Te usługi są rozpoznawane przez [wstrzykiwanie zależności](xref:fundamentals/dependency-injection) lub [IApplicationBuilder.ApplicationServices](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder.applicationservices).

Host sieci web może skonfigurować niektóre usługi przed `Startup` metody są wywoływane. Szczegółowe informacje są dostępne w [hostów w programie ASP.NET Core](xref:fundamentals/host/index) tematu.

W przypadku funkcji, które wymagają znacznej Instalatora, istnieją `Add[Service]` metod rozszerzenia [IServiceCollection](/dotnet/api/Microsoft.Extensions.DependencyInjection.IServiceCollection). Aplikację internetową typowe rejestruje usługi Entity Framework, tożsamości i MVC:

[!code-csharp[](../common/samples/WebApplication1/Startup.cs?highlight=4,7,11&start=40&end=55)]

## <a name="the-configure-method"></a>Metoda Konfiguruj

[Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.startupbase.configure) metoda jest używana do określenia, w jaki sposób aplikacja reaguje na żądania HTTP. Potok żądań jest skonfigurowany, dodając [oprogramowania pośredniczącego](xref:fundamentals/middleware/index) składników [IApplicationBuilder](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) wystąpienia. `IApplicationBuilder` jest dostępna dla `Configure` metody, ale nie jest zarejestrowany w kontenerze usługi. Hostingu tworzy `IApplicationBuilder` i przekazuje je bezpośrednio do `Configure` ([źródło odwołania](https://github.com/aspnet/Hosting/blob/release/2.0.0/src/Microsoft.AspNetCore.Hosting/Internal/WebHost.cs#L179-L192)).

[Szablony ASP.NET Core](/dotnet/core/tools/dotnet-new) Konfiguruje potok dzięki obsłudze stroną wyjątków dla deweloperów [BrowserLink](http://vswebessentials.com/features/browserlink), stron błędów, pliki statyczne i ASP.NET Core MVC:

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Startup.cs?range=28-48&highlight=5,6,10,13,15)]

Każdy `Use` — metoda rozszerzenia dodaje składnik oprogramowania pośredniczącego do potoku żądania. Na przykład `UseMvc` dodaje metody rozszerzenia [routingu oprogramowania pośredniczącego](xref:fundamentals/routing) do potoku żądania i konfiguruje [MVC](xref:mvc/overview) jako domyślny program obsługi.

Każdy składnik oprogramowania pośredniczącego w potoku żądania jest odpowiedzialny za wywoływanie następny składnik w potoku lub zwarcie łańcucha, jeśli to stosowne. Jeśli zwarcie nie występuje prowadzącą oprogramowanie pośredniczące, każdy oprogramowania pośredniczącego ma drugą szansę, aby przetworzyć żądanie przed wysłaniem go do klienta.

Dodatkowe usługi, takie jak `IHostingEnvironment` i `ILoggerFactory`, mogą być również określone w oznaczeniu metody. Jeśli zostanie określony, są wprowadzane dodatkowe usługi, jeśli są one dostępne.

Aby uzyskać więcej informacji na temat sposobu użycia `IApplicationBuilder` i kolejność przetwarzania oprogramowanie pośredniczące, zobacz [oprogramowania pośredniczącego](xref:fundamentals/middleware/index).

## <a name="convenience-methods"></a>Wygodne metody

[ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder.configureservices) i [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderextensions.configure) wygodne metody, mogą być używane zamiast określania `Startup` klasy. Wiele wywołań `ConfigureServices` dołączenia do siebie nawzajem. Wiele wywołań `Configure` Użyj ostatniego wywołania metody.

[!code-csharp[](startup/snapshot_sample/Program.cs?highlight=18,22)]

## <a name="startup-filters"></a>Filtrów uruchamiania

Użyj [IStartupFilter](/dotnet/api/microsoft.aspnetcore.hosting.istartupfilter) do konfiguracji oprogramowania pośredniczącego na początku lub na końcu aplikacji [Konfiguruj](#the-configure-method) potoku oprogramowania pośredniczącego. `IStartupFilter` jest przydatne upewnić się, że oprogramowanie pośredniczące jest uruchamiany przed lub po oprogramowanie pośredniczące dodane przez biblioteki na początku lub końcu potoku przetwarzania żądań aplikacji.

`IStartupFilter` implementuje jedną metodę [Konfiguruj](/dotnet/api/microsoft.aspnetcore.hosting.istartupfilter.configure), który odbiera i zwraca `Action<IApplicationBuilder>`. [IApplicationBuilder](/dotnet/api/microsoft.aspnetcore.builder.iapplicationbuilder) definiuje klasę, aby skonfigurować Potok żądań aplikacji. Aby uzyskać więcej informacji, zobacz [tworzenie potoku oprogramowania pośredniczącego z IApplicationBuilder](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder).

Każdy `IStartupFilter` implementuje middlewares co najmniej jeden w Potok żądań. Filtry są wywoływane w kolejności, w której zostały one dodane do kontenera usług. Filtry mogą dodawać oprogramowanie pośredniczące przed lub po przekazanie sterowania do następnego filtru, dlatego ich dołączania na początku lub końcu potoku aplikacji.

[Przykładową aplikację](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/startup/sample/) ([sposobu pobierania](xref:tutorials/index#how-to-download-a-sample)) pokazuje, jak zarejestrować oprogramowanie pośredniczące z `IStartupFilter`. Przykładowa aplikacja zawiera oprogramowanie pośredniczące, która ustawia wartości opcji z parametru ciągu zapytania:

[!code-csharp[](startup/sample/RequestSetOptionsMiddleware.cs?name=snippet1)]

`RequestSetOptionsMiddleware` Jest skonfigurowana w `RequestSetOptionsStartupFilter` klasy:

[!code-csharp[](startup/sample/RequestSetOptionsStartupFilter.cs?name=snippet1&highlight=7)]

`IStartupFilter` Jest zarejestrowany w kontenerze usługi w `ConfigureServices`:

[!code-csharp[](startup/sample/Startup.cs?name=snippet1&highlight=3)]

Gdy parametr ciągu zapytania dla `option` zostanie podana, oprogramowanie pośredniczące przetwarza przypisanie wartości, zanim oprogramowanie pośredniczące MVC renderuje odpowiedzi:

![Okno przeglądarki, przedstawiające renderowanej strony indeksu. Wartość opcji jest renderowany jako "Z oprogramowania pośredniczącego" oparte na żądanie strony za pomocą parametru ciągu zapytania i wartość równa "Z oprogramowania pośredniczącego" opcji.](startup/_static/index.png)

Oprogramowanie pośredniczące kolejność wykonywania jest ustawiona według kolejności z `IStartupFilter` rejestracji:

* Wiele `IStartupFilter` implementacji może współpracować z tych samych obiektów. Jeśli kolejność jest ważna, kolejność ich `IStartupFilter` usługi rejestracji, aby dopasować kolejności ich middlewares powinno być ono uruchomione.
* Biblioteki mogą dodawać oprogramowanie pośredniczące z co najmniej `IStartupFilter` implementacji, które są uruchamiane przed lub po innym oprogramowaniu pośredniczącym aplikacji zarejestrowane w usłudze `IStartupFilter`. Aby wywołać `IStartupFilter` oprogramowania pośredniczącego przed oprogramowanie pośredniczące dodane przez bibliotekę `IStartupFilter`, umieść rejestracji usługi, zanim biblioteki jest dodawany do kontenera usługi. Po dodaniu biblioteki, aby wywołać później, umieść rejestracji usługi.

## <a name="add-configuration-at-startup-from-an-external-assembly"></a>Dodaj konfigurację podczas uruchamiania z zewnętrznego zestawu

[Interfejsu IHostingStartup](/dotnet/api/microsoft.aspnetcore.hosting.ihostingstartup) implementacji umożliwia dodawanie rozszerzeń do aplikacji przy uruchamianiu z zewnętrznego zestawu poza aplikacji `Startup` klasy. Aby uzyskać więcej informacji, zobacz [zwiększanie możliwości aplikacji z zewnętrznego zestawu](xref:fundamentals/configuration/platform-specific-configuration).

## <a name="additional-resources"></a>Dodatkowe zasoby

* <xref:fundamentals/host/index>
* <xref:fundamentals/environments>
* <xref:fundamentals/middleware/index>
* <xref:fundamentals/logging/index>
* <xref:fundamentals/configuration/index>
* [Klasa StartupLoader: metoda FindStartupType (źródło odwołania)](https://github.com/aspnet/Hosting/blob/rel/2.0.0/src/Microsoft.AspNetCore.Hosting/Internal/StartupLoader.cs#L66-L116)
