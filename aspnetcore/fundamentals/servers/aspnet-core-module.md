---
title: Moduł platformy ASP.NET Core
author: rick-anderson
description: Dowiedz się, jak moduł platformy ASP.NET Core umożliwia Kestrel serwer sieci web dla usług IIS lub usług IIS Express jako serwera zwrotnego serwera proxy.
ms.author: tdykstra
ms.custom: mvc
ms.date: 02/23/2018
uid: fundamentals/servers/aspnet-core-module
ms.openlocfilehash: 319ab80f20f3917dae921f43566e368b51c29f0b
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/20/2018
ms.locfileid: "36272338"
---
# <a name="aspnet-core-module"></a>Moduł platformy ASP.NET Core

Przez [Dykstra Tomasz](https://github.com/tdykstra), [Rick Strahl](https://github.com/RickStrahl), i [Roaming Krzysztof](https://github.com/Tratcher) 

Moduł platformy ASP.NET Core umożliwia platformy ASP.NET Core uruchamiania za usług IIS w konfiguracji zwrotny serwer proxy aplikacji. Usługi IIS oferują aplikacji web zaawansowane funkcje zabezpieczeń i możliwości zarządzania.

Obsługiwane wersje systemu Windows:

* Windows 7 lub nowszy
* Windows Server 2008 R2 lub nowszy

Moduł platformy ASP.NET Core działa tylko z Kestrel. Moduł nie jest zgodna z [HTTP.sys](xref:fundamentals/servers/httpsys) (wcześniej nazywanych [WebListener](xref:fundamentals/servers/weblistener)).

## <a name="aspnet-core-module-description"></a>Opis podstawowych modułu ASP.NET

Moduł platformy ASP.NET Core jest macierzysty moduł usług IIS, które podłącza się do potoku usług IIS do przekierowywania żądań sieci web do zaplecza aplikacji platformy ASP.NET Core. Wiele modułów macierzystych, takich jak uwierzytelnianie systemu Windows, pozostają aktywne. Aby dowiedzieć się, jak aktywne moduły usług IIS w module, zobacz [moduły IIS](xref:host-and-deploy/iis/modules).

Ponieważ aplikacje platformy ASP.NET Core, uruchom w procesie oddzielić od proces roboczy usług IIS, moduł obsługuje także zarządzanie procesem. Moduł uruchamia proces dla aplikacji platformy ASP.NET Core po pierwsze żądanie dociera i ponowne uruchomienie aplikacji, jeśli uległa awarii. Jest to zasadniczo takie samo zachowanie jako ASP.NET 4.x aplikacje, które są uruchamiane w procesie w usługach IIS, które są zarządzane przez [usługi aktywacji procesów systemu Windows (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was).

Na poniższym diagramie przedstawiono relację między usługami IIS, platformy ASP.NET Core modułu a aplikacje platformy ASP.NET Core:

![Moduł platformy ASP.NET Core](aspnet-core-module/_static/ancm.png)

Żądania odbierania z sieci web do sterownik HTTP.sys trybu jądra. Sterownik kieruje żądania do usług IIS na skonfigurowanym porcie witryny sieci Web, zwykle 80 (HTTP) lub 443 (HTTPS). Moduł przekazuje żądania do Kestrel na losowy port dla aplikacji, która nie jest port 80/443.

Moduł Określa port, za pomocą zmiennej środowiskowej podczas uruchamiania i oprogramowanie pośredniczące integracji usług IIS umożliwia skonfigurowanie serwera do nasłuchiwania `http://localhost:{port}`. Dodatkowe testy są wykonywane, i zostały odrzucone żądania, które nie pochodzą z modułu. Moduł nie obsługuje przekazywanie protokołu HTTPS, więc żądania są przekazywane za pośrednictwem protokołu HTTP, nawet jeśli odebranych przez usługi IIS przy użyciu protokołu HTTPS.

Po Kestrel przejmuje żądania z modułu, żądanie zostanie przypisany do potoku oprogramowanie pośredniczące platformy ASP.NET Core. Potoku oprogramowania pośredniczącego obsługuje żądanie i przekazuje ją jako `HttpContext` wystąpienie aplikacji logiki. Odpowiedzi aplikacji jest przekazywane z powrotem do usług IIS, które umieszcza on Wycofaj do klienta HTTP, który zainicjował żądanie.

Moduł platformy ASP.NET Core ma kilka innych funkcji. Moduł może:

* Ustawianie zmiennych środowiskowych dla procesu roboczego.
* Zaloguj się stdout dane wyjściowe do przechowywania plików podczas rozwiązywania problemów uruchamiania.
* Tokeny uwierzytelniania systemu Windows do przodu.

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>Jak zainstalować i używać modułu platformy ASP.NET Core

Aby uzyskać szczegółowe instrukcje na temat instalowania i używania moduł platformy ASP.NET Core, zobacz [hosta w systemie Windows z programem IIS](xref:host-and-deploy/iis/index). Aby uzyskać informacje na temat konfigurowania modułu, zobacz [odwołania konfiguracji platformy ASP.NET Core modułu](xref:host-and-deploy/aspnet-core-module).

## <a name="additional-resources"></a>Dodatkowe zasoby

* [Hosting w systemie Windows z usługami IIS](xref:host-and-deploy/iis/index)
* [Odwołania do konfiguracji modułu platformy ASP.NET Core](xref:host-and-deploy/aspnet-core-module)
* [Repozytorium GitHub modułu podstawowej platformy ASP.NET (kodu źródłowego)](https://github.com/aspnet/AspNetCoreModule)
