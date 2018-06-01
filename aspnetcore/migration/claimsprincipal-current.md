---
title: Migrar do ClaimsPrincipal.Current
author: mjrousos
description: Saiba como afastar ClaimsPrincipal.Current para recuperar declarações no núcleo do ASP.NET e a identidade do usuário autenticado atual.
manager: wpickett
ms.author: scaddie
ms.custom: mvc
ms.date: 05/04/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: migration/claimsprincipal-current
ms.openlocfilehash: ea43d17e76380baf57cd9debbc508e8812cfa4a6
ms.sourcegitcommit: 477d38e33530a305405eaf19faa29c6d805273aa
ms.contentlocale: pt-BR
ms.lasthandoff: 05/08/2018
---
# <a name="migrate-from-claimsprincipalcurrent"></a><span data-ttu-id="29f2c-103">Migrar do ClaimsPrincipal.Current</span><span class="sxs-lookup"><span data-stu-id="29f2c-103">Migrate from ClaimsPrincipal.Current</span></span>

<span data-ttu-id="29f2c-104">Em projetos do ASP.NET, é comum usar [ClaimsPrincipal.Current](/dotnet/api/system.security.claims.claimsprincipal.current) recuperar atual autenticado a identidade do usuário e declarações.</span><span class="sxs-lookup"><span data-stu-id="29f2c-104">In ASP.NET projects, it was common to use [ClaimsPrincipal.Current](/dotnet/api/system.security.claims.claimsprincipal.current) to retrieve the current authenticated user's identity and claims.</span></span> <span data-ttu-id="29f2c-105">No núcleo do ASP.NET, esta propriedade não está definida.</span><span class="sxs-lookup"><span data-stu-id="29f2c-105">In ASP.NET Core, this property is no longer set.</span></span> <span data-ttu-id="29f2c-106">Código que foi dependem precisa ser atualizado para obter a identidade do usuário autenticado atual por meio de uma maneira diferente.</span><span class="sxs-lookup"><span data-stu-id="29f2c-106">Code that was depending on it needs to be updated to get the current authenticated user's identity through a different means.</span></span>

## <a name="context-specific-data-instead-of-static-data"></a><span data-ttu-id="29f2c-107">Dados de contexto específico em vez de dados estáticos</span><span class="sxs-lookup"><span data-stu-id="29f2c-107">Context-specific data instead of static data</span></span>

