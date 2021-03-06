---
title: Oprogramowanie pośredniczące platformy ASP.NET Core
author: rick-anderson
description: Więcej informacji na temat oprogramowania pośredniczącego platformy ASP.NET Core i potok żądań.
ms.author: riande
ms.custom: mvc
ms.date: 08/21/2018
uid: fundamentals/middleware/index
ms.openlocfilehash: e6dc76b7cb80e0dfda102df5aefb5d9ce9b821ed
ms.sourcegitcommit: 847cc1de5526ff42a7303491e6336c2dbdb45de4
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 08/27/2018
ms.locfileid: "43055809"
---
# <a name="aspnet-core-middleware"></a>Oprogramowanie pośredniczące platformy ASP.NET Core

Przez [Rick Anderson](https://twitter.com/RickAndMSFT) i [Steve Smith](https://ardalis.com/)

Oprogramowanie pośredniczące to oprogramowanie, które jest umieszczone w potoku aplikacji do obsługi żądań i odpowiedzi. Poszczególne składniki:

* Pozwala wybrać, czy przekazywać żądania do następnego składnika w potoku.
* Można wykonywać pracę, przed i po wywołaniu następny składnik w potoku.

Delegaty żądania są używane do tworzenia potoku żądania. Delegaty żądania obsługi danego żądania HTTP.

Żądania, obiekty delegowane są skonfigurowane przy użyciu <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run*>, <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map*>, i <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*> metody rozszerzenia. Delegat pojedynczego żądania może być określony w tekście jako metodę anonimową (nazywane w tekście oprogramowania pośredniczącego), lub można zdefiniować klasy wielokrotnego użytku. Te klasy wielokrotnego użytku i metod anonimowych w tekście są *oprogramowania pośredniczącego*, nazywane również *składników oprogramowania pośredniczącego*. Każdy składnik oprogramowania pośredniczącego w potoku żądania jest odpowiedzialny za wywoływanie następny składnik w potoku lub zwarcie potoku.

<xref:migration/http-modules> wyjaśnia różnicę pomiędzy potoki żądania w programie ASP.NET Core i ASP.NET 4.x i udostępnia więcej przykładów oprogramowania pośredniczącego.

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>Tworzenie potoku oprogramowania pośredniczącego z IApplicationBuilder

Potok żądań ASP.NET Core składa się z sekwencji obiektów delegowanych żądania, o nazwie jedna po drugiej. Poniższy diagram ilustruje pojęcia. Wątek wykonywania następuje zamiast czarnych strzałek.

![Wyświetlanie żądań przychodzących, przetwarzania za pomocą trzech middlewares i odpowiedzi, wychodzenia z aplikacji wzorca przetwarzania żądania. Każdy oprogramowania pośredniczącego uruchamia swojej logiki i przekazywało żądanie do następnego oprogramowania pośredniczącego w instrukcji next(). Po trzecie oprogramowanie pośredniczące przetwarza żądanie, żądanie Przechodzi wstecz przez wcześniejsze middlewares dwa w odwrotnej kolejności dla dodatkowego przetwarzania po deklaracji metody next() przed opuszczeniem aplikację jako odpowiedzi do klienta.](index/_static/request-delegate-pipeline.png)

Każdy delegat mogą wykonywać operacje, przed i po następnym delegata. Obiekt delegowany można też przekazuje żądania do następnej delegata, która jest wywoływana *zwarcie Potok żądań*. Zwarcie jest często pożądane, ponieważ takie rozwiązanie pomaga uniknąć niepotrzebnych pracy. Na przykład statyczne pliki oprogramowania pośredniczącego wrócić żądanie dotyczące pliku statycznego i zwarcie pozostałego potoku. Delegatów obsługi wyjątków są nazywane na wczesnym etapie potoku, dlatego ich może przechwytywać wyjątki, które występują w późniejszym etapie w potoku.

Najprostsza możliwa aplikacji ASP.NET Core ustawia delegat pojedynczego żądania, który obsługuje wszystkie żądania. Ten przypadek nie zawiera rzeczywistego żądania potoku. Zamiast tego jednego funkcja anonimowa jest wywoływana w odpowiedzi na każde żądanie HTTP.

[!code-csharp[](index/snapshot/Middleware/Startup.cs?name=snippet1)]

Pierwszy <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run*> delegata kończy potoku.

Utworzyć łańcuch wielu delegatów żądanie, wraz z <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*>. `next` Parametr reprezentuje dalej delegata w potoku. Można zwarcie potok według *nie* wywoływania *dalej* parametru. Zazwyczaj można wykonywać akcje przed i po następnym delegata, tak jak pokazano w poniższym przykładzie:

[!code-csharp[](index/snapshot/Chain/Startup.cs?name=snippet1)]

> [!WARNING]
> Nie wywołuj `next.Invoke` po wysłaniu odpowiedzi do klienta. Zmienia się na <xref:Microsoft.AspNetCore.Http.HttpResponse> po rozpoczęciu odpowiedzi zgłoszenie wyjątku. Na przykład zmiany, takie jak ustawianie nagłówków i kod stanu zgłosić wyjątek. Zapisywanie w treści odpowiedzi po wywołaniu `next`:
>
> * Może to spowodować naruszenie protokołu. Na przykład zapisywanie ponad podanej `Content-Length`.
> * Może spowodować uszkodzenie format treści. Na przykład zapis stopki HTML do pliku CSS.
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted*> jest to przydatne wskazówka, aby wskazać, czy nagłówki zostały wysłane, czy jednostka została zapisana.

## <a name="order"></a>Kolejność

Kolejność dodaną składników oprogramowania pośredniczącego w `Startup.Configure` metoda definiuje kolejność, w którym są wywoływane składników oprogramowania pośredniczącego na żądania i odwrotnej kolejności dla odpowiedzi. Kolejność jest krytyczny dla bezpieczeństwa, wydajności i funkcjonalności.

Następujące `Configure` metoda dodaje następujące składniki oprogramowania pośredniczącego:

1. Obsługa wyjątku/błędów
2. Serwer plików statycznych
3. Uwierzytelnianie
4. MVC

::: moniker range=">= aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    if (env.IsDevelopment())
    {
        // When the app runs in the Development environment:
        //   Use the Developer Exception Page to report app runtime errors.
        //   Use the Database Error Page to report database runtime errors.
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        // When the app doesn't run in the Development environment:
        //   Enable the Exception Handler Middleware to catch exceptions
        //     thrown in the following middlewares.
        //   Use the HTTP Strict Transport Security Protocol (HSTS)
        //     Middleware.
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    // Use HTTPS Redirection Middleware to redirect HTTP requests to HTTPS.
    app.UseHttpsRedirection();

    // Return static files and end the pipeline.
    app.UseStaticFiles();

    // Use Cookie Policy Middleware to conform to EU General Data 
    //   Protection Regulation (GDPR) regulations.
    app.UseCookiePolicy();

    // Authenticate before the user accesses secure resources.
    app.UseAuthentication();

    // Add MVC to the request pipeline.
    app.UseMvc();
}
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    // Enable the Exception Handler Middleware to catch exceptions
    //   thrown in the following middlewares.
    app.UseExceptionHandler("/Home/Error");

    // Return static files and end the pipeline.
    app.UseStaticFiles();

    // Authenticate before you access secure resources.
    app.UseIdentity();

    // Add MVC to the request pipeline.
    app.UseMvcWithDefaultRoute();
}
```

::: moniker-end

W poprzednim kodzie przykład, każda metoda rozszerzenia oprogramowanie pośredniczące jest uwidaczniany w <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> za pośrednictwem <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName> przestrzeni nazw.

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler*> Pierwszy składnik oprogramowania pośredniczącego jest dodawany do potoku. W związku z tym oprogramowanie pośredniczące programu obsługi wyjątków przechwytuje wszystkie wyjątki, które występują w nowszym wywołuje.

Statyczne oprogramowania pośredniczącego plików jest wywoływana na wczesnym etapie potoku, dzięki czemu mogą obsługiwać żądania i zwarcie bez pośrednictwa pozostałe składniki. Statyczne oprogramowanie pośredniczące plikach udostępnia **nie** sprawdzeń autoryzacji. Wszystkich plików obsługiwanych przez, łącznie z tymi w ramach *wwwroot*, są publicznie dostępne. Podejścia do zabezpieczania plików statycznych, zobacz <xref:fundamentals/static-files>.

::: moniker range=">= aspnetcore-2.0"

Jeśli żądanie nie jest obsługiwany przez statyczne oprogramowanie pośredniczące plików, będzie przekazany z oprogramowaniem pośredniczącym uwierzytelniania (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*>), który przeprowadza uwierzytelnianie. Uwierzytelnianie nie powodują pominięcie nieuwierzytelnione żądania. Mimo że uwierzytelniania oprogramowania pośredniczącego uwierzytelniania na podstawie żądań autoryzacji (i odrzucenia) występuje tylko wtedy, gdy MVC wybiera określonych akcji i kontrolerów MVC lub strony Razor.

::: moniker-end

::: moniker range="< aspnetcore-2.0"

Jeśli żądanie nie jest obsługiwany przez statyczne pliki oprogramowanie pośredniczące, przekazaniem do oprogramowania pośredniczącego tożsamości (<xref:Microsoft.AspNetCore.Builder.BuilderExtensions.UseIdentity*>), który przeprowadza uwierzytelnianie. Tożsamość nie powodują pominięcie nieuwierzytelnione żądania. Mimo że tożsamość uwierzytelniania na podstawie żądań autoryzacji (i odrzucenia) występuje tylko wtedy, gdy MVC wybiera określonego kontrolera i akcji.

::: moniker-end

Poniższy przykład pokazuje kolejność oprogramowania pośredniczącego, w którym żądań dotyczących plików statycznych są obsługiwane przez statyczne pliki oprogramowania pośredniczącego, zanim oprogramowanie pośredniczące kompresji odpowiedzi. Pliki statyczne nie są kompresowane z tym zamówieniem oprogramowania pośredniczącego. Odpowiedzi MVC z <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute*> można skompresować.

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files not compressed by Static Files Middleware.
    app.UseStaticFiles();
    app.UseResponseCompression();
    app.UseMvcWithDefaultRoute();
}
```

