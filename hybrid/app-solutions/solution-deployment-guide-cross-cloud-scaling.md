---
title: Felhőbeli felhőalapú alkalmazások üzembe helyezése az Azure-ban és Azure Stack hub
description: Megtudhatja, hogyan helyezhet üzembe olyan alkalmazást, amely az Azure-ban és Azure Stack hub-ban is méretezhető.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895414"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a><span data-ttu-id="e104a-103">Felhőben futó alkalmazások üzembe helyezése az Azure-ban és Azure Stack hub használatával</span><span class="sxs-lookup"><span data-stu-id="e104a-103">Deploy an app that scales cross-cloud using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="e104a-104">Megtudhatja, hogyan hozhat létre többfelhős megoldást egy olyan manuálisan aktivált folyamat megadására, amely egy Azure Stack hub által üzemeltetett webalkalmazásból egy Azure-ban üzemeltetett webalkalmazásba való váltásra szolgál az automatikus skálázással a Traffic Manager használatával.</span><span class="sxs-lookup"><span data-stu-id="e104a-104">Learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app with autoscaling via traffic manager.</span></span> <span data-ttu-id="e104a-105">Ez a folyamat rugalmas és méretezhető felhőalapú segédprogramot biztosít a betöltés alatt.</span><span class="sxs-lookup"><span data-stu-id="e104a-105">This process ensures flexible and scalable cloud utility when under load.</span></span>

<span data-ttu-id="e104a-106">Ezzel a mintával előfordulhat, hogy a bérlő nem áll készen az alkalmazás futtatására a nyilvános felhőben.</span><span class="sxs-lookup"><span data-stu-id="e104a-106">With this pattern, your tenant may not be ready to run your app in the public cloud.</span></span> <span data-ttu-id="e104a-107">Előfordulhat azonban, hogy nem valósítható meg gazdaságosan a vállalat számára, hogy fenntartsa a helyszíni környezetben szükséges kapacitást, hogy kezelni tudja az alkalmazás iránti igényt.</span><span class="sxs-lookup"><span data-stu-id="e104a-107">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="e104a-108">A bérlő a nyilvános felhő rugalmasságát a helyszíni megoldással is használhatja.</span><span class="sxs-lookup"><span data-stu-id="e104a-108">Your tenant can make use of the elasticity of the public cloud with their on-premises solution.</span></span>

<span data-ttu-id="e104a-109">Ebben a megoldásban egy példaként szolgáló környezetet fog kiépíteni a következőkre:</span><span class="sxs-lookup"><span data-stu-id="e104a-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e104a-110">Hozzon létre egy több csomópontot tartalmazó webalkalmazást.</span><span class="sxs-lookup"><span data-stu-id="e104a-110">Create a multi-node web app.</span></span>
> - <span data-ttu-id="e104a-111">Konfigurálja és felügyelje a folyamatos üzembe helyezés (CD) folyamatát.</span><span class="sxs-lookup"><span data-stu-id="e104a-111">Configure and manage the Continuous Deployment (CD) process.</span></span>
> - <span data-ttu-id="e104a-112">Tegye közzé a webalkalmazást Azure Stack hubhoz.</span><span class="sxs-lookup"><span data-stu-id="e104a-112">Publish the web app to Azure Stack Hub.</span></span>
> - <span data-ttu-id="e104a-113">Hozzon létre egy kiadást.</span><span class="sxs-lookup"><span data-stu-id="e104a-113">Create a release.</span></span>
> - <span data-ttu-id="e104a-114">Ismerje meg az üzemelő példányok figyelését és nyomon követését.</span><span class="sxs-lookup"><span data-stu-id="e104a-114">Learn to monitor and track your deployments.</span></span>