<span data-ttu-id="29f2c-108">Ao usar o ASP.NET Core, os valores de `ClaimsPrincipal.Current` e `Thread.CurrentPrincipal` não estão definidos.</span><span class="sxs-lookup"><span data-stu-id="29f2c-108">When using ASP.NET Core, the values of both `ClaimsPrincipal.Current` and `Thread.CurrentPrincipal` aren't set.</span></span> <span data-ttu-id="29f2c-109">Essas propriedades representam o estado estático, que normalmente evita o ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="29f2c-109">These properties both represent static state, which ASP.NET Core generally avoids.</span></span> <span data-ttu-id="29f2c-110">Em vez disso, arquitetura do ASP.NET Core é recuperar dependências (como a identidade do usuário atual) de coleções de contexto específico de serviço (usando sua [injeção de dependência](xref:fundamentals/dependency-injection) modelo (DI)).</span><span class="sxs-lookup"><span data-stu-id="29f2c-110">Instead, ASP.NET Core's architecture is to retrieve dependencies (like the current user's identity) from context-specific service collections (using its [dependency injection](xref:fundamentals/dependency-injection) (DI) model).</span></span> <span data-ttu-id="29f2c-111">Além disso, `Thread.CurrentPrincipal` é thread estático, portanto ele não pode persistir as alterações em alguns cenários assíncronas (e `ClaimsPrincipal.Current` apenas chama `Thread.CurrentPrincipal` por padrão).</span><span class="sxs-lookup"><span data-stu-id="29f2c-111">What's more, `Thread.CurrentPrincipal` is thread static, so it may not persist changes in some asynchronous scenarios (and `ClaimsPrincipal.Current` just calls `Thread.CurrentPrincipal` by default).</span></span>

<span data-ttu-id="29f2c-112">Para entender as classificações do thread de problemas de membros estáticos podem levar a cenários assíncronos, considere o seguinte trecho de código:</span><span class="sxs-lookup"><span data-stu-id="29f2c-112">To understand the sorts of problems thread static members can lead to in asynchronous scenarios, consider the following code snippet:</span></span>

```csharp
// Create a ClaimsPrincipal and set Thread.CurrentPrincipal
var identity = new ClaimsIdentity();
identity.AddClaim(new Claim(ClaimTypes.Name, "User1"));
Thread.CurrentPrincipal = new ClaimsPrincipal(identity);

// Check the current user
Console.WriteLine($"Current user: {Thread.CurrentPrincipal?.Identity.Name}");

// For the method to complete asynchronously
await Task.Yield();

// Check the current user after
Console.WriteLine($"Current user: {Thread.CurrentPrincipal?.Identity.Name}");
```

<span data-ttu-id="29f2c-113">Os conjuntos de código de exemplo anterior `Thread.CurrentPrincipal` e verifica seu valor antes e depois aguardando uma chamada assíncrona.</span><span class="sxs-lookup"><span data-stu-id="29f2c-113">The preceding sample code sets `Thread.CurrentPrincipal` and checks its value before and after awaiting an asynchronous call.</span></span> <span data-ttu-id="29f2c-114">`Thread.CurrentPrincipal` é específico para o *thread* no qual ele está definido e o método provavelmente retomar a execução em um thread diferente após o await.</span><span class="sxs-lookup"><span data-stu-id="29f2c-114">`Thread.CurrentPrincipal` is specific to the *thread* on which it's set, and the method is likely to resume execution on a different thread after the await.</span></span> <span data-ttu-id="29f2c-115">Consequentemente, `Thread.CurrentPrincipal` está presente quando ele é verificado, mas nulo após a chamada a `await Task.Yield()`.</span><span class="sxs-lookup"><span data-stu-id="29f2c-115">Consequently, `Thread.CurrentPrincipal` is present when it's first checked but is null after the call to `await Task.Yield()`.</span></span>

<span data-ttu-id="29f2c-116">Obter a identidade do usuário atual da coleção de serviço de injeção de dependência do aplicativo é mais testável, muito, desde que as identidades de teste podem ser facilmente inseridas.</span><span class="sxs-lookup"><span data-stu-id="29f2c-116">Getting the current user's identity from the app's DI service collection is more testable, too, since test identities can be easily injected.</span></span>

## <a name="retrieve-the-current-user-in-an-aspnet-core-app"></a><span data-ttu-id="29f2c-117">Recuperar o usuário atual em um aplicativo do ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="29f2c-117">Retrieve the current user in an ASP.NET Core app</span></span>

<span data-ttu-id="29f2c-118">Há várias opções para recuperar o atual usuário autenticado `ClaimsPrincipal` no ASP.NET Core no lugar de `ClaimsPrincipal.Current`:</span><span class="sxs-lookup"><span data-stu-id="29f2c-118">There are several options for retrieving the current authenticated user's `ClaimsPrincipal` in ASP.NET Core in place of `ClaimsPrincipal.Current`:</span></span>

* <span data-ttu-id="29f2c-119">**ControllerBase.User**.</span><span class="sxs-lookup"><span data-stu-id="29f2c-119">**ControllerBase.User**.</span></span> <span data-ttu-id="29f2c-120">Controladores MVC podem acessar o usuário autenticado atual com seus [usuário](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.user) propriedade.</span><span class="sxs-lookup"><span data-stu-id="29f2c-120">MVC controllers can access the current authenticated user with their [User](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.user) property.</span></span>
* <span data-ttu-id="29f2c-121">**HttpContext**.</span><span class="sxs-lookup"><span data-stu-id="29f2c-121">**HttpContext.User**.</span></span> <span data-ttu-id="29f2c-122">Componentes com acesso ao atual `HttpContext` (middleware, por exemplo) pode obter o usuário atual `ClaimsPrincipal` de [HttpContext](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user).</span><span class="sxs-lookup"><span data-stu-id="29f2c-122">Components with access to the current `HttpContext` (middleware, for example) can get the current user's `ClaimsPrincipal` from [HttpContext.User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user).</span></span>
* <span data-ttu-id="29f2c-123">**Passado do chamador**.</span><span class="sxs-lookup"><span data-stu-id="29f2c-123">**Passed in from caller**.</span></span> <span data-ttu-id="29f2c-124">Bibliotecas sem acesso ao atual `HttpContext` são chamados de controladores ou componentes de middleware e pode ter a identidade do usuário atual passada como um argumento.</span><span class="sxs-lookup"><span data-stu-id="29f2c-124">Libraries without access to the current `HttpContext` are often called from controllers or middleware components and can have the current user's identity passed as an argument.</span></span>
* <span data-ttu-id="29f2c-125">**IHttpContextAccessor**.</span><span class="sxs-lookup"><span data-stu-id="29f2c-125">**IHttpContextAccessor**.</span></span> <span data-ttu-id="29f2c-126">O projeto ASP.NET que estão sendo migrado para o ASP.NET Core pode ser muito grande para passar facilmente a identidade do usuário atual para todos os locais necessários.</span><span class="sxs-lookup"><span data-stu-id="29f2c-126">The ASP.NET project being migrated to ASP.NET Core may be too large to easily pass the current user's identity to all necessary locations.</span></span> <span data-ttu-id="29f2c-127">Nesses casos, [IHttpContextAccessor](/dotnet/api/microsoft.aspnetcore.http.ihttpcontextaccessor) pode ser usado como uma solução alternativa.</span><span class="sxs-lookup"><span data-stu-id="29f2c-127">In such cases, [IHttpContextAccessor](/dotnet/api/microsoft.aspnetcore.http.ihttpcontextaccessor) can be used as a workaround.</span></span> <span data-ttu-id="29f2c-128">`IHttpContextAccessor` é capaz de acessar atual `HttpContext` (se houver).</span><span class="sxs-lookup"><span data-stu-id="29f2c-128">`IHttpContextAccessor` is able to access the current `HttpContext` (if one exists).</span></span> <span data-ttu-id="29f2c-129">Uma solução de curto prazo para obter a identidade do usuário atual no código que ainda não foi atualizado para funcionar com a arquitetura de controlado por DI do ASP.NET Core seria:</span><span class="sxs-lookup"><span data-stu-id="29f2c-129">A short-term solution to getting the current user's identity in code that hasn't yet been updated to work with ASP.NET Core's DI-driven architecture would be:</span></span>

  * <span data-ttu-id="29f2c-130">Verifique `IHttpContextAccessor` disponível no contêiner de DI chamando [AddHttpContextAccessor](https://github.com/aspnet/Hosting/issues/793) em `Startup.ConfigureServices`.</span><span class="sxs-lookup"><span data-stu-id="29f2c-130">Make `IHttpContextAccessor` available in the DI container by calling [AddHttpContextAccessor](https://github.com/aspnet/Hosting/issues/793) in `Startup.ConfigureServices`.</span></span>
  * <span data-ttu-id="29f2c-131">Obter uma instância de `IHttpContextAccessor` durante a inicialização e armazená-lo em uma variável estática.</span><span class="sxs-lookup"><span data-stu-id="29f2c-131">Get an instance of `IHttpContextAccessor` during startup and store it in a static variable.</span></span> <span data-ttu-id="29f2c-132">A instância é disponibilizada para o código que anteriormente estava recuperando o usuário atual de uma propriedade estática.</span><span class="sxs-lookup"><span data-stu-id="29f2c-132">The instance is made available to code that was previously retrieving the current user from a static property.</span></span>
  * <span data-ttu-id="29f2c-133">Recuperar o usuário atual `ClaimsPrincipal` usando `HttpContextAccessor.HttpContext?.User`.</span><span class="sxs-lookup"><span data-stu-id="29f2c-133">Retrieve the current user's `ClaimsPrincipal` using `HttpContextAccessor.HttpContext?.User`.</span></span> <span data-ttu-id="29f2c-134">Se esse código for usado fora do contexto de uma solicitação HTTP, o `HttpContext` é nulo.</span><span class="sxs-lookup"><span data-stu-id="29f2c-134">If this code is used outside of the context of an HTTP request, the `HttpContext` is null.</span></span>

<span data-ttu-id="29f2c-135">O último opção usando `IHttpContextAccessor`, infringir princípios do ASP.NET Core (preferindo dependências injetadas dependências estáticas).</span><span class="sxs-lookup"><span data-stu-id="29f2c-135">The final option, using `IHttpContextAccessor`, is contrary to ASP.NET Core principles (preferring injected dependencies to static dependencies).</span></span> <span data-ttu-id="29f2c-136">Planejar eventualmente remover a dependência estática `IHttpContextAccessor` auxiliar.</span><span class="sxs-lookup"><span data-stu-id="29f2c-136">Plan to eventually remove the dependency on the static `IHttpContextAccessor` helper.</span></span> <span data-ttu-id="29f2c-137">Pode ser uma ponte útil, no entanto, ao migrar grandes aplicativos ASP.NET existentes que foram anteriormente usando `ClaimsPrincipal.Current`.</span><span class="sxs-lookup"><span data-stu-id="29f2c-137">It can be a useful bridge, though, when migrating large existing ASP.NET apps that were previously using `ClaimsPrincipal.Current`.</span></span>