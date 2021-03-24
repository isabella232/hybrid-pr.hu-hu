---
title: Közvetlen forgalom egy földrajzilag elosztott alkalmazással az Azure és a Azure Stack hub használatával
description: Megtudhatja, hogyan irányíthatja át a forgalmat adott végpontokra egy földrajzilag elosztott alkalmazás-megoldással az Azure és a Azure Stack hub használatával.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 9fa2c351d2c13d85fe1adb17a35e165de96ea2a2
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895431"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a><span data-ttu-id="e06a6-103">Közvetlen forgalom egy földrajzilag elosztott alkalmazással az Azure és a Azure Stack hub használatával</span><span class="sxs-lookup"><span data-stu-id="e06a6-103">Direct traffic with a geo-distributed app using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="e06a6-104">Megtudhatja, hogyan irányíthatja át a forgalmat adott végpontokra különböző mérőszámok alapján a földrajzilag elosztott alkalmazások mintájának használatával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-104">Learn how to direct traffic to specific endpoints based on various metrics using the geo-distributed apps pattern.</span></span> <span data-ttu-id="e06a6-105">A Traffic Manager-profilok földrajzi alapú útválasztási és végponti konfigurációval való létrehozása biztosítja az információk átirányítását a végpontok számára a regionális követelmények, a vállalati és a nemzetközi szabályozás, valamint az adatok igényei alapján.</span><span class="sxs-lookup"><span data-stu-id="e06a6-105">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>

<span data-ttu-id="e06a6-106">Ebben a megoldásban egy példaként szolgáló környezetet fog kiépíteni a következőkre:</span><span class="sxs-lookup"><span data-stu-id="e06a6-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e06a6-107">Hozzon létre egy földrajzilag elosztott alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-107">Create a geo-distributed app.</span></span>
> - <span data-ttu-id="e06a6-108">Az alkalmazás célzásához használja a Traffic Manager.</span><span class="sxs-lookup"><span data-stu-id="e06a6-108">Use Traffic Manager to target your app.</span></span>

## <a name="use-the-geo-distributed-apps-pattern"></a><span data-ttu-id="e06a6-109">A földrajzilag elosztott alkalmazások mintájának használata</span><span class="sxs-lookup"><span data-stu-id="e06a6-109">Use the geo-distributed apps pattern</span></span>

<span data-ttu-id="e06a6-110">A földrajzilag elosztott mintázattal az alkalmazás a régiókat öleli fel.</span><span class="sxs-lookup"><span data-stu-id="e06a6-110">With the geo-distributed pattern, your app spans regions.</span></span> <span data-ttu-id="e06a6-111">Alapértelmezés szerint a nyilvános felhőbe kerül, de előfordulhat, hogy egyes felhasználók az adataikat a saját régiójába kell megtartaniuk.</span><span class="sxs-lookup"><span data-stu-id="e06a6-111">You can default to the public cloud, but some of your users may require that their data remain in their region.</span></span> <span data-ttu-id="e06a6-112">A felhasználókat a legmegfelelőbb felhőre irányíthatja a követelmények alapján.</span><span class="sxs-lookup"><span data-stu-id="e06a6-112">You can direct users to the most suitable cloud based on their requirements.</span></span>

### <a name="issues-and-considerations"></a><span data-ttu-id="e06a6-113">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="e06a6-113">Issues and considerations</span></span>

#### <a name="scalability-considerations"></a><span data-ttu-id="e06a6-114">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="e06a6-114">Scalability considerations</span></span>

<span data-ttu-id="e06a6-115">Az ezzel a cikkel felépített megoldás nem alkalmas a méretezhetőségre.</span><span class="sxs-lookup"><span data-stu-id="e06a6-115">The solution you'll build with this article isn't to accommodate scalability.</span></span> <span data-ttu-id="e06a6-116">Ha azonban más Azure-és helyszíni megoldásokkal együtt használja, a méretezhetőségre vonatkozó követelményeket is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-116">However, if used in combination with other Azure and on-premises solutions, you can accommodate scalability requirements.</span></span> <span data-ttu-id="e06a6-117">A Traffic Manager használatával történő automatikus skálázással rendelkező hibrid megoldások létrehozásával kapcsolatos információkért lásd: [felhőalapú méretezési megoldások létrehozása az Azure-](solution-deployment-guide-cross-cloud-scaling.md)ban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-117">For information on creating a hybrid solution with autoscaling via traffic manager, see [Create cross-cloud scaling solutions with Azure](solution-deployment-guide-cross-cloud-scaling.md).</span></span>

#### <a name="availability-considerations"></a><span data-ttu-id="e06a6-118">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="e06a6-118">Availability considerations</span></span>

<span data-ttu-id="e06a6-119">A méretezhetőséggel kapcsolatos szempontoknak megfelelően ez a megoldás nem foglalkozik közvetlenül a rendelkezésre állással.</span><span class="sxs-lookup"><span data-stu-id="e06a6-119">As is the case with scalability considerations, this solution doesn't directly address availability.</span></span> <span data-ttu-id="e06a6-120">Az Azure és a helyszíni megoldások azonban a megoldáson belül valósíthatók meg, így biztosítva az összes érintett összetevő magas rendelkezésre állását.</span><span class="sxs-lookup"><span data-stu-id="e06a6-120">However, Azure and on-premises solutions can be implemented within this solution to ensure high availability for all components involved.</span></span>

### <a name="when-to-use-this-pattern"></a><span data-ttu-id="e06a6-121">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="e06a6-121">When to use this pattern</span></span>

- <span data-ttu-id="e06a6-122">A szervezet olyan nemzetközi ágakat tartalmaz, amelyek egyéni regionális biztonsági és terjesztési házirendeket igényelnek.</span><span class="sxs-lookup"><span data-stu-id="e06a6-122">Your organization has international branches requiring custom regional security and distribution policies.</span></span>

- <span data-ttu-id="e06a6-123">A szervezet minden irodája lekéri az alkalmazottakat, az üzleti és a létesítmény adatait, amelyek helyi szabályozással és időzónákkal kapcsolatos jelentési tevékenységet igényelnek.</span><span class="sxs-lookup"><span data-stu-id="e06a6-123">Each of your organization's offices pulls employee, business, and facility data, which requires reporting activity per local regulations and time zones.</span></span>

- <span data-ttu-id="e06a6-124">A nagy léptékű követelmények teljesítése az alkalmazások horizontális felskálázásával történik, amely egyetlen régión belül több alkalmazás-telepítéssel rendelkezik, és a szélsőséges terhelési követelmények kezelése érdekében a régiók között.</span><span class="sxs-lookup"><span data-stu-id="e06a6-124">High-scale requirements are met by horizontally scaling out apps with multiple app deployments within a single region and across regions to handle extreme load requirements.</span></span>

### <a name="planning-the-topology"></a><span data-ttu-id="e06a6-125">A topológia megtervezése</span><span class="sxs-lookup"><span data-stu-id="e06a6-125">Planning the topology</span></span>

<span data-ttu-id="e06a6-126">Az elosztott alkalmazás-lábnyom kiépítése előtt a következő dolgokat ismerheti meg:</span><span class="sxs-lookup"><span data-stu-id="e06a6-126">Before building out a distributed app footprint, it helps to know the following things:</span></span>

- <span data-ttu-id="e06a6-127">**Egyéni tartomány az alkalmazáshoz:** Mi az az Egyéni tartománynév, amelyet az ügyfelek az alkalmazás eléréséhez használni fognak?</span><span class="sxs-lookup"><span data-stu-id="e06a6-127">**Custom domain for the app:** What's the custom domain name that customers will use to access the app?</span></span> <span data-ttu-id="e06a6-128">A minta alkalmazás esetében az Egyéni tartománynév a *www \. scalableasedemo.com.*</span><span class="sxs-lookup"><span data-stu-id="e06a6-128">For the sample app, the custom domain name is *www\.scalableasedemo.com.*</span></span>

- <span data-ttu-id="e06a6-129">**Traffic Manager tartomány:** Az [Azure Traffic Manager-profil](/azure/traffic-manager/traffic-manager-manage-profiles)létrehozásakor egy tartománynév van kiválasztva.</span><span class="sxs-lookup"><span data-stu-id="e06a6-129">**Traffic Manager domain:** A domain name is chosen when creating an [Azure Traffic Manager profile](/azure/traffic-manager/traffic-manager-manage-profiles).</span></span> <span data-ttu-id="e06a6-130">Ezt a nevet a *trafficmanager.net* utótaggal kombinálva regisztrálja Traffic Manager által felügyelt tartományi bejegyzést.</span><span class="sxs-lookup"><span data-stu-id="e06a6-130">This name is combined with the *trafficmanager.net* suffix to register a domain entry that's managed by Traffic Manager.</span></span> <span data-ttu-id="e06a6-131">A minta alkalmazás esetében a választott név a *skálázható – a bemutató*.</span><span class="sxs-lookup"><span data-stu-id="e06a6-131">For the sample app, the name chosen is *scalable-ase-demo*.</span></span> <span data-ttu-id="e06a6-132">Ennek eredményeképpen a Traffic Manager által felügyelt teljes tartománynév *Scalable-ASE-demo.trafficmanager.net*.</span><span class="sxs-lookup"><span data-stu-id="e06a6-132">As a result, the full domain name that's managed by Traffic Manager is *scalable-ase-demo.trafficmanager.net*.</span></span>

- <span data-ttu-id="e06a6-133">**Az alkalmazás helyigényének méretezésére szolgáló stratégia:** Döntse el, hogy az alkalmazás adatlábnyoma több App Service-környezetben legyen elosztva egyetlen régióban, több régióban vagy mindkét megközelítés kombinációjában.</span><span class="sxs-lookup"><span data-stu-id="e06a6-133">**Strategy for scaling the app footprint:** Decide whether the app footprint will be distributed across multiple App Service environments in a single region, multiple regions, or a mix of both approaches.</span></span> <span data-ttu-id="e06a6-134">Ennek a döntésnek az alapján kell megjelennie, hogy az ügyfelek forgalmának hol kell származnia, és milyen mértékben méretezhető az alkalmazás további támogatása.</span><span class="sxs-lookup"><span data-stu-id="e06a6-134">The decision should be based on expectations of where customer traffic will originate and how well the rest of an app's supporting back-end infrastructure can scale.</span></span> <span data-ttu-id="e06a6-135">Például egy 100%-os állapot nélküli alkalmazás esetében az alkalmazások nagy mértékben méretezhetők az Azure-régiók több App Service környezetének kombinációjával, és a több Azure-régióban üzembe helyezett App Service környezetek szorzatával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-135">For example, with a 100% stateless app, an app can be massively scaled using a combination of multiple App Service environments per Azure region, multiplied by App Service environments deployed across multiple Azure regions.</span></span> <span data-ttu-id="e06a6-136">A 15 és a globális Azure-régiók közül választhatnak, így az ügyfelek az egész világra kiterjedő, Hyper-Scale szintű alkalmazás-lábnyomot hozhatnak létre.</span><span class="sxs-lookup"><span data-stu-id="e06a6-136">With 15+ global Azure regions available to choose from, customers can truly build a world-wide hyper-scale app footprint.</span></span> <span data-ttu-id="e06a6-137">Az itt használt minta alkalmazáshoz három App Service környezet lett létrehozva egyetlen Azure-régióban (USA déli középső régiója).</span><span class="sxs-lookup"><span data-stu-id="e06a6-137">For the sample app used here, three App Service environments were created in a single Azure region (South Central US).</span></span>

