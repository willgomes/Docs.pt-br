---
title: "Pesquisa rápida de outros provedores de autenticação."
author: rick-anderson
ms.author: riande
manager: wpickett
ms.date: 11/03/2016
ms.topic: article
ms.prod: asp.net-core
uid: security/authentication/otherlogins
ms.openlocfilehash: 9b061c66c8bf81f275ac35dc2e160feaf79eb21e
ms.sourcegitcommit: 3e303620a125325bb9abd4b2d315c106fb8c47fd
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/19/2018
---
# <a name="short-survey-of-other-authentication-providers"></a><span data-ttu-id="a272e-102">Pesquisa rápida de outros provedores de autenticação</span><span class="sxs-lookup"><span data-stu-id="a272e-102">Short survey of other authentication providers</span></span>

<a name="security-authentication-other-logins"></a>

<span data-ttu-id="a272e-103">Por [Rick Anderson](https://twitter.com/RickAndMSFT), [Pranav Rastogi](https://github.com/rustd), e [Valeriy Novytskyy](https://github.com/01binary)</span><span class="sxs-lookup"><span data-stu-id="a272e-103">By [Rick Anderson](https://twitter.com/RickAndMSFT), [Pranav Rastogi](https://github.com/rustd), and [Valeriy Novytskyy](https://github.com/01binary)</span></span>

<span data-ttu-id="a272e-104">Aqui estão configurados instruções para alguns outros provedores OAuth comuns.</span><span class="sxs-lookup"><span data-stu-id="a272e-104">Here are set up instructions for some other common OAuth providers.</span></span> <span data-ttu-id="a272e-105">Pacotes do NuGet de terceiros, como aqueles mantidos pelo [aspnet Contribuidor](https://www.nuget.org/packages?q=owners%3Aaspnet-contrib+title%3AOAuth) pode ser usada para complementar os provedores de autenticação implementados pela equipe do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="a272e-105">Third-party NuGet packages such as the ones maintained by [aspnet-contrib](https://www.nuget.org/packages?q=owners%3Aaspnet-contrib+title%3AOAuth) can be used to complement authentication providers implemented by the ASP.NET Core team.</span></span>

* <span data-ttu-id="a272e-106">Configurar **LinkedIn** entrar: [https://www.linkedin.com/developer/apps](https://www.linkedin.com/developer/apps).</span><span class="sxs-lookup"><span data-stu-id="a272e-106">Set up **LinkedIn** sign in: [https://www.linkedin.com/developer/apps](https://www.linkedin.com/developer/apps).</span></span> <span data-ttu-id="a272e-107">Consulte [etapas oficiais](https://developer.linkedin.com/docs/oauth2).</span><span class="sxs-lookup"><span data-stu-id="a272e-107">See [official steps](https://developer.linkedin.com/docs/oauth2).</span></span>

* <span data-ttu-id="a272e-108">Configurar **Instagram** entrar: [https://www.instagram.com/developer/register/](https://www.instagram.com/developer/register/).</span><span class="sxs-lookup"><span data-stu-id="a272e-108">Set up **Instagram** sign in: [https://www.instagram.com/developer/register/](https://www.instagram.com/developer/register/).</span></span> <span data-ttu-id="a272e-109">Consulte [etapas oficiais](https://www.instagram.com/developer/authentication/).</span><span class="sxs-lookup"><span data-stu-id="a272e-109">See [official steps](https://www.instagram.com/developer/authentication/).</span></span>

* <span data-ttu-id="a272e-110">Configurar **Reddit** entrar: [https://www.reddit.com/login?dest=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps](https://www.reddit.com/login?dest=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps).</span><span class="sxs-lookup"><span data-stu-id="a272e-110">Set up **Reddit** sign in: [https://www.reddit.com/login?dest=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps](https://www.reddit.com/login?dest=https%3A%2F%2Fwww.reddit.com%2Fprefs%2Fapps).</span></span> <span data-ttu-id="a272e-111">Consulte [etapas oficiais](https://github.com/reddit/reddit/wiki/OAuth2-Quick-Start-Example).</span><span class="sxs-lookup"><span data-stu-id="a272e-111">See [official steps](https://github.com/reddit/reddit/wiki/OAuth2-Quick-Start-Example).</span></span>

* <span data-ttu-id="a272e-112">Configurar **Github** entrar: [https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fapplications%2Fnew](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fapplications%2Fnew).</span><span class="sxs-lookup"><span data-stu-id="a272e-112">Set up **Github** sign in: [https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fapplications%2Fnew](https://github.com/login?return_to=https%3A%2F%2Fgithub.com%2Fsettings%2Fapplications%2Fnew).</span></span> <span data-ttu-id="a272e-113">Consulte [etapas oficiais](https://developer.github.com/v3/oauth/).</span><span class="sxs-lookup"><span data-stu-id="a272e-113">See [official steps](https://developer.github.com/v3/oauth/).</span></span>

* <span data-ttu-id="a272e-114">Configurar **Yahoo** entrar: [https://login.yahoo.com/config/login?src=devnet&.done=http%3A%2F%2Fdeveloper.yahoo.com%2Fapps%2Fcreate%2F](https://login.yahoo.com/config/login?src=devnet&.done=http%3A%2F%2Fdeveloper.yahoo.com%2Fapps%2Fcreate%2F).</span><span class="sxs-lookup"><span data-stu-id="a272e-114">Set up **Yahoo** sign in: [https://login.yahoo.com/config/login?src=devnet&.done=http%3A%2F%2Fdeveloper.yahoo.com%2Fapps%2Fcreate%2F](https://login.yahoo.com/config/login?src=devnet&.done=http%3A%2F%2Fdeveloper.yahoo.com%2Fapps%2Fcreate%2F).</span></span> <span data-ttu-id="a272e-115">Consulte [etapas oficiais](https://developer.yahoo.com/bbauth/user.html).</span><span class="sxs-lookup"><span data-stu-id="a272e-115">See [official steps](https://developer.yahoo.com/bbauth/user.html).</span></span>

* <span data-ttu-id="a272e-116">Configurar **Tumblr** entrar: [https://www.tumblr.com/oauth/apps](https://www.tumblr.com/oauth/apps).</span><span class="sxs-lookup"><span data-stu-id="a272e-116">Set up **Tumblr** sign in: [https://www.tumblr.com/oauth/apps](https://www.tumblr.com/oauth/apps).</span></span> <span data-ttu-id="a272e-117">Consulte [etapas oficiais](https://www.tumblr.com/docs/api/v2#auth).</span><span class="sxs-lookup"><span data-stu-id="a272e-117">See [official steps](https://www.tumblr.com/docs/api/v2#auth).</span></span>

* <span data-ttu-id="a272e-118">Configurar **Pinterest** entrar: [https://www.pinterest.com/login/?next=http%3A%2F%2Fdevsite%2Fapps%2F](https://www.pinterest.com/login/?next=http%3A%2F%2Fdevsite%2Fapps%2F).</span><span class="sxs-lookup"><span data-stu-id="a272e-118">Set up **Pinterest** sign in: [https://www.pinterest.com/login/?next=http%3A%2F%2Fdevsite%2Fapps%2F](https://www.pinterest.com/login/?next=http%3A%2F%2Fdevsite%2Fapps%2F).</span></span> <span data-ttu-id="a272e-119">Consulte [etapas oficiais](https://developers.pinterest.com/docs/api/overview/?).</span><span class="sxs-lookup"><span data-stu-id="a272e-119">See [official steps](https://developers.pinterest.com/docs/api/overview/?).</span></span>

* <span data-ttu-id="a272e-120">Configurar **Pocket** entrar: [https://getpocket.com/developer/apps/new](https://getpocket.com/developer/apps/new).</span><span class="sxs-lookup"><span data-stu-id="a272e-120">Set up **Pocket** sign in: [https://getpocket.com/developer/apps/new](https://getpocket.com/developer/apps/new).</span></span> <span data-ttu-id="a272e-121">Consulte [etapas oficiais](https://getpocket.com/developer/docs/authentication).</span><span class="sxs-lookup"><span data-stu-id="a272e-121">See [official steps](https://getpocket.com/developer/docs/authentication).</span></span>

* <span data-ttu-id="a272e-122">Configurar **Flickr** entrar: [https://www.flickr.com/services/apps/create](https://www.flickr.com/services/apps/create).</span><span class="sxs-lookup"><span data-stu-id="a272e-122">Set up **Flickr** sign in: [https://www.flickr.com/services/apps/create](https://www.flickr.com/services/apps/create).</span></span> <span data-ttu-id="a272e-123">Consulte [etapas oficiais](https://www.flickr.com/services/api/auth.oauth.html).</span><span class="sxs-lookup"><span data-stu-id="a272e-123">See [official steps](https://www.flickr.com/services/api/auth.oauth.html).</span></span>

* <span data-ttu-id="a272e-124">Configurar **Dribble** entrar: [https://dribbble.com/signup](https://dribbble.com/signup).</span><span class="sxs-lookup"><span data-stu-id="a272e-124">Set up **Dribble** sign in: [https://dribbble.com/signup](https://dribbble.com/signup).</span></span> <span data-ttu-id="a272e-125">Consulte [etapas oficiais](http://developer.dribbble.com/v1/oauth/).</span><span class="sxs-lookup"><span data-stu-id="a272e-125">See [official steps](http://developer.dribbble.com/v1/oauth/).</span></span>

* <span data-ttu-id="a272e-126">Configurar **Vimeo** entrar: [https://vimeo.com/join](https://vimeo.com/join).</span><span class="sxs-lookup"><span data-stu-id="a272e-126">Set up **Vimeo** sign in: [https://vimeo.com/join](https://vimeo.com/join).</span></span> <span data-ttu-id="a272e-127">Consulte [etapas oficiais](https://developer.vimeo.com/api/authentication).</span><span class="sxs-lookup"><span data-stu-id="a272e-127">See [official steps](https://developer.vimeo.com/api/authentication).</span></span>

* <span data-ttu-id="a272e-128">Configurar **SoundCloud** entrar: [https://soundcloud.com/you/apps/new](https://soundcloud.com/you/apps/new).</span><span class="sxs-lookup"><span data-stu-id="a272e-128">Set up **SoundCloud** sign in: [https://soundcloud.com/you/apps/new](https://soundcloud.com/you/apps/new).</span></span> <span data-ttu-id="a272e-129">Consulte [etapas oficiais](https://developers.soundcloud.com/blog/we-love-oauth-2).</span><span class="sxs-lookup"><span data-stu-id="a272e-129">See [official steps](https://developers.soundcloud.com/blog/we-love-oauth-2).</span></span>

* <span data-ttu-id="a272e-130">Configurar **VK** entrar: [https://vk.com/apps?act=manage](https://vk.com/apps?act=manage).</span><span class="sxs-lookup"><span data-stu-id="a272e-130">Set up **VK** sign in: [https://vk.com/apps?act=manage](https://vk.com/apps?act=manage).</span></span> <span data-ttu-id="a272e-131">Consulte [etapas oficiais](https://vk.com/pages?oid=-17680044&p=Authorizing_Sites).</span><span class="sxs-lookup"><span data-stu-id="a272e-131">See [official steps](https://vk.com/pages?oid=-17680044&p=Authorizing_Sites).</span></span>