### <a name="use-run-and-map"></a>Użyj, uruchom i mapy

Konfigurowanie przy użyciu potoku HTTP `Use`, `Run`, i `Map`. `Use` Metoda może zwarcie potoku (to znaczy, jeśli on nie wywołać `next` delegata żądania). `Run` jest to Konwencja i niektórych składników oprogramowania pośredniczącego może narazić `Run[Middleware]` metod, które są uruchamiane na końcu potoku.

<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map*> rozszerzenia są używane jako Konwencję do tworzenia odgałęzień potoku. `Map*` gałęzie potoku żądania na podstawie dopasowań podanej ścieżki żądania. Jeśli ścieżka żądania rozpoczyna się od podanej ścieżce, jest wykonywana gałąź.

[!code-csharp[](index/snapshot/Chain/StartupMap.cs?name=snippet1)]

W poniższej tabeli przedstawiono żądań i odpowiedzi z `http://localhost:1234` przy użyciu poprzedniego kodu.

| Żądanie             | Odpowiedź                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Pozdrowienia ze-Map delegata. |
| localhost:1234/map1 | Mapa badania 1                   |
| localhost:1234/map2 | Mapa badania 2                   |
| localhost:1234/map3 | Pozdrowienia ze-Map delegata. |

Gdy `Map` jest używane, ścieżka dopasowane jej ewentualny związek są usuwane z `HttpRequest.Path` i dołączonych do `HttpRequest.PathBase` dla każdego żądania.