- <span data-ttu-id="e06a6-138">**A app Service környezetek elnevezési konvenciója:** Minden App Service környezethez egyedi név szükséges.</span><span class="sxs-lookup"><span data-stu-id="e06a6-138">**Naming convention for the App Service environments:** Each App Service environment requires a unique name.</span></span> <span data-ttu-id="e06a6-139">Egy vagy két App Service környezeten kívül hasznos lehet elnevezési konvenciója az egyes App Service-környezetek azonosításához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-139">Beyond one or two App Service environments, it's helpful to have a naming convention to help identify each App Service environment.</span></span> <span data-ttu-id="e06a6-140">Az itt használt minta alkalmazáshoz egyszerű elnevezési konvenciót használunk.</span><span class="sxs-lookup"><span data-stu-id="e06a6-140">For the sample app used here, a simple naming convention was used.</span></span> <span data-ttu-id="e06a6-141">A három App Service környezet neve *fe1ase*, *fe2ase* és *fe3ase*.</span><span class="sxs-lookup"><span data-stu-id="e06a6-141">The names of the three App Service environments are *fe1ase*, *fe2ase*, and *fe3ase*.</span></span>

- <span data-ttu-id="e06a6-142">**Az alkalmazások elnevezési konvenciója:** Mivel az alkalmazás több példánya is telepítve lesz, a központilag telepített alkalmazás minden példányához nevet kell megadni.</span><span class="sxs-lookup"><span data-stu-id="e06a6-142">**Naming convention for the apps:** Since multiple instances of the app will be deployed, a name is needed for each instance of the deployed app.</span></span> <span data-ttu-id="e06a6-143">A Power apps App Service Environment esetében ugyanaz az alkalmazásnév több környezetben is használható.</span><span class="sxs-lookup"><span data-stu-id="e06a6-143">With App Service Environment for Power Apps, the same app name can be used across multiple environments.</span></span> <span data-ttu-id="e06a6-144">Mivel minden App Service környezet egyedi tartományi utótaggal rendelkezik, a fejlesztők úgy dönthetnek, hogy ugyanazt az alkalmazást használják az egyes környezetekben.</span><span class="sxs-lookup"><span data-stu-id="e06a6-144">Since each App Service environment has a unique domain suffix, developers can choose to reuse the exact same app name in each environment.</span></span> <span data-ttu-id="e06a6-145">Előfordulhat például, hogy egy fejlesztőnek a következőképpen kell megneveznie az alkalmazásokat: *MyApp.Foo1.p.azurewebsites.net*, *MyApp.Foo2.p.azurewebsites.net*, *MyApp.Foo3.p.azurewebsites.net* stb.</span><span class="sxs-lookup"><span data-stu-id="e06a6-145">For example, a developer could have apps named as follows: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net*, and so on.</span></span> <span data-ttu-id="e06a6-146">Az itt használt alkalmazás esetében minden alkalmazás-példány egyedi névvel rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="e06a6-146">For the app used here, each app instance has a unique name.</span></span> <span data-ttu-id="e06a6-147">Az *webfrontend1*, a *webfrontend2* és a *webfrontend3* használt alkalmazás-példányok nevei.</span><span class="sxs-lookup"><span data-stu-id="e06a6-147">The app instance names used are *webfrontend1*, *webfrontend2*, and *webfrontend3*.</span></span>

