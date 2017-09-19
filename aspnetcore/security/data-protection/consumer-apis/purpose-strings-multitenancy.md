---
title: "Cadeias de caracteres de finalidade no núcleo do ASP.NET"
author: rick-anderson
description: 
keywords: ASP.NET Core
ms.author: riande
manager: wpickett
ms.date: 10/14/2016
ms.topic: article
ms.assetid: 9d18c287-e0e6-4541-b09c-7fed45c902d9
ms.technology: aspnet
ms.prod: asp.net-core
uid: security/data-protection/consumer-apis/purpose-strings-multitenancy
ms.openlocfilehash: dd87d8bcaf0056b322908e9a3ef75678f603e1e6
ms.sourcegitcommit: 0b6c8e6d81d2b3c161cd375036eecbace46a9707
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 08/11/2017
---
# <a name="purpose-hierarchy-and-multi-tenancy-in-aspnet-core"></a><span data-ttu-id="52992-103">Hierarquia de propósito e multilocação no núcleo do ASP.NET</span><span class="sxs-lookup"><span data-stu-id="52992-103">Purpose hierarchy and multi-tenancy in ASP.NET Core</span></span>

<span data-ttu-id="52992-104">Como um IDataProtector também é implicitamente um IDataProtectionProvider, fins podem ser encadeada.</span><span class="sxs-lookup"><span data-stu-id="52992-104">Since an IDataProtector is also implicitly an IDataProtectionProvider, purposes can be chained together.</span></span> <span data-ttu-id="52992-105">Este provedor sentido. CreateProtector (["purpose1", "purpose2"]) é equivalente ao provedor. CreateProtector("purpose1"). CreateProtector("purpose2").</span><span class="sxs-lookup"><span data-stu-id="52992-105">In this sense provider.CreateProtector([ "purpose1", "purpose2" ]) is equivalent to provider.CreateProtector("purpose1").CreateProtector("purpose2").</span></span>