[MapWhen](/dotnet/api/microsoft.aspnetcore.builder.mapwhenextensions) odgałęzień potoku żądania na podstawie wyniku danego predykatu. Wszelkie predykatu typu `Func<HttpContext, bool>` może być używany do mapowania żądania do nowej gałęzi w potoku. W poniższym przykładzie, predykat jest używany do wykrywania obecności zmiennej ciągu zapytania `branch`:

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs?name=snippet1)]

W poniższej tabeli przedstawiono żądań i odpowiedzi z `http://localhost:1234` przy użyciu poprzedniego kodu.

| Żądanie                       | Odpowiedź                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Pozdrowienia ze-Map delegata. |
| localhost:1234/?branch=master | Używana gałąź = master         |

`Map` obsługuje zagnieżdżania, na przykład:

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
   ```

`Map` można również przeprowadzać dopasowywanie wiele segmentów jednocześnie:

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?name=snippet1&highlight=13)]

## <a name="built-in-middleware"></a>Wbudowane oprogramowania pośredniczącego

Platforma ASP.NET Core jest dostarczany z następujących składników oprogramowania pośredniczącego. *Kolejności* kolumna zawiera uwagi dotyczące umieszczenia oprogramowanie pośredniczące w potoku żądanie i na jakich warunkach oprogramowanie pośredniczące może zakończyć żądania i uniemożliwić innym oprogramowaniu pośredniczącym przetwarzania żądania.

| Oprogramowanie pośredniczące | Opis | Kolejność |
| ---------- | ----------- | ----- |
| [Uwierzytelnianie](xref:security/authentication/identity) | Zapewnia obsługę uwierzytelniania. | Przed `HttpContext.User` jest wymagana. Terminala dla wywołania zwrotne OAuth. |
| [CORS](xref:security/cors) | Konfiguruje, Cross-Origin Resource Sharing. | Przed składników, które korzystają z mechanizmu CORS. |
| [Diagnostyka](xref:fundamentals/error-handling) | Konfiguruje diagnostyki. | Przed składniki, które generują błędy. |
| [Nagłówki przekazywane](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions) | Przekazuje nagłówki przekazywane do bieżącego żądania. | Przed składników korzystających z zaktualizowanymi polami (przykłady: schematu i hosta, adres IP klienta, metoda). |
| [Zastąpienie metody HTTP](/dotnet/api/microsoft.aspnetcore.builder.httpmethodoverrideextensions) | Zezwala na przychodzące żądania POST przesłonić metodę. | Przed składniki, które zużywają zaktualizowana metoda. |
| [Przekierowania protokołu HTTPS](xref:security/enforcing-ssl#require-https) | Przekieruj wszystkie żądania HTTP do HTTPS (platformy ASP.NET Core 2.1 lub nowszej). | Przed składniki, których wartość użycia adresu URL. |
| [Zabezpieczenia transportu Strict HTTP (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | Oprogramowanie ulepszenie pośredniczące zabezpieczeń, który dodaje nagłówek odpowiedzi specjalne (platformy ASP.NET Core 2.1 lub nowszej). | Przed wysłaniem odpowiedzi i po składniki, które modyfikują żądania (na przykład nagłówki przekazywane, ponownego zapisywania adresów URL). |
| [MVC](xref:mvc/overview) | Przetwarza żądania przy użyciu stron MVC i Razor (platformy ASP.NET Core w wersji 2.0 lub nowszej). | Terminal, jeśli żądanie jest zgodny z trasą. |
| [OWIN](xref:fundamentals/owin) | Współdziałanie z aplikacji opartych na OWIN, serwerów i oprogramowania pośredniczącego. | Terminal, jeśli oprogramowanie pośredniczące OWIN w pełni przetwarza żądanie. |
| [Buforowanie odpowiedzi](xref:performance/caching/middleware) | Zapewnia obsługę buforowania odpowiedzi. | Przed składniki, które wymagają buforowania. |
| [Kompresja odpowiedzi](xref:performance/response-compression) | Zapewnia obsługę kompresowania odpowiedzi. | Przed składniki, które wymagają kompresji. |
| [Lokalizacja żądania](xref:fundamentals/localization) | Zapewnia obsługę lokalizacji. | Przed składniki poufnych lokalizacji. |
| [Routing](xref:fundamentals/routing) | Definiuje i ogranicza trasy żądania. | Terminalu zgodnych tras. |
| [Sesja](xref:fundamentals/app-state) | Zapewnia obsługę zarządzania sesjami użytkownika. | Przed składniki, które wymagają sesji. |
| [Pliki statyczne](xref:fundamentals/static-files) | Zapewnia obsługę obsługujących pliki statyczne oraz przeglądanie katalogów. | Terminal, gdy żądanie pasuje do pliku. |
| [Ponownego zapisywania adresów URL](xref:fundamentals/url-rewriting) | Zapewnia obsługę ponownego zapisywania adresów URL przekierowywania żądań. | Przed składniki, których wartość użycia adresu URL. |
| [Obiekty WebSocket](xref:fundamentals/websockets) | Włącza protokół Websocket. | Przed składniki, które są wymagane w celu umożliwienia akceptowania żądań protokołu WebSocket. |

## <a name="write-middleware"></a>Pisanie oprogramowania pośredniczącego

Oprogramowanie pośredniczące jest zazwyczaj hermetyzowane w klasie i widoczne z metodą rozszerzenia. Należy wziąć pod uwagę następujące oprogramowanie pośredniczące, które ustawia kulturę bieżącego żądania z ciągu zapytania:

[!code-csharp[](index/snapshot/Culture/StartupCulture.cs?name=snippet1)]

Poprzedni przykładowy kod jest używany do zademonstrowania tworzenia składników oprogramowania pośredniczącego. Obsługę wbudowanych lokalizacja platformy ASP.NET Core, zobacz <xref:fundamentals/localization>.

Możesz przetestować oprogramowanie pośredniczące, przekazując w kulturze, na przykład `http://localhost:7997/?culture=no`.

