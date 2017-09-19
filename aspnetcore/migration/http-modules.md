---
title: "Migrando manipuladores HTTP e módulos ASP.NET Core middleware"
author: rick-anderson
description: 
keywords: ASP.NET Core,
ms.author: tdykstra
manager: wpickett
ms.date: 12/07/2016
ms.topic: article
ms.assetid: 9c826a76-fbd2-46b5-978d-6ca6df53531a
ms.technology: aspnet
ms.prod: asp.net-core
uid: migration/http-modules
ms.openlocfilehash: e14664133abf010b80374036e4855fdff71d1d5f
ms.sourcegitcommit: 9cdbfd0d670d70b9c354216aabee260c52dad5ee
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/12/2017
---
# <a name="migrating-http-handlers-and-modules-to-aspnet-core-middleware"></a><span data-ttu-id="474fa-103">Migrando manipuladores HTTP e módulos ASP.NET Core middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-103">Migrating HTTP handlers and modules to ASP.NET Core middleware</span></span> 

<span data-ttu-id="474fa-104">Por [Matt Perdeck](https://www.linkedin.com/in/mattperdeck)</span><span class="sxs-lookup"><span data-stu-id="474fa-104">By [Matt Perdeck](https://www.linkedin.com/in/mattperdeck)</span></span>

<span data-ttu-id="474fa-105">Este artigo mostra como migrar ASP.NET existente [módulos HTTP e manipuladores de System. webServer](https://docs.microsoft.com/iis/configuration/system.webserver/) para ASP.NET Core [middleware](../fundamentals/middleware.md).</span><span class="sxs-lookup"><span data-stu-id="474fa-105">This article shows how to migrate existing ASP.NET [HTTP modules and handlers from system.webserver](https://docs.microsoft.com/iis/configuration/system.webserver/) to ASP.NET Core [middleware](../fundamentals/middleware.md).</span></span>

## <a name="modules-and-handlers-revisited"></a><span data-ttu-id="474fa-106">Manipuladores revisitados e módulos</span><span class="sxs-lookup"><span data-stu-id="474fa-106">Modules and handlers revisited</span></span>

<span data-ttu-id="474fa-107">Antes de prosseguir para o ASP.NET Core middleware, vejamos primeiro novamente como manipuladores e módulos HTTP funcionam:</span><span class="sxs-lookup"><span data-stu-id="474fa-107">Before proceeding to ASP.NET Core middleware, let's first recap how HTTP modules and handlers work:</span></span>

![Manipulador de módulos](http-modules/_static/moduleshandlers.png)

<span data-ttu-id="474fa-109">**Manipuladores são:**</span><span class="sxs-lookup"><span data-stu-id="474fa-109">**Handlers are:**</span></span>

   * <span data-ttu-id="474fa-110">As classes que implementam [IHttpHandler](https://docs.microsoft.com/dotnet/api/system.web.ihttphandler)</span><span class="sxs-lookup"><span data-stu-id="474fa-110">Classes that implement [IHttpHandler](https://docs.microsoft.com/dotnet/api/system.web.ihttphandler)</span></span>

   * <span data-ttu-id="474fa-111">Usado para manipular solicitações com uma extensão, ou o nome de arquivo fornecido como *relatório*</span><span class="sxs-lookup"><span data-stu-id="474fa-111">Used to handle requests with a given file name or extension, such as *.report*</span></span>

   * <span data-ttu-id="474fa-112">[Configurado](https://docs.microsoft.com//iis/configuration/system.webserver/handlers/) em *Web. config*</span><span class="sxs-lookup"><span data-stu-id="474fa-112">[Configured](https://docs.microsoft.com//iis/configuration/system.webserver/handlers/) in *Web.config*</span></span>

<span data-ttu-id="474fa-113">**Os módulos são:**</span><span class="sxs-lookup"><span data-stu-id="474fa-113">**Modules are:**</span></span>

   * <span data-ttu-id="474fa-114">As classes que implementam [IHttpModule](https://docs.microsoft.com/dotnet/api/system.web.ihttpmodule)</span><span class="sxs-lookup"><span data-stu-id="474fa-114">Classes that implement [IHttpModule](https://docs.microsoft.com/dotnet/api/system.web.ihttpmodule)</span></span>

   * <span data-ttu-id="474fa-115">Chamado para cada solicitação</span><span class="sxs-lookup"><span data-stu-id="474fa-115">Invoked for every request</span></span>

   * <span data-ttu-id="474fa-116">Capaz de curto-circuito (Interromper processamento adicional de uma solicitação)</span><span class="sxs-lookup"><span data-stu-id="474fa-116">Able to short-circuit (stop further processing of a request)</span></span>

   * <span data-ttu-id="474fa-117">Capaz de adicionar a resposta HTTP, ou criar seus próprios</span><span class="sxs-lookup"><span data-stu-id="474fa-117">Able to add to the HTTP response, or create their own</span></span>

   * <span data-ttu-id="474fa-118">[Configurado](https://docs.microsoft.com//iis/configuration/system.webserver/modules/) em *Web. config*</span><span class="sxs-lookup"><span data-stu-id="474fa-118">[Configured](https://docs.microsoft.com//iis/configuration/system.webserver/modules/) in *Web.config*</span></span>

<span data-ttu-id="474fa-119">**A ordem em que os módulos de processam solicitações de entrada é determinada por:**</span><span class="sxs-lookup"><span data-stu-id="474fa-119">**The order in which modules process incoming requests is determined by:**</span></span>

   1. <span data-ttu-id="474fa-120">O [ciclo de vida do aplicativo](https://msdn.microsoft.com/library/ms227673.aspx), que é um eventos série acionado pelo ASP.NET: [BeginRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.beginrequest), [AuthenticateRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.authenticaterequest), etc. Cada módulo pode criar um manipulador de eventos de um ou mais.</span><span class="sxs-lookup"><span data-stu-id="474fa-120">The [application life cycle](https://msdn.microsoft.com/library/ms227673.aspx), which is a series events fired by ASP.NET: [BeginRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.beginrequest), [AuthenticateRequest](https://docs.microsoft.com/dotnet/api/system.web.httpapplication.authenticaterequest), etc. Each module can create a handler for one or more events.</span></span>

   2. <span data-ttu-id="474fa-121">Para o mesmo evento, a ordem na qual eles são configurados em *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="474fa-121">For the same event, the order in which they are configured in *Web.config*.</span></span>

<span data-ttu-id="474fa-122">Além de módulos, você pode adicionar manipuladores para os eventos de ciclo de vida para o *Global.asax.cs* arquivo.</span><span class="sxs-lookup"><span data-stu-id="474fa-122">In addition to modules, you can add handlers for the life cycle events to your *Global.asax.cs* file.</span></span> <span data-ttu-id="474fa-123">Esses manipuladores executar após os manipuladores nos módulos configurados.</span><span class="sxs-lookup"><span data-stu-id="474fa-123">These handlers run after the handlers in the configured modules.</span></span>

## <a name="from-handlers-and-modules-to-middleware"></a><span data-ttu-id="474fa-124">Manipuladores e módulos de middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-124">From handlers and modules to middleware</span></span>

<span data-ttu-id="474fa-125">**Middleware são mais simples do que manipuladores e módulos HTTP:**</span><span class="sxs-lookup"><span data-stu-id="474fa-125">**Middleware are simpler than HTTP modules and handlers:**</span></span>

   * <span data-ttu-id="474fa-126">Módulos, manipuladores, *Global.asax.cs*, *Web. config* (exceto para a configuração do IIS) e o ciclo de vida do aplicativo serão excluídos</span><span class="sxs-lookup"><span data-stu-id="474fa-126">Modules, handlers, *Global.asax.cs*, *Web.config* (except for IIS configuration) and the application life cycle are gone</span></span>

   * <span data-ttu-id="474fa-127">As funções dos módulos e manipuladores de tem sido controladas por middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-127">The roles of both modules and handlers have been taken over by middleware</span></span>

   * <span data-ttu-id="474fa-128">Middleware são configurados usando o código em vez de em *Web. config*</span><span class="sxs-lookup"><span data-stu-id="474fa-128">Middleware are configured using code rather than in *Web.config*</span></span>

   * <span data-ttu-id="474fa-129">[Ramificação de pipeline](../fundamentals/middleware.md#middleware-run-map-use) permite que você envia solicitações ao middleware específico, com base na URL não só mas também em cabeçalhos de solicitação, cadeias de caracteres de consulta, etc.</span><span class="sxs-lookup"><span data-stu-id="474fa-129">[Pipeline branching](../fundamentals/middleware.md#middleware-run-map-use) lets you send requests to specific middleware, based on not only the URL but also on request headers, query strings, etc.</span></span>

<span data-ttu-id="474fa-130">**Middleware são muito semelhantes aos módulos:**</span><span class="sxs-lookup"><span data-stu-id="474fa-130">**Middleware are very similar to modules:**</span></span>

   * <span data-ttu-id="474fa-131">Invocado em princípio para cada solicitação</span><span class="sxs-lookup"><span data-stu-id="474fa-131">Invoked in principle for every request</span></span>

   * <span data-ttu-id="474fa-132">Capaz de curto-circuito a uma solicitação por [não passar a solicitação para o próximo middleware](#http-modules-shortcircuiting-middleware)</span><span class="sxs-lookup"><span data-stu-id="474fa-132">Able to short-circuit a request, by [not passing the request to the next middleware](#http-modules-shortcircuiting-middleware)</span></span>

   * <span data-ttu-id="474fa-133">Capaz de criar sua própria resposta HTTP</span><span class="sxs-lookup"><span data-stu-id="474fa-133">Able to create their own HTTP response</span></span>

<span data-ttu-id="474fa-134">**Middleware e módulos são processados em uma ordem diferente:**</span><span class="sxs-lookup"><span data-stu-id="474fa-134">**Middleware and modules are processed in a different order:**</span></span>

   * <span data-ttu-id="474fa-135">Ordem de middleware é baseada na ordem em que são inseridos no pipeline de solicitação, enquanto a ordem dos módulos baseia-se principalmente em [ciclo de vida do aplicativo](https://msdn.microsoft.com/library/ms227673.aspx) eventos</span><span class="sxs-lookup"><span data-stu-id="474fa-135">Order of middleware is based on the order in which they are inserted into the request pipeline, while order of modules is mainly based on [application life cycle](https://msdn.microsoft.com/library/ms227673.aspx) events</span></span>

   * <span data-ttu-id="474fa-136">Ordem de middleware para respostas é o oposto do que para solicitações, enquanto a ordem dos módulos é o mesmo para solicitações e respostas</span><span class="sxs-lookup"><span data-stu-id="474fa-136">Order of middleware for responses is the reverse from that for requests, while order of modules is the same for requests and responses</span></span>

   * <span data-ttu-id="474fa-137">Consulte [criando um pipeline de middleware com IApplicationBuilder](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder)</span><span class="sxs-lookup"><span data-stu-id="474fa-137">See [Creating a middleware pipeline with IApplicationBuilder](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder)</span></span>

![Middleware](http-modules/_static/middleware.png)

<span data-ttu-id="474fa-139">Observe como na imagem acima, o middleware de autenticação curto-circuito a solicitação.</span><span class="sxs-lookup"><span data-stu-id="474fa-139">Note how in the image above, the authentication middleware short-circuited the request.</span></span>

## <a name="migrating-module-code-to-middleware"></a><span data-ttu-id="474fa-140">Migrando o código de módulo para middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-140">Migrating module code to middleware</span></span>

<span data-ttu-id="474fa-141">Um módulo HTTP existente será semelhante a este:</span><span class="sxs-lookup"><span data-stu-id="474fa-141">An existing HTTP module will look similar to this:</span></span>

<span data-ttu-id="474fa-142">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyModule.cs?highlight=6,8,24,31)]</span><span class="sxs-lookup"><span data-stu-id="474fa-142">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyModule.cs?highlight=6,8,24,31)]</span></span>

<span data-ttu-id="474fa-143">Conforme mostrado no [Middleware](../fundamentals/middleware.md) página, um middleware ASP.NET Core é uma classe que expõe um `Invoke` colocando método um `HttpContext` e retornar um `Task`.</span><span class="sxs-lookup"><span data-stu-id="474fa-143">As shown in the [Middleware](../fundamentals/middleware.md) page, an ASP.NET Core middleware is a class that exposes an `Invoke` method taking an `HttpContext` and returning a `Task`.</span></span> <span data-ttu-id="474fa-144">Seu novo middleware será assim:</span><span class="sxs-lookup"><span data-stu-id="474fa-144">Your new middleware will look like this:</span></span>

<a name=http-modules-usemiddleware></a>

<span data-ttu-id="474fa-145">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddleware.cs?highlight=9,13,20,24,28,30,32)]</span><span class="sxs-lookup"><span data-stu-id="474fa-145">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddleware.cs?highlight=9,13,20,24,28,30,32)]</span></span>

<span data-ttu-id="474fa-146">O modelo de middleware acima foi obtido da seção de [gravar middleware](../fundamentals/middleware.md#middleware-writing-middleware).</span><span class="sxs-lookup"><span data-stu-id="474fa-146">The above middleware template was taken from the section on [writing middleware](../fundamentals/middleware.md#middleware-writing-middleware).</span></span>

<span data-ttu-id="474fa-147">O *MyMiddlewareExtensions* classe auxiliar torna mais fácil de configurar o middleware em seu `Startup` classe.</span><span class="sxs-lookup"><span data-stu-id="474fa-147">The *MyMiddlewareExtensions* helper class makes it easier to configure your middleware in your `Startup` class.</span></span> <span data-ttu-id="474fa-148">O `UseMyMiddleware` método adiciona sua classe de middleware no pipeline de solicitação.</span><span class="sxs-lookup"><span data-stu-id="474fa-148">The `UseMyMiddleware` method adds your middleware class to the request pipeline.</span></span> <span data-ttu-id="474fa-149">Serviços necessários para o middleware são injetados no construtor do middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-149">Services required by the middleware get injected in the middleware's constructor.</span></span>

<a name=http-modules-shortcircuiting-middleware></a>

<span data-ttu-id="474fa-150">O módulo pode encerrar uma solicitação, por exemplo, se o usuário não está autorizado:</span><span class="sxs-lookup"><span data-stu-id="474fa-150">Your module might terminate a request, for example if the user is not authorized:</span></span>

<span data-ttu-id="474fa-151">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyTerminatingModule.cs?highlight=9,10,11,12,13&name=snippet_Terminate)]</span><span class="sxs-lookup"><span data-stu-id="474fa-151">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyTerminatingModule.cs?highlight=9,10,11,12,13&name=snippet_Terminate)]</span></span>