> [!Tip]  
> <span data-ttu-id="e06a6-148">![Hibrid oszlopok diagramja](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="e06a6-148">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="e06a6-149">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="e06a6-149">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="e06a6-150">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="e06a6-150">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="e06a6-151">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-151">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="e06a6-152">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="e06a6-152">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="part-1-create-a-geo-distributed-app"></a><span data-ttu-id="e06a6-153">1. rész: földrajzilag elosztott alkalmazás létrehozása</span><span class="sxs-lookup"><span data-stu-id="e06a6-153">Part 1: Create a geo-distributed app</span></span>

<span data-ttu-id="e06a6-154">Ebben a részben egy webalkalmazást fog létrehozni.</span><span class="sxs-lookup"><span data-stu-id="e06a6-154">In this part, you'll create a web app.</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e06a6-155">Webes alkalmazások létrehozása és közzététele.</span><span class="sxs-lookup"><span data-stu-id="e06a6-155">Create web apps and publish.</span></span>
> - <span data-ttu-id="e06a6-156">Kód hozzáadása az Azure Reposhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-156">Add code to Azure Repos.</span></span>
> - <span data-ttu-id="e06a6-157">Az alkalmazás kiépítése több Felhőbeli célpontra.</span><span class="sxs-lookup"><span data-stu-id="e06a6-157">Point the app build to multiple cloud targets.</span></span>
> - <span data-ttu-id="e06a6-158">A CD-folyamat kezelése és konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="e06a6-158">Manage and configure the CD process.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e06a6-159">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="e06a6-159">Prerequisites</span></span>

<span data-ttu-id="e06a6-160">Azure-előfizetésre és Azure Stack hub telepítésre van szükség.</span><span class="sxs-lookup"><span data-stu-id="e06a6-160">An Azure subscription and Azure Stack Hub installation are required.</span></span>

### <a name="geo-distributed-app-steps"></a><span data-ttu-id="e06a6-161">Földrajzilag elosztott alkalmazás lépései</span><span class="sxs-lookup"><span data-stu-id="e06a6-161">Geo-distributed app steps</span></span>

### <a name="obtain-a-custom-domain-and-configure-dns"></a><span data-ttu-id="e06a6-162">Egyéni tartomány beszerzése és a DNS konfigurálása</span><span class="sxs-lookup"><span data-stu-id="e06a6-162">Obtain a custom domain and configure DNS</span></span>

<span data-ttu-id="e06a6-163">Frissítse a tartományhoz tartozó DNS-zónafájl fájlját.</span><span class="sxs-lookup"><span data-stu-id="e06a6-163">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="e06a6-164">Az Azure AD ekkor ellenőrizheti az Egyéni tartománynév tulajdonjogát.</span><span class="sxs-lookup"><span data-stu-id="e06a6-164">Azure AD can then verify ownership of the custom domain name.</span></span> <span data-ttu-id="e06a6-165">Az Azure-ban az Azure/Microsoft 365/külső DNS-rekordok [Azure DNS](/azure/dns/dns-getstarted-portal) használhatók, vagy a DNS-bejegyzést [egy másik DNS-regisztrálónál](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)adja hozzá.</span><span class="sxs-lookup"><span data-stu-id="e06a6-165">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

1. <span data-ttu-id="e06a6-166">Egyéni tartomány regisztrálása nyilvános regisztrálóval.</span><span class="sxs-lookup"><span data-stu-id="e06a6-166">Register a custom domain with a public registrar.</span></span>

2. <span data-ttu-id="e06a6-167">Jelentkezzen be a tartomány tartománynév-regisztrálójába.</span><span class="sxs-lookup"><span data-stu-id="e06a6-167">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="e06a6-168">A DNS-frissítések elvégzéséhez szükség lehet egy jóváhagyott rendszergazdára.</span><span class="sxs-lookup"><span data-stu-id="e06a6-168">An approved admin may be required to make the DNS updates.</span></span>

3. <span data-ttu-id="e06a6-169">Frissítse a tartományhoz tartozó DNS-zónát az Azure AD által biztosított DNS-bejegyzés hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-169">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span> <span data-ttu-id="e06a6-170">A DNS-bejegyzés nem változtatja meg a viselkedést, például a posta vagy a webes üzemeltetést.</span><span class="sxs-lookup"><span data-stu-id="e06a6-170">The DNS entry doesn't change behaviors such as mail routing or web hosting.</span></span>

### <a name="create-web-apps-and-publish"></a><span data-ttu-id="e06a6-171">Webalkalmazások létrehozása és közzététel</span><span class="sxs-lookup"><span data-stu-id="e06a6-171">Create web apps and publish</span></span>

<span data-ttu-id="e06a6-172">Hibrid folyamatos integráció/folyamatos teljesítés (CI/CD) beállítása a webalkalmazás üzembe helyezéséhez az Azure-ban és Azure Stack hub-ban, valamint az automatikus leküldéses módosítások mindkét felhőben.</span><span class="sxs-lookup"><span data-stu-id="e06a6-172">Set up Hybrid Continuous Integration/Continuous Delivery (CI/CD) to deploy Web App to Azure and Azure Stack Hub, and auto push changes to both clouds.</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-173">Azure Stack központ futtatásához (Windows Server és SQL) és a App Service üzembe helyezéshez szükséges megfelelő rendszerképekkel.</span><span class="sxs-lookup"><span data-stu-id="e06a6-173">Azure Stack Hub with proper images syndicated to run (Windows Server and SQL) and App Service deployment are required.</span></span> <span data-ttu-id="e06a6-174">További információkért lásd: [az App Service üzembe helyezésének Előfeltételei Azure stack központban](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span><span class="sxs-lookup"><span data-stu-id="e06a6-174">For more information, see [Prerequisites for deploying App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-before-you-get-started).</span></span>

#### <a name="add-code-to-azure-repos"></a><span data-ttu-id="e06a6-175">Kód hozzáadása az Azure Reposhez</span><span class="sxs-lookup"><span data-stu-id="e06a6-175">Add Code to Azure Repos</span></span>

1. <span data-ttu-id="e06a6-176">Jelentkezzen be a Visual studióba egy olyan **fiókkal, amely projekt-létrehozási jogokkal rendelkezik** az Azure reposban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-176">Sign in to Visual Studio with an **account that has project creation rights** on Azure Repos.</span></span>

    <span data-ttu-id="e06a6-177">A CI/CD az alkalmazás kódjára és az infrastruktúra kódjára is alkalmazható.</span><span class="sxs-lookup"><span data-stu-id="e06a6-177">CI/CD can apply to both app code and infrastructure code.</span></span> <span data-ttu-id="e06a6-178">A saját és a szolgáltatott felhő fejlesztéséhez [Azure Resource Manager sablonokat](https://azure.microsoft.com/resources/templates/) is használhat.</span><span class="sxs-lookup"><span data-stu-id="e06a6-178">Use [Azure Resource Manager templates](https://azure.microsoft.com/resources/templates/) for both private and hosted cloud development.</span></span>

    ![Kapcsolódás egy projekthez a Visual Studióban](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. <span data-ttu-id="e06a6-180">**A tárház klónozása** az alapértelmezett webalkalmazás létrehozásával és megnyitásával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-180">**Clone the repository** by creating and opening the default web app.</span></span>

    ![A klónozott adattár a Visual Studióban](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a><span data-ttu-id="e06a6-182">Webalkalmazás-telepítés létrehozása mindkét felhőben</span><span class="sxs-lookup"><span data-stu-id="e06a6-182">Create web app deployment in both clouds</span></span>

1. <span data-ttu-id="e06a6-183">Szerkessze a **webalkalmazás. csproj** fájlt: válassza ki `Runtimeidentifier` és adja hozzá a elemet `win10-x64` .</span><span class="sxs-lookup"><span data-stu-id="e06a6-183">Edit the **WebApplication.csproj** file: Select `Runtimeidentifier` and add `win10-x64`.</span></span> <span data-ttu-id="e06a6-184">(Lásd az [önálló központi telepítési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) dokumentációt.)</span><span class="sxs-lookup"><span data-stu-id="e06a6-184">(See [Self-contained Deployment](/dotnet/core/deploying/deploy-with-vs#simpleSelf) documentation.)</span></span>

    ![Webalkalmazás-projektfájl szerkesztése a Visual Studióban](media/solution-deployment-guide-geo-distributed/image3.png)

2. <span data-ttu-id="e06a6-186">**Az Team Explorer használatával keresse meg a kódot az Azure reposban** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-186">**Check in the code to Azure Repos** using Team Explorer.</span></span>

3. <span data-ttu-id="e06a6-187">Győződjön meg róla, hogy az **alkalmazás kódja** be lett jelölve az Azure reposban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-187">Confirm that the **application code** has been checked into Azure Repos.</span></span>

### <a name="create-the-build-definition"></a><span data-ttu-id="e06a6-188">A Build definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e06a6-188">Create the build definition</span></span>

1. <span data-ttu-id="e06a6-189">**Jelentkezzen be az Azure-folyamatokba** , és erősítse meg a létrehozási definíciók létrehozásának képességét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-189">**Sign in to Azure Pipelines** to confirm ability to create build definitions.</span></span>

2. <span data-ttu-id="e06a6-190">`-r win10-x64`Kód hozzáadása.</span><span class="sxs-lookup"><span data-stu-id="e06a6-190">Add `-r win10-x64` code.</span></span> <span data-ttu-id="e06a6-191">Ez a Hozzáadás szükséges a .NET Core-hoz készült önálló telepítés elindításához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-191">This addition is necessary to trigger a self-contained deployment with .NET Core.</span></span>

    ![Kód hozzáadása a Build definícióhoz az Azure-folyamatokban](media/solution-deployment-guide-geo-distributed/image4.png)

3. <span data-ttu-id="e06a6-193">**Futtassa a buildet**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-193">**Run the build**.</span></span> <span data-ttu-id="e06a6-194">A [saját üzemeltetésű üzembe helyezési](/dotnet/core/deploying/deploy-with-vs#simpleSelf) folyamat olyan összetevőket tesz közzé, amelyek az Azure-ban és a Azure stack hub-ban is futtathatók.</span><span class="sxs-lookup"><span data-stu-id="e06a6-194">The [self-contained deployment build](/dotnet/core/deploying/deploy-with-vs#simpleSelf) process will publish artifacts that can run on Azure and Azure Stack Hub.</span></span>

#### <a name="using-an-azure-hosted-agent"></a><span data-ttu-id="e06a6-195">Azure-beli üzemeltetett ügynök használata</span><span class="sxs-lookup"><span data-stu-id="e06a6-195">Using an Azure Hosted Agent</span></span>

<span data-ttu-id="e06a6-196">Az üzemeltetett ügynök használata az Azure-folyamatokban kényelmes megoldás webalkalmazások létrehozására és üzembe helyezésére.</span><span class="sxs-lookup"><span data-stu-id="e06a6-196">Using a hosted agent in Azure Pipelines is a convenient option to build and deploy web apps.</span></span> <span data-ttu-id="e06a6-197">A karbantartást és a frissítéseket a Microsoft Azure automatikusan hajtja végre, ami lehetővé teszi a folyamatos fejlesztést, tesztelést és üzembe helyezést.</span><span class="sxs-lookup"><span data-stu-id="e06a6-197">Maintenance and upgrades are automatically performed by Microsoft Azure, which enables uninterrupted development, testing, and deployment.</span></span>

### <a name="manage-and-configure-the-cd-process"></a><span data-ttu-id="e06a6-198">A CD-folyamat kezelése és konfigurálása</span><span class="sxs-lookup"><span data-stu-id="e06a6-198">Manage and configure the CD process</span></span>

<span data-ttu-id="e06a6-199">Az Azure DevOps Services kiválóan konfigurálható és kezelhető folyamatot biztosít több környezet, például fejlesztési, átmeneti, MINŐSÉGBIZTOSÍTÁSi és éles környezetek kiadásához; többek között a jóváhagyások megkövetelése adott fázisokban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-199">Azure DevOps Services provide a highly configurable and manageable pipeline for releases to multiple environments such as development, staging, QA, and production environments; including requiring approvals at specific stages.</span></span>

## <a name="create-release-definition"></a><span data-ttu-id="e06a6-200">Kiadás definíciójának létrehozása</span><span class="sxs-lookup"><span data-stu-id="e06a6-200">Create release definition</span></span>

1. <span data-ttu-id="e06a6-201">A **plusz** gomb kiválasztásával új kiadást adhat hozzá az Azure DevOps Services **Build és Release** szakaszának **kiadások** lapján.</span><span class="sxs-lookup"><span data-stu-id="e06a6-201">Select the **plus** button to add a new release under the **Releases** tab in the **Build and Release** section of Azure DevOps Services.</span></span>

    ![Kiadási definíció létrehozása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image5.png)

2. <span data-ttu-id="e06a6-203">Alkalmazza a Azure App Service központi telepítési sablont.</span><span class="sxs-lookup"><span data-stu-id="e06a6-203">Apply the Azure App Service Deployment template.</span></span>

   ![Azure App Service telepítési sablon alkalmazása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image6.png)

3. <span data-ttu-id="e06a6-205">Az összetevő **hozzáadása** területen adja hozzá az Azure Cloud Build alkalmazáshoz tartozó összetevőt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-205">Under **Add artifact**, add the artifact for the Azure Cloud build app.</span></span>

   ![Összetevő hozzáadása az Azure Cloud buildhez az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image7.png)

4. <span data-ttu-id="e06a6-207">A folyamat lapon válassza ki a **fázist, a feladat** hivatkozását, és állítsa be az Azure Cloud Environment értékeit.</span><span class="sxs-lookup"><span data-stu-id="e06a6-207">Under Pipeline tab, select the **Phase, Task** link of the environment and set the Azure cloud environment values.</span></span>

   ![Azure Cloud Environment-értékek beállítása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image8.png)

5. <span data-ttu-id="e06a6-209">Adja meg a **környezet nevét** , és válassza ki az Azure-beli Felhőbeli végponthoz tartozó **Azure-előfizetést** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-209">Set the **environment name** and select the **Azure subscription** for the Azure Cloud endpoint.</span></span>

      ![Azure-előfizetés kiválasztása Azure-beli felhőalapú végponthoz az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image9.png)

6. <span data-ttu-id="e06a6-211">Az **app Service neve** alatt állítsa be a szükséges Azure app Service-nevet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-211">Under **App service name**, set the required Azure app service name.</span></span>

      ![Azure app Service-név beállítása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image10.png)

7. <span data-ttu-id="e06a6-213">Adja meg a "üzemeltetett VS2017" kifejezést az Azure Cloud üzemeltetett környezet **ügynök-várólistájában** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-213">Enter "Hosted VS2017" under **Agent queue** for Azure cloud hosted environment.</span></span>

      ![Az Azure DevOps Services Azure Cloud üzemeltetett környezetének ügynök-várólistájának beállítása](media/solution-deployment-guide-geo-distributed/image11.png)

8. <span data-ttu-id="e06a6-215">A telepítés Azure App Service menüben válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-215">In Deploy Azure App Service menu, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e06a6-216">Kattintson **az OK** gombra a **mappa helyének** megadásához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-216">Select **OK** to **folder location**.</span></span>
  
      ![Azure App Service-környezethez tartozó csomag vagy mappa kiválasztása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Mappa-választó párbeszédpanel 1](media/solution-deployment-guide-geo-distributed/image13.png)

9. <span data-ttu-id="e06a6-219">Mentse az összes módosítást, és térjen vissza a **kiadási folyamathoz**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-219">Save all changes and go back to **release pipeline**.</span></span>

    ![A kiadási folyamat változásainak mentése az Azure DevOps Services szolgáltatásban](media/solution-deployment-guide-geo-distributed/image14.png)

10. <span data-ttu-id="e06a6-221">Adjon hozzá egy új összetevőt, amely kiválasztja az Azure Stack hub alkalmazás buildjét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-221">Add a new artifact selecting the build for the Azure Stack Hub app.</span></span>

    ![Új összetevő hozzáadása Azure Stack hub-alkalmazáshoz az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image15.png)


11. <span data-ttu-id="e06a6-223">Vegyen fel még egy környezetet a Azure App Service központi telepítés alkalmazásával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-223">Add one more environment by applying the Azure App Service Deployment.</span></span>

    ![Környezet hozzáadása Azure App Service üzembe helyezéséhez az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image16.png)

12. <span data-ttu-id="e06a6-225">Nevezze el az új környezeti Azure Stack hubot.</span><span class="sxs-lookup"><span data-stu-id="e06a6-225">Name the new environment Azure Stack Hub.</span></span>

    ![Név környezet Azure App Service üzembe helyezés az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image17.png)

13. <span data-ttu-id="e06a6-227">Keresse meg a Azure Stack hub-környezetet a **feladat** lapon.</span><span class="sxs-lookup"><span data-stu-id="e06a6-227">Find the Azure Stack Hub environment under **Task** tab.</span></span>

    ![Azure Stack hub-környezet az Azure DevOps Services szolgáltatásban az Azure DevOps Services szolgáltatásban](media/solution-deployment-guide-geo-distributed/image18.png)

14. <span data-ttu-id="e06a6-229">Válassza ki az Azure Stack hub-végpont előfizetését.</span><span class="sxs-lookup"><span data-stu-id="e06a6-229">Select the subscription for the Azure Stack Hub endpoint.</span></span>

    ![Válassza ki az Azure Stack hub-végpont előfizetését az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image19.png)

15. <span data-ttu-id="e06a6-231">Adja meg az Azure Stack hub-webalkalmazás nevét az App Service neveként.</span><span class="sxs-lookup"><span data-stu-id="e06a6-231">Set the Azure Stack Hub web app name as the App service name.</span></span>

    ![Azure Stack hub-webalkalmazás nevének beállítása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image20.png)

16. <span data-ttu-id="e06a6-233">Válassza ki az Azure Stack hub-ügynököt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-233">Select the Azure Stack Hub agent.</span></span>

    ![Az Azure Stack hub-ügynök kiválasztása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image21.png)

17. <span data-ttu-id="e06a6-235">A Azure App Service telepítése szakaszban válassza ki a környezet érvényes **csomagját vagy mappáját** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-235">Under the Deploy Azure App Service section, select the valid **Package or Folder** for the environment.</span></span> <span data-ttu-id="e06a6-236">Kattintson **az OK** gombra a mappa helyének megadásához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-236">Select **OK** to folder location.</span></span>

    ![Mappa kiválasztása Azure App Service üzembe helyezéséhez az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Mappa-választó párbeszédpanel 2](media/solution-deployment-guide-geo-distributed/image23.png)

18. <span data-ttu-id="e06a6-239">A változó lapon adjon hozzá egy nevű változót `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , állítsa az értékét **igaz** értékre, és hatókörét Azure stack hubhoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-239">Under Variable tab add a variable named `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, set its value as **true**, and scope to Azure Stack Hub.</span></span>

    ![Változó hozzáadása az Azure-alkalmazások üzembe helyezéséhez az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image24.png)

19. <span data-ttu-id="e06a6-241">Válassza ki a **folyamatos** üzembe helyezési trigger ikont mindkét összetevőben, és **engedélyezze a folytatás** üzembe helyezési triggert.</span><span class="sxs-lookup"><span data-stu-id="e06a6-241">Select the **Continuous** deployment trigger icon in both artifacts and enable the **Continues** deployment trigger.</span></span>

    ![Folyamatos üzembe helyezési trigger kiválasztása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image25.png)

20. <span data-ttu-id="e06a6-243">Válassza ki az **üzembe helyezés előtti** feltételek ikont az Azure stack hub-környezetben, és állítsa be a triggert a **kiadás után.**</span><span class="sxs-lookup"><span data-stu-id="e06a6-243">Select the **Pre-deployment** conditions icon in the Azure Stack Hub environment and set the trigger to **After release.**</span></span>

    ![Üzembe helyezés előtti feltételek kiválasztása az Azure DevOps Servicesben](media/solution-deployment-guide-geo-distributed/image26.png)

21. <span data-ttu-id="e06a6-245">Mentse az összes módosítást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-245">Save all changes.</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-246">Előfordulhat, hogy a feladatok egyes beállításai automatikusan [környezeti változókként](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) vannak definiálva, amikor kiadási definíciót hoz létre egy sablonból.</span><span class="sxs-lookup"><span data-stu-id="e06a6-246">Some settings for the tasks may have been automatically defined as [environment variables](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) when creating a release definition from a template.</span></span> <span data-ttu-id="e06a6-247">Ezek a beállítások nem módosíthatók a feladat beállításaiban. Ehelyett a szülő környezeti elemet kell kiválasztani a beállítások szerkesztéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-247">These settings can't be modified in the task settings; instead, the parent environment item must be selected to edit these settings.</span></span>

## <a name="part-2-update-web-app-options"></a><span data-ttu-id="e06a6-248">2. rész: a webalkalmazás beállításainak frissítése</span><span class="sxs-lookup"><span data-stu-id="e06a6-248">Part 2: Update web app options</span></span>

<span data-ttu-id="e06a6-249">Az [Azure App Service](/azure/app-service/overview) egy hatékonyan méretezhető, önjavító webes üzemeltetési szolgáltatás.</span><span class="sxs-lookup"><span data-stu-id="e06a6-249">[Azure App Service](/azure/app-service/overview) provides a highly scalable, self-patching web hosting service.</span></span>

![Azure App Service](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - <span data-ttu-id="e06a6-251">Meglévő egyéni DNS-név leképezése az Azure Web Appsra.</span><span class="sxs-lookup"><span data-stu-id="e06a6-251">Map an existing custom DNS name to Azure Web Apps.</span></span>
> - <span data-ttu-id="e06a6-252">CNAME- **rekord** és egy **rekord** használatával rendelje hozzá az egyéni DNS-nevet a app Servicehoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-252">Use a **CNAME record** and an **A record** to map a custom DNS name to App Service.</span></span>

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a><span data-ttu-id="e06a6-253">Meglévő egyéni DNS-név leképezése az Azure Web Appsra</span><span class="sxs-lookup"><span data-stu-id="e06a6-253">Map an existing custom DNS name to Azure Web Apps</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-254">Használjon CNAME-t az összes egyéni DNS-névhez, kivéve a legfelső szintű tartományt (például northwind.com).</span><span class="sxs-lookup"><span data-stu-id="e06a6-254">Use a CNAME for all custom DNS names except a root domain (for example, northwind.com).</span></span>

<span data-ttu-id="e06a6-255">Élő webhely és hozzá tartozó DNS-tartománynév migrálása az App Service-be: [Aktív DNS-név migrálása az Azure App Service-be](/azure/app-service/manage-custom-dns-migrate-domain).</span><span class="sxs-lookup"><span data-stu-id="e06a6-255">To migrate a live site and its DNS domain name to App Service, see [Migrate an active DNS name to Azure App Service](/azure/app-service/manage-custom-dns-migrate-domain).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e06a6-256">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="e06a6-256">Prerequisites</span></span>

<span data-ttu-id="e06a6-257">A megoldás elvégzéséhez:</span><span class="sxs-lookup"><span data-stu-id="e06a6-257">To complete this solution:</span></span>

- <span data-ttu-id="e06a6-258">[Hozzon létre egy app Service alkalmazást](/azure/app-service/), vagy használjon egy másik megoldáshoz létrehozott alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-258">[Create an App Service app](/azure/app-service/), or use an app created for another  solution.</span></span>

- <span data-ttu-id="e06a6-259">Adjon meg egy tartománynevet, és győződjön meg arról, hogy a tartományi szolgáltató DNS-beállításjegyzéke elérhető.</span><span class="sxs-lookup"><span data-stu-id="e06a6-259">Purchase a domain name and ensure access to the DNS registry for the domain provider.</span></span>

<span data-ttu-id="e06a6-260">Frissítse a tartományhoz tartozó DNS-zónafájl fájlját.</span><span class="sxs-lookup"><span data-stu-id="e06a6-260">Update the DNS zone file for the domain.</span></span> <span data-ttu-id="e06a6-261">Az Azure AD ellenőrzi az Egyéni tartománynév tulajdonjogát.</span><span class="sxs-lookup"><span data-stu-id="e06a6-261">Azure AD will verify ownership of the custom domain name.</span></span> <span data-ttu-id="e06a6-262">Az Azure-ban az Azure/Microsoft 365/külső DNS-rekordok [Azure DNS](/azure/dns/dns-getstarted-portal) használhatók, vagy a DNS-bejegyzést [egy másik DNS-regisztrálónál](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)adja hozzá.</span><span class="sxs-lookup"><span data-stu-id="e06a6-262">Use [Azure DNS](/azure/dns/dns-getstarted-portal) for Azure/Microsoft 365/external DNS records within Azure, or add the DNS entry at [a different DNS registrar](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).</span></span>

- <span data-ttu-id="e06a6-263">Egyéni tartomány regisztrálása nyilvános regisztrálóval.</span><span class="sxs-lookup"><span data-stu-id="e06a6-263">Register a custom domain with a public registrar.</span></span>

- <span data-ttu-id="e06a6-264">Jelentkezzen be a tartomány tartománynév-regisztrálójába.</span><span class="sxs-lookup"><span data-stu-id="e06a6-264">Sign in to the domain name registrar for the domain.</span></span> <span data-ttu-id="e06a6-265">(A DNS-frissítések elvégzéséhez egy jóváhagyott rendszergazdára lehet szükség.)</span><span class="sxs-lookup"><span data-stu-id="e06a6-265">(An approved admin may be required to make DNS updates.)</span></span>

- <span data-ttu-id="e06a6-266">Frissítse a tartományhoz tartozó DNS-zónát az Azure AD által biztosított DNS-bejegyzés hozzáadásával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-266">Update the DNS zone file for the domain by adding the DNS entry provided by Azure AD.</span></span>

<span data-ttu-id="e06a6-267">Például a northwindcloud.com és a www northwindcloud.com DNS-bejegyzéseinek hozzáadásához \. konfigurálja a northwindcloud.com DNS-beállításait.</span><span class="sxs-lookup"><span data-stu-id="e06a6-267">For example, to add DNS entries for northwindcloud.com and www\.northwindcloud.com, configure DNS settings for the northwindcloud.com root domain.</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-268">A [Azure Portal](/azure/app-service/manage-custom-dns-buy-domain)használatával megvásárolható egy tartománynév.</span><span class="sxs-lookup"><span data-stu-id="e06a6-268">A domain name may be purchased using the [Azure portal](/azure/app-service/manage-custom-dns-buy-domain).</span></span> <span data-ttu-id="e06a6-269">Egy egyéni DNS-név webalkalmazásra való leképezéséhez a webalkalmazás [App Service-csomagjának](https://azure.microsoft.com/pricing/details/app-service/) fizetős rétegben kell lennie (**megosztott**, **alapvető**, **szabványos** vagy **prémium szintű**).</span><span class="sxs-lookup"><span data-stu-id="e06a6-269">To map a custom DNS name to a web app, the web app's [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be a paid tier (**Shared**, **Basic**, **Standard**, or **Premium**).</span></span>

### <a name="create-and-map-cname-and-a-records"></a><span data-ttu-id="e06a6-270">CNAME és rekordok létrehozása és leképezése</span><span class="sxs-lookup"><span data-stu-id="e06a6-270">Create and map CNAME and A records</span></span>

#### <a name="access-dns-records-with-domain-provider"></a><span data-ttu-id="e06a6-271">DNS-rekordok elérése tartományszolgáltató esetén</span><span class="sxs-lookup"><span data-stu-id="e06a6-271">Access DNS records with domain provider</span></span>

> [!Note]  
>  <span data-ttu-id="e06a6-272">A Azure DNS használatával konfigurálhatja az Azure Web Apps egyéni DNS-nevét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-272">Use Azure DNS to configure a custom DNS name for Azure Web Apps.</span></span> <span data-ttu-id="e06a6-273">További információt az [egyéni tartománybeállítások egy Azure-szolgáltatáshoz az Azure DNS használatával történő megadását](/azure/dns/dns-custom-domain) ismertető cikkben talál.</span><span class="sxs-lookup"><span data-stu-id="e06a6-273">For more information, see [Use Azure DNS to provide custom domain settings for an Azure service](/azure/dns/dns-custom-domain).</span></span>

1. <span data-ttu-id="e06a6-274">Jelentkezzen be a fő szolgáltató webhelyére.</span><span class="sxs-lookup"><span data-stu-id="e06a6-274">Sign in to the website of the main provider.</span></span>

2. <span data-ttu-id="e06a6-275">Keresse meg a DNS-rekordok kezelésére szolgáló oldalt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-275">Find the page for managing DNS records.</span></span> <span data-ttu-id="e06a6-276">Minden tartományi szolgáltató saját DNS-rekordok felülettel rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="e06a6-276">Every domain provider has its own DNS records interface.</span></span> <span data-ttu-id="e06a6-277">A webhely **Tartománynév**, **DNS** vagy **Névkiszolgáló kezelése** címkével ellátott területeit keresse.</span><span class="sxs-lookup"><span data-stu-id="e06a6-277">Look for areas of the site labeled **Domain Name**, **DNS**, or **Name Server Management**.</span></span>

<span data-ttu-id="e06a6-278">A DNS-rekordok oldala megtekinthető a **saját tartományokban**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-278">DNS records page can be viewed in **My domains**.</span></span> <span data-ttu-id="e06a6-279">Keresse meg a **zónafájl**, **DNS-rekordok** vagy **Speciális konfiguráció** nevű hivatkozást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-279">Find the link named **Zone file**, **DNS Records**, or **Advanced configuration**.</span></span>

<span data-ttu-id="e06a6-280">A következő képernyőkép egy DNS-rekordokat tartalmazó oldalra mutat példát:</span><span class="sxs-lookup"><span data-stu-id="e06a6-280">The following screenshot is an example of a DNS records page:</span></span>

![DNS-rekordokat tartalmazó oldal példája](media/solution-deployment-guide-geo-distributed/image28.png)

1. <span data-ttu-id="e06a6-282">A tartománynév-Regisztrálóban válassza a **Hozzáadás vagy a létrehozás** elemet a rekord létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-282">In Domain Name Registrar, select **Add or Create** to create a record.</span></span> <span data-ttu-id="e06a6-283">Egyes szolgáltatók eltérő hivatkozásokat használnak a különböző rekordtípusok hozzáadásához.</span><span class="sxs-lookup"><span data-stu-id="e06a6-283">Some providers have different links to add different record types.</span></span> <span data-ttu-id="e06a6-284">A szolgáltató dokumentációjában tájékozódhat.</span><span class="sxs-lookup"><span data-stu-id="e06a6-284">Consult the provider's documentation.</span></span>

2. <span data-ttu-id="e06a6-285">Adjon hozzá egy CNAME rekordot, amely altartományt rendel az alkalmazás alapértelmezett állomásnevét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-285">Add a CNAME record to map a subdomain to the app's default hostname.</span></span>

   <span data-ttu-id="e06a6-286">A www \. northwindcloud.com-tartományhoz példaként adjon hozzá egy CNAME-rekordot, amely leképezi a nevet a következőhöz: `<app_name>.azurewebsites.net` .</span><span class="sxs-lookup"><span data-stu-id="e06a6-286">For the www\.northwindcloud.com domain example, add a CNAME record that maps the name to `<app_name>.azurewebsites.net`.</span></span>

<span data-ttu-id="e06a6-287">A CNAME hozzáadása után a DNS-rekordok oldal a következő példához hasonlóan néz ki:</span><span class="sxs-lookup"><span data-stu-id="e06a6-287">After adding the CNAME, the DNS records page looks like the following example:</span></span>

![Navigálás a portálon egy Azure-alkalmazáshoz](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a><span data-ttu-id="e06a6-289">A CNAME rekord hozzárendelésének engedélyezése az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="e06a6-289">Enable the CNAME record mapping in Azure</span></span>

1. <span data-ttu-id="e06a6-290">Egy új lapon jelentkezzen be a Azure Portalba.</span><span class="sxs-lookup"><span data-stu-id="e06a6-290">In a new tab, sign in to the Azure portal.</span></span>

2. <span data-ttu-id="e06a6-291">Lépjen App Services.</span><span class="sxs-lookup"><span data-stu-id="e06a6-291">Go to App Services.</span></span>

3. <span data-ttu-id="e06a6-292">Válassza a webalkalmazás lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-292">Select web app.</span></span>

4. <span data-ttu-id="e06a6-293">Az Azure Portal bal oldali navigációs sávján válassza ki az **Egyéni tartományok** elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-293">In the left navigation of the app page in the Azure portal, select **Custom domains**.</span></span>

5. <span data-ttu-id="e06a6-294">Jelölje be az **+** **állomásnév hozzáadása** elem melletti ikont.</span><span class="sxs-lookup"><span data-stu-id="e06a6-294">Select the **+** icon next to **Add hostname**.</span></span>

6. <span data-ttu-id="e06a6-295">Írja be a teljes tartománynevet, például: `www.northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="e06a6-295">Type the fully qualified domain name, like `www.northwindcloud.com`.</span></span>

7. <span data-ttu-id="e06a6-296">Válassza az **Érvényesítés** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-296">Select **Validate**.</span></span>

8. <span data-ttu-id="e06a6-297">Ha meg van jelölve, további rekordokat adhat hozzá más típusokhoz ( `A` vagy `TXT` ) a tartománynév-regisztráló DNS-rekordjaihoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-297">If indicated, add additional records of other types (`A` or `TXT`) to the domain name registrars DNS records.</span></span> <span data-ttu-id="e06a6-298">Az Azure megadja a rekordok értékeit és típusát:</span><span class="sxs-lookup"><span data-stu-id="e06a6-298">Azure will provide the values and types of these records:</span></span>

   <span data-ttu-id="e06a6-299">a.</span><span class="sxs-lookup"><span data-stu-id="e06a6-299">a.</span></span>  <span data-ttu-id="e06a6-300">egy **A** rekordra, amelyet leképezhet az alkalmazás IP-címére.</span><span class="sxs-lookup"><span data-stu-id="e06a6-300">An **A** record to map to the app's IP address.</span></span>

   <span data-ttu-id="e06a6-301">b.</span><span class="sxs-lookup"><span data-stu-id="e06a6-301">b.</span></span>  <span data-ttu-id="e06a6-302">egy **TXT** típusú rekordra, amelyet leképezhet az alkalmazás alapértelmezett `<app_name>.azurewebsites.net` gazdagépnevére.</span><span class="sxs-lookup"><span data-stu-id="e06a6-302">A **TXT** record to map to the app's default hostname `<app_name>.azurewebsites.net`.</span></span> <span data-ttu-id="e06a6-303">App Service ezt a rekordot csak a konfiguráció idejére használja az egyéni tartomány tulajdonjogának ellenőrzéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-303">App Service uses this record only at configuration time to verify custom domain ownership.</span></span> <span data-ttu-id="e06a6-304">Az ellenőrzés után törölje a TXT-rekordot.</span><span class="sxs-lookup"><span data-stu-id="e06a6-304">After verification, delete the TXT record.</span></span>

9. <span data-ttu-id="e06a6-305">Hajtsa végre ezt a feladatot a tartományregisztráló lapon, majd az **állomásnév hozzáadása** gomb aktiválása után ellenőrizze újra a műveletet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-305">Complete this task in the domain registrar tab and revalidate until the **Add hostname** button is activated.</span></span>

10. <span data-ttu-id="e06a6-306">Győződjön meg arról, hogy az **állomásnév bejegyzéstípusa** **CNAME** (www.example.com vagy bármely altartomány) értékre van beállítva.</span><span class="sxs-lookup"><span data-stu-id="e06a6-306">Make sure that **Hostname record type** is set to **CNAME** (www.example.com or any subdomain).</span></span>

11. <span data-ttu-id="e06a6-307">Válassza a **Gazdagépnév hozzáadása** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-307">Select **Add hostname**.</span></span>

12. <span data-ttu-id="e06a6-308">Írja be a teljes tartománynevet, például: `northwindcloud.com` .</span><span class="sxs-lookup"><span data-stu-id="e06a6-308">Type the fully qualified domain name, like `northwindcloud.com`.</span></span>

13. <span data-ttu-id="e06a6-309">Válassza az **Érvényesítés** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-309">Select **Validate**.</span></span> <span data-ttu-id="e06a6-310">A **Hozzáadás** aktiválva van.</span><span class="sxs-lookup"><span data-stu-id="e06a6-310">The **Add** is activated.</span></span>

14. <span data-ttu-id="e06a6-311">Győződjön meg arról, hogy az **állomásnév bejegyzéstípus** **egy rekordra** (example.com) van beállítva.</span><span class="sxs-lookup"><span data-stu-id="e06a6-311">Make sure that **Hostname record type** is set to **A record** (example.com).</span></span>

15. <span data-ttu-id="e06a6-312">**Adja hozzá a gazdagépet**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-312">**Add hostname**.</span></span>

    <span data-ttu-id="e06a6-313">Eltarthat egy ideig, amíg az új állomásnevek megjelennek az alkalmazás **Egyéni tartományok** lapján.</span><span class="sxs-lookup"><span data-stu-id="e06a6-313">It might take some time for the new hostnames to be reflected in the app's **Custom domains** page.</span></span> <span data-ttu-id="e06a6-314">Próbálja meg frissíteni a böngészőt az adatok frissítéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-314">Try refreshing the browser to update the data.</span></span>
  
    ![Egyéni tartományok](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    <span data-ttu-id="e06a6-316">Ha hiba történik, a lap alján egy ellenőrző hibaüzenet jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="e06a6-316">If there's an error, a verification error notification will appear at the bottom of the page.</span></span> ![Tartomány-ellenőrzési hiba](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  <span data-ttu-id="e06a6-318">A fenti lépések megismétlődnek a helyettesítő karakteres tartomány ( \* . northwindcloud.com) leképezéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-318">The above steps may be repeated to map a wildcard domain (\*.northwindcloud.com).</span></span> <span data-ttu-id="e06a6-319">Ez lehetővé teszi további altartományok hozzáadását az App Service-hez anélkül, hogy mindegyikhez külön CNAME rekordot kellene létrehoznia.</span><span class="sxs-lookup"><span data-stu-id="e06a6-319">This allows the addition of any additional subdomains to this app service without having to create a separate CNAME record for each one.</span></span> <span data-ttu-id="e06a6-320">A beállítás konfigurálásához kövesse a regisztrátor utasításait.</span><span class="sxs-lookup"><span data-stu-id="e06a6-320">Follow the registrar instructions to configure this setting.</span></span>

#### <a name="test-in-a-browser"></a><span data-ttu-id="e06a6-321">Tesztelés böngészőben</span><span class="sxs-lookup"><span data-stu-id="e06a6-321">Test in a browser</span></span>

<span data-ttu-id="e06a6-322">Tallózással keresse meg a korábban konfigurált DNS-név (oka) t (például `northwindcloud.com` vagy `www.northwindcloud.com` ).</span><span class="sxs-lookup"><span data-stu-id="e06a6-322">Browse to the DNS name(s) configured earlier (for example, `northwindcloud.com` or `www.northwindcloud.com`).</span></span>

## <a name="part-3-bind-a-custom-ssl-cert"></a><span data-ttu-id="e06a6-323">3. rész: egyéni SSL-tanúsítvány kötése</span><span class="sxs-lookup"><span data-stu-id="e06a6-323">Part 3: Bind a custom SSL cert</span></span>

<span data-ttu-id="e06a6-324">Ebben a részben a következőket tesszük:</span><span class="sxs-lookup"><span data-stu-id="e06a6-324">In this part, we will:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="e06a6-325">Az egyéni SSL-tanúsítvány kötése App Servicehoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-325">Bind the custom SSL certificate to App Service.</span></span>
> - <span data-ttu-id="e06a6-326">HTTPS kényszerített alkalmazása az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-326">Enforce HTTPS for the app.</span></span>
> - <span data-ttu-id="e06a6-327">SSL-tanúsítvány kötésének automatizálása parancsfájlok használatával.</span><span class="sxs-lookup"><span data-stu-id="e06a6-327">Automate SSL certificate binding with scripts.</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-328">Ha szükséges, szerezze be a Azure Portal ügyfél SSL-tanúsítványát, és kösse azt a webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-328">If needed, obtain a customer SSL certificate in the Azure portal and bind it to the web app.</span></span> <span data-ttu-id="e06a6-329">További információkért tekintse meg a [app Service Certificates oktatóanyagot](/azure/app-service/web-sites-purchase-ssl-web-site).</span><span class="sxs-lookup"><span data-stu-id="e06a6-329">For more information, see the [App Service Certificates tutorial](/azure/app-service/web-sites-purchase-ssl-web-site).</span></span>

### <a name="prerequisites"></a><span data-ttu-id="e06a6-330">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="e06a6-330">Prerequisites</span></span>

<span data-ttu-id="e06a6-331">A megoldás elvégzéséhez:</span><span class="sxs-lookup"><span data-stu-id="e06a6-331">To complete this  solution:</span></span>

- [<span data-ttu-id="e06a6-332">Hozzon létre egy App Service alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-332">Create an App Service app.</span></span>](/azure/app-service/)
- [<span data-ttu-id="e06a6-333">Rendelje hozzá az egyéni DNS-nevet a webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-333">Map a custom DNS name to your web app.</span></span>](/azure/app-service/app-service-web-tutorial-custom-domain)
- <span data-ttu-id="e06a6-334">Szerezzen be egy SSL-tanúsítványt egy megbízható hitelesítésszolgáltatótól, és a kulcs használatával írja alá a kérelmet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-334">Acquire an SSL certificate from a trusted certificate authority and use the key to sign the request.</span></span>

### <a name="requirements-for-your-ssl-certificate"></a><span data-ttu-id="e06a6-335">Az SSL-tanúsítvány követelményei</span><span class="sxs-lookup"><span data-stu-id="e06a6-335">Requirements for your SSL certificate</span></span>

<span data-ttu-id="e06a6-336">A tanúsítvány App Service-ben történő használatához a tanúsítványnak meg kell felelnie az alábbi követelmények mindegyikének:</span><span class="sxs-lookup"><span data-stu-id="e06a6-336">To use a certificate in App Service, the certificate must meet all the following requirements:</span></span>

- <span data-ttu-id="e06a6-337">Egy megbízható hitelesítésszolgáltató írta alá.</span><span class="sxs-lookup"><span data-stu-id="e06a6-337">Signed by a trusted certificate authority.</span></span>

- <span data-ttu-id="e06a6-338">Jelszóval védett PFX-fájlként lett exportálva.</span><span class="sxs-lookup"><span data-stu-id="e06a6-338">Exported as a password-protected PFX file.</span></span>

- <span data-ttu-id="e06a6-339">Legalább 2048 bit hosszúságú titkos kulcsot tartalmaz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-339">Contains private key at least 2048 bits long.</span></span>

- <span data-ttu-id="e06a6-340">A tanúsítványlánc összes köztes tanúsítványát tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="e06a6-340">Contains all intermediate certificates in the certificate chain.</span></span>

> [!Note]  
> <span data-ttu-id="e06a6-341">Az **elliptikus görbe titkosítási (ECC-) tanúsítványok** app Service, de nem szerepelnek ebben az útmutatóban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-341">**Elliptic Curve Cryptography (ECC) certificates** work with App Service but aren't included in this guide.</span></span> <span data-ttu-id="e06a6-342">Az ECC-tanúsítványok létrehozásával kapcsolatos segítségért forduljon a hitelesítésszolgáltatóhoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-342">Consult a certificate authority for assistance in creating ECC certificates.</span></span>

#### <a name="prepare-the-web-app"></a><span data-ttu-id="e06a6-343">A webalkalmazás előkészítése</span><span class="sxs-lookup"><span data-stu-id="e06a6-343">Prepare the web app</span></span>

<span data-ttu-id="e06a6-344">Ha egyéni SSL-tanúsítványt szeretne kötni a webalkalmazáshoz, a [app Service tervnek](https://azure.microsoft.com/pricing/details/app-service/) **alapszintű**, **standard** vagy **prémium** szintűnek kell lennie.</span><span class="sxs-lookup"><span data-stu-id="e06a6-344">To bind a custom SSL certificate to the web app, the [App Service plan](https://azure.microsoft.com/pricing/details/app-service/) must be in the **Basic**, **Standard**, or **Premium** tier.</span></span>

#### <a name="sign-in-to-azure"></a><span data-ttu-id="e06a6-345">Bejelentkezés az Azure-ba</span><span class="sxs-lookup"><span data-stu-id="e06a6-345">Sign in to Azure</span></span>

1. <span data-ttu-id="e06a6-346">Nyissa meg a [Azure Portalt](https://portal.azure.com/) , és lépjen a webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-346">Open the [Azure portal](https://portal.azure.com/) and go to the web app.</span></span>

2. <span data-ttu-id="e06a6-347">A bal oldali menüben válassza a **app Services** lehetőséget, majd válassza ki a webalkalmazás nevét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-347">From the left menu, select **App Services**, and then select the web app name.</span></span>

![Webalkalmazás kiválasztása Azure Portal](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a><span data-ttu-id="e06a6-349">A tarifacsomag ellenőrzése</span><span class="sxs-lookup"><span data-stu-id="e06a6-349">Check the pricing tier</span></span>

1. <span data-ttu-id="e06a6-350">A webalkalmazás bal oldali navigációs sávján görgessen a **Beállítások** szakaszra, és válassza a vertikális **felskálázás (App Service terv)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-350">In the left-hand navigation of the web app page, scroll to the **Settings** section and select **Scale up (App Service plan)**.</span></span>

    ![Vertikális Felskálázási menü a web app-ban](media/solution-deployment-guide-geo-distributed/image34.png)

1. <span data-ttu-id="e06a6-352">Győződjön meg arról, hogy a webalkalmazás nem az **ingyenes** vagy a **közös** szinten van.</span><span class="sxs-lookup"><span data-stu-id="e06a6-352">Ensure the web app isn't in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="e06a6-353">A webalkalmazás jelenlegi szintje sötét kék mezőben van kiemelve.</span><span class="sxs-lookup"><span data-stu-id="e06a6-353">The web app's current tier is highlighted in a dark blue box.</span></span>

    ![A Web App díjszabási szintjeinek keresése](media/solution-deployment-guide-geo-distributed/image35.png)

<span data-ttu-id="e06a6-355">Az egyéni SSL nem támogatott az **ingyenes** vagy a **közös** szinten.</span><span class="sxs-lookup"><span data-stu-id="e06a6-355">Custom SSL isn't supported in the **Free** or **Shared** tier.</span></span> <span data-ttu-id="e06a6-356">A felskálázáshoz kövesse a következő szakaszban leírt lépéseket, vagy a **válassza ki a díjszabási szintet** lapot, és ugorjon az [SSL-tanúsítvány feltöltése és kötése](/azure/app-service/app-service-web-tutorial-custom-ssl)lehetőségre.</span><span class="sxs-lookup"><span data-stu-id="e06a6-356">To upscale, follow the steps in the next section or the **Choose your pricing tier** page and skip to [Upload and bind your SSL certificate](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

#### <a name="scale-up-your-app-service-plan"></a><span data-ttu-id="e06a6-357">Az App Service-csomag vertikális felskálázása</span><span class="sxs-lookup"><span data-stu-id="e06a6-357">Scale up your App Service plan</span></span>

1. <span data-ttu-id="e06a6-358">Válassza az **Alapszintű**, a **Standard** vagy a **Prémium** szintet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-358">Select one of the **Basic**, **Standard**, or **Premium** tiers.</span></span>

2. <span data-ttu-id="e06a6-359">Válassza a **Kiválasztás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-359">Select **Select**.</span></span>

![A webalkalmazás díjszabási szintjeinek kiválasztása](media/solution-deployment-guide-geo-distributed/image36.png)

<span data-ttu-id="e06a6-361">A skálázási művelet akkor fejeződik be, ha az értesítés megjelenik.</span><span class="sxs-lookup"><span data-stu-id="e06a6-361">The scale operation is complete when notification is displayed.</span></span>

![Vertikális felskálázási értesítés](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a><span data-ttu-id="e06a6-363">Az SSL-tanúsítvány kötése és a köztes tanúsítványok egyesítése</span><span class="sxs-lookup"><span data-stu-id="e06a6-363">Bind your SSL certificate and merge intermediate certificates</span></span>

<span data-ttu-id="e06a6-364">Több tanúsítvány egyesítése a láncban.</span><span class="sxs-lookup"><span data-stu-id="e06a6-364">Merge multiple certificates in the chain.</span></span>

1. <span data-ttu-id="e06a6-365">**Nyisson meg minden olyan tanúsítványt** , amelyet egy szövegszerkesztőben kapott.</span><span class="sxs-lookup"><span data-stu-id="e06a6-365">**Open each certificate** you received in a text editor.</span></span>

2. <span data-ttu-id="e06a6-366">Hozzon létre egy fájlt a *mergedcertificate. CRT* nevű egyesített tanúsítványhoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-366">Create a file for the merged certificate called *mergedcertificate.crt*.</span></span> <span data-ttu-id="e06a6-367">Egy szövegszerkesztőben másolja ebbe a fájlba az egyes tanúsítványok tartalmát.</span><span class="sxs-lookup"><span data-stu-id="e06a6-367">In a text editor, copy the content of each certificate into this file.</span></span> <span data-ttu-id="e06a6-368">A tanúsítványok sorrendjének egyeznie kell a tanúsítványláncban lévő sorrenddel, a saját tanúsítvánnyal kezdve és a főtanúsítvánnyal végződve.</span><span class="sxs-lookup"><span data-stu-id="e06a6-368">The order of your certificates should follow the order in the certificate chain, beginning with your certificate and ending with the root certificate.</span></span> <span data-ttu-id="e06a6-369">Az alábbi példához hasonlóan néz ki:</span><span class="sxs-lookup"><span data-stu-id="e06a6-369">It looks like the following example:</span></span>

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a><span data-ttu-id="e06a6-370">Tanúsítvány exportálása PFX-fájlba</span><span class="sxs-lookup"><span data-stu-id="e06a6-370">Export certificate to PFX</span></span>

<span data-ttu-id="e06a6-371">Exportálja az egyesített SSL-tanúsítványt a tanúsítvány által generált titkos kulccsal.</span><span class="sxs-lookup"><span data-stu-id="e06a6-371">Export the merged SSL certificate with the private key generated by the certificate.</span></span>

<span data-ttu-id="e06a6-372">A titkos kulcsfájl az OpenSSL-n keresztül jön létre.</span><span class="sxs-lookup"><span data-stu-id="e06a6-372">A private key file is created via OpenSSL.</span></span> <span data-ttu-id="e06a6-373">A tanúsítvány PFX fájlba való exportálásához futtassa a következő parancsot, és cserélje le a helyőrzőket `<private-key-file>` és a `<merged-certificate-file>` titkos kulcs elérési útját és az egyesített tanúsítványfájl:</span><span class="sxs-lookup"><span data-stu-id="e06a6-373">To export the certificate to PFX, run the following command and replace the placeholders `<private-key-file>` and `<merged-certificate-file>` with the private key path and the merged certificate file:</span></span>

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

<span data-ttu-id="e06a6-374">Ha a rendszer kéri, adjon meg egy exportálási jelszót az SSL-tanúsítvány feltöltéséhez App Service később.</span><span class="sxs-lookup"><span data-stu-id="e06a6-374">When prompted, define an export password for uploading your SSL certificate to App Service later.</span></span>

<span data-ttu-id="e06a6-375">Ha az IIS vagy **Certreq.exe** a tanúsítványkérelem előállítására szolgál, telepítse a tanúsítványt egy helyi gépre, majd [exportálja a tanúsítványt a pfx-be](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span><span class="sxs-lookup"><span data-stu-id="e06a6-375">When IIS or **Certreq.exe** are used to generate the certificate request, install the certificate to a local machine and then [export the certificate to PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).</span></span>

#### <a name="upload-the-ssl-certificate"></a><span data-ttu-id="e06a6-376">Az SSL-tanúsítvány feltöltése</span><span class="sxs-lookup"><span data-stu-id="e06a6-376">Upload the SSL certificate</span></span>

1. <span data-ttu-id="e06a6-377">Válassza az **SSL-beállítások** elemet a webalkalmazás bal oldali navigációs sávján.</span><span class="sxs-lookup"><span data-stu-id="e06a6-377">Select **SSL settings** in the left navigation of the web app.</span></span>

2. <span data-ttu-id="e06a6-378">Válassza a **tanúsítvány feltöltése** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-378">Select **Upload Certificate**.</span></span>

3. <span data-ttu-id="e06a6-379">A **pfx-tanúsítványfájl** területen válassza a pfx-fájl lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-379">In **PFX Certificate File**, select PFX file.</span></span>

4. <span data-ttu-id="e06a6-380">A **tanúsítvány jelszava** mezőben adja meg a pfx-fájl exportálásakor létrehozott jelszót.</span><span class="sxs-lookup"><span data-stu-id="e06a6-380">In **Certificate password**, type the password created when exporting the PFX file.</span></span>

5. <span data-ttu-id="e06a6-381">Válassza a **Feltöltés** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-381">Select **Upload**.</span></span>

    ![SSL-tanúsítvány feltöltése](media/solution-deployment-guide-geo-distributed/image38.png)

<span data-ttu-id="e06a6-383">Amikor App Service befejezi a tanúsítvány feltöltését, az SSL- **Beállítások** lapon jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="e06a6-383">When App Service finishes uploading the certificate, it appears in the **SSL settings** page.</span></span>

![SSL Settings (SSL-beállítások)](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a><span data-ttu-id="e06a6-385">Az SSL-tanúsítvány kötése</span><span class="sxs-lookup"><span data-stu-id="e06a6-385">Bind your SSL certificate</span></span>

1. <span data-ttu-id="e06a6-386">Az **SSL-kötések** szakaszban válassza a **kötés hozzáadása** elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-386">In the **SSL bindings** section, select **Add binding**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="e06a6-387">Ha a tanúsítvány fel lett töltve, de nem jelenik meg az **állomásnév** legördülő listájában, akkor próbálja meg frissíteni a böngésző oldalát.</span><span class="sxs-lookup"><span data-stu-id="e06a6-387">If the certificate has been uploaded, but doesn't appear in domain name(s) in the **Hostname** dropdown, try refreshing the browser page.</span></span>

2. <span data-ttu-id="e06a6-388">Az **SSL-kötés hozzáadása** lapon a legördülő listából válassza ki a védeni kívánt tartománynevet, és a használni kívánt tanúsítványt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-388">In the **Add SSL Binding** page, use the drop downs to select the domain name to secure and the certificate to use.</span></span>

3. <span data-ttu-id="e06a6-389">Az **SSL Type** (SSL típusa) területen válassza ki, hogy a [**kiszolgálónév jelzésén (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) alapuló vagy IP-alapú SSL-t kíván-e használni.</span><span class="sxs-lookup"><span data-stu-id="e06a6-389">In **SSL Type**, select whether to use [**Server Name Indication (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) or IP-based SSL.</span></span>

    - <span data-ttu-id="e06a6-390">**SNI-alapú SSL**: több SNI-alapú SSL-kötés is felvehető.</span><span class="sxs-lookup"><span data-stu-id="e06a6-390">**SNI-based SSL**: Multiple SNI-based SSL bindings may be added.</span></span> <span data-ttu-id="e06a6-391">Ez a beállítás lehetővé teszi, hogy több SSL-tanúsítvány biztosítson védelmet több tartomány számára ugyanazon az IP-címen.</span><span class="sxs-lookup"><span data-stu-id="e06a6-391">This option allows multiple SSL certificates to secure multiple domains on the same IP address.</span></span> <span data-ttu-id="e06a6-392">A legtöbb modern böngésző (beleértve az Internet Explorert, a Chrome-ot, a Firefox-ot és az Operát) támogatja az SNI-t (átfogóbb böngészőtámogatási információkat a [Kiszolgálónév jelzése](https://wikipedia.org/wiki/Server_Name_Indication) című szakaszban talál).</span><span class="sxs-lookup"><span data-stu-id="e06a6-392">Most modern browsers (including Internet Explorer, Chrome, Firefox, and Opera) support SNI (find more comprehensive browser support information at [Server Name Indication](https://wikipedia.org/wiki/Server_Name_Indication)).</span></span>

    - <span data-ttu-id="e06a6-393">**IP-alapú SSL**: a rendszer csak egy IP-alapú SSL-kötést adhat hozzá.</span><span class="sxs-lookup"><span data-stu-id="e06a6-393">**IP-based SSL**: Only one IP-based SSL binding may be added.</span></span> <span data-ttu-id="e06a6-394">Ez a beállítás csak egy SSL-tanúsítványnak engedélyezi egy dedikált nyilvános IP-cím védelmét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-394">This option allows only one SSL certificate to secure a dedicated public IP address.</span></span> <span data-ttu-id="e06a6-395">Több tartomány biztonságossá tételéhez gondoskodjon arról, hogy mindegyik ugyanazt az SSL-tanúsítványt használja.</span><span class="sxs-lookup"><span data-stu-id="e06a6-395">To secure multiple domains, secure them all using the same SSL certificate.</span></span> <span data-ttu-id="e06a6-396">Az IP-alapú SSL az SSL-kötés hagyományos beállítása.</span><span class="sxs-lookup"><span data-stu-id="e06a6-396">IP-based SSL is the traditional option for SSL binding.</span></span>

4. <span data-ttu-id="e06a6-397">Válassza a **kötés hozzáadása** elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-397">Select **Add Binding**.</span></span>

    ![SSL-kötés hozzáadása](media/solution-deployment-guide-geo-distributed/image40.png)

<span data-ttu-id="e06a6-399">Amikor App Service befejezi a tanúsítvány feltöltését, az az **SSL-kötések** szakaszában jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="e06a6-399">When App Service finishes uploading the certificate, it appears in the **SSL bindings** sections.</span></span>

![Az SSL-kötések feltöltése befejeződött](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a><span data-ttu-id="e06a6-401">Az A rekord újratársítása IP SSL</span><span class="sxs-lookup"><span data-stu-id="e06a6-401">Remap the A record for IP SSL</span></span>

<span data-ttu-id="e06a6-402">Ha az IP-alapú SSL nincs használatban a webalkalmazásban, ugorjon a [https tesztelése az egyéni tartományhoz](/azure/app-service/app-service-web-tutorial-custom-ssl)elemre.</span><span class="sxs-lookup"><span data-stu-id="e06a6-402">If IP-based SSL isn't used in the web app, skip to [Test HTTPS for your custom domain](/azure/app-service/app-service-web-tutorial-custom-ssl).</span></span>

<span data-ttu-id="e06a6-403">Alapértelmezés szerint a webalkalmazás megosztott nyilvános IP-címet használ.</span><span class="sxs-lookup"><span data-stu-id="e06a6-403">By default, the web app uses a shared public IP address.</span></span> <span data-ttu-id="e06a6-404">Ha a tanúsítvány IP-alapú SSL-sel van kötve, App Service létrehoz egy új és dedikált IP-címet a webalkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="e06a6-404">When the certificate is bound with IP-based SSL, App Service creates a new and dedicated IP address for the web app.</span></span>

<span data-ttu-id="e06a6-405">Ha egy rekord a webalkalmazásra van leképezve, a tartomány beállításjegyzékét frissíteni kell a dedikált IP-címmel.</span><span class="sxs-lookup"><span data-stu-id="e06a6-405">When an A record is mapped to the web app, the domain registry must be updated with the dedicated IP address.</span></span>

<span data-ttu-id="e06a6-406">Az **egyéni tartomány** lapot az új, dedikált IP-címmel frissíti a rendszer.</span><span class="sxs-lookup"><span data-stu-id="e06a6-406">The **Custom domain** page is updated with the new, dedicated IP address.</span></span> <span data-ttu-id="e06a6-407">Másolja ezt az [IP-címet](/azure/app-service/app-service-web-tutorial-custom-domain), majd a [rekordot](/azure/app-service/app-service-web-tutorial-custom-domain) az új IP-címhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-407">Copy this [IP address](/azure/app-service/app-service-web-tutorial-custom-domain), then remap the [A record](/azure/app-service/app-service-web-tutorial-custom-domain) to this new IP address.</span></span>

#### <a name="test-https"></a><span data-ttu-id="e06a6-408">HTTPS tesztelése</span><span class="sxs-lookup"><span data-stu-id="e06a6-408">Test HTTPS</span></span>

<span data-ttu-id="e06a6-409">Különböző böngészőkben `https://<your.custom.domain>` a webalkalmazás kiszolgálása érdekében nyissa meg a következőt:.</span><span class="sxs-lookup"><span data-stu-id="e06a6-409">In different browsers, go to `https://<your.custom.domain>` to ensure the web app is served.</span></span>

![webalkalmazás tallózása](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> <span data-ttu-id="e06a6-411">Ha a tanúsítvány-érvényesítési hibák merülnek fel, az önaláírt tanúsítvány lehet az OK, vagy előfordulhat, hogy a köztes tanúsítványok a PFX-fájlba való exportáláskor megmaradtak.</span><span class="sxs-lookup"><span data-stu-id="e06a6-411">If certificate validation errors occur, a self-signed certificate may be the cause, or intermediate certificates may have been left off when exporting to the PFX file.</span></span>

#### <a name="enforce-https"></a><span data-ttu-id="e06a6-412">HTTPS kényszerítése</span><span class="sxs-lookup"><span data-stu-id="e06a6-412">Enforce HTTPS</span></span>

<span data-ttu-id="e06a6-413">Alapértelmezés szerint bárki elérheti a webalkalmazást HTTP-n keresztül.</span><span class="sxs-lookup"><span data-stu-id="e06a6-413">By default, anyone can access the web app using HTTP.</span></span> <span data-ttu-id="e06a6-414">Lehet, hogy a HTTPS-portra irányuló összes HTTP-kérelem át lesz irányítva.</span><span class="sxs-lookup"><span data-stu-id="e06a6-414">All HTTP requests to the HTTPS port may be redirected.</span></span>

<span data-ttu-id="e06a6-415">A Web App (webalkalmazás) lapon válassza az **SL-beállítások** elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-415">In the web app page, select **SL settings**.</span></span> <span data-ttu-id="e06a6-416">Ezután a **HTTPS Only** (Csak HTTPS) területen válassza az **On** (Be) elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-416">Then, in **HTTPS Only**, select **On**.</span></span>

![HTTPS kényszerítése](media/solution-deployment-guide-geo-distributed/image43.png)

<span data-ttu-id="e06a6-418">Ha a művelet befejeződött, lépjen az alkalmazásra mutató HTTP URL-címek bármelyikére.</span><span class="sxs-lookup"><span data-stu-id="e06a6-418">When the operation is complete, go to any of the HTTP URLs that point to the app.</span></span> <span data-ttu-id="e06a6-419">Például:</span><span class="sxs-lookup"><span data-stu-id="e06a6-419">For example:</span></span>

- <span data-ttu-id="e06a6-420"> https://<app_name>. azurewebsites.net</span><span class="sxs-lookup"><span data-stu-id="e06a6-420">https://<app_name>.azurewebsites.net</span></span>
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a><span data-ttu-id="e06a6-421">A TLS 1.1/1.2 kényszerítése</span><span class="sxs-lookup"><span data-stu-id="e06a6-421">Enforce TLS 1.1/1.2</span></span>

<span data-ttu-id="e06a6-422">Az alkalmazás alapértelmezés szerint engedélyezi a [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0-et, amely már nem tekinthető biztonságosnak az iparági szabványok (például a [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)) számára.</span><span class="sxs-lookup"><span data-stu-id="e06a6-422">The app allows [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0 by default, which is no longer considered secure by industry standards (like [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard)).</span></span> <span data-ttu-id="e06a6-423">A TLS újabb verziójának kényszerítéséhez kövesse az alábbi lépéseket:</span><span class="sxs-lookup"><span data-stu-id="e06a6-423">To enforce higher TLS versions, follow these steps:</span></span>

1. <span data-ttu-id="e06a6-424">A webalkalmazás lap bal oldali navigációs sávján válassza az **SSL-beállítások** elemet.</span><span class="sxs-lookup"><span data-stu-id="e06a6-424">In the web app page, in the left navigation, select **SSL settings**.</span></span>

2. <span data-ttu-id="e06a6-425">A **TLS verziónál** válassza ki a TLS minimális verzióját.</span><span class="sxs-lookup"><span data-stu-id="e06a6-425">In **TLS version**, select the minimum TLS version.</span></span>

    ![A TLS 1.1 vagy 1.2 kényszerítése](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a><span data-ttu-id="e06a6-427">Traffic Manager-profil létrehozása</span><span class="sxs-lookup"><span data-stu-id="e06a6-427">Create a Traffic Manager profile</span></span>

1. <span data-ttu-id="e06a6-428">Válassza **az erőforrás létrehozása**  >  **hálózatkezelés**  >  **Traffic Manager profil**  >  **létrehozása** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-428">Select **Create a resource** > **Networking** > **Traffic Manager profile** > **Create**.</span></span>

2. <span data-ttu-id="e06a6-429">A **Traffic Manager-profil létrehozása** területen adja meg a következőket:</span><span class="sxs-lookup"><span data-stu-id="e06a6-429">In the **Create Traffic Manager profile**, complete as follows:</span></span>

    1. <span data-ttu-id="e06a6-430">A **név mezőben** adja meg a profil nevét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-430">In **Name**, provide a name for the profile.</span></span> <span data-ttu-id="e06a6-431">Ennek a névnek egyedinek kell lennie a forgalmi manager.net zónán belül, és a trafficmanager.net DNS-nevet kell használnia, amely a Traffic Manager profil elérésére szolgál.</span><span class="sxs-lookup"><span data-stu-id="e06a6-431">This name needs to be unique within the traffic manager.net zone and results in the DNS name, trafficmanager.net, which is used to access the Traffic Manager profile.</span></span>

    2. <span data-ttu-id="e06a6-432">Az **útválasztási módszer** területen válassza ki a **földrajzi útválasztási módszert**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-432">In **Routing method**, select the **Geographic routing method**.</span></span>

    3. <span data-ttu-id="e06a6-433">Az **előfizetés** területen válassza ki azt az előfizetést, amelyben létre szeretné hozni a profilt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-433">In **Subscription**, select the subscription under which to create this profile.</span></span>

    4. <span data-ttu-id="e06a6-434">Az **Erőforráscsoport** mezőben hozzon létre egy új erőforráscsoportot, amely alá ezt a profilt helyezi.</span><span class="sxs-lookup"><span data-stu-id="e06a6-434">In **Resource Group**, create a new resource group to place this profile under.</span></span>

    5. <span data-ttu-id="e06a6-435">Az **Erőforráscsoport helye** területen válassza ki az erőforráscsoport helyét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-435">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="e06a6-436">Ez a beállítás az erőforráscsoport helyére vonatkozik, és nincs hatással a globálisan telepített Traffic Manager-profilra.</span><span class="sxs-lookup"><span data-stu-id="e06a6-436">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile deployed globally.</span></span>

    6. <span data-ttu-id="e06a6-437">Válassza a **Létrehozás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-437">Select **Create**.</span></span>

    7. <span data-ttu-id="e06a6-438">Ha a Traffic Manager-profil globális telepítése befejeződött, az a megfelelő erőforráscsoporthoz kerül, mint az egyik erőforrás.</span><span class="sxs-lookup"><span data-stu-id="e06a6-438">When the global deployment of the Traffic Manager profile is complete, it's listed in the respective resource group as one of the resources.</span></span>

        ![Erőforráscsoportok a Create Traffic Manager Profile](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="e06a6-440">Traffic Manager-végpontok hozzáadása</span><span class="sxs-lookup"><span data-stu-id="e06a6-440">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="e06a6-441">A portálon keresse meg az előző szakaszban létrehozott **Traffic Manager profil** nevét, és válassza ki a Traffic Manager-profilt a megjelenített eredmények között.</span><span class="sxs-lookup"><span data-stu-id="e06a6-441">In the portal search bar, search for the **Traffic Manager profile** name created in the preceding section and select the traffic manager profile in the displayed results.</span></span>

2. <span data-ttu-id="e06a6-442">**Traffic Manager profilban** a **Beállítások** szakaszban válassza a **végpontok** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-442">In **Traffic Manager profile**, in the **Settings** section, select **Endpoints**.</span></span>

3. <span data-ttu-id="e06a6-443">Válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-443">Select **Add**.</span></span>

4. <span data-ttu-id="e06a6-444">Az Azure Stack hub-végpont hozzáadása.</span><span class="sxs-lookup"><span data-stu-id="e06a6-444">Adding the Azure Stack Hub Endpoint.</span></span>

5. <span data-ttu-id="e06a6-445">A **Típus mezőben** válassza a **külső végpont** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-445">For **Type**, select **External endpoint**.</span></span>

6. <span data-ttu-id="e06a6-446">Adja meg a végpont **nevét** , ideális esetben az Azure stack hub nevét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-446">Provide a **Name** for this endpoint, ideally the name of the Azure Stack Hub.</span></span>

7. <span data-ttu-id="e06a6-447">Teljes tartománynév (**FQDN**) esetén használja az Azure stack hub webalkalmazás külső URL-címét.</span><span class="sxs-lookup"><span data-stu-id="e06a6-447">For fully qualified domain name (**FQDN**), use the external URL for the Azure Stack Hub Web App.</span></span>

8. <span data-ttu-id="e06a6-448">A földrajzi leképezés területen válassza ki azt a régiót/kontinenst, ahol az erőforrás található.</span><span class="sxs-lookup"><span data-stu-id="e06a6-448">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="e06a6-449">Például: **Európa.**</span><span class="sxs-lookup"><span data-stu-id="e06a6-449">For example, **Europe.**</span></span>

9. <span data-ttu-id="e06a6-450">A megjelenő ország/régió legördülő listából válassza ki azt az országot, amely erre a végpontra vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="e06a6-450">Under the Country/Region drop-down that appears, select the country that applies to this endpoint.</span></span> <span data-ttu-id="e06a6-451">Például: **Németország**.</span><span class="sxs-lookup"><span data-stu-id="e06a6-451">For example, **Germany**.</span></span>

10. <span data-ttu-id="e06a6-452">A **Beállítás letiltottként** jelölőnégyzetet ne jelölje ki.</span><span class="sxs-lookup"><span data-stu-id="e06a6-452">Keep **Add as disabled** unchecked.</span></span>

11. <span data-ttu-id="e06a6-453">Válassza az **OK** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-453">Select **OK**.</span></span>

12. <span data-ttu-id="e06a6-454">Az Azure-végpont hozzáadása:</span><span class="sxs-lookup"><span data-stu-id="e06a6-454">Adding the Azure Endpoint:</span></span>

    1. <span data-ttu-id="e06a6-455">A **Típus mezőben** válassza az **Azure-végpont** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-455">For **Type**, select **Azure endpoint**.</span></span>

    2. <span data-ttu-id="e06a6-456">Adja meg a végpont **nevét** .</span><span class="sxs-lookup"><span data-stu-id="e06a6-456">Provide a **Name** for the endpoint.</span></span>

    3. <span data-ttu-id="e06a6-457">A **cél erőforrástípus mezőben** válassza a **app Service** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-457">For **Target resource type**, select **App Service**.</span></span>

    4. <span data-ttu-id="e06a6-458">A **cél erőforrásnál** válassza az **app Service kiválasztása** lehetőséget az azonos előfizetéshez tartozó Web Apps listájának megjelenítéséhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-458">For **Target resource**, select **Choose an app service** to show the listing of the Web Apps under the same subscription.</span></span> <span data-ttu-id="e06a6-459">Az **erőforrás** területen válassza ki az első végpontként használt app Service-t.</span><span class="sxs-lookup"><span data-stu-id="e06a6-459">In **Resource**, pick the App service used as the first endpoint.</span></span>

13. <span data-ttu-id="e06a6-460">A földrajzi leképezés területen válassza ki azt a régiót/kontinenst, ahol az erőforrás található.</span><span class="sxs-lookup"><span data-stu-id="e06a6-460">Under Geo-mapping, select a region/continent where the resource is located.</span></span> <span data-ttu-id="e06a6-461">Például: **Észak-Amerika/Közép-Amerika/Karib-térség.**</span><span class="sxs-lookup"><span data-stu-id="e06a6-461">For example, **North America/Central America/Caribbean.**</span></span>

14. <span data-ttu-id="e06a6-462">A megjelenő ország/régió legördülő menüben hagyja üresen ezt a helyet, hogy kiválassza az összes fenti regionális csoportosítást.</span><span class="sxs-lookup"><span data-stu-id="e06a6-462">Under the Country/Region drop-down that appears, leave this spot blank to select all of the above regional grouping.</span></span>

15. <span data-ttu-id="e06a6-463">A **Beállítás letiltottként** jelölőnégyzetet ne jelölje ki.</span><span class="sxs-lookup"><span data-stu-id="e06a6-463">Keep **Add as disabled** unchecked.</span></span>

16. <span data-ttu-id="e06a6-464">Válassza az **OK** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="e06a6-464">Select **OK**.</span></span>

    > [!Note]  
    >  <span data-ttu-id="e06a6-465">Hozzon létre legalább egy olyan végpontot, amelynek földrajzi hatóköre az összes (világ), hogy az erőforrás alapértelmezett végpontja legyen.</span><span class="sxs-lookup"><span data-stu-id="e06a6-465">Create at least one endpoint with a geographic scope of All (World) to serve as the default endpoint for the resource.</span></span>

17. <span data-ttu-id="e06a6-466">Mindkét végpont hozzáadásakor a rendszer a **Traffic Manager profilban** jeleníti meg a figyelési állapotukat **online** állapottal együtt.</span><span class="sxs-lookup"><span data-stu-id="e06a6-466">When the addition of both endpoints is complete, they're displayed in **Traffic Manager profile** along with their monitoring status as **Online**.</span></span>

    ![Traffic Manager profil végpontjának állapota](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a><span data-ttu-id="e06a6-468">A globális vállalat az Azure geo-eloszlási képességeire támaszkodik</span><span class="sxs-lookup"><span data-stu-id="e06a6-468">Global Enterprise relies on Azure geo-distribution capabilities</span></span>

<span data-ttu-id="e06a6-469">Az adatforgalom Azure-Traffic Manager és földrajzilag specifikus végpontokon keresztüli átirányítása lehetővé teszi a globális vállalatok számára a regionális szabályozások betartását és az adatok megfelelő és biztonságos megőrzését, ami elengedhetetlen a helyi és a távoli üzleti telephelyek sikerességéhez.</span><span class="sxs-lookup"><span data-stu-id="e06a6-469">Directing data traffic via Azure Traffic Manager and geography-specific endpoints enables global enterprises to adhere to regional regulations and keep data compliant and secure, which is crucial to the success of local and remote business locations.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e06a6-470">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="e06a6-470">Next steps</span></span>

- <span data-ttu-id="e06a6-471">Az Azure Cloud Patterns szolgáltatással kapcsolatos további információkért lásd: [Felhőbeli tervezési minták](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="e06a6-471">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