Poniższy kod powoduje delegata oprogramowania pośredniczącego do klasy:

[!code-csharp[](index/snapshot/Culture/RequestCultureMiddleware.cs)]

::: moniker range="< aspnetcore-2.0"

Oprogramowanie pośredniczące `Task` nazwę metody musi być `Invoke`. W programie ASP.NET Core 2.0 lub nowszej, może to być albo `Invoke` lub `InvokeAsync`.

::: moniker-end

Następujące metody rozszerzenia udostępnia oprogramowanie pośredniczące za pośrednictwem <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder>:

[!code-csharp[](index/snapshot/Culture/RequestCultureMiddlewareExtensions.cs)]

Poniższy kod wywołuje oprogramowanie pośredniczące z `Startup.Configure`:

[!code-csharp[](index/snapshot/Culture/Startup.cs?name=snippet1&highlight=5)]

Oprogramowanie pośredniczące powinno wykonać [jawne zależności zasady](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies) uwidaczniając jego zależności w jego konstruktorze. Oprogramowanie pośredniczące jest tworzona raz na *okres istnienia aplikacji*. Zobacz [zależności żądania](#per-request-dependencies) sekcji, jeśli zachodzi potrzeba udostępniania usług za pomocą oprogramowania pośredniczącego w żądaniu.

Składniki oprogramowania pośredniczącego może rozpoznać ich zależności z [wstrzykiwanie zależności (DI)](xref:fundamentals/dependency-injection) za pośrednictwem parametry konstruktora. [UseMiddleware&lt;T&gt; ](/dotnet/api/microsoft.aspnetcore.builder.usemiddlewareextensions.usemiddleware#Microsoft_AspNetCore_Builder_UseMiddlewareExtensions_UseMiddleware_Microsoft_AspNetCore_Builder_IApplicationBuilder_System_Type_System_Object___) mogą również akceptować dodatkowe parametry bezpośrednio.

### <a name="per-request-dependencies"></a>Zależności żądania

Ponieważ oprogramowanie pośredniczące jest konstruowany przy uruchamianiu aplikacji, nie dla poszczególnych żądań, *zakresie* okres istnienia, obejmujący usługi używane przez oprogramowanie pośredniczące konstruktory nie są udostępniane przy użyciu innych typów zależności, wprowadzony podczas każdego żądania. Jeśli musisz udostępnić *zakresie* usługi między oprogramowania pośredniczącego a innymi typami danych, należy dodać tych usług `Invoke` podpis metody. `Invoke` Metoda może obsługiwać dodatkowe parametry, które są wypełniane przez DI:

```csharp
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    // IMyScopedService is injected into Invoke
    public async Task Invoke(HttpContext httpContext, IMyScopedService svc)
    {
        svc.MyProperty = 1000;
        await _next(httpContext);
    }
}
```

## <a name="additional-resources"></a>Dodatkowe zasoby

* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>