<span data-ttu-id="474fa-152">Um middleware lida com isso chamando não `Invoke` no próximo middleware no pipeline.</span><span class="sxs-lookup"><span data-stu-id="474fa-152">A middleware handles this by not calling `Invoke` on the next middleware in the pipeline.</span></span> <span data-ttu-id="474fa-153">Tenha em mente que isso não totalmente encerra a solicitação, pois middlewares anterior ainda será invocado quando a resposta torna sua maneira novamente por meio do pipeline.</span><span class="sxs-lookup"><span data-stu-id="474fa-153">Keep in mind that this does not fully terminate the request, because previous middlewares will still be invoked when the response makes its way back through the pipeline.</span></span>

<span data-ttu-id="474fa-154">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyTerminatingMiddleware.cs?highlight=7,8&name=snippet_Terminate)]</span><span class="sxs-lookup"><span data-stu-id="474fa-154">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyTerminatingMiddleware.cs?highlight=7,8&name=snippet_Terminate)]</span></span>

<span data-ttu-id="474fa-155">Ao migrar a funcionalidade do módulo para o seu middleware de novo, você pode achar que seu código não compilado porque o `HttpContext` classe tiverem sido alterados significativamente no núcleo do ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="474fa-155">When you migrate your module's functionality to your new middleware, you may find that your code doesn't compile because the `HttpContext` class has significantly changed in ASP.NET Core.</span></span> <span data-ttu-id="474fa-156">[Mais tarde](#migrating-to-the-new-httpcontext), você verá como migrar para o ASP.NET Core HttpContext novo.</span><span class="sxs-lookup"><span data-stu-id="474fa-156">[Later on](#migrating-to-the-new-httpcontext), you'll see how to migrate to the new ASP.NET Core HttpContext.</span></span>

## <a name="migrating-module-insertion-into-the-request-pipeline"></a><span data-ttu-id="474fa-157">Migrando inserção de módulo no pipeline de solicitação</span><span class="sxs-lookup"><span data-stu-id="474fa-157">Migrating module insertion into the request pipeline</span></span>

<span data-ttu-id="474fa-158">Módulos HTTP normalmente são adicionados ao pipeline de solicitação usando *Web. config*:</span><span class="sxs-lookup"><span data-stu-id="474fa-158">HTTP modules are typically added to the request pipeline using *Web.config*:</span></span>

<span data-ttu-id="474fa-159">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32-33,36,43,50,101)]</span><span class="sxs-lookup"><span data-stu-id="474fa-159">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32-33,36,43,50,101)]</span></span>

