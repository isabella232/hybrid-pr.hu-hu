---
title: Hibrid felhőalapú identitás konfigurálása az Azure-hoz és Azure Stack hub-alkalmazásokhoz
description: Ismerje meg, hogyan konfigurálhatja az Azure és Azure Stack hub-alkalmazások hibrid felhőalapú identitását.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911313"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="6a26c-103">Hibrid felhőalapú identitás konfigurálása az Azure-hoz és Azure Stack hub-alkalmazásokhoz</span><span class="sxs-lookup"><span data-stu-id="6a26c-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="6a26c-104">Megtudhatja, hogyan konfigurálhat hibrid Felhőbeli identitást Azure-és Azure Stack hub-alkalmazásaihoz.</span><span class="sxs-lookup"><span data-stu-id="6a26c-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="6a26c-105">Két lehetősége van arra, hogy hozzáférjen az alkalmazásaihoz a globális Azure-ban és Azure Stack hub-ban is.</span><span class="sxs-lookup"><span data-stu-id="6a26c-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="6a26c-106">Ha Azure Stack hub folyamatos internetkapcsolattal rendelkezik, használhatja a Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="6a26c-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="6a26c-107">Ha Azure Stack hub le van választva az internetről, használhatja az Azure Directory összevont szolgáltatások (AD FS) szolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="6a26c-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="6a26c-108">Az egyszerű szolgáltatásokkal hozzáférést biztosíthat a Azure Stack hub-alkalmazásokhoz az Azure Stack hub Azure Resource Manager használatával történő üzembe helyezéshez vagy konfiguráláshoz.</span><span class="sxs-lookup"><span data-stu-id="6a26c-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="6a26c-109">Ebben a megoldásban egy példaként szolgáló környezetet fog kiépíteni a következőkre:</span><span class="sxs-lookup"><span data-stu-id="6a26c-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="6a26c-110">Hibrid identitás létrehozása a globális Azure-ban és Azure Stack hub-ban</span><span class="sxs-lookup"><span data-stu-id="6a26c-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="6a26c-111">Token lekérése az Azure Stack hub API eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="6a26c-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="6a26c-112">Ennek a megoldásnak a lépéseihez Azure Stack hub-kezelő engedélyekkel kell rendelkeznie.</span><span class="sxs-lookup"><span data-stu-id="6a26c-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="6a26c-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="6a26c-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="6a26c-114">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="6a26c-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="6a26c-115">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="6a26c-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="6a26c-116">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="6a26c-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="6a26c-117">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="6a26c-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="6a26c-118">Egyszerű szolgáltatásnév létrehozása az Azure AD-hez a portálon</span><span class="sxs-lookup"><span data-stu-id="6a26c-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="6a26c-119">Ha Azure Stack hub-t az Azure AD-ben telepítette az identitás-tárolóként, az Azure-hoz hasonlóan az egyszerű szolgáltatásokat is létrehozhatja.</span><span class="sxs-lookup"><span data-stu-id="6a26c-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="6a26c-120">Az [erőforrások eléréséhez használt alkalmazás-identitással](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) megtudhatja, hogyan hajthatja végre a lépéseket a portálon.</span><span class="sxs-lookup"><span data-stu-id="6a26c-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="6a26c-121">A Kezdés előtt győződjön meg arról, hogy rendelkezik a [szükséges Azure ad-engedélyekkel](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .</span><span class="sxs-lookup"><span data-stu-id="6a26c-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="6a26c-122">Egyszerű szolgáltatásnév létrehozása AD FShoz a PowerShell használatával</span><span class="sxs-lookup"><span data-stu-id="6a26c-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="6a26c-123">Ha AD FS használatával telepített Azure Stack hubot, a PowerShell segítségével létrehozhat egy egyszerű szolgáltatásnevet, hozzárendelhet egy szerepkört a hozzáféréshez, és bejelentkezhet a PowerShellből az identitás használatával.</span><span class="sxs-lookup"><span data-stu-id="6a26c-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="6a26c-124">Az [erőforrások eléréséhez használt alkalmazás-identitással](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) megtudhatja, hogyan hajthatja végre a szükséges lépéseket a PowerShell használatával.</span><span class="sxs-lookup"><span data-stu-id="6a26c-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="6a26c-125">Az Azure Stack hub API használata</span><span class="sxs-lookup"><span data-stu-id="6a26c-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="6a26c-126">Az [Azure stack hub API](/azure-stack/user/azure-stack-rest-api-use.md) -megoldás végigvezeti a token lekérésének folyamatán az Azure stack hub API eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="6a26c-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="6a26c-127">Kapcsolódás Azure Stack hubhoz a PowerShell használatával</span><span class="sxs-lookup"><span data-stu-id="6a26c-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="6a26c-128">Az [Azure stack hub PowerShell-](/azure-stack/operator/azure-stack-powershell-install.md) lel való üzembe helyezéséhez szükséges gyors útmutató végigvezeti a Azure PowerShell telepítésének és a Azure stack hub-telepítéshez való kapcsolódásnak a lépésein.</span><span class="sxs-lookup"><span data-stu-id="6a26c-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="6a26c-129">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="6a26c-129">Prerequisites</span></span>

<span data-ttu-id="6a26c-130">Egy, az Azure AD-hez csatlakoztatott Azure Stack hub-telepítésre van szükség, amelyhez hozzá tud férni.</span><span class="sxs-lookup"><span data-stu-id="6a26c-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="6a26c-131">Ha nem rendelkezik Azure Stack hub-telepítéssel, a következő utasításokat követve állíthatja be a [Azure stack Development Kitt (ASDK)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="6a26c-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="6a26c-132">Kapcsolódás Azure Stack hubhoz kód használatával</span><span class="sxs-lookup"><span data-stu-id="6a26c-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="6a26c-133">Ha kódot használ a Azure Stack hubhoz való kapcsolódáshoz, használja a Azure Resource Manager endpoints API-t az Azure Stack hub-telepítéshez tartozó hitelesítési és gráf végpontok beszerzéséhez.</span><span class="sxs-lookup"><span data-stu-id="6a26c-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="6a26c-134">Ezután végezze el a hitelesítést a REST-kérelmek használatával.</span><span class="sxs-lookup"><span data-stu-id="6a26c-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="6a26c-135">A [githubon](https://github.com/shriramnat/HybridARMApplication)megtalálhatja a minta ügyfélalkalmazás alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="6a26c-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="6a26c-136">Ha a választott nyelvhez készült Azure SDK támogatja az Azure API-profilokat, előfordulhat, hogy az SDK nem működik együtt Azure Stack hub-val.</span><span class="sxs-lookup"><span data-stu-id="6a26c-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="6a26c-137">Az Azure API-profilokkal kapcsolatos további tudnivalókért tekintse meg az [API-verzió profiljainak kezelése](/azure-stack/user/azure-stack-version-profiles.md) című cikket.</span><span class="sxs-lookup"><span data-stu-id="6a26c-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6a26c-138">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="6a26c-138">Next steps</span></span>

- <span data-ttu-id="6a26c-139">Ha többet szeretne megtudni arról, hogyan kezeli az identitást Azure Stack központban, tekintse meg a [Azure stack hub Identity Architecture](/azure-stack/operator/azure-stack-identity-architecture.md)című témakört.</span><span class="sxs-lookup"><span data-stu-id="6a26c-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="6a26c-140">Az Azure Cloud Patterns szolgáltatással kapcsolatos további információkért lásd: [Felhőbeli tervezési minták](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="6a26c-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
