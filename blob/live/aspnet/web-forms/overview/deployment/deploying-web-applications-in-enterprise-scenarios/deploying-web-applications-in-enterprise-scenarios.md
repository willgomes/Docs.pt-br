---
uid: web-forms/overview/deployment/deploying-web-applications-in-enterprise-scenarios/deploying-web-applications-in-enterprise-scenarios
title: "Implantando aplicativos da Web em cenários corporativos usando o Visual Studio 2010 | Microsoft Docs"
author: jrjlee
description: "Esse conjunto de tutoriais descreve ferramentas e técnicas que você pode usar para implantar aplicativos da web em vários cenários corporativos. Explica como fazer o melhor uso..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 05/03/2012
ms.topic: article
ms.assetid: 48cfe378-d62a-48c6-a4db-6be3cead6898
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/deployment/deploying-web-applications-in-enterprise-scenarios/deploying-web-applications-in-enterprise-scenarios
msc.type: authoredcontent
ms.openlocfilehash: 99bab371dd34b30f3554843e49bbec7f57c3f96c
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/10/2017
---
<a name="deploying-web-applications-in-enterprise-scenarios-using-visual-studio-2010"></a><span data-ttu-id="b1da4-104">Implantando aplicativos da Web em cenários corporativos usando o Visual Studio 2010</span><span class="sxs-lookup"><span data-stu-id="b1da4-104">Deploying Web Applications in Enterprise Scenarios using Visual Studio 2010</span></span>
====================
<span data-ttu-id="b1da4-105">por [Jason Lee](https://github.com/jrjlee)</span><span class="sxs-lookup"><span data-stu-id="b1da4-105">by [Jason Lee](https://github.com/jrjlee)</span></span>

[<span data-ttu-id="b1da4-106">Baixar PDF</span><span class="sxs-lookup"><span data-stu-id="b1da4-106">Download PDF</span></span>](https://msdnshared.blob.core.windows.net/media/MSDNBlogsFS/prod.evol.blogs.msdn.com/CommunityServer.Blogs.Components.WeblogFiles/00/00/00/63/56/8130.DeployingWebAppsInEnterpriseScenarios.pdf)

> <span data-ttu-id="b1da4-107">Esse conjunto de tutoriais descreve ferramentas e técnicas que você pode usar para implantar aplicativos da web em vários cenários corporativos.</span><span class="sxs-lookup"><span data-stu-id="b1da4-107">This set of tutorials describes tools and techniques you can use to deploy web applications in various enterprise scenarios.</span></span> <span data-ttu-id="b1da4-108">Explica como fazer melhor uso de tecnologias como o Visual Studio 2010, a Microsoft Build Engine (MSBuild), serviços de informações da Internet (IIS) 7.5, a ferramenta de implantação da Web de IIS (implantação da Web), Web Farm Framework (WFF) e utilitários, como VSDBCMD.exe para simplificar e gerenciar o processo de implantação.</span><span class="sxs-lookup"><span data-stu-id="b1da4-108">It explains how to make best use of technologies like Visual Studio 2010, the Microsoft Build Engine (MSBuild), Internet Information Services (IIS) 7.5, the IIS Web Deployment Tool (Web Deploy), the Web Farm Framework (WFF), and utilities like VSDBCMD.exe to simplify and manage the deployment process.</span></span> <span data-ttu-id="b1da4-109">Ele inclui uma visão geral conceitual e orientação e orientada a tarefas que ajudarão você a:</span><span class="sxs-lookup"><span data-stu-id="b1da4-109">It includes conceptual overviews and task-oriented guidance that will help you to:</span></span>
> 
> - <span data-ttu-id="b1da4-110">Examine e estabelecer os requisitos de implantação para um aplicativo da web de grande porte.</span><span class="sxs-lookup"><span data-stu-id="b1da4-110">Review and establish the deployment requirements for an enterprise-scale web application.</span></span>
> - <span data-ttu-id="b1da4-111">Configure ambientes de servidor web teste, preparação e produção para dar suporte à implantação da web.</span><span class="sxs-lookup"><span data-stu-id="b1da4-111">Configure test, staging, and production web server environments to support web deployment.</span></span>
> - <span data-ttu-id="b1da4-112">Configure os processos de CI (integração contínua) do Team Foundation Server (TFS) para dar suporte à implantação de web automatizada.</span><span class="sxs-lookup"><span data-stu-id="b1da4-112">Configure Team Foundation Server (TFS) continuous integration (CI) processes to support automated web deployment.</span></span>
> - <span data-ttu-id="b1da4-113">Implante aplicativos da web de nível corporativo para ambientes de servidor diferentes com diferentes requisitos e restrições.</span><span class="sxs-lookup"><span data-stu-id="b1da4-113">Deploy enterprise-scale web applications to different server environments with varying requirements and restrictions.</span></span>
> - <span data-ttu-id="b1da4-114">Implante alterações em aplicativos da web que são executados em ambientes de servidor diferente.</span><span class="sxs-lookup"><span data-stu-id="b1da4-114">Deploy changes to web applications that are running in different server environments.</span></span>
> 
> > [!NOTE]
> > <span data-ttu-id="b1da4-115">Enquanto esses tutoriais descrevem o uso do TFS, como um servidor de CI, a orientação é facilmente adaptada para qualquer servidor de CI.</span><span class="sxs-lookup"><span data-stu-id="b1da4-115">While these tutorials describe the use of TFS as a CI server, the guidance is easily adapted to any CI server.</span></span> <span data-ttu-id="b1da4-116">Não é necessário um conhecimento detalhado do TFS para compreender e aproveitar os tutoriais.</span><span class="sxs-lookup"><span data-stu-id="b1da4-116">You don't need a detailed knowledge of TFS to understand and leverage the tutorials.</span></span>
> 
> 
> <span data-ttu-id="b1da4-117">Para obter uma tradução italiana destes tutoriais, visite [http://www.lucamorelli.it](http://www.lucamorelli.it).</span><span class="sxs-lookup"><span data-stu-id="b1da4-117">For an Italian translation of these tutorials, visit [http://www.lucamorelli.it](http://www.lucamorelli.it).</span></span>


## <a name="about-the-authors"></a><span data-ttu-id="b1da4-118">Sobre os autores</span><span class="sxs-lookup"><span data-stu-id="b1da4-118">About the Authors</span></span>

<span data-ttu-id="b1da4-119">Jason Lee é tecnólogo principal com [conteúdo mestre](http://www.contentmaster.com/) onde ele trabalha com produtos e tecnologias, especialmente o SharePoint e o ASP.NET, Microsoft por vários anos.</span><span class="sxs-lookup"><span data-stu-id="b1da4-119">Jason Lee is a principal technologist with [Content Master](http://www.contentmaster.com/) where he has been working with Microsoft products and technologies, especially SharePoint and ASP.NET, for several years.</span></span> <span data-ttu-id="b1da4-120">Jason possui PhD na computação e está atualmente Credenciais e MCTS certificados.</span><span class="sxs-lookup"><span data-stu-id="b1da4-120">Jason holds a PhD in computing and is currently MCPD and MCTS certified.</span></span> <span data-ttu-id="b1da4-121">Você pode ler o blog de técnica de Jason em [www.jrjlee.com](http://www.jrjlee.com/).</span><span class="sxs-lookup"><span data-stu-id="b1da4-121">You can read Jason's technical blog at [www.jrjlee.com](http://www.jrjlee.com/).</span></span>

<span data-ttu-id="b1da4-122">Benjamin Curry é tecnólogo principal com [conteúdo mestre](http://www.contentmaster.com/) que gravou white papers, documentação do SDK, apresentações do PowerPoint e cursos online e instrutor durante sua carreiras.</span><span class="sxs-lookup"><span data-stu-id="b1da4-122">Benjamin Curry is a principal technologist with [Content Master](http://www.contentmaster.com/) who has written whitepapers, SDK documentation, PowerPoint presentations, and instructor-led and online training courses during his career.</span></span> <span data-ttu-id="b1da4-123">Um membro original da equipe de documentação do ASP.NET, ele trabalha com tecnologias web da Microsoft mais de uma década.</span><span class="sxs-lookup"><span data-stu-id="b1da4-123">An original member of the ASP.NET documentation team, he has worked with Microsoft's web technologies for over a decade.</span></span>

## <a name="target-audience"></a><span data-ttu-id="b1da4-124">Público-alvo</span><span class="sxs-lookup"><span data-stu-id="b1da4-124">Target Audience</span></span>

<span data-ttu-id="b1da4-125">Esse conjunto de tutoriais é para desenvolvedores de aplicativos web ASP.NET e arquitetos de soluções que usam o Visual Studio 2010 para criar aplicativos da web de grande porte.</span><span class="sxs-lookup"><span data-stu-id="b1da4-125">This set of tutorials is for ASP.NET web application developers and solution architects who use Visual Studio 2010 to create enterprise-scale web applications.</span></span> <span data-ttu-id="b1da4-126">Para obter o valor máximo do conteúdo, você deve estar familiarizado com o Visual Studio 2010 e tem uma familiaridade básica com o TFS, junto com reconhecimento de tecnologias da plataforma Microsoft web ASP.NET MVC 3, o Windows Communication Foundation (WCF), o IIS, o SQL O servidor e projetos de banco de dados do Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="b1da4-126">To get the most value from the content, you should be comfortable using Visual Studio 2010 and have a basic familiarity with TFS, together with an awareness of Microsoft web platform technologies like ASP.NET MVC 3, Windows Communication Foundation (WCF), IIS, SQL Server, and Visual Studio database projects.</span></span> <span data-ttu-id="b1da4-127">No entanto, você não precisa estar familiarizado com as tecnologias e ferramentas de implantação ou precisa saber como configurar sistemas de CI.</span><span class="sxs-lookup"><span data-stu-id="b1da4-127">However, you do not need to be familiar with deployment tools and technologies or need to know how to set up CI systems.</span></span>

## <a name="requirements"></a><span data-ttu-id="b1da4-128">Requisitos</span><span class="sxs-lookup"><span data-stu-id="b1da4-128">Requirements</span></span>

<span data-ttu-id="b1da4-129">Para seguir a instruções passo a passo e executar as tarefas que descrevem esses tutoriais, você precisará instalar esse software em seu computador de desenvolvimento:</span><span class="sxs-lookup"><span data-stu-id="b1da4-129">To follow the walkthroughs and perform the tasks that these tutorials describe, you'll need to install this software on your development computer:</span></span>

- <span data-ttu-id="b1da4-130">Visual Studio 2010 Premium ou Ultimate Edition com Service Pack 1</span><span class="sxs-lookup"><span data-stu-id="b1da4-130">Visual Studio 2010 Premium or Ultimate Edition with Service Pack 1</span></span>
- <span data-ttu-id="b1da4-131">.NET framework 4.0</span><span class="sxs-lookup"><span data-stu-id="b1da4-131">.NET Framework 4.0</span></span>
- <span data-ttu-id="b1da4-132">.NET framework 3.5 com Service Pack 1</span><span class="sxs-lookup"><span data-stu-id="b1da4-132">.NET Framework 3.5 with Service Pack 1</span></span>
- <span data-ttu-id="b1da4-133">ASP.NET MVC 3.0</span><span class="sxs-lookup"><span data-stu-id="b1da4-133">ASP.NET MVC 3.0</span></span>
- <span data-ttu-id="b1da4-134">O IIS 7.5 Express</span><span class="sxs-lookup"><span data-stu-id="b1da4-134">IIS 7.5 Express</span></span>
- <span data-ttu-id="b1da4-135">SQL Server Express 2008 R2</span><span class="sxs-lookup"><span data-stu-id="b1da4-135">SQL Server Express 2008 R2</span></span>

<span data-ttu-id="b1da4-136">Para executar as etapas de implantação descritas em todo esse passo a passo, você precisará ter acesso aos ambientes de implantação do aplicativo de Web de exemplo.</span><span class="sxs-lookup"><span data-stu-id="b1da4-136">To perform the deployment steps described throughout these walkthroughs, you'll need to have access to sample Web application deployment environments.</span></span> <span data-ttu-id="b1da4-137">Para obter melhores resultados, esses ambientes devem refletir o padrão de implantação de empresa da sua organização.</span><span class="sxs-lookup"><span data-stu-id="b1da4-137">For best results, these environments should reflect your organization's enterprise deployment pattern.</span></span> <span data-ttu-id="b1da4-138">Você pode modificar as orientações fornecidas nesta documentação para refletir a ambientes de implantação e os requisitos da sua organização.</span><span class="sxs-lookup"><span data-stu-id="b1da4-138">You can then modify the walkthroughs provided in this documentation to reflect the deployment environments and requirements of your own organization.</span></span>

## <a name="series-contents"></a><span data-ttu-id="b1da4-139">Conteúdo de série</span><span class="sxs-lookup"><span data-stu-id="b1da4-139">Series Contents</span></span>

<span data-ttu-id="b1da4-140">Esta seção Introdução consiste em dois tópicos adicionais.</span><span class="sxs-lookup"><span data-stu-id="b1da4-140">This introductory section consists of two further topics.</span></span> <span data-ttu-id="b1da4-141">Eles são projetados para fornecer algum contexto mais amplo para os tutoriais a seguir:</span><span class="sxs-lookup"><span data-stu-id="b1da4-141">These are designed to provide some broader context for the tutorials that follow:</span></span>

- <span data-ttu-id="b1da4-142">[Implantação da Web corporativa: Visão geral do cenário](enterprise-web-deployment-scenario-overview.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-142">[Enterprise Web Deployment: Scenario Overview](enterprise-web-deployment-scenario-overview.md).</span></span> <span data-ttu-id="b1da4-143">Este tópico descreve o cenário que serve de base para cada um dos tutoriais na série.</span><span class="sxs-lookup"><span data-stu-id="b1da4-143">This topic describes the scenario that underpins each of the tutorials in this series.</span></span> <span data-ttu-id="b1da4-144">O cenário se concentra nos requisitos de gerenciamento de ciclo de vida do aplicativo (ALM) de uma empresa fictícia chamada Fabrikam, Inc. como ele desenvolve um aplicativo da web de grande porte.</span><span class="sxs-lookup"><span data-stu-id="b1da4-144">The scenario focuses on the Application Lifecycle Management (ALM) requirements of a fictional company named Fabrikam, Inc. as it develops an enterprise-scale web application.</span></span>
- <span data-ttu-id="b1da4-145">[Gerenciamento de ciclo de vida do aplicativo: De desenvolvimento para produção](application-lifecycle-management-from-development-to-production.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-145">[Application Lifecycle Management: From Development to Production](application-lifecycle-management-from-development-to-production.md).</span></span> <span data-ttu-id="b1da4-146">Este tópico fornece uma visão geral de alto nível, de ponta a ponta de um processo de implantação.</span><span class="sxs-lookup"><span data-stu-id="b1da4-146">This topic provides a high-level, end-to-end overview of a deployment process.</span></span> <span data-ttu-id="b1da4-147">Ele ilustra como a Fabrikam, Inc. se move um aplicativo web ASP.NET escala corporativa por meio de ambientes de teste, preparação e produção como parte de um processo de desenvolvimento contínuo.</span><span class="sxs-lookup"><span data-stu-id="b1da4-147">It illustrates how Fabrikam,Inc. moves an enterprise-scale ASP.NET web application through test, staging, and production environments as part of a continuous development process.</span></span>

<span data-ttu-id="b1da4-148">A série inclui quatro conjuntos de tutorial.</span><span class="sxs-lookup"><span data-stu-id="b1da4-148">The series includes four tutorial sets.</span></span> <span data-ttu-id="b1da4-149">Cada uma se concentra nos diferentes aspectos da implantação da web:</span><span class="sxs-lookup"><span data-stu-id="b1da4-149">Each focuses on different aspects of web deployment:</span></span>

- <span data-ttu-id="b1da4-150">[Implantação na empresa Web](../web-deployment-in-the-enterprise/web-deployment-in-the-enterprise.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-150">[Web Deployment in the Enterprise](../web-deployment-in-the-enterprise/web-deployment-in-the-enterprise.md).</span></span> <span data-ttu-id="b1da4-151">Este tutorial fornece uma introdução conceitual para arquivos de projeto do MSBuild, o Pipeline de publicação na Web, implantação da Web e outras tecnologias relacionadas.</span><span class="sxs-lookup"><span data-stu-id="b1da4-151">This tutorial provides a conceptual introduction to MSBuild project files, the Web Publishing Pipeline, Web Deploy, and other related technologies.</span></span> <span data-ttu-id="b1da4-152">Ele explica como você pode usar essas ferramentas em conjunto para gerenciar processos complexos de implantação.</span><span class="sxs-lookup"><span data-stu-id="b1da4-152">It explains how you can use these tools together to manage complex deployment processes.</span></span>
- <span data-ttu-id="b1da4-153">[Configurando ambientes de servidor para a implantação da Web](../configuring-server-environments-for-web-deployment/configuring-server-environments-for-web-deployment.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-153">[Configuring Server Environments for Web Deployment](../configuring-server-environments-for-web-deployment/configuring-server-environments-for-web-deployment.md).</span></span> <span data-ttu-id="b1da4-154">Este tutorial descreve como configurar os servidores Windows para oferecer suporte a vários cenários de implantação, incluindo a implantação de pacote via web remoto usando o serviço de agente de implantação da Web (o "agente remoto") ou o manipulador de implantação da Web e implantação de banco de dados remoto.</span><span class="sxs-lookup"><span data-stu-id="b1da4-154">This tutorial describes how to configure Windows servers to support various deployment scenarios, including remote web package deployment using the Web Deployment Agent Service (the "remote agent") or the Web Deploy Handler and remote database deployment.</span></span> <span data-ttu-id="b1da4-155">Fornece orientação sobre como escolher o método de implantação apropriadas para seu próprio ambiente, e descreve como usar o WFF para replicar os aplicativos web implantados em todos os servidores web em um farm de servidores.</span><span class="sxs-lookup"><span data-stu-id="b1da4-155">It provides guidance on choosing the appropriate deployment method for your own environment, and it describes how to use the WFF to replicate deployed web applications across all the web servers in a server farm.</span></span>
- <span data-ttu-id="b1da4-156">[Configurando o Team Foundation Server para a implantação da Web](../configuring-team-foundation-server-for-web-deployment/configuring-team-foundation-server-for-web-deployment.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-156">[Configuring Team Foundation Server for Web Deployment](../configuring-team-foundation-server-for-web-deployment/configuring-team-foundation-server-for-web-deployment.md).</span></span> <span data-ttu-id="b1da4-157">Este tutorial descreve como configurar o TFS para dar suporte a vários cenários de implantação, incluindo a implantação automatizada como parte de um processo de CI e disparada manualmente as implantações de compilações específicas.</span><span class="sxs-lookup"><span data-stu-id="b1da4-157">This tutorial describes how to configure TFS to support various deployment scenarios, including automated deployment as part of a CI process and manually triggered deployments of specific builds.</span></span>
- <span data-ttu-id="b1da4-158">[Enterprise Web implantação avançada](../advanced-enterprise-web-deployment/advanced-enterprise-web-deployment.md).</span><span class="sxs-lookup"><span data-stu-id="b1da4-158">[Advanced Enterprise Web Deployment](../advanced-enterprise-web-deployment/advanced-enterprise-web-deployment.md).</span></span> <span data-ttu-id="b1da4-159">Este tutorial descreve como realizar várias tarefas de implantação mais avançadas, como personalizar as implantações de banco de dados para vários ambientes, excluindo arquivos e pastas de implantação e que os aplicativos de web offline durante o processo de implantação .</span><span class="sxs-lookup"><span data-stu-id="b1da4-159">This tutorial describes how to accomplish various more advanced deployment tasks, like customizing database deployments for multiple environments, excluding files and folders from deployment, and taking web applications offline during the deployment process.</span></span>

## <a name="where-to-start"></a><span data-ttu-id="b1da4-160">Onde começar</span><span class="sxs-lookup"><span data-stu-id="b1da4-160">Where to Start</span></span>

<span data-ttu-id="b1da4-161">Esse conjunto de tutoriais usa uma solução de exemplo com um nível realista de complexidade, junto com um cenário de implantação da empresa fictícia, para fornecer uma implementação de referência e conceder a tarefas e instruções passo a passo um contexto de comum.</span><span class="sxs-lookup"><span data-stu-id="b1da4-161">This set of tutorials uses a sample solution with a realistic level of complexity, together with a fictional enterprise deployment scenario, to provide a reference implementation and to give the tasks and walkthroughs a common context.</span></span> <span data-ttu-id="b1da4-162">O próximo tópico, [implantação de Web corporativa: Visão geral do cenário](enterprise-web-deployment-scenario-overview.md), apresenta o cenário e a solução de exemplo.</span><span class="sxs-lookup"><span data-stu-id="b1da4-162">The next topic, [Enterprise Web Deployment: Scenario Overview](enterprise-web-deployment-scenario-overview.md), introduces the scenario and the sample solution.</span></span> <span data-ttu-id="b1da4-163">A partir daí, você pode trabalhar nos tutoriais e tópicos que melhor atendem às suas necessidades.</span><span class="sxs-lookup"><span data-stu-id="b1da4-163">From there you can work through the tutorials and topics that most closely match your needs.</span></span>

>[!div class="step-by-step"]
[<span data-ttu-id="b1da4-164">Avançar</span><span class="sxs-lookup"><span data-stu-id="b1da4-164">Next</span></span>](enterprise-web-deployment-scenario-overview.md)