<span data-ttu-id="474fa-160">Converter isso por [adicionando seu novo middleware](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder) para o pipeline de solicitação no seu `Startup` classe:</span><span class="sxs-lookup"><span data-stu-id="474fa-160">Convert this by [adding your new middleware](../fundamentals/middleware.md#creating-a-middleware-pipeline-with-iapplicationbuilder) to the request pipeline in your `Startup` class:</span></span>

<span data-ttu-id="474fa-161">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=16)]</span><span class="sxs-lookup"><span data-stu-id="474fa-161">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=16)]</span></span>

<span data-ttu-id="474fa-162">Ponto no pipeline, em que você insere seu novo middleware exato depende do evento que ela tratada como um módulo (`BeginRequest`, `EndRequest`, etc.) e sua ordem na lista de módulos em *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="474fa-162">The exact spot in the pipeline where you insert your new middleware depends on the event that it handled as a module (`BeginRequest`, `EndRequest`, etc.) and its order in your list of modules in *Web.config*.</span></span>

<span data-ttu-id="474fa-163">Como anteriormente não mencionado, há mais nenhum ciclo de vida do aplicativo no núcleo do ASP.NET e a ordem na qual as respostas são processadas pelo middleware diferente daquela usada pelos módulos.</span><span class="sxs-lookup"><span data-stu-id="474fa-163">As previously stated, there is no application life cycle in ASP.NET Core and the order in which responses are processed by middleware differs from the order used by modules.</span></span> <span data-ttu-id="474fa-164">Isso pode tornar sua decisão de ordenação mais difícil.</span><span class="sxs-lookup"><span data-stu-id="474fa-164">This could make your ordering decision more challenging.</span></span>

<span data-ttu-id="474fa-165">Se ordenação se torna um problema, você pode dividir seu módulo em vários componentes de middleware que podem ser ordenados de forma independente.</span><span class="sxs-lookup"><span data-stu-id="474fa-165">If ordering becomes a problem, you could split your module into multiple middleware components that can be ordered independently.</span></span>

## <a name="migrating-handler-code-to-middleware"></a><span data-ttu-id="474fa-166">Migrando o código de manipulador de middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-166">Migrating handler code to middleware</span></span>

<span data-ttu-id="474fa-167">Um manipulador HTTP parecida com esta:</span><span class="sxs-lookup"><span data-stu-id="474fa-167">An HTTP handler looks something like this:</span></span>

<span data-ttu-id="474fa-168">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/HttpHandlers/ReportHandler.cs?highlight=5,7,13,14,15,16)]</span><span class="sxs-lookup"><span data-stu-id="474fa-168">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/HttpHandlers/ReportHandler.cs?highlight=5,7,13,14,15,16)]</span></span>

