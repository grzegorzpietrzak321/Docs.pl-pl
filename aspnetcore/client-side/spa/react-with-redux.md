---
title: Szablon projektu platformy React z Redux za pomocą platformy ASP.NET Core
author: SteveSandersonMS
description: Dowiedz się, jak rozpocząć pracę z szablonu projektu platformy ASP.NET Core jednej strony aplikacji JEDNOSTRONICOWEJ dla platformy React z Redux i utworzyć platformy react aplikacji.
monikerRange: '>= aspnetcore-2.0'
ms.author: scaddie
ms.custom: mvc
ms.date: 02/21/2018
uid: spa/react-with-redux
ms.openlocfilehash: dab3d20865250aae548bff4614e631dd7c73b46f
ms.sourcegitcommit: a1afd04758e663d7062a5bfa8a0d4dca38f42afc
ms.translationtype: MT
ms.contentlocale: pl-PL
ms.lasthandoff: 06/20/2018
ms.locfileid: "36291637"
---
# <a name="use-the-react-with-redux-project-template-with-aspnet-core"></a><span data-ttu-id="bd7ad-103">Szablon projektu platformy React z Redux za pomocą platformy ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="bd7ad-103">Use the React-with-Redux project template with ASP.NET Core</span></span>

::: moniker range="= aspnetcore-2.0"

> [!NOTE]
> <span data-ttu-id="bd7ad-104">O bibliotece React z Redux szablonu projektu tej dokumentacji nie jest zawarty w programie ASP.NET 2.0 Core.</span><span class="sxs-lookup"><span data-stu-id="bd7ad-104">This documentation isn't about the React-with-Redux project template included in ASP.NET Core 2.0.</span></span> <span data-ttu-id="bd7ad-105">Chodzi o nowszej platformy React z Redux szablonu, do której można zaktualizować ręcznie.</span><span class="sxs-lookup"><span data-stu-id="bd7ad-105">It's about the newer React-with-Redux template to which you can update manually.</span></span> <span data-ttu-id="bd7ad-106">Domyślnie znajduje szablonu platformy ASP.NET Core 2.1.</span><span class="sxs-lookup"><span data-stu-id="bd7ad-106">The template is included in ASP.NET Core 2.1 by default.</span></span>

::: moniker-end

<span data-ttu-id="bd7ad-107">Zaktualizowany szablon projektu platformy React z Redux udostępnia dogodny punkt początkowy dla aplikacji platformy ASP.NET Core za pomocą zareagować Redux, i [utworzyć platformy react aplikacji](https://github.com/facebookincubator/create-react-app) konwencje (CRA) do zaimplementowania rozbudowanego klienta interfejsu użytkownika (UI).</span><span class="sxs-lookup"><span data-stu-id="bd7ad-107">The updated React-with-Redux project template provides a convenient starting point for ASP.NET Core apps using React, Redux, and [create-react-app](https://github.com/facebookincubator/create-react-app) (CRA) conventions to implement a rich, client-side user interface (UI).</span></span>

<span data-ttu-id="bd7ad-108">Z wyjątkiem polecenia tworzenia projektu wszystkie informacje o bibliotece React z Redux szablonu jest taka sama jak szablon platformy React.</span><span class="sxs-lookup"><span data-stu-id="bd7ad-108">With the exception of the project creation command, all information about the React-with-Redux template is the same as the React template.</span></span> <span data-ttu-id="bd7ad-109">Aby utworzyć ten typ projektu, uruchom `dotnet new reactredux` zamiast `dotnet new react`.</span><span class="sxs-lookup"><span data-stu-id="bd7ad-109">To create this project type, run `dotnet new reactredux` instead of `dotnet new react`.</span></span> <span data-ttu-id="bd7ad-110">Aby uzyskać więcej informacji o funkcjach wspólne dla obu szablony oparte na bibliotece React, zobacz [zareagować dokumentacji dotyczącej szablonu](xref:spa/react).</span><span class="sxs-lookup"><span data-stu-id="bd7ad-110">For more information about the functionality common to both React-based templates, see [React template documentation](xref:spa/react).</span></span>