> [!Tip]  
> <span data-ttu-id="e104a-115">![hibrid oszlopok diagramja](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="e104a-115">![hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="e104a-116">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="e104a-116">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="e104a-117">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="e104a-117">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="e104a-118">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="e104a-118">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="e104a-119">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="e104a-119">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="e104a-120">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="e104a-120">Prerequisites</span></span>

- <span data-ttu-id="e104a-121">Egy Azure-előfizetés.</span><span class="sxs-lookup"><span data-stu-id="e104a-121">Azure subscription.</span></span> <span data-ttu-id="e104a-122">Ha szükséges, hozzon létre egy [ingyenes fiókot](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) a Kezdés előtt.</span><span class="sxs-lookup"><span data-stu-id="e104a-122">If needed, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before beginning.</span></span>
- <span data-ttu-id="e104a-123">Azure Stack hub integrált rendszer vagy Azure Stack Development Kit (ASDK) üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="e104a-123">An Azure Stack Hub integrated system or deployment of Azure Stack Development Kit (ASDK).</span></span>
  - <span data-ttu-id="e104a-124">Az Azure Stack hub telepítésére vonatkozó utasításokért lásd: [a ASDK telepítése](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="e104a-124">For instructions on installing Azure Stack Hub, see [Install the ASDK](/azure-stack/asdk/asdk-install).</span></span>
  - <span data-ttu-id="e104a-125">Az üzembe helyezés utáni automatizálási szkriptek ASDK válassza a következőt: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span><span class="sxs-lookup"><span data-stu-id="e104a-125">For an ASDK post-deployment automation script, go to: [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)</span></span>
  - <span data-ttu-id="e104a-126">Előfordulhat, hogy a telepítés elvégzéséhez néhány óra szükséges.</span><span class="sxs-lookup"><span data-stu-id="e104a-126">This installation may require a few hours to complete.</span></span>
- <span data-ttu-id="e104a-127">[App Service](/azure-stack/operator/azure-stack-app-service-deploy) Péter-szolgáltatások üzembe helyezése Azure stack hubhoz.</span><span class="sxs-lookup"><span data-stu-id="e104a-127">Deploy [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS services to Azure Stack Hub.</span></span>
- <span data-ttu-id="e104a-128">[Hozzon létre terveket/ajánlatokat](/azure-stack/operator/service-plan-offer-subscription-overview) a Azure stack hub-környezetben.</span><span class="sxs-lookup"><span data-stu-id="e104a-128">[Create plans/offers](/azure-stack/operator/service-plan-offer-subscription-overview) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="e104a-129">[Bérlői előfizetés létrehozása](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) az Azure stack hub-környezeten belül.</span><span class="sxs-lookup"><span data-stu-id="e104a-129">[Create tenant subscription](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) within the Azure Stack Hub environment.</span></span>
- <span data-ttu-id="e104a-130">Hozzon létre egy webalkalmazást a bérlői előfizetésen belül.</span><span class="sxs-lookup"><span data-stu-id="e104a-130">Create a web app within the tenant subscription.</span></span> <span data-ttu-id="e104a-131">Jegyezze fel az új webalkalmazás URL-címét későbbi használatra.</span><span class="sxs-lookup"><span data-stu-id="e104a-131">Make note of the new web app URL for later use.</span></span>
- <span data-ttu-id="e104a-132">Az Azure-folyamatok virtuális gép (VM) üzembe helyezése a bérlői előfizetésen belül.</span><span class="sxs-lookup"><span data-stu-id="e104a-132">Deploy Azure Pipelines virtual machine (VM) within the tenant subscription.</span></span>
- <span data-ttu-id="e104a-133">A Windows Server 2016 rendszerű virtuális gépeket .NET 3,5-tel kell megadnia.</span><span class="sxs-lookup"><span data-stu-id="e104a-133">Windows Server 2016 VM with .NET 3.5 is required.</span></span> <span data-ttu-id="e104a-134">Ez a virtuális gép az Azure Stack hub bérlői előfizetésében lesz felépítve, mint a privát Build ügynök.</span><span class="sxs-lookup"><span data-stu-id="e104a-134">This VM will be built in the tenant subscription on Azure Stack Hub as the private build agent.</span></span>
- <span data-ttu-id="e104a-135">A [Windows Server 2016 és az SQL 2017](/azure-stack/operator/azure-stack-add-vm-image) virtuálisgép-rendszerkép a Azure stack hub piactéren érhető el.</span><span class="sxs-lookup"><span data-stu-id="e104a-135">[Windows Server 2016 with SQL 2017 VM image](/azure-stack/operator/azure-stack-add-vm-image) is available in the Azure Stack Hub Marketplace.</span></span> <span data-ttu-id="e104a-136">Ha ez a rendszerkép nem érhető el, működjön együtt egy Azure Stack hub-kezelővel, és győződjön meg arról, hogy hozzá van adva a környezethez.</span><span class="sxs-lookup"><span data-stu-id="e104a-136">If this image isn't available, work with an Azure Stack Hub Operator to ensure it's added to the environment.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="e104a-137">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="e104a-137">Issues and considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="e104a-138">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="e104a-138">Scalability</span></span>

<span data-ttu-id="e104a-139">A többfelhős méretezés kulcsfontosságú összetevője az, hogy a nyilvános és a helyszíni felhőalapú infrastruktúra között azonnali és igény szerinti skálázást lehessen biztosítani, amely konzisztens és megbízható szolgáltatást biztosít.</span><span class="sxs-lookup"><span data-stu-id="e104a-139">The key component of cross-cloud scaling is the ability to deliver immediate and on-demand scaling between public and on-premises cloud infrastructure, providing consistent and reliable service.</span></span>

### <a name="availability"></a><span data-ttu-id="e104a-140">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="e104a-140">Availability</span></span>

<span data-ttu-id="e104a-141">Győződjön meg arról, hogy a helyileg telepített alkalmazások magas rendelkezésre állásra vannak konfigurálva a helyszíni hardverkonfiguráció és a szoftverek központi telepítése révén.</span><span class="sxs-lookup"><span data-stu-id="e104a-141">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="e104a-142">Kezelhetőség</span><span class="sxs-lookup"><span data-stu-id="e104a-142">Manageability</span></span>

<span data-ttu-id="e104a-143">A többfelhős megoldás zökkenőmentes felügyeletet és ismerős felületet biztosít a környezetek között.</span><span class="sxs-lookup"><span data-stu-id="e104a-143">The cross-cloud solution ensures seamless management and familiar interface between environments.</span></span> <span data-ttu-id="e104a-144">A PowerShell használata a platformok közötti felügyelethez ajánlott.</span><span class="sxs-lookup"><span data-stu-id="e104a-144">PowerShell is recommended for cross-platform management.</span></span>

## <a name="cross-cloud-scaling"></a><span data-ttu-id="e104a-145">Több felhőre kiterjedő méretezés</span><span class="sxs-lookup"><span data-stu-id="e104a-145">Cross-cloud scaling</span></span>

### <a name="get-a-custom-domain-and-configure-dns"></a><span data-ttu-id="e104a-146">Egyéni tartomány beszerzése és a DNS konfigurálása</span><span class="sxs-lookup"><span data-stu-id="e104a-146">Get a custom domain and configure DNS</span></span>

<span data-ttu-id="e104a-147">Frissítse a tartományhoz tartozó DNS-zónafájl fájlját.</span><span class="sxs-lookup"><span data-stu-id="e104a-147">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="e104a-148">Az Azure AD ellenőrzi az Egyéni tartománynév tulajdonjogát.</span><span class="sxs-lookup"><span data-stu-id="e104a-148">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="e104a-149">Az Azure-ban az Azure/Microsoft 365/külső DNS-rekordok [Azure DNS](/azure/dns/dns-getstarted-portal) használhatók, vagy a DNS-bejegyzést [egy másik DNS-regisztrálónál](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)adja hozzá.</span><span class="sxs-lookup"><span data-stu-id="e104a-149">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="e104a-150">Egyéni tartomány regisztrálása nyilvános regisztrálóval.</span><span class="sxs-lookup"><span data-stu-id="e104a-150">Register a custom domain with a public registrar.</span></span>
2. <span data-ttu-id="e104a-151">Jelentkezzen be a tartomány tartománynév-regisztrálójába.</span><span class="sxs-lookup"><span data-stu-id="e104a-151">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="e104a-152">A DNS-frissítések elvégzéséhez egy jóváhagyott rendszergazdára lehet szükség.</span><span class="sxs-lookup"><span data-stu-id="e104a-152">An approved admin may be required to make DNS updates.</span></span>
3. <span data-ttu-id="e104a-153">Frissítse a tartományhoz tartozó DNS-zónát az Azure AD által biztosított DNS-bejegyzés hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="e104a-153">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="e104a-154">(A DNS-bejegyzés nem befolyásolja az e-mailek útválasztását vagy a webes üzemeltetési viselkedést.)</span><span class="sxs-lookup"><span data-stu-id="e104a-154">(The DNS entry won't affect email routing or web hosting behaviors.)</span></span>

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a><span data-ttu-id="e104a-155">Alapértelmezett több csomópontos Webalkalmazás létrehozása Azure Stack hub-ban</span><span class="sxs-lookup"><span data-stu-id="e104a-155">Create a default multi-node web app in Azure Stack Hub</span></span>

<span data-ttu-id="e104a-156">Hibrid folyamatos integráció és folyamatos üzembe helyezés (CI/CD) beállításával webalkalmazásokat telepíthet az Azure-ba és Azure Stack hub-ra, valamint a felhőbe történő autopush-módosításokat.</span><span class="sxs-lookup"><span data-stu-id="e104a-156">Set up hybrid continuous integration and continuous deployment (CI/CD) to deploy web apps to Azure and Azure Stack Hub and to autopush changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="e104a-157">Azure Stack központ futtatásához (Windows Server és SQL) és a App Service üzembe helyezéshez szükséges megfelelő rendszerképekkel.</span><span class="sxs-lookup"><span data-stu-id="e104a-157">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="e104a-158">További információkért tekintse át a [app Service üzembe helyezéséhez szükséges](/azure-stack/operator/azure-stack-app-service-before-you-get-started)app Service dokumentációt az Azure stack hub-on.</span><span class="sxs-lookup"><span data-stu-id="e104a-158">For more information, review the App Service documentation [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

### <a name="add-code-to-azure-repos"></a><span data-ttu-id="e104a-159">Kód hozzáadása az Azure Reposhez</span><span class="sxs-lookup"><span data-stu-id="e104a-159">Add Code to Azure Repos</span></span>

<span data-ttu-id="e104a-160">Azure Repos</span><span class="sxs-lookup"><span data-stu-id="e104a-160">Azure Repos</span></span>

1. <span data-ttu-id="e104a-161">Jelentkezzen be az Azure Reposba egy olyan fiókkal, amely projekt-létrehozási jogokkal rendelkezik az Azure Reposban.</span><span class="sxs-lookup"><span data-stu-id="e104a-161">Sign in to Azure Repos with an account that has project creation rights on Azure Repos.</span></span>

    <span data-ttu-id="e104a-162">A hibrid CI/CD az alkalmazás kódjára és az infrastruktúra kódjára is alkalmazható.</span><span class="sxs-lookup"><span data-stu-id="e104a-162">Hybrid CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="e104a-163">A saját és a szolgáltatott felhő fejlesztéséhez [Azure Resource Manager sablonokat](https://azure.microsoft.com/resources/templates/) is használhat.</span><span class="sxs-lookup"><span data-stu-id="e104a-163">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Kapcsolódás egy projekthez az Azure Reposban](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. <span data-ttu-id="e104a-165">**A tárház klónozása** az alapértelmezett webalkalmazás létrehozásával és megnyitásával.</span><span class="sxs-lookup"><span data-stu-id="e104a-165">**Clone the repository** by creating and opening the default web app.</span></span>

    ![Adattár klónozása az Azure-webalkalmazásban](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="e104a-167">Saját tulajdonú webalkalmazások létrehozása a App Services mindkét felhőben</span><span class="sxs-lookup"><span data-stu-id="e104a-167">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="e104a-168">Szerkessze a **webalkalmazás. csproj** fájlt.</span><span class="sxs-lookup"><span data-stu-id="e104a-168">Edit the **WebApplication.csproj** file.</span></span> <span data-ttu-id="e104a-169">Válassza ki `Runtimeidentifier` és adja hozzá a elemet `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="e104a-169">Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="e104a-170">(Lásd az [önálló központi telepítési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) dokumentációt.)</span><span class="sxs-lookup"><span data-stu-id="e104a-170">(See [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Webalkalmazás-projektfájl szerkesztése](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. <span data-ttu-id="e104a-172">Az Team Explorer használatával keresse meg a kódot az Azure Reposban.</span><span class="sxs-lookup"><span data-stu-id="e104a-172">Check in the code to Azure Repos using Team Explorer.</span></span>

3. <span data-ttu-id="e104a-173">Győződjön meg róla, hogy az alkalmazás kódja be lett jelölve az Azure Reposban.</span><span class="sxs-lookup"><span data-stu-id="e104a-173">Confirm that the app code has been checked into Azure Repos.</span></span>

## <a name="create-the-build-definition"></a><span data-ttu-id="e104a-174">A Build definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e104a-174">Create the build definition</span></span>

1. <span data-ttu-id="e104a-175">Jelentkezzen be az Azure-folyamatokba, és erősítse meg a Build-definíciók létrehozásának lehetőségét.</span><span class="sxs-lookup"><span data-stu-id="e104a-175">Sign in to Azure Pipelines to confirm the ability to create build definitions.</span></span>

2. <span data-ttu-id="e104a-176">Add **-r win10-x64** kód.</span><span class="sxs-lookup"><span data-stu-id="e104a-176">Add **-r win10-x64** code.</span></span> <span data-ttu-id="e104a-177">Ez a Hozzáadás szükséges a .NET Core-hoz készült önálló telepítés elindításához.</span><span class="sxs-lookup"><span data-stu-id="e104a-177">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Kód hozzáadása a webalkalmazáshoz](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. <span data-ttu-id="e104a-179">Futtassa a buildet.</span><span class="sxs-lookup"><span data-stu-id="e104a-179">Run the build.</span></span> <span data-ttu-id="e104a-180">A [saját üzemeltetésű üzembe helyezési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) folyamat az Azure-ban és Azure stack hub-on futó összetevőket teszi közzé.</span><span class="sxs-lookup"><span data-stu-id="e104a-180">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that run on Azure and Azure Stack Hub.</span></span>

## <a name="use-an-azure-hosted-agent"></a><span data-ttu-id="e104a-181">Azure-beli üzemeltetett ügynök használata</span><span class="sxs-lookup"><span data-stu-id="e104a-181">Use an Azure hosted agent</span></span>

<span data-ttu-id="e104a-182">Az üzemeltetett Build-ügynök használata az Azure-folyamatokban kényelmes megoldás webalkalmazások létrehozására és üzembe helyezésére.</span><span class="sxs-lookup"><span data-stu-id="e104a-182">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="e104a-183">A karbantartást és a frissítéseket a Microsoft Azure automatikusan végzi el, így folyamatos és zavartalan fejlesztési ciklust tesz lehetővé.</span><span class="sxs-lookup"><span data-stu-id="e104a-183">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="e104a-184">A CD-folyamat kezelése és konfigurálása</span><span class="sxs-lookup"><span data-stu-id="e104a-184">Manage and configure the CD process</span></span>

<span data-ttu-id="e104a-185">Az Azure-folyamatok és az Azure DevOps-szolgáltatások kiválóan konfigurálható és kezelhető folyamatokat biztosítanak több környezet, például fejlesztési, átmeneti, MINŐSÉGBIZTOSÍTÁSi és éles környezetek kiadásához; többek között a jóváhagyások megkövetelése adott fázisokban.</span><span class="sxs-lookup"><span data-stu-id="e104a-185">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="e104a-186">Kiadás definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e104a-186">Create release definition</span></span>

1. <span data-ttu-id="e104a-187">A **plusz** gomb kiválasztásával új kiadást adhat hozzá az Azure DevOps Services **Build és Release** szakaszának **kiadások** lapján.</span><span class="sxs-lookup"><span data-stu-id="e104a-187">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Kiadási definíció létrehozása](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. <span data-ttu-id="e104a-189">Alkalmazza a Azure App Service központi telepítési sablont.</span><span class="sxs-lookup"><span data-stu-id="e104a-189">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service központi telepítési sablon alkalmazása](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. <span data-ttu-id="e104a-191">Az összetevő **hozzáadása** területen adja hozzá az Azure Cloud Build alkalmazáshoz tartozó összetevőt.</span><span class="sxs-lookup"><span data-stu-id="e104a-191">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Összetevő hozzáadása az Azure Cloud buildhez](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. <span data-ttu-id="e104a-193">A folyamat lapon válassza ki a **fázist, a feladat** hivatkozását, és állítsa be az Azure Cloud Environment értékeit.</span><span class="sxs-lookup"><span data-stu-id="e104a-193">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Az Azure Cloud Environment értékeinek beállítása](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. <span data-ttu-id="e104a-195">Adja meg a **környezet nevét** , és válassza ki az Azure-beli Felhőbeli végponthoz tartozó **Azure-előfizetést** .</span><span class="sxs-lookup"><span data-stu-id="e104a-195">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure-előfizetés kiválasztása Azure Felhőbeli végponthoz](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. <span data-ttu-id="e104a-197">Az **app Service neve** alatt állítsa be a szükséges Azure app Service-nevet.</span><span class="sxs-lookup"><span data-stu-id="e104a-197">Under **App service name**, set the required Azure app service name.</span></span>

      ![Az Azure app Service nevének beállítása](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. <span data-ttu-id="e104a-199">Adja meg a "üzemeltetett VS2017" kifejezést az Azure Cloud üzemeltetett környezet **ügynök-várólistájában** .</span><span class="sxs-lookup"><span data-stu-id="e104a-199">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Az Azure Cloud üzemeltetett környezet ügynök-várólistájának beállítása](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. <span data-ttu-id="e104a-201">A telepítés Azure App Service menüben válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e104a-201">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e104a-202">Kattintson **az OK** gombra a **mappa helyének** megadásához.</span><span class="sxs-lookup"><span data-stu-id="e104a-202">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service-környezethez tartozó csomag vagy mappa kiválasztása](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Mappa-választó párbeszédpanel 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. <span data-ttu-id="e104a-205">Mentse az összes módosítást, és térjen vissza a **kiadási folyamathoz**.</span><span class="sxs-lookup"><span data-stu-id="e104a-205">Save all changes and go back to **release pipeline**.</span></span>

    ![A kiadási folyamat módosításainak mentése](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. <span data-ttu-id="e104a-207">Adjon hozzá egy új összetevőt, amely kiválasztja az Azure Stack hub alkalmazás buildjét.</span><span class="sxs-lookup"><span data-stu-id="e104a-207">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Új összetevő hozzáadása Azure Stack hub-alkalmazáshoz](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. <span data-ttu-id="e104a-209">Vegyen fel még egy környezetet a Azure App Service központi telepítés alkalmazásával.</span><span class="sxs-lookup"><span data-stu-id="e104a-209">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Környezet hozzáadása Azure App Service központi telepítéshez](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. <span data-ttu-id="e104a-211">Nevezze el az új "Azure Stack" környezetet.</span><span class="sxs-lookup"><span data-stu-id="e104a-211">Name the new environment "Azure Stack".</span></span>

    ![Név környezet Azure App Service üzemelő példányban](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. <span data-ttu-id="e104a-213">Keresse meg a Azure Stack környezetet a **feladat** lapon.</span><span class="sxs-lookup"><span data-stu-id="e104a-213">Find the Azure Stack environment under **Task** tab.</span></span>

    ![Azure Stack környezet](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. <span data-ttu-id="e104a-215">Válassza ki az Azure Stack-végpont előfizetését.</span><span class="sxs-lookup"><span data-stu-id="e104a-215">Select the subscription for the Azure Stack endpoint.</span></span>

    ![Válassza ki az Azure Stack-végpont előfizetését](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. <span data-ttu-id="e104a-217">Adja meg a Azure Stack webalkalmazás nevét az App Service neveként.</span><span class="sxs-lookup"><span data-stu-id="e104a-217">Set the Azure Stack web app name as the App service name.</span></span>
    <span data-ttu-id="e104a-218">![Azure Stack webalkalmazás nevének beállítása](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span><span class="sxs-lookup"><span data-stu-id="e104a-218">![Set Azure Stack web app name](media/solution-deployment-guide-cross-cloud-scaling/image20.png)</span></span>

16. <span data-ttu-id="e104a-219">Válassza ki a Azure Stack ügynököt.</span><span class="sxs-lookup"><span data-stu-id="e104a-219">Select the Azure Stack agent.</span></span>

    ![Azure Stack ügynök kiválasztása](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. <span data-ttu-id="e104a-221">A Azure App Service telepítése szakaszban válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e104a-221">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e104a-222">Kattintson **az OK** gombra a mappa helyének megadásához.</span><span class="sxs-lookup"><span data-stu-id="e104a-222">Select **OK** to folder location.</span></span>

    ![Mappa kiválasztása Azure App Service központi telepítéshez](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Mappa-választó párbeszédpanel 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. <span data-ttu-id="e104a-225">A változó lapon adjon hozzá egy nevű változót `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , állítsa az értékét **igaz** értékre, a hatókört pedig Azure Stackra.</span><span class="sxs-lookup"><span data-stu-id="e104a-225">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack.</span></span>

    ![Változó hozzáadása az Azure-alkalmazások üzembe helyezéséhez](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. <span data-ttu-id="e104a-227">Válassza ki a **folyamatos** üzembe helyezési trigger ikont mindkét összetevőben, és **engedélyezze a folytatás** üzembe helyezési triggert.</span><span class="sxs-lookup"><span data-stu-id="e104a-227">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Folyamatos üzembe helyezési trigger kiválasztása](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. <span data-ttu-id="e104a-229">Válassza ki az **üzembe helyezés előtti** feltételek ikont a Azure stack környezetben, és állítsa be a triggert a **kiadás után.**</span><span class="sxs-lookup"><span data-stu-id="e104a-229">Select the **Pre-deployment** conditions icon in the Azure Stack environment and set the trigger to **After release.**</span></span>

    ![Központi telepítés előtti feltételek kiválasztása](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. <span data-ttu-id="e104a-231">Mentse az összes módosítást.</span><span class="sxs-lookup"><span data-stu-id="e104a-231">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="e104a-232">Előfordulhat, hogy a feladatok egyes beállításai automatikusan [környezeti változókként](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) vannak definiálva, amikor kiadási definíciót hoz létre egy sablonból.</span><span class="sxs-lookup"><span data-stu-id="e104a-232">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="e104a-233">Ezek a beállítások nem módosíthatók a feladat beállításaiban. Ehelyett a szülő környezeti elemet kell kiválasztani a beállítások szerkesztéséhez.</span><span class="sxs-lookup"><span data-stu-id="e104a-233">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a><span data-ttu-id="e104a-234">Közzététel Azure Stack hub-on a Visual Studióval</span><span class="sxs-lookup"><span data-stu-id="e104a-234">Publish to Azure Stack Hub via Visual Studio</span></span>

<span data-ttu-id="e104a-235">A végpontok létrehozásával az Azure DevOps Services buildek üzembe helyezhetik az Azure-szolgáltatások alkalmazásait Azure Stack hubhoz.</span><span class="sxs-lookup"><span data-stu-id="e104a-235">By creating endpoints, an Azure DevOps Services build can deploy Azure Service apps to Azure Stack Hub.</span></span> <span data-ttu-id="e104a-236">Az Azure-folyamatok összekapcsolják a Build ügynököt, amely Azure Stack hubhoz csatlakozik.</span><span class="sxs-lookup"><span data-stu-id="e104a-236">Azure Pipelines connects to the build agent, which connects to Azure Stack Hub.</span></span>

1. <span data-ttu-id="e104a-237">Jelentkezzen be az Azure DevOps Services szolgáltatásba, és lépjen az Alkalmazásbeállítások lapra.</span><span class="sxs-lookup"><span data-stu-id="e104a-237">Sign in to Azure DevOps Services and go to the app settings page.</span></span>

2. <span data-ttu-id="e104a-238">A **Beállítások** területen válassza a **Biztonság** elemet.</span><span class="sxs-lookup"><span data-stu-id="e104a-238">On **Settings**, select **Security**.</span></span>

3. <span data-ttu-id="e104a-239">A **vsts-csoportok** területen válassza a **végpont-létrehozók** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-239">In **VSTS Groups**, select **Endpoint Creators**.</span></span>

4. <span data-ttu-id="e104a-240">A **tagok** lapon válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-240">On the **Members** tab, select **Add**.</span></span>

5. <span data-ttu-id="e104a-241">A **felhasználók és csoportok hozzáadása** lapon adja meg a felhasználónevet, és válassza ki a felhasználót a felhasználók listájából.</span><span class="sxs-lookup"><span data-stu-id="e104a-241">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

6. <span data-ttu-id="e104a-242">Válassza a **Módosítások mentése** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-242">Select **Save changes**.</span></span>

7. <span data-ttu-id="e104a-243">A **vsts-csoportok** listában válassza ki a **végponti rendszergazdák** elemet.</span><span class="sxs-lookup"><span data-stu-id="e104a-243">In the **VSTS Groups** list, select **Endpoint Administrators**.</span></span>

8. <span data-ttu-id="e104a-244">A **tagok** lapon válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-244">On the **Members** tab, select **Add**.</span></span>

9. <span data-ttu-id="e104a-245">A **felhasználók és csoportok hozzáadása** lapon adja meg a felhasználónevet, és válassza ki a felhasználót a felhasználók listájából.</span><span class="sxs-lookup"><span data-stu-id="e104a-245">In **Add users and groups**, enter a user name and select that user from the list of users.</span></span>

10. <span data-ttu-id="e104a-246">Válassza a **Módosítások mentése** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-246">Select **Save changes**.</span></span>

<span data-ttu-id="e104a-247">Most, hogy a végponti információ már létezik, az Azure-folyamatok Azure Stack hub-kapcsolatok használatra készen állnak.</span><span class="sxs-lookup"><span data-stu-id="e104a-247">Now that the endpoint information exists, the Azure Pipelines to Azure Stack Hub connection is ready to use.</span></span> <span data-ttu-id="e104a-248">Az Azure Stack hub Build ügynöke beolvassa az Azure-folyamatok utasításait, majd az ügynök az Azure Stack hub-vel folytatott kommunikációhoz végponti információkat továbbít.</span><span class="sxs-lookup"><span data-stu-id="e104a-248">The build agent in Azure Stack Hub gets instructions from Azure Pipelines and then the agent conveys endpoint information for communication with Azure Stack Hub.</span></span>

## <a name="develop-the-app-build"></a><span data-ttu-id="e104a-249">Az alkalmazás kiépítésének fejlesztése</span><span class="sxs-lookup"><span data-stu-id="e104a-249">Develop the app build</span></span>

> [!Note]  
> <span data-ttu-id="e104a-250">Azure Stack központ futtatásához (Windows Server és SQL) és a App Service üzembe helyezéshez szükséges megfelelő rendszerképekkel.</span><span class="sxs-lookup"><span data-stu-id="e104a-250">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="e104a-251">További információkért lásd: [az App Service üzembe helyezésének Előfeltételei Azure stack központban](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="e104a-251">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

<span data-ttu-id="e104a-252">A felhőben való üzembe helyezéshez használjon [Azure Resource Manager sablonokat](https://azure.microsoft.com/resources/templates/) , például a webalkalmazás kódját az Azure reposből.</span><span class="sxs-lookup"><span data-stu-id="e104a-252">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) like web app code from Azure Repos to deploy to both clouds.</span></span>

### <a name="add-code-to-an-azure-repos-project"></a><span data-ttu-id="e104a-253">Kód hozzáadása egy Azure Repos-projekthez</span><span class="sxs-lookup"><span data-stu-id="e104a-253">Add code to an Azure Repos project</span></span>

1. <span data-ttu-id="e104a-254">Jelentkezzen be az Azure Reposba egy olyan fiókkal, amely Azure Stack hub-beli projekt-létrehozási jogokkal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="e104a-254">Sign in to Azure Repos with an account that has project creation rights on Azure Stack Hub.</span></span>

2. <span data-ttu-id="e104a-255">**A tárház klónozása** az alapértelmezett webalkalmazás létrehozásával és megnyitásával.</span><span class="sxs-lookup"><span data-stu-id="e104a-255">**Clone the repository** by creating and opening the default web app.</span></span>

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a><span data-ttu-id="e104a-256">Saját tulajdonú webalkalmazások létrehozása a App Services mindkét felhőben</span><span class="sxs-lookup"><span data-stu-id="e104a-256">Create self-contained web app deployment for App Services in both clouds</span></span>

1. <span data-ttu-id="e104a-257">Szerkessze a **webalkalmazás. csproj** fájlt: válassza ki `Runtimeidentifier` , majd adja hozzá a elemet `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="e104a-257">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and then add `win10-x64`.</span></span> <span data-ttu-id="e104a-258">További információ: [önálló telepítési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) dokumentáció.</span><span class="sxs-lookup"><span data-stu-id="e104a-258">For more information, see [Self-contained deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.</span></span>

2. <span data-ttu-id="e104a-259">Az Team Explorer segítségével keresse meg a kódot az Azure Reposban.</span><span class="sxs-lookup"><span data-stu-id="e104a-259">Use Team Explorer to check the code into Azure Repos.</span></span>

3. <span data-ttu-id="e104a-260">Győződjön meg róla, hogy az alkalmazás kódja be lett jelölve az Azure Repos-ben.</span><span class="sxs-lookup"><span data-stu-id="e104a-260">Confirm that the app code was checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="e104a-261">A Build definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e104a-261">Create the build definition</span></span>

1. <span data-ttu-id="e104a-262">Jelentkezzen be az Azure-folyamatokba egy olyan fiókkal, amely létrehoz egy Build-definíciót.</span><span class="sxs-lookup"><span data-stu-id="e104a-262">Sign in to Azure Pipelines with an account that can create a build definition.</span></span>

2. <span data-ttu-id="e104a-263">Lépjen a projekthez tartozó **Webalkalmazás létrehozása** lapra.</span><span class="sxs-lookup"><span data-stu-id="e104a-263">Go to the **Build Web Application** page for the project.</span></span>

3. <span data-ttu-id="e104a-264">Az **argumentumokban** adja hozzá az **-r win10-x64** kódot.</span><span class="sxs-lookup"><span data-stu-id="e104a-264">In **Arguments**, add **-r win10-x64** code.</span></span> <span data-ttu-id="e104a-265">Ez a kiegészítés szükséges a .NET Core-ban lévő, önálló telepítés elindításához.</span><span class="sxs-lookup"><span data-stu-id="e104a-265">This addition is required to trigger a self-contained deployment with .NET Core.</span></span>

4. <span data-ttu-id="e104a-266">Futtassa a buildet.</span><span class="sxs-lookup"><span data-stu-id="e104a-266">Run the build.</span></span> <span data-ttu-id="e104a-267">A [saját üzemeltetésű üzembe helyezési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) folyamat olyan összetevőket tesz közzé, amelyek az Azure-ban és a Azure stack hub-ban is futtathatók.</span><span class="sxs-lookup"><span data-stu-id="e104a-267">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="use-an-azure-hosted-build-agent"></a><span data-ttu-id="e104a-268">Azure-beli üzemeltetett Build-ügynök használata</span><span class="sxs-lookup"><span data-stu-id="e104a-268">Use an Azure hosted build agent</span></span>

<span data-ttu-id="e104a-269">Az üzemeltetett Build-ügynök használata az Azure-folyamatokban kényelmes megoldás webalkalmazások létrehozására és üzembe helyezésére.</span><span class="sxs-lookup"><span data-stu-id="e104a-269">Using a hosted build agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="e104a-270">A karbantartást és a frissítéseket a Microsoft Azure automatikusan végzi el, így folyamatos és zavartalan fejlesztési ciklust tesz lehetővé.</span><span class="sxs-lookup"><span data-stu-id="e104a-270">Maintenance and upgrades are done automatically by Microsoft Azure, enabling a continuous and uninterrupted development cycle.</span></span>

### <a name="configure-the-continuous-deployment-cd-process"></a><span data-ttu-id="e104a-271">A folyamatos üzembe helyezés (CD) folyamatának konfigurálása</span><span class="sxs-lookup"><span data-stu-id="e104a-271">Configure the continuous deployment (CD) process</span></span>

<span data-ttu-id="e104a-272">Az Azure-folyamatok és az Azure DevOps-szolgáltatások kiválóan konfigurálható és kezelhető folyamatokat biztosítanak több környezet, például a fejlesztés, az előkészítés, a minőségbiztosítás (QA) és a gyártás számára.</span><span class="sxs-lookup"><span data-stu-id="e104a-272">Azure Pipelines and Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments like development, staging, quality assurance (QA), and production.</span></span> <span data-ttu-id="e104a-273">Ez a folyamat a jóváhagyások megkövetelését is magában foglalhatja az alkalmazás életciklusának adott szakaszaiban.</span><span class="sxs-lookup"><span data-stu-id="e104a-273">This process can include requiring approvals at specific stages of the app life cycle.</span></span>

#### <a name="create-release-definition"></a><span data-ttu-id="e104a-274">Kiadás definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e104a-274">Create release definition</span></span>

<span data-ttu-id="e104a-275">A kiadás definíciójának létrehozása az alkalmazás-létrehozási folyamat utolsó lépése.</span><span class="sxs-lookup"><span data-stu-id="e104a-275">Creating a release definition is the final step in the app build process.</span></span> <span data-ttu-id="e104a-276">Ez a kiadási definíció a kiadás létrehozásához és a buildek üzembe helyezéséhez használható.</span><span class="sxs-lookup"><span data-stu-id="e104a-276">This release definition is used to create a release and deploy a build.</span></span>

1. <span data-ttu-id="e104a-277">Jelentkezzen be az Azure-folyamatokba, és nyissa meg a projekt **felépítését és kiadását** .</span><span class="sxs-lookup"><span data-stu-id="e104a-277">Sign in to Azure Pipelines and go to **Build and Release** for the project.</span></span>

2. <span data-ttu-id="e104a-278">A **kiadások** lapon válassza a **[+]** elemet, majd válassza a **Létrehozás kiadás definíciója** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-278">On the **Releases** tab, select **[ + ]** and then pick **Create release definition**.</span></span>

3. <span data-ttu-id="e104a-279">A **sablon kiválasztása** lapon válassza **Azure app Service központi telepítés** lehetőséget, majd kattintson az **alkalmaz** gombra.</span><span class="sxs-lookup"><span data-stu-id="e104a-279">On **Select a Template**, choose **Azure App Service Deployment**, and then select **Apply**.</span></span>

4. <span data-ttu-id="e104a-280">Az összetevő **hozzáadása** lapon a **forrás (összeállítás definíciója)** területen válassza ki az Azure Cloud Build alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="e104a-280">On **Add artifact**, from the **Source (Build definition)**, select the Azure Cloud build app.</span></span>

5. <span data-ttu-id="e104a-281">A **folyamat** lapon válassza az **1 fázis**, **1 feladat** hivatkozás lehetőséget a **környezeti feladatok megtekintéséhez**.</span><span class="sxs-lookup"><span data-stu-id="e104a-281">On the **Pipeline** tab, select the **1 Phase**, **1 Task** link to **View environment tasks**.</span></span>

6. <span data-ttu-id="e104a-282">A **feladatok** lapon adja meg az Azure-t a **környezet neveként** , majd válassza ki a AzureCloud Traders-Web EP-t az **Azure-előfizetések** listájából.</span><span class="sxs-lookup"><span data-stu-id="e104a-282">On the **Tasks** tab, enter Azure as the **Environment name** and select the AzureCloud Traders-Web EP from the **Azure subscription** list.</span></span>

7. <span data-ttu-id="e104a-283">Adja meg az **Azure app Service nevét**, amely `northwindtraders` a következő képernyőfelvételen szerepel.</span><span class="sxs-lookup"><span data-stu-id="e104a-283">Enter the **Azure app service name**, which is `northwindtraders` in the next screen capture.</span></span>

8. <span data-ttu-id="e104a-284">Az ügynök fázisa esetében az **ügynök-várólista** listából válassza az **üzemeltetett VS2017** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-284">For the Agent phase, select **Hosted VS2017** from the **Agent queue** list.</span></span>

9. <span data-ttu-id="e104a-285">A **telepítés Azure app Service** területen válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e104a-285">In **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span>

10. <span data-ttu-id="e104a-286">A **fájl vagy mappa kiválasztása** lapon kattintson **az OK** gombra a **helyhez**.</span><span class="sxs-lookup"><span data-stu-id="e104a-286">In **Select File or Folder**, select **OK** to **Location**.</span></span>

11. <span data-ttu-id="e104a-287">Mentse az összes módosítást, és térjen vissza a **folyamathoz**.</span><span class="sxs-lookup"><span data-stu-id="e104a-287">Save all changes and go back to **Pipeline**.</span></span>

12. <span data-ttu-id="e104a-288">A **folyamat** lapon válassza az összetevő **hozzáadása** lehetőséget, majd válassza ki a **NorthwindCloud Traders-hajót** a **forrás (összeállítás definíciója)** listából.</span><span class="sxs-lookup"><span data-stu-id="e104a-288">On the **Pipeline** tab, select **Add artifact**, and choose the **NorthwindCloud Traders-Vessel** from the **Source (Build Definition)** list.</span></span>

13. <span data-ttu-id="e104a-289">A **sablon kiválasztása** lapon adjon hozzá egy másik környezetet.</span><span class="sxs-lookup"><span data-stu-id="e104a-289">On **Select a Template**, add another environment.</span></span> <span data-ttu-id="e104a-290">Válassza a **Azure app Service központi telepítés** lehetőséget, majd kattintson az **alkalmaz** gombra.</span><span class="sxs-lookup"><span data-stu-id="e104a-290">Pick **Azure App Service Deployment** and then select **Apply**.</span></span>

14. <span data-ttu-id="e104a-291">Adja meg `Azure Stack Hub` a **környezet nevét**.</span><span class="sxs-lookup"><span data-stu-id="e104a-291">Enter `Azure Stack Hub` as the **Environment name**.</span></span>

15. <span data-ttu-id="e104a-292">A **feladatok** lapon keresse meg és válassza ki Azure stack hubot.</span><span class="sxs-lookup"><span data-stu-id="e104a-292">On the **Tasks** tab, find and select Azure Stack Hub.</span></span>

16. <span data-ttu-id="e104a-293">Az **Azure-előfizetések** listájában válassza a **AzureStack Traders-Vessel EP** elemet az Azure stack hub-végponthoz.</span><span class="sxs-lookup"><span data-stu-id="e104a-293">From the **Azure subscription** list, select **AzureStack Traders-Vessel EP** for the Azure Stack Hub endpoint.</span></span>

17. <span data-ttu-id="e104a-294">Adja meg az Azure Stack hub-webalkalmazás nevét az **app Service neveként**.</span><span class="sxs-lookup"><span data-stu-id="e104a-294">Enter the Azure Stack Hub web app name as the **App service name**.</span></span>

18. <span data-ttu-id="e104a-295">Az **ügynök kiválasztása** területen válassza ki az **AzureStack-b Douglas Fir** elemet az **ügynök-várólista** listából.</span><span class="sxs-lookup"><span data-stu-id="e104a-295">Under **Agent selection**, pick **AzureStack -b Douglas Fir** from the **Agent queue** list.</span></span>

19. <span data-ttu-id="e104a-296">**Azure app Service üzembe helyezéséhez** válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e104a-296">For **Deploy Azure App Service**, select the valid **Package or folder** for the environment.</span></span> <span data-ttu-id="e104a-297">A **fájl vagy mappa kiválasztása** lapon válassza az **OK** lehetőséget a mappa **helyeként**.</span><span class="sxs-lookup"><span data-stu-id="e104a-297">On **Select File Or Folder**, select **OK** for the folder **Location**.</span></span>

20. <span data-ttu-id="e104a-298">A **változó** lapon keresse meg a nevű változót `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` .</span><span class="sxs-lookup"><span data-stu-id="e104a-298">On the **Variable** tab, find the variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`.</span></span> <span data-ttu-id="e104a-299">Állítsa a változó értékét **igaz** értékre, és állítsa a hatókörét **Azure stack hubhoz**.</span><span class="sxs-lookup"><span data-stu-id="e104a-299">Set the variable value to **true**, and set its scope to **Azure Stack Hub**.</span></span>

21. <span data-ttu-id="e104a-300">A **folyamat** lapon válassza ki a **folyamatos üzembe helyezési trigger** ikont a NorthwindCloud Traders-Web-összetevőhöz, és állítsa be a **folyamatos üzembe helyezési triggert** **engedélyezve** értékre.</span><span class="sxs-lookup"><span data-stu-id="e104a-300">On the **Pipeline** tab, select the **Continuous deployment trigger** icon for the NorthwindCloud Traders-Web artifact and set the **Continuous deployment trigger** to **Enabled**.</span></span> <span data-ttu-id="e104a-301">Tegye ugyanezt a **NorthwindCloud-kereskedők** számára.</span><span class="sxs-lookup"><span data-stu-id="e104a-301">Do the same thing for the **NorthwindCloud Traders-Vessel** artifact.</span></span>

22. <span data-ttu-id="e104a-302">Az Azure Stack hub-környezethez válassza az indítás **előtti feltételek** ikont az aktiválás a **kiadás után** beállításnál.</span><span class="sxs-lookup"><span data-stu-id="e104a-302">For the Azure Stack Hub environment, select the **Pre-deployment conditions** icon set the trigger to **After release**.</span></span>

23. <span data-ttu-id="e104a-303">Mentse az összes módosítást.</span><span class="sxs-lookup"><span data-stu-id="e104a-303">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="e104a-304">A kiadási feladatok egyes beállításai automatikusan [környezeti változókként](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) vannak definiálva a kiadási definíciók sablonból való létrehozásakor.</span><span class="sxs-lookup"><span data-stu-id="e104a-304">Some settings for release tasks are automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="e104a-305">Ezek a beállítások nem módosíthatók a feladat beállításaiban, de módosíthatók a szülő környezeti elemekben.</span><span class="sxs-lookup"><span data-stu-id="e104a-305">These settings can't be modified in the task settings but can be modified in the parent environment items.</span></span>

## <a name="create-a-release"></a><span data-ttu-id="e104a-306">Kiadás létrehozása</span><span class="sxs-lookup"><span data-stu-id="e104a-306">Create a release</span></span>

1. <span data-ttu-id="e104a-307">A **folyamat** lapon nyissa meg a **kiadási** listát, és válassza a **kiadás létrehozása** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-307">On the **Pipeline** tab, open the **Release** list and select **Create release**.</span></span>

2. <span data-ttu-id="e104a-308">Adja meg a kiadás leírását, ellenőrizze, hogy a megfelelő összetevők van-e kiválasztva, majd válassza a **Létrehozás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-308">Enter a description for the release, check to see that the correct artifacts are selected, and then select **Create**.</span></span> <span data-ttu-id="e104a-309">Néhány pillanat múlva megjelenik egy szalagcím, amely azt jelzi, hogy az új kiadás létrejött, és a kiadás neve hivatkozásként jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-309">After a few moments, a banner appears indicating that the new release was created and the release name is displayed as a link.</span></span> <span data-ttu-id="e104a-310">Válassza ki a hivatkozást, és tekintse meg a kiadás összegzése lapot.</span><span class="sxs-lookup"><span data-stu-id="e104a-310">Select the link to see the release summary page.</span></span>

3. <span data-ttu-id="e104a-311">A kiadás összegzése lap a kiadás részleteit jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-311">The release summary page shows details about the release.</span></span> <span data-ttu-id="e104a-312">A "Release-2" képernyőn az alábbi képernyőfelvételen a **környezetek** szakasz a "folyamatban" állapotú Azure **telepítési állapotát** jeleníti meg, az Azure stack hub állapota pedig "sikeres".</span><span class="sxs-lookup"><span data-stu-id="e104a-312">In the following screen capture for "Release-2", the **Environments** section shows the **Deployment status** for Azure as "IN PROGRESS", and the status for Azure Stack Hub is "SUCCEEDED".</span></span> <span data-ttu-id="e104a-313">Ha az Azure-környezet központi telepítési állapota "sikeres" állapotúra változik, megjelenik egy szalagcím, amely azt jelzi, hogy a kiadás készen áll a jóváhagyásra.</span><span class="sxs-lookup"><span data-stu-id="e104a-313">When the deployment status for the Azure environment changes to "SUCCEEDED", a banner appears indicating that the release is ready for approval.</span></span> <span data-ttu-id="e104a-314">Ha egy üzemelő példány függőben van vagy sikertelen, kék **(i)** információs ikon jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-314">When a deployment is pending or has failed, a blue **(i)** information icon is shown.</span></span> <span data-ttu-id="e104a-315">Vigye az egérmutatót a ikon fölé egy előugró ablak megjelenítéséhez, amely a késés vagy a hiba okát tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="e104a-315">Hover over the icon to see a pop-up that contains the reason for delay or failure.</span></span>

4. <span data-ttu-id="e104a-316">Más nézetek, például a kiadások listája, egy ikon is megjelenik, amely jelzi, hogy a jóváhagyás függőben van.</span><span class="sxs-lookup"><span data-stu-id="e104a-316">Other views, like the list of releases, also display an icon that indicates approval is pending.</span></span> <span data-ttu-id="e104a-317">Az ikon előugró ablaka a környezet nevét és a telepítéssel kapcsolatos további részleteket jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-317">The pop-up for this icon shows the environment name and more details related to the deployment.</span></span> <span data-ttu-id="e104a-318">A rendszergazda egyszerűen megtekintheti a kiadások általános előrehaladását, és megtekintheti, hogy mely kiadások várnak jóváhagyásra.</span><span class="sxs-lookup"><span data-stu-id="e104a-318">It's easy for an admin see the overall progress of releases and see which releases are waiting for approval.</span></span>

## <a name="monitor-and-track-deployments"></a><span data-ttu-id="e104a-319">Központi telepítések figyelése és nyomon követése</span><span class="sxs-lookup"><span data-stu-id="e104a-319">Monitor and track deployments</span></span>

1. <span data-ttu-id="e104a-320">A **Release-2** Összegzés lapon válassza a **naplók** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-320">On the **Release-2** summary page, select **Logs**.</span></span> <span data-ttu-id="e104a-321">Az üzembe helyezés során ez az oldal az ügynökből származó élő naplót jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-321">During a deployment, this page shows the live log from the agent.</span></span> <span data-ttu-id="e104a-322">A bal oldali ablaktábla megjeleníti az egyes műveletek állapotát a központi telepítésben az egyes környezetekben.</span><span class="sxs-lookup"><span data-stu-id="e104a-322">The left pane shows the status of each operation in the deployment for each environment.</span></span>

2. <span data-ttu-id="e104a-323">A **művelet** oszlopban jelölje ki a személy ikont az üzembe helyezés előtti vagy a telepítés utáni jóváhagyáshoz, és ellenőrizze, hogy ki hagyta jóvá (vagy utasította el) a központi telepítést és a megadott üzenetet.</span><span class="sxs-lookup"><span data-stu-id="e104a-323">Select the person icon in the **Action** column for a pre-deployment or post-deployment approval to see who approved (or rejected) the deployment and the message they provided.</span></span>

3. <span data-ttu-id="e104a-324">Az üzembe helyezés befejeződése után a teljes naplófájl megjelenik a jobb oldali ablaktáblán.</span><span class="sxs-lookup"><span data-stu-id="e104a-324">After the deployment finishes, the entire log file is displayed in the right pane.</span></span> <span data-ttu-id="e104a-325">A bal oldali ablaktábla bármelyik **lépését** kiválasztva megtekintheti a naplófájlt egyetlen lépéshez, például az **inicializálási feladatot**.</span><span class="sxs-lookup"><span data-stu-id="e104a-325">Select any **Step** in the left pane to see the log file for a single step, like **Initialize Job**.</span></span> <span data-ttu-id="e104a-326">Az egyes naplók megtekintésének képessége megkönnyíti a teljes telepítés részeinek nyomon követését és hibakeresését.</span><span class="sxs-lookup"><span data-stu-id="e104a-326">The ability to see individual logs makes it easier to trace and debug parts of the overall deployment.</span></span> <span data-ttu-id="e104a-327">**Mentse** a naplófájlt egy lépéshez, vagy **töltse le az összes naplót zip**-fájlként.</span><span class="sxs-lookup"><span data-stu-id="e104a-327">**Save** the log file for a step or **Download all logs as zip**.</span></span>

4. <span data-ttu-id="e104a-328">Nyissa meg az **Összefoglalás** lapot a kiadással kapcsolatos általános információk megtekintéséhez.</span><span class="sxs-lookup"><span data-stu-id="e104a-328">Open the **Summary** tab to see general information about the release.</span></span> <span data-ttu-id="e104a-329">Ez a nézet a kiépítés, a környezetek üzembe helyezési állapota és a kiadással kapcsolatos egyéb információk részleteit jeleníti meg.</span><span class="sxs-lookup"><span data-stu-id="e104a-329">This view shows details about the build, the environments it was deployed to, deployment status, and other information about the release.</span></span>

5. <span data-ttu-id="e104a-330">Válasszon egy környezeti hivatkozást (**Azure** vagy **Azure stack hub**) a meglévő és a függőben lévő központi telepítésekkel kapcsolatos információk megtekintéséhez egy adott környezetben.</span><span class="sxs-lookup"><span data-stu-id="e104a-330">Select an environment link (**Azure** or **Azure Stack Hub**) to see information about existing and pending deployments to a specific environment.</span></span> <span data-ttu-id="e104a-331">Ezekkel a nézetekkel gyorsan ellenőrizhető, hogy ugyanazt a buildet mindkét környezetbe telepítették-e.</span><span class="sxs-lookup"><span data-stu-id="e104a-331">Use these views as a quick way to check that the same build was deployed to both environments.</span></span>

6. <span data-ttu-id="e104a-332">Nyissa meg az **üzembe helyezett éles alkalmazást** egy böngészőben.</span><span class="sxs-lookup"><span data-stu-id="e104a-332">Open the **deployed production app** in a browser.</span></span> <span data-ttu-id="e104a-333">Az Azure App Services webhelyén például nyissa meg az URL-címet `https://[your-app-name\].azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="e104a-333">For example, for the Azure App Services website, open the URL `https://[your-app-name\].azurewebsites.net`.</span></span>

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a><span data-ttu-id="e104a-334">Az Azure és a Azure Stack hub integrációja méretezhető, többfelhős megoldást kínál</span><span class="sxs-lookup"><span data-stu-id="e104a-334">Integration of Azure and Azure Stack Hub provides a scalable cross-cloud solution</span></span>

<span data-ttu-id="e104a-335">A rugalmas és robusztus többfelhős szolgáltatás adatbiztonságot, biztonsági mentést és redundanciát, egységes és gyors rendelkezésre állást, méretezhető tárolást és elosztást, valamint geo-kompatibilis útválasztást biztosít.</span><span class="sxs-lookup"><span data-stu-id="e104a-335">A flexible and robust multi-cloud service provides data security, back up and redundancy, consistent and rapid availability, scalable storage and distribution, and geo-compliant routing.</span></span> <span data-ttu-id="e104a-336">Ez a manuálisan aktivált folyamat megbízható és hatékony terhelést biztosít a szolgáltatott webalkalmazások között, és azonnal elérhetővé teszi a kritikus fontosságú adatmennyiséget.</span><span class="sxs-lookup"><span data-stu-id="e104a-336">This manually triggered process ensures reliable and efficient load switching between hosted web apps and immediate availability of crucial data.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e104a-337">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="e104a-337">Next steps</span></span>

- <span data-ttu-id="e104a-338">Az Azure Cloud Patterns szolgáltatással kapcsolatos további információkért lásd: [Felhőbeli tervezési minták](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="e104a-338">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