<span data-ttu-id="474fa-169">No seu projeto do ASP.NET Core, isso pode ser traduzido para um middleware semelhante a este:</span><span class="sxs-lookup"><span data-stu-id="474fa-169">In your ASP.NET Core project, you would translate this to a middleware similar to this:</span></span>

<span data-ttu-id="474fa-170">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/ReportHandlerMiddleware.cs?highlight=7,9,13,20,21,22,23,40,42,44)]</span><span class="sxs-lookup"><span data-stu-id="474fa-170">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/ReportHandlerMiddleware.cs?highlight=7,9,13,20,21,22,23,40,42,44)]</span></span>

<span data-ttu-id="474fa-171">Este middleware é muito semelhante para o middleware correspondente a módulos.</span><span class="sxs-lookup"><span data-stu-id="474fa-171">This middleware is very similar to the middleware corresponding to modules.</span></span> <span data-ttu-id="474fa-172">A única diferença é que não há aqui nenhuma chamada para `_next.Invoke(context)`.</span><span class="sxs-lookup"><span data-stu-id="474fa-172">The only real difference is that here there is no call to `_next.Invoke(context)`.</span></span> <span data-ttu-id="474fa-173">Isso faz sentido, porque o manipulador não é no final do pipeline de solicitação, que será a nenhum próximo middleware para invocar.</span><span class="sxs-lookup"><span data-stu-id="474fa-173">That makes sense, because the handler is at the end of the request pipeline, so there will be no next middleware to invoke.</span></span>

## <a name="migrating-handler-insertion-into-the-request-pipeline"></a><span data-ttu-id="474fa-174">Migrando inserção manipulador no pipeline de solicitação</span><span class="sxs-lookup"><span data-stu-id="474fa-174">Migrating handler insertion into the request pipeline</span></span>

<span data-ttu-id="474fa-175">Configurar um manipulador HTTP é feita em *Web. config* e tem a seguinte aparência:</span><span class="sxs-lookup"><span data-stu-id="474fa-175">Configuring an HTTP handler is done in *Web.config* and looks something like this:</span></span>

<span data-ttu-id="474fa-176">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32,46-48,50,101)]</span><span class="sxs-lookup"><span data-stu-id="474fa-176">[!code-xml[Main](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32,46-48,50,101)]</span></span>

<span data-ttu-id="474fa-177">Você pode convertê-lo adicionando seu novo middleware de manipulador para o pipeline de solicitação no seu `Startup` classe, semelhante ao middleware convertido de módulos.</span><span class="sxs-lookup"><span data-stu-id="474fa-177">You could convert this by adding your new handler middleware to the request pipeline in your `Startup` class, similar to middleware converted from modules.</span></span> <span data-ttu-id="474fa-178">O problema com essa abordagem é que ele poderia enviar todas as solicitações para seu novo middleware de manipulador.</span><span class="sxs-lookup"><span data-stu-id="474fa-178">The problem with that approach is that it would send all requests to your new handler middleware.</span></span> <span data-ttu-id="474fa-179">No entanto, você precisará apenas solicitações com uma determinada extensão para alcançar seu middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-179">However, you only want requests with a given extension to reach your middleware.</span></span> <span data-ttu-id="474fa-180">Essa seria a mesma funcionalidade que tinha com o manipulador HTTP.</span><span class="sxs-lookup"><span data-stu-id="474fa-180">That would give you the same functionality you had with your HTTP handler.</span></span>

<span data-ttu-id="474fa-181">Uma solução é ramificar o pipeline de solicitações com uma determinada extensão, usando o `MapWhen` método de extensão.</span><span class="sxs-lookup"><span data-stu-id="474fa-181">One solution is to branch the pipeline for requests with a given extension, using the `MapWhen` extension method.</span></span> <span data-ttu-id="474fa-182">Você pode fazer isso no mesmo `Configure` método em que você adiciona o middleware outros:</span><span class="sxs-lookup"><span data-stu-id="474fa-182">You do this in the same `Configure` method where you add the other middleware:</span></span>

<span data-ttu-id="474fa-183">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=27-34)]</span><span class="sxs-lookup"><span data-stu-id="474fa-183">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=27-34)]</span></span>

<span data-ttu-id="474fa-184">`MapWhen`usa esses parâmetros:</span><span class="sxs-lookup"><span data-stu-id="474fa-184">`MapWhen` takes these parameters:</span></span>

1. <span data-ttu-id="474fa-185">Uma expressão lambda que usa o `HttpContext` e retorna `true` se a solicitação deve ir para a ramificação.</span><span class="sxs-lookup"><span data-stu-id="474fa-185">A lambda that takes the `HttpContext` and returns `true` if the request should go down the branch.</span></span> <span data-ttu-id="474fa-186">Isso significa que você pode se ramificar solicitações não apenas com base em sua extensão, mas também em cabeçalhos de solicitação, parâmetros de cadeia de caracteres de consulta, etc.</span><span class="sxs-lookup"><span data-stu-id="474fa-186">This means you can branch requests not just based on their extension, but also on request headers, query string parameters, etc.</span></span>

2. <span data-ttu-id="474fa-187">Uma expressão lambda que leva um `IApplicationBuilder` e adiciona todos o middleware para a ramificação.</span><span class="sxs-lookup"><span data-stu-id="474fa-187">A lambda that takes an `IApplicationBuilder` and adds all the middleware for the branch.</span></span> <span data-ttu-id="474fa-188">Isso significa que você pode adicionar middleware adicional para a ramificação na frente do seu manipulador middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-188">This means you can add additional middleware to the branch in front of your handler middleware.</span></span>

<span data-ttu-id="474fa-189">Middleware adicionados ao pipeline antes da ramificação será invocada em todas as solicitações; a ramificação não terá impacto sobre eles.</span><span class="sxs-lookup"><span data-stu-id="474fa-189">Middleware added to the pipeline before the branch will be invoked on all requests; the branch will have no impact on them.</span></span>

## <a name="loading-middleware-options-using-the-options-pattern"></a><span data-ttu-id="474fa-190">Opções de middleware usando o padrão de opções de carregamento</span><span class="sxs-lookup"><span data-stu-id="474fa-190">Loading middleware options using the options pattern</span></span>

<span data-ttu-id="474fa-191">Alguns módulos e manipuladores têm opções de configuração que são armazenadas em *Web. config*. No entanto, no ASP.NET Core um novo modelo de configuração é usado no lugar de *Web. config*.</span><span class="sxs-lookup"><span data-stu-id="474fa-191">Some modules and handlers have configuration options that are stored in *Web.config*. However, in ASP.NET Core a new configuration model is used in place of *Web.config*.</span></span>