<span data-ttu-id="52992-106">Isso permite que algumas relações hierárquicas interessantes através do sistema de proteção de dados.</span><span class="sxs-lookup"><span data-stu-id="52992-106">This allows for some interesting hierarchical relationships through the data protection system.</span></span> <span data-ttu-id="52992-107">No exemplo anterior de [Contoso.Messaging.SecureMessage](purpose-strings.md#data-protection-contoso-purpose), o componente SecureMessage pode chamar o provedor. CreateProtector("Contoso.Messaging.SecureMessage") quando inicial e o resultado em um campo particular _myProvider do cache.</span><span class="sxs-lookup"><span data-stu-id="52992-107">In the earlier example of [Contoso.Messaging.SecureMessage](purpose-strings.md#data-protection-contoso-purpose), the SecureMessage component can call provider.CreateProtector("Contoso.Messaging.SecureMessage") once upfront and cache the result into a private _myProvider field.</span></span> <span data-ttu-id="52992-108">Protetores futuras, em seguida, podem ser criados por meio de chamadas para _myProvider.CreateProtector ("usuário: nome de usuário"), e os protetores são usados para proteger as mensagens individuais.</span><span class="sxs-lookup"><span data-stu-id="52992-108">Future protectors can then be created via calls to _myProvider.CreateProtector("User: username"), and these protectors would be used for securing the individual messages.</span></span>

<span data-ttu-id="52992-109">Isso também pode ser invertido.</span><span class="sxs-lookup"><span data-stu-id="52992-109">This can also be flipped.</span></span> <span data-ttu-id="52992-110">Considere que hospeda vários locatários (um CMS parece razoável) e cada locatário podem ser configurado com seu próprio sistema de gerenciamento de autenticação e o estado de um único aplicativo lógico.</span><span class="sxs-lookup"><span data-stu-id="52992-110">Consider a single logical application which hosts multiple tenants (a CMS seems reasonable), and each tenant can be configured with its own authentication and state management system.</span></span> <span data-ttu-id="52992-111">O aplicativo de proteção tem um provedor de mestre único, e ele chama o provedor. CreateProtector ("locatário 1") e o provedor. CreateProtector ("locatário 2") para fornecer seu próprio fatia isolada do sistema de proteção de dados de cada locatário.</span><span class="sxs-lookup"><span data-stu-id="52992-111">The umbrella application has a single master provider, and it calls provider.CreateProtector("Tenant 1") and provider.CreateProtector("Tenant 2") to give each tenant its own isolated slice of the data protection system.</span></span> <span data-ttu-id="52992-112">Os locatários, em seguida, podem derivar seus próprio protetores individuais com base em suas necessidades, mas, independentemente de como eles tentam não é possível criar protetores que entrarem em conflito com qualquer outro locatário no sistema.</span><span class="sxs-lookup"><span data-stu-id="52992-112">The tenants could then derive their own individual protectors based on their own needs, but no matter how hard they try they cannot create protectors which collide with any other tenant in the system.</span></span> <span data-ttu-id="52992-113">Graficamente é representado como abaixo.</span><span class="sxs-lookup"><span data-stu-id="52992-113">Graphically this is represented as below.</span></span>

![Vários fins de aluguel](purpose-strings-multitenancy/_static/purposes-multi-tenancy.png)

>[!WARNING]
> <span data-ttu-id="52992-115">Isso pressupõe que a proteção controles de aplicativo que APIs estão disponíveis para locatários individuais e que os locatários não podem executar código arbitrário no servidor.</span><span class="sxs-lookup"><span data-stu-id="52992-115">This assumes the umbrella application controls which APIs are available to individual tenants and that tenants cannot execute arbitrary code on the server.</span></span> <span data-ttu-id="52992-116">Se um locatário pode executar código arbitrário, eles executaria privada reflexão para interromper as garantias de isolamento ou pode apenas ler o material de chave mestre diretamente e derivar qualquer subchaves que desejarem.</span><span class="sxs-lookup"><span data-stu-id="52992-116">If a tenant can execute arbitrary code, they could perform private reflection to break the isolation guarantees, or they could just read the master keying material directly and derive whatever subkeys they desire.</span></span>

<span data-ttu-id="52992-117">Na verdade, o sistema de proteção de dados usa uma classificação de multilocação na configuração padrão da caixa.</span><span class="sxs-lookup"><span data-stu-id="52992-117">The data protection system actually uses a sort of multi-tenancy in its default out-of-the-box configuration.</span></span> <span data-ttu-id="52992-118">Por padrão, o material de chave mestra é armazenado na pasta de perfil de usuário da conta de processo do operador (ou o registro, para identidades de pool de aplicativos do IIS).</span><span class="sxs-lookup"><span data-stu-id="52992-118">By default master keying material is stored in the worker process account's user profile folder (or the registry, for IIS application pool identities).</span></span> <span data-ttu-id="52992-119">Mas é na verdade bastante comum para usar uma única conta para executar vários aplicativos e, portanto, todos esses aplicativos acabar compartilhando a material da chave mestra.</span><span class="sxs-lookup"><span data-stu-id="52992-119">But it is actually fairly common to use a single account to run multiple applications, and thus all these applications would end up sharing the master keying material.</span></span> <span data-ttu-id="52992-120">Para resolver isso, o sistema de proteção de dados automaticamente insere um identificador exclusivo por aplicativo como o primeiro elemento da cadeia de finalidade geral.</span><span class="sxs-lookup"><span data-stu-id="52992-120">To solve this, the data protection system automatically inserts a unique-per-application identifier as the first element in the overall purpose chain.</span></span> <span data-ttu-id="52992-121">Essa finalidade implícita serve para [isolar aplicativos individuais](../configuration/overview.md#data-protection-configuration-per-app-isolation) uns dos outros tratando efetivamente cada aplicativo como um locatário exclusivo dentro do sistema e o processo de criação de protetor é idêntico à imagem acima.</span><span class="sxs-lookup"><span data-stu-id="52992-121">This implicit purpose serves to [isolate individual applications](../configuration/overview.md#data-protection-configuration-per-app-isolation) from one another by effectively treating each application as a unique tenant within the system, and the protector creation process looks identical to the image above.</span></span>