<span data-ttu-id="474fa-192">O novo [sistema de configuração](../fundamentals/configuration.md) oferece essas opções para resolver isso:</span><span class="sxs-lookup"><span data-stu-id="474fa-192">The new [configuration system](../fundamentals/configuration.md) gives you these options to solve this:</span></span>

* <span data-ttu-id="474fa-193">Injetar diretamente as opções para o middleware, conforme o [próxima seção](#loading-middleware-options-through-direct-injection).</span><span class="sxs-lookup"><span data-stu-id="474fa-193">Directly inject the options into the middleware, as shown in the [next section](#loading-middleware-options-through-direct-injection).</span></span>

* <span data-ttu-id="474fa-194">Use o [padrão de opções](../fundamentals/configuration.md#options-config-objects):</span><span class="sxs-lookup"><span data-stu-id="474fa-194">Use the [options pattern](../fundamentals/configuration.md#options-config-objects):</span></span>

1.  <span data-ttu-id="474fa-195">Crie uma classe para manter suas opções de middleware, por exemplo:</span><span class="sxs-lookup"><span data-stu-id="474fa-195">Create a class to hold your middleware options, for example:</span></span>

    <span data-ttu-id="474fa-196">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Options)]</span><span class="sxs-lookup"><span data-stu-id="474fa-196">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Options)]</span></span>

2.  <span data-ttu-id="474fa-197">Armazenar os valores de opção</span><span class="sxs-lookup"><span data-stu-id="474fa-197">Store the option values</span></span>

    <span data-ttu-id="474fa-198">O sistema de configuração permite que você armazene valores de opção em qualquer local desejado.</span><span class="sxs-lookup"><span data-stu-id="474fa-198">The configuration system allows you to store option values anywhere you want.</span></span> <span data-ttu-id="474fa-199">No entanto, a maioria dos locais use *appSettings. JSON*, portanto, vamos essa abordagem:</span><span class="sxs-lookup"><span data-stu-id="474fa-199">However, most sites use *appsettings.json*, so we'll take that approach:</span></span>

    <span data-ttu-id="474fa-200">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,14-18)]</span><span class="sxs-lookup"><span data-stu-id="474fa-200">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,14-18)]</span></span>

    <span data-ttu-id="474fa-201">*MyMiddlewareOptionsSection* aqui é um nome de seção.</span><span class="sxs-lookup"><span data-stu-id="474fa-201">*MyMiddlewareOptionsSection* here is a section name.</span></span> <span data-ttu-id="474fa-202">Ele não precisa ser igual ao nome da sua classe de opções.</span><span class="sxs-lookup"><span data-stu-id="474fa-202">It doesn't have to be the same as the name of your options class.</span></span>

3. <span data-ttu-id="474fa-203">Associe os valores de opção com a classe de opções</span><span class="sxs-lookup"><span data-stu-id="474fa-203">Associate the option values with the options class</span></span>

    <span data-ttu-id="474fa-204">O padrão de opções usa a estrutura de injeção de dependência do ASP.NET Core para associar o tipo de opções (como `MyMiddlewareOptions`) com um `MyMiddlewareOptions` objeto que tem as opções reais.</span><span class="sxs-lookup"><span data-stu-id="474fa-204">The options pattern uses ASP.NET Core's dependency injection framework to associate the options type (such as `MyMiddlewareOptions`) with a `MyMiddlewareOptions` object that has the actual options.</span></span>

    <span data-ttu-id="474fa-205">Atualização de seu `Startup` classe:</span><span class="sxs-lookup"><span data-stu-id="474fa-205">Update your `Startup` class:</span></span>

    1.  <span data-ttu-id="474fa-206">Se você estiver usando *appSettings. JSON*, adicioná-lo para o construtor de configuração no `Startup` construtor:</span><span class="sxs-lookup"><span data-stu-id="474fa-206">If you're using *appsettings.json*, add it to the configuration builder in the `Startup` constructor:</span></span>

      <span data-ttu-id="474fa-207">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Ctor&highlight=5-6)]</span><span class="sxs-lookup"><span data-stu-id="474fa-207">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Ctor&highlight=5-6)]</span></span>

    2.  <span data-ttu-id="474fa-208">Configure o serviço de opções:</span><span class="sxs-lookup"><span data-stu-id="474fa-208">Configure the options service:</span></span>

      <span data-ttu-id="474fa-209">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=4)]</span><span class="sxs-lookup"><span data-stu-id="474fa-209">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=4)]</span></span>

    3.  <span data-ttu-id="474fa-210">Associe as opções de sua classe de opções:</span><span class="sxs-lookup"><span data-stu-id="474fa-210">Associate your options with your options class:</span></span>

      <span data-ttu-id="474fa-211">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=6-8)]</span><span class="sxs-lookup"><span data-stu-id="474fa-211">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=6-8)]</span></span>

4.  <span data-ttu-id="474fa-212">Inserir as opções para o construtor de middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-212">Inject the options into your middleware constructor.</span></span> <span data-ttu-id="474fa-213">Isso é semelhante a injeção de opções em um controlador.</span><span class="sxs-lookup"><span data-stu-id="474fa-213">This is similar to injecting options into a controller.</span></span>

  <span data-ttu-id="474fa-214">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_MiddlewareWithParams&highlight=4,7,10,15-16)]</span><span class="sxs-lookup"><span data-stu-id="474fa-214">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_MiddlewareWithParams&highlight=4,7,10,15-16)]</span></span>

  <span data-ttu-id="474fa-215">O [UseMiddleware](#http-modules-usemiddleware) método de extensão que adiciona o middleware para o `IApplicationBuilder` cuida de injeção de dependência.</span><span class="sxs-lookup"><span data-stu-id="474fa-215">The [UseMiddleware](#http-modules-usemiddleware) extension method that adds your middleware to the `IApplicationBuilder` takes care of dependency injection.</span></span>

  <span data-ttu-id="474fa-216">Isso não é limitado a `IOptions` objetos.</span><span class="sxs-lookup"><span data-stu-id="474fa-216">This is not limited to `IOptions` objects.</span></span> <span data-ttu-id="474fa-217">Qualquer objeto que requer o middleware pode ser inserido dessa maneira.</span><span class="sxs-lookup"><span data-stu-id="474fa-217">Any other object that your middleware requires can be injected this way.</span></span>

## <a name="loading-middleware-options-through-direct-injection"></a><span data-ttu-id="474fa-218">Carregando opções de middleware injeção direto</span><span class="sxs-lookup"><span data-stu-id="474fa-218">Loading middleware options through direct injection</span></span>

<span data-ttu-id="474fa-219">O padrão de opções tem a vantagem de que ele cria flexível acoplamento entre valores de opções e seus consumidores.</span><span class="sxs-lookup"><span data-stu-id="474fa-219">The options pattern has the advantage that it creates loose coupling between options values and their consumers.</span></span> <span data-ttu-id="474fa-220">Depois que você associou a uma classe de opções com os valores reais de opções, qualquer outra classe pode obter acesso às opções por meio da estrutura de injeção de dependência.</span><span class="sxs-lookup"><span data-stu-id="474fa-220">Once you've associated an options class with the actual options values, any other class can get access to the options through the dependency injection framework.</span></span> <span data-ttu-id="474fa-221">Não é necessário para passar em torno de valores de opções.</span><span class="sxs-lookup"><span data-stu-id="474fa-221">There is no need to pass around options values.</span></span>

<span data-ttu-id="474fa-222">Isso divide Embora se você quiser usar o mesmo middleware duas vezes, com opções diferentes.</span><span class="sxs-lookup"><span data-stu-id="474fa-222">This breaks down though if you want to use the same middleware twice, with different options.</span></span> <span data-ttu-id="474fa-223">Por exemplo um autorização middleware usado em diferentes ramificações, permitindo que diferentes funções.</span><span class="sxs-lookup"><span data-stu-id="474fa-223">For example an authorization middleware used in different branches allowing different roles.</span></span> <span data-ttu-id="474fa-224">Você não pode associar dois objetos diferentes opções com a classe de opções de um.</span><span class="sxs-lookup"><span data-stu-id="474fa-224">You can't associate two different options objects with the one options class.</span></span>

<span data-ttu-id="474fa-225">A solução é obter os objetos de opções com os valores de opções reais no seu `Startup` classe e passe-os diretamente a cada instância de seu middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-225">The solution is to get the options objects with the actual options values in your `Startup` class and pass those directly to each instance of your middleware.</span></span>

1.  <span data-ttu-id="474fa-226">Adicionar uma segunda chave para *appSettings. JSON*</span><span class="sxs-lookup"><span data-stu-id="474fa-226">Add a second key to *appsettings.json*</span></span>

    <span data-ttu-id="474fa-227">Para adicionar um segundo conjunto de opções para o *appSettings. JSON* de arquivo, use uma nova chave para identificá-lo exclusivamente:</span><span class="sxs-lookup"><span data-stu-id="474fa-227">To add a second set of options to the *appsettings.json* file, use a new key to uniquely identify it:</span></span>

    <span data-ttu-id="474fa-228">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,10-18&highlight=2-5)]</span><span class="sxs-lookup"><span data-stu-id="474fa-228">[!code-json[Main](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,10-18&highlight=2-5)]</span></span>

2.  <span data-ttu-id="474fa-229">Recuperar valores de opções e os transmite para o middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-229">Retrieve options values and pass them to middleware.</span></span> <span data-ttu-id="474fa-230">O `Use...` método de extensão (o que adiciona o middleware no pipeline) é um local lógico para transmitir os valores de opção:</span><span class="sxs-lookup"><span data-stu-id="474fa-230">The `Use...` extension method (which adds your middleware to the pipeline) is a logical place to pass in the option values:</span></span> 

    <span data-ttu-id="474fa-231">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=20-23)]</span><span class="sxs-lookup"><span data-stu-id="474fa-231">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=20-23)]</span></span>

4.  <span data-ttu-id="474fa-232">Habilite middleware para utilizar um parâmetro de opções.</span><span class="sxs-lookup"><span data-stu-id="474fa-232">Enable middleware to take an options parameter.</span></span> <span data-ttu-id="474fa-233">Forneça uma sobrecarga do `Use...` método de extensão (que usa o parâmetro options e passá-lo para `UseMiddleware`).</span><span class="sxs-lookup"><span data-stu-id="474fa-233">Provide an overload of the `Use...` extension method (that takes the options parameter and passes it to `UseMiddleware`).</span></span> <span data-ttu-id="474fa-234">Quando `UseMiddleware` é chamado com parâmetros, ele passa os parâmetros para o construtor de middleware quando ele cria uma instância do objeto de middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-234">When `UseMiddleware` is called with parameters, it passes the parameters to your middleware constructor when it instantiates the middleware object.</span></span>

    <span data-ttu-id="474fa-235">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Extensions&highlight=9-14)]</span><span class="sxs-lookup"><span data-stu-id="474fa-235">[!code-csharp[Main](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Extensions&highlight=9-14)]</span></span>

    <span data-ttu-id="474fa-236">Observe como isso encapsula o objeto de opções em um `OptionsWrapper` objeto.</span><span class="sxs-lookup"><span data-stu-id="474fa-236">Note how this wraps the options object in an `OptionsWrapper` object.</span></span> <span data-ttu-id="474fa-237">Isso implementa `IOptions`, conforme o esperado pelo construtor de middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-237">This implements `IOptions`, as expected by the middleware constructor.</span></span>

## <a name="migrating-to-the-new-httpcontext"></a><span data-ttu-id="474fa-238">Migrando para o novo HttpContext</span><span class="sxs-lookup"><span data-stu-id="474fa-238">Migrating to the new HttpContext</span></span>

<span data-ttu-id="474fa-239">Anteriormente, você viu que o `Invoke` método no seu middleware usa um parâmetro de tipo `HttpContext`:</span><span class="sxs-lookup"><span data-stu-id="474fa-239">You saw earlier that the `Invoke` method in your middleware takes a parameter of type `HttpContext`:</span></span>

```csharp
public async Task Invoke(HttpContext context)
```

<span data-ttu-id="474fa-240">`HttpContext`foi alterado significativamente no núcleo do ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="474fa-240">`HttpContext` has significantly changed in ASP.NET Core.</span></span> <span data-ttu-id="474fa-241">Esta seção mostra como converter as propriedades mais usadas de [System.Web.HttpContext](https://docs.microsoft.com/dotnet/api/system.web.httpcontext) para o novo `Microsoft.AspNetCore.Http.HttpContext`.</span><span class="sxs-lookup"><span data-stu-id="474fa-241">This section shows how to translate the most commonly used properties of [System.Web.HttpContext](https://docs.microsoft.com/dotnet/api/system.web.httpcontext) to the new `Microsoft.AspNetCore.Http.HttpContext`.</span></span>

### <a name="httpcontext"></a><span data-ttu-id="474fa-242">HttpContext</span><span class="sxs-lookup"><span data-stu-id="474fa-242">HttpContext</span></span>

<span data-ttu-id="474fa-243">**HttpContext** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-243">**HttpContext.Items** translates to:</span></span>

<span data-ttu-id="474fa-244">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Items)]</span><span class="sxs-lookup"><span data-stu-id="474fa-244">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Items)]</span></span>

<span data-ttu-id="474fa-245">**ID da solicitação exclusiva (nenhum equivalente System.Web.HttpContext)**</span><span class="sxs-lookup"><span data-stu-id="474fa-245">**Unique request ID (no System.Web.HttpContext counterpart)**</span></span>

<span data-ttu-id="474fa-246">Fornece uma id exclusiva para cada solicitação.</span><span class="sxs-lookup"><span data-stu-id="474fa-246">Gives you a unique id for each request.</span></span> <span data-ttu-id="474fa-247">Muito útil para incluir nos logs.</span><span class="sxs-lookup"><span data-stu-id="474fa-247">Very useful to include in your logs.</span></span>

<span data-ttu-id="474fa-248">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Trace)]</span><span class="sxs-lookup"><span data-stu-id="474fa-248">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Trace)]</span></span>

### <a name="httpcontextrequest"></a><span data-ttu-id="474fa-249">HttpContext.Request</span><span class="sxs-lookup"><span data-stu-id="474fa-249">HttpContext.Request</span></span>

<span data-ttu-id="474fa-250">**HttpContext.Request.HttpMethod** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-250">**HttpContext.Request.HttpMethod** translates to:</span></span>

<span data-ttu-id="474fa-251">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Method)]</span><span class="sxs-lookup"><span data-stu-id="474fa-251">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Method)]</span></span>

<span data-ttu-id="474fa-252">**HttpContext.Request.QueryString** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-252">**HttpContext.Request.QueryString** translates to:</span></span>

<span data-ttu-id="474fa-253">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Query)]</span><span class="sxs-lookup"><span data-stu-id="474fa-253">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Query)]</span></span>

<span data-ttu-id="474fa-254">**HttpContext.Request.Url** e **HttpContext.Request.RawUrl** traduzir para:</span><span class="sxs-lookup"><span data-stu-id="474fa-254">**HttpContext.Request.Url** and **HttpContext.Request.RawUrl** translate to:</span></span>

<span data-ttu-id="474fa-255">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Url)]</span><span class="sxs-lookup"><span data-stu-id="474fa-255">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Url)]</span></span>

<span data-ttu-id="474fa-256">**HttpContext.Request.IsSecureConnection** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-256">**HttpContext.Request.IsSecureConnection** translates to:</span></span>

<span data-ttu-id="474fa-257">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Secure)]</span><span class="sxs-lookup"><span data-stu-id="474fa-257">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Secure)]</span></span>

<span data-ttu-id="474fa-258">**HttpContext.Request.UserHostAddress** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-258">**HttpContext.Request.UserHostAddress** translates to:</span></span>

<span data-ttu-id="474fa-259">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Host)]</span><span class="sxs-lookup"><span data-stu-id="474fa-259">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Host)]</span></span>

<span data-ttu-id="474fa-260">**HttpContext.Request.Cookies** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-260">**HttpContext.Request.Cookies** translates to:</span></span>

<span data-ttu-id="474fa-261">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Cookies)]</span><span class="sxs-lookup"><span data-stu-id="474fa-261">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Cookies)]</span></span>

<span data-ttu-id="474fa-262">**HttpContext.Request.RequestContext.RouteData** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-262">**HttpContext.Request.RequestContext.RouteData** translates to:</span></span>

<span data-ttu-id="474fa-263">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Route)]</span><span class="sxs-lookup"><span data-stu-id="474fa-263">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Route)]</span></span>

<span data-ttu-id="474fa-264">**HttpContext.Request.Headers** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-264">**HttpContext.Request.Headers** translates to:</span></span>

<span data-ttu-id="474fa-265">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Headers)]</span><span class="sxs-lookup"><span data-stu-id="474fa-265">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Headers)]</span></span>

<span data-ttu-id="474fa-266">**HttpContext.Request.UserAgent** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-266">**HttpContext.Request.UserAgent** translates to:</span></span>

<span data-ttu-id="474fa-267">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Agent)]</span><span class="sxs-lookup"><span data-stu-id="474fa-267">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Agent)]</span></span>

<span data-ttu-id="474fa-268">**HttpContext.Request.UrlReferrer** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-268">**HttpContext.Request.UrlReferrer** translates to:</span></span>

<span data-ttu-id="474fa-269">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Referrer)]</span><span class="sxs-lookup"><span data-stu-id="474fa-269">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Referrer)]</span></span>

<span data-ttu-id="474fa-270">**HttpContext.Request.ContentType** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-270">**HttpContext.Request.ContentType** translates to:</span></span>

<span data-ttu-id="474fa-271">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Type)]</span><span class="sxs-lookup"><span data-stu-id="474fa-271">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Type)]</span></span>

<span data-ttu-id="474fa-272">**HttpContext.Request.Form** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-272">**HttpContext.Request.Form** translates to:</span></span>

<span data-ttu-id="474fa-273">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Form)]</span><span class="sxs-lookup"><span data-stu-id="474fa-273">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Form)]</span></span>

> [!WARNING]
> <span data-ttu-id="474fa-274">Valores de formulário de leitura somente se o tipo de conteúdo sub é *x-www-form-urlencoded* ou *dados do formulário*.</span><span class="sxs-lookup"><span data-stu-id="474fa-274">Read form values only if the content sub type is *x-www-form-urlencoded* or *form-data*.</span></span>

<span data-ttu-id="474fa-275">**HttpContext.Request.InputStream** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-275">**HttpContext.Request.InputStream** translates to:</span></span>

<span data-ttu-id="474fa-276">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Input)]</span><span class="sxs-lookup"><span data-stu-id="474fa-276">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Input)]</span></span>

> [!WARNING]
> <span data-ttu-id="474fa-277">Use esse código somente em um middleware de tipo de manipulador, no final de um pipeline.</span><span class="sxs-lookup"><span data-stu-id="474fa-277">Use this code only in a handler type middleware, at the end of a pipeline.</span></span>
>
><span data-ttu-id="474fa-278">Você pode ler o corpo bruto conforme mostrado acima apenas uma vez por solicitação.</span><span class="sxs-lookup"><span data-stu-id="474fa-278">You can read the raw body as shown above only once per request.</span></span> <span data-ttu-id="474fa-279">Middleware tentar ler o corpo após a primeira leitura lê um corpo vazio.</span><span class="sxs-lookup"><span data-stu-id="474fa-279">Middleware trying to read the body after the first read will read an empty body.</span></span>
>
><span data-ttu-id="474fa-280">Isso não se aplica a leitura de um formulário como mostrado anteriormente, porque isso é feito de um buffer.</span><span class="sxs-lookup"><span data-stu-id="474fa-280">This does not apply to reading a form as shown earlier, because that is done from a buffer.</span></span>

### <a name="httpcontextresponse"></a><span data-ttu-id="474fa-281">HttpContext.Response</span><span class="sxs-lookup"><span data-stu-id="474fa-281">HttpContext.Response</span></span>

<span data-ttu-id="474fa-282">**HttpContext.Response.Status** e **HttpContext.Response.StatusDescription** traduzir para:</span><span class="sxs-lookup"><span data-stu-id="474fa-282">**HttpContext.Response.Status** and **HttpContext.Response.StatusDescription** translate to:</span></span>

<span data-ttu-id="474fa-283">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Status)]</span><span class="sxs-lookup"><span data-stu-id="474fa-283">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Status)]</span></span>

<span data-ttu-id="474fa-284">**HttpContext.Response.ContentEncoding** e **HttpContext.Response.ContentType** traduzir para:</span><span class="sxs-lookup"><span data-stu-id="474fa-284">**HttpContext.Response.ContentEncoding** and **HttpContext.Response.ContentType** translate to:</span></span>

<span data-ttu-id="474fa-285">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespType)]</span><span class="sxs-lookup"><span data-stu-id="474fa-285">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespType)]</span></span>

<span data-ttu-id="474fa-286">**HttpContext.Response.ContentType** em seu próprio também se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-286">**HttpContext.Response.ContentType** on its own also translates to:</span></span>

<span data-ttu-id="474fa-287">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespTypeOnly)]</span><span class="sxs-lookup"><span data-stu-id="474fa-287">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespTypeOnly)]</span></span>

<span data-ttu-id="474fa-288">**HttpContext.Response.Output** se traduz em:</span><span class="sxs-lookup"><span data-stu-id="474fa-288">**HttpContext.Response.Output** translates to:</span></span>

<span data-ttu-id="474fa-289">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Output)]</span><span class="sxs-lookup"><span data-stu-id="474fa-289">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Output)]</span></span>

<span data-ttu-id="474fa-290">**HttpContext.Response.TransmitFile**</span><span class="sxs-lookup"><span data-stu-id="474fa-290">**HttpContext.Response.TransmitFile**</span></span>

<span data-ttu-id="474fa-291">Serviços de um arquivo é discutida [aqui](../fundamentals/request-features.md#middleware-and-request-features).</span><span class="sxs-lookup"><span data-stu-id="474fa-291">Serving up a file is discussed [here](../fundamentals/request-features.md#middleware-and-request-features).</span></span>

<span data-ttu-id="474fa-292">**HttpContext.Response.Headers**</span><span class="sxs-lookup"><span data-stu-id="474fa-292">**HttpContext.Response.Headers**</span></span>

<span data-ttu-id="474fa-293">Enviar cabeçalhos de resposta é complicada pelo fato de que se você defini-las depois que nada foi gravado no corpo da resposta, ele não serão enviados.</span><span class="sxs-lookup"><span data-stu-id="474fa-293">Sending response headers is complicated by the fact that if you set them after anything has been written to the response body, they will not be sent.</span></span>

<span data-ttu-id="474fa-294">A solução é definir um método de retorno de chamada que será chamado direita antes da gravação do início da resposta.</span><span class="sxs-lookup"><span data-stu-id="474fa-294">The solution is to set a callback method that will be called right before writing to the response starts.</span></span> <span data-ttu-id="474fa-295">Isso é feito melhor no início do `Invoke` método no seu middleware.</span><span class="sxs-lookup"><span data-stu-id="474fa-295">This is best done at the start of the `Invoke` method in your middleware.</span></span> <span data-ttu-id="474fa-296">É desse método de retorno de chamada que define os cabeçalhos de resposta.</span><span class="sxs-lookup"><span data-stu-id="474fa-296">It is this callback method that sets your response headers.</span></span>

<span data-ttu-id="474fa-297">O código a seguir define um método de retorno de chamada chamado `SetHeaders`:</span><span class="sxs-lookup"><span data-stu-id="474fa-297">The following code sets a callback method called `SetHeaders`:</span></span>

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

<span data-ttu-id="474fa-298">O `SetHeaders` método de retorno de chamada terá esta aparência:</span><span class="sxs-lookup"><span data-stu-id="474fa-298">The `SetHeaders` callback method would look like this:</span></span>

<span data-ttu-id="474fa-299">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetHeaders)]</span><span class="sxs-lookup"><span data-stu-id="474fa-299">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetHeaders)]</span></span>

<span data-ttu-id="474fa-300">**HttpContext.Response.Cookies**</span><span class="sxs-lookup"><span data-stu-id="474fa-300">**HttpContext.Response.Cookies**</span></span>

<span data-ttu-id="474fa-301">Cookies de viagem para o navegador em um *Set-Cookie* cabeçalho de resposta.</span><span class="sxs-lookup"><span data-stu-id="474fa-301">Cookies travel to the browser in a *Set-Cookie* response header.</span></span> <span data-ttu-id="474fa-302">Como resultado, envio de cookies requer o retorno de chamada mesmo usada para enviar cabeçalhos de resposta:</span><span class="sxs-lookup"><span data-stu-id="474fa-302">As a result, sending cookies requires the same callback as used for sending response headers:</span></span>

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetCookies, state: httpContext);
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

<span data-ttu-id="474fa-303">O `SetCookies` método de retorno de chamada deve ser semelhante ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="474fa-303">The `SetCookies` callback method would look like the following:</span></span>

<span data-ttu-id="474fa-304">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetCookies)]</span><span class="sxs-lookup"><span data-stu-id="474fa-304">[!code-csharp[Main](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetCookies)]</span></span>

## <a name="additional-resources"></a><span data-ttu-id="474fa-305">Recursos adicionais</span><span class="sxs-lookup"><span data-stu-id="474fa-305">Additional Resources</span></span>

* [<span data-ttu-id="474fa-306">Visão geral de módulos HTTP e de manipuladores HTTP</span><span class="sxs-lookup"><span data-stu-id="474fa-306">HTTP Handlers and HTTP Modules Overview</span></span>](https://docs.microsoft.com/iis/configuration/system.webserver/)

* [<span data-ttu-id="474fa-307">Configuração</span><span class="sxs-lookup"><span data-stu-id="474fa-307">Configuration</span></span>](../fundamentals/configuration.md)

* [<span data-ttu-id="474fa-308">Inicialização de aplicativos</span><span class="sxs-lookup"><span data-stu-id="474fa-308">Application Startup</span></span>](../fundamentals/startup.md)

* [<span data-ttu-id="474fa-309">Middleware</span><span class="sxs-lookup"><span data-stu-id="474fa-309">Middleware</span></span>](../fundamentals/middleware.md)