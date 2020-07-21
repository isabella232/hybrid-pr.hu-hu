---
title: Hibrid alkalmazás üzembe helyezése a felhőn átívelő helyszíni adatszolgáltatásokkal
description: Megtudhatja, hogyan helyezhet üzembe olyan alkalmazást, amely helyszíni információkat használ, és az Azure-ban és Azure Stack hub-on keresztül méretezi a felhőket.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6de35cb55c4c35a2a9927f9ffc2516ccb00cd89f
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477320"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="fef1f-103">Hibrid alkalmazás üzembe helyezése a felhőn átívelő helyszíni adatszolgáltatásokkal</span><span class="sxs-lookup"><span data-stu-id="fef1f-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="fef1f-104">Ez a megoldási útmutató bemutatja, hogyan helyezhet üzembe egy olyan hibrid alkalmazást, amely az Azure-t és Azure Stack hub-t is felöleli, és egyetlen helyszíni adatforrást használ.</span><span class="sxs-lookup"><span data-stu-id="fef1f-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="fef1f-105">A hibrid felhőalapú megoldásokkal kombinálhatja a privát felhő megfelelőségi előnyeit a nyilvános felhő méretezhetőségével.</span><span class="sxs-lookup"><span data-stu-id="fef1f-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="fef1f-106">A fejlesztők kihasználhatják a Microsoft fejlesztői ökoszisztémáját is, és alkalmazhatják tudásukat a felhőben és a helyszíni környezetekben.</span><span class="sxs-lookup"><span data-stu-id="fef1f-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="fef1f-107">Áttekintés és feltételezések</span><span class="sxs-lookup"><span data-stu-id="fef1f-107">Overview and assumptions</span></span>

<span data-ttu-id="fef1f-108">Ezt az oktatóanyagot követve beállíthat egy olyan munkafolyamatot, amely lehetővé teszi, hogy a fejlesztők egy azonos webalkalmazást telepítsenek egy nyilvános felhőbe és egy privát felhőbe.</span><span class="sxs-lookup"><span data-stu-id="fef1f-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="fef1f-109">Ez az alkalmazás elérheti a privát felhőben üzemeltetett, nem internetes útválasztású hálózatot.</span><span class="sxs-lookup"><span data-stu-id="fef1f-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="fef1f-110">Ezeket a webalkalmazásokat a rendszer figyeli, és ha van egy tüske a forgalomban, a program módosítja a DNS-rekordokat, hogy átirányítsa a forgalmat a nyilvános felhőbe.</span><span class="sxs-lookup"><span data-stu-id="fef1f-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="fef1f-111">Ha a forgalom a csúcs előtti szintre esik, a forgalmat a rendszer visszairányítja a privát felhőbe.</span><span class="sxs-lookup"><span data-stu-id="fef1f-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="fef1f-112">Ez az oktatóanyag a következő feladatokat mutatja be:</span><span class="sxs-lookup"><span data-stu-id="fef1f-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fef1f-113">Hibrid csatlakozású SQL Server adatbázis-kiszolgáló üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="fef1f-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="fef1f-114">Egy webalkalmazás összekötése a globális Azure-ban egy hibrid hálózattal.</span><span class="sxs-lookup"><span data-stu-id="fef1f-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="fef1f-115">Konfigurálja a DNS-t a Felhőbeli skálázáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fef1f-116">Konfigurálja az SSL-tanúsítványokat a Felhőbeli skálázáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fef1f-117">Konfigurálja és telepítse a webalkalmazást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="fef1f-118">Hozzon létre egy Traffic Manager profilt, és konfigurálja a Felhőbeli skálázáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="fef1f-119">Application Insights monitorozás és riasztások beállítása a megnövekedett forgalomhoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="fef1f-120">Konfigurálja az automatikus forgalmat a globális Azure és az Azure Stack hub között.</span><span class="sxs-lookup"><span data-stu-id="fef1f-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="fef1f-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fef1f-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fef1f-122">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="fef1f-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fef1f-123">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="fef1f-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fef1f-124">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fef1f-125">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="fef1f-126">Feltételezések</span><span class="sxs-lookup"><span data-stu-id="fef1f-126">Assumptions</span></span>

<span data-ttu-id="fef1f-127">Ez az oktatóanyag feltételezi, hogy rendelkezik a globális Azure és Azure Stack hub alapszintű ismeretével.</span><span class="sxs-lookup"><span data-stu-id="fef1f-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fef1f-128">Ha többet szeretne megtudni az oktatóanyag megkezdése előtt, tekintse át a következő cikkeket:</span><span class="sxs-lookup"><span data-stu-id="fef1f-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="fef1f-129">Bevezetés az Azure használatába</span><span class="sxs-lookup"><span data-stu-id="fef1f-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="fef1f-130">Azure Stack hub főbb fogalmak</span><span class="sxs-lookup"><span data-stu-id="fef1f-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="fef1f-131">Ez az oktatóanyag azt is feltételezi, hogy rendelkezik Azure-előfizetéssel.</span><span class="sxs-lookup"><span data-stu-id="fef1f-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="fef1f-132">Ha nem rendelkezik előfizetéssel, [hozzon létre egy ingyenes fiókot a](https://azure.microsoft.com/free/) Kezdés előtt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="fef1f-133">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="fef1f-133">Prerequisites</span></span>

<span data-ttu-id="fef1f-134">A megoldás elindítása előtt győződjön meg arról, hogy megfelel a következő követelményeknek:</span><span class="sxs-lookup"><span data-stu-id="fef1f-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="fef1f-135">Egy Azure Stack Development Kit (ASDK) vagy előfizetés egy Azure Stack hub integrált rendszeren.</span><span class="sxs-lookup"><span data-stu-id="fef1f-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="fef1f-136">A ASDK üzembe helyezéséhez kövesse a [ASDK telepítése a telepítő használatával](/azure-stack/asdk/asdk-install.md)című témakör utasításait.</span><span class="sxs-lookup"><span data-stu-id="fef1f-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="fef1f-137">A Azure Stack hub telepítése a következőkkel telepíthető:</span><span class="sxs-lookup"><span data-stu-id="fef1f-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="fef1f-138">A Azure App Service.</span><span class="sxs-lookup"><span data-stu-id="fef1f-138">The Azure App Service.</span></span> <span data-ttu-id="fef1f-139">Az Azure Stack hub operátorral együttműködve telepítheti és konfigurálhatja a Azure App Servicet a környezetében.</span><span class="sxs-lookup"><span data-stu-id="fef1f-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="fef1f-140">Ez az oktatóanyag megköveteli, hogy a App Service legalább egy (1) elérhető dedikált feldolgozói szerepkörrel rendelkezzen.</span><span class="sxs-lookup"><span data-stu-id="fef1f-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="fef1f-141">Egy Windows Server 2016-rendszerkép.</span><span class="sxs-lookup"><span data-stu-id="fef1f-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="fef1f-142">Egy Microsoft SQL Server rendszerképpel rendelkező Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="fef1f-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="fef1f-143">A megfelelő csomagok és ajánlatok.</span><span class="sxs-lookup"><span data-stu-id="fef1f-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="fef1f-144">A webalkalmazáshoz tartozó tartománynév.</span><span class="sxs-lookup"><span data-stu-id="fef1f-144">A domain name for your web app.</span></span> <span data-ttu-id="fef1f-145">Ha nem rendelkezik tartománynévvel, vásárolhat egyet egy olyan tartományi szolgáltatótól, mint a GoDaddy, a Bluehost és a InMotion.</span><span class="sxs-lookup"><span data-stu-id="fef1f-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="fef1f-146">Egy megbízható hitelesítésszolgáltatótól (például LetsEncrypt) származó SSL-tanúsítvány a tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="fef1f-147">Egy SQL Server-adatbázissal kommunikáló webalkalmazás, amely támogatja a Application Insights.</span><span class="sxs-lookup"><span data-stu-id="fef1f-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="fef1f-148">A [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) minta alkalmazást letöltheti a githubról.</span><span class="sxs-lookup"><span data-stu-id="fef1f-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="fef1f-149">Hibrid hálózat egy Azure-beli virtuális hálózat és Azure Stack hub virtuális hálózat között.</span><span class="sxs-lookup"><span data-stu-id="fef1f-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="fef1f-150">Részletes útmutatás: [hibrid felhőalapú kapcsolat konfigurálása az Azure-ban és Azure stack hub](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="fef1f-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="fef1f-151">Hibrid folyamatos integrációs/folyamatos üzembe helyezési (CI/CD) folyamat egy Azure Stack hub-beli privát Build-ügynökkel.</span><span class="sxs-lookup"><span data-stu-id="fef1f-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="fef1f-152">Részletes útmutatás: [hibrid Felhőbeli identitás konfigurálása az Azure-ban és Azure stack hub-alkalmazások](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="fef1f-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="fef1f-153">Hibrid kapcsolattal rendelkező SQL Server adatbázis-kiszolgáló üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="fef1f-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="fef1f-154">Jelentkezzen be az Azure Stack hub felhasználói portálra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="fef1f-155">Az **irányítópulton**válassza a **piactér**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Azure Stack hub piactér](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="fef1f-157">A **piactéren**válassza a **számítás**lehetőséget, majd válassza a **továbbiak**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="fef1f-158">A **további**területen válassza ki az **ingyenes SQL Server licencet: SQL Server 2017 Developer Windows Server** rendszerképre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Virtuálisgép-rendszerkép kiválasztása Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="fef1f-160">**Ingyenes SQL Server licenc: SQL Server 2017 Developer on Windows Server**, válassza a **Létrehozás**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="fef1f-161">Az **alapvető beállítások konfigurálása > alapszintű beállításnál**adja meg a virtuális gép (VM) **nevét** , a SQL Server SA **felhasználónevét** és a biztonsági társításhoz tartozó **jelszót** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="fef1f-162">Az **előfizetés** legördülő listából válassza ki azt az előfizetést, amelyre telepíteni kívánja.</span><span class="sxs-lookup"><span data-stu-id="fef1f-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="fef1f-163">Az **erőforráscsoport**esetében használja a **meglévő kiválasztása lehetőséget** , és helyezze a virtuális gépet ugyanabba az erőforráscsoporthoz, mint a Azure stack hub webalkalmazás.</span><span class="sxs-lookup"><span data-stu-id="fef1f-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![A virtuális gép alapszintű beállításainak konfigurálása Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="fef1f-165">A **méret**területen válasszon egy méretet a virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="fef1f-166">Ebben az oktatóanyagban A2_Standard vagy DS2_V2_Standard használatát javasoljuk.</span><span class="sxs-lookup"><span data-stu-id="fef1f-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="fef1f-167">A **beállítások > a választható funkciók konfigurálása**területen adja meg a következő beállításokat:</span><span class="sxs-lookup"><span data-stu-id="fef1f-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="fef1f-168">**Storage-fiók**: hozzon létre egy új fiókot, ha szüksége van rá.</span><span class="sxs-lookup"><span data-stu-id="fef1f-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="fef1f-169">**Virtuális hálózat**:</span><span class="sxs-lookup"><span data-stu-id="fef1f-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="fef1f-170">Győződjön meg arról, hogy a SQL Server VM ugyanazon a virtuális hálózaton van telepítve, mint a VPN-átjárók.</span><span class="sxs-lookup"><span data-stu-id="fef1f-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="fef1f-171">**Nyilvános IP-cím**: használja az alapértelmezett beállításokat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="fef1f-172">**Hálózati biztonsági csoport**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="fef1f-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="fef1f-173">Hozzon létre egy új NSG.</span><span class="sxs-lookup"><span data-stu-id="fef1f-173">Create a new NSG.</span></span>
   - <span data-ttu-id="fef1f-174">**Bővítmények és figyelés**: tartsa meg az alapértelmezett beállításokat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="fef1f-175">**Diagnosztikai Storage-fiók**: hozzon létre egy új fiókot, ha szüksége van rá.</span><span class="sxs-lookup"><span data-stu-id="fef1f-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="fef1f-176">A konfiguráció mentéséhez kattintson **az OK gombra** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-176">Select **OK** to save your configuration.</span></span>

     ![Opcionális virtuálisgép-funkciók konfigurálása Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="fef1f-178">A **SQL Server beállítások**területen adja meg a következő beállításokat:</span><span class="sxs-lookup"><span data-stu-id="fef1f-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="fef1f-179">**SQL-kapcsolat**esetén válassza a **nyilvános (Internet)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="fef1f-180">A **port**esetében tartsa meg az alapértelmezett **1433**-as értéket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="fef1f-181">**SQL-hitelesítés**esetén válassza az **Engedélyezés**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="fef1f-182">Az SQL-hitelesítés engedélyezésekor a rendszer automatikusan feltölti az **alapokban**konfigurált "SQLAdmin" információkkal.</span><span class="sxs-lookup"><span data-stu-id="fef1f-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="fef1f-183">A többi beállításnál tartsa meg az alapértelmezett értékeket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="fef1f-184">Kattintson az **OK** gombra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-184">Select **OK**.</span></span>

     ![SQL Server beállítások konfigurálása Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="fef1f-186">Az **Összefoglalás**lapon tekintse át a virtuális gép konfigurációját, majd a telepítés elindításához kattintson **az OK gombra** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Konfiguráció összegzése Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="fef1f-188">Némi időbe telik az új virtuális gép létrehozása.</span><span class="sxs-lookup"><span data-stu-id="fef1f-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="fef1f-189">A **virtuális gépek állapota a Virtual Machines**szolgáltatásban tekinthető meg.</span><span class="sxs-lookup"><span data-stu-id="fef1f-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Virtuális gépek állapota Azure Stack hub felhasználói portálon](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="fef1f-191">Webalkalmazások létrehozása az Azure-ban és Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="fef1f-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-192">A Azure App Service leegyszerűsíti a webalkalmazások futtatását és felügyeletét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="fef1f-193">Mivel Azure Stack hub konzisztens az Azure-ban, a App Service mindkét környezetben futhat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="fef1f-194">Az alkalmazás futtatásához a App Service fogja használni.</span><span class="sxs-lookup"><span data-stu-id="fef1f-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="fef1f-195">Webalkalmazások létrehozása</span><span class="sxs-lookup"><span data-stu-id="fef1f-195">Create web apps</span></span>

1. <span data-ttu-id="fef1f-196">Hozzon létre egy webalkalmazást az Azure-ban a [app Service-csomag kezelése az Azure-ban](/azure/app-service/app-service-plan-manage#create-an-app-service-plan)című témakör útmutatásait követve.</span><span class="sxs-lookup"><span data-stu-id="fef1f-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="fef1f-197">Győződjön meg arról, hogy a webalkalmazást a hibrid hálózattal megegyező előfizetésben és erőforráscsoporthoz helyezi el.</span><span class="sxs-lookup"><span data-stu-id="fef1f-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="fef1f-198">Ismételje meg az előző lépést (1) az Azure Stack központban.</span><span class="sxs-lookup"><span data-stu-id="fef1f-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="fef1f-199">Útvonal hozzáadása Azure Stack hubhoz</span><span class="sxs-lookup"><span data-stu-id="fef1f-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-200">Az Azure Stack hub App Service a nyilvános internetről irányíthatónak kell lennie ahhoz, hogy a felhasználók hozzáférhessenek az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="fef1f-201">Ha az Azure Stack hub elérhető az internetről, jegyezze fel az Azure Stack hub-webalkalmazás nyilvános IP-címét vagy URL-címét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="fef1f-202">Ha ASDK használ, beállíthatja [a statikus NAT-leképezést](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) úgy, hogy a virtuális környezeten kívül is elérhetővé tegye a app Service.</span><span class="sxs-lookup"><span data-stu-id="fef1f-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="fef1f-203">Webalkalmazás összekötése az Azure-ban hibrid hálózattal</span><span class="sxs-lookup"><span data-stu-id="fef1f-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="fef1f-204">Az Azure webes kezelőfelülete és a Azure Stack hub SQL Server adatbázisa közötti kapcsolat biztosításához a webalkalmazásnak csatlakoznia kell a hibrid hálózathoz az Azure és a Azure Stack hub között.</span><span class="sxs-lookup"><span data-stu-id="fef1f-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fef1f-205">A kapcsolat engedélyezéséhez a következőkre lesz szüksége:</span><span class="sxs-lookup"><span data-stu-id="fef1f-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="fef1f-206">Pont – hely kapcsolat konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="fef1f-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="fef1f-207">Konfigurálja a webalkalmazást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-207">Configure the web app.</span></span>
- <span data-ttu-id="fef1f-208">Módosítsa Azure Stack hub helyi hálózati átjáróját.</span><span class="sxs-lookup"><span data-stu-id="fef1f-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="fef1f-209">Az Azure-beli virtuális hálózat konfigurálása pont – hely kapcsolathoz</span><span class="sxs-lookup"><span data-stu-id="fef1f-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="fef1f-210">A hibrid hálózat Azure oldalán lévő virtuális hálózati átjárónak lehetővé kell tennie a pont – hely kapcsolatoknak a Azure App Service való integrálását.</span><span class="sxs-lookup"><span data-stu-id="fef1f-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="fef1f-211">Az Azure-ban nyissa meg a virtuális hálózati átjáró lapot.</span><span class="sxs-lookup"><span data-stu-id="fef1f-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="fef1f-212">A **Beállítások**területen válassza a **pont – hely konfiguráció**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Pont – hely beállítás az Azure Virtual Network gatewayben](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="fef1f-214">Válassza a **Konfigurálás most** lehetőséget a pont – hely konfigurálásához.</span><span class="sxs-lookup"><span data-stu-id="fef1f-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Pont – hely konfiguráció indítása az Azure Virtual Network gatewayben](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="fef1f-216">A **pont – hely** konfiguráció lapon adja meg **a címkészlet számára**használni kívánt magánhálózati IP-címtartományt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="fef1f-217">Ügyeljen arra, hogy a megadott tartomány ne legyen átfedésben a hibrid hálózat globális Azure-ban vagy Azure Stack hub-összetevőjében már használt alhálózatok egyikével sem.</span><span class="sxs-lookup"><span data-stu-id="fef1f-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="fef1f-218">Az **alagút típusa**területen törölje a **IKEv2 VPN-** t.</span><span class="sxs-lookup"><span data-stu-id="fef1f-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="fef1f-219">Válassza a **Mentés** lehetőséget a pont – hely konfigurálásának befejezéséhez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Pont – hely beállítások az Azure Virtual Network-átjárón](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="fef1f-221">A Azure App Service alkalmazás integrálása a hibrid hálózattal</span><span class="sxs-lookup"><span data-stu-id="fef1f-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="fef1f-222">Az alkalmazás Azure VNet való összekapcsolásához kövesse az [átjáró szükséges VNet-integrációjának](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration)utasításait.</span><span class="sxs-lookup"><span data-stu-id="fef1f-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="fef1f-223">Lépjen a webalkalmazást üzemeltető App Service-csomag **beállításaihoz** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="fef1f-224">A **Beállítások**területen válassza a **hálózatkezelés**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-224">In **Settings**, select **Networking**.</span></span>

    ![A App Service-csomag hálózatkezelésének konfigurálása](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="fef1f-226">A **VNET-integráció**területen válassza a **kattintson ide a kezeléséhez**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![A App Service-csomag VNET-integrációjának kezelése](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="fef1f-228">Válassza ki a konfigurálni kívánt VNET.</span><span class="sxs-lookup"><span data-stu-id="fef1f-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="fef1f-229">A **VNET felé irányított IP-címek**területen adja meg az Azure VNET, az Azure stack hub VNET és a pont – hely címtartomány IP-címtartományt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="fef1f-230">A beállítások ellenőrzéséhez és mentéséhez kattintson a **Mentés** gombra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-230">Select **Save** to validate and save these settings.</span></span>

    ![Az Virtual Network-integrációban útválasztásra kerülő IP-címtartományok](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="fef1f-232">Ha többet szeretne megtudni arról, hogy a App Service hogyan integrálódik az Azure virtuális hálózatok, tekintse meg az [alkalmazás integrálása Azure](/azure/app-service/web-sites-integrate-with-vnet)-beli Virtual Network című témakört.</span><span class="sxs-lookup"><span data-stu-id="fef1f-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="fef1f-233">Az Azure Stack hub virtuális hálózat konfigurálása</span><span class="sxs-lookup"><span data-stu-id="fef1f-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="fef1f-234">Az Azure Stack hub virtuális hálózatában lévő helyi hálózati átjárót úgy kell konfigurálni, hogy a forgalmat a App Service pont – hely címtartomány alapján irányítsa.</span><span class="sxs-lookup"><span data-stu-id="fef1f-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="fef1f-235">Az Azure Stack központban lépjen a **helyi hálózati átjáró**elemre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="fef1f-236">A **Beállítások** területen válassza a **Konfiguráció** elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-236">Under **Settings**, select **Configuration**.</span></span>

    ![Átjáró konfigurációs beállítása Azure Stack hub helyi hálózati átjárójában](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="fef1f-238">A **címterület**mezőben adja meg az Azure-beli virtuális hálózati átjáró pont – hely típusú címtartományt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Pont – hely címtartomány a Azure Stack hub helyi hálózati átjárójában](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="fef1f-240">A konfiguráció ellenőrzéséhez és mentéséhez kattintson a **Mentés** gombra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="fef1f-241">A DNS konfigurálása a Felhőbeli skálázáshoz</span><span class="sxs-lookup"><span data-stu-id="fef1f-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="fef1f-242">A DNS Felhőbeli alkalmazások számára történő megfelelő konfigurálásával a felhasználók hozzáférhetnek a webalkalmazás globális Azure-és Azure Stack hub-példányaihoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="fef1f-243">Az oktatóanyag DNS-konfigurációja azt is lehetővé teszi, hogy az Azure Traffic Manager irányítsa a forgalmat, amikor a terhelés növekszik vagy csökken.</span><span class="sxs-lookup"><span data-stu-id="fef1f-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="fef1f-244">Ez az oktatóanyag a Azure DNS használatával kezeli a DNS-t, mert App Service tartományok nem működnek.</span><span class="sxs-lookup"><span data-stu-id="fef1f-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="fef1f-245">Altartományok létrehozása</span><span class="sxs-lookup"><span data-stu-id="fef1f-245">Create subdomains</span></span>

<span data-ttu-id="fef1f-246">Mivel Traffic Manager DNS-CNAME-re támaszkodik, egy altartományra van szükség ahhoz, hogy megfelelően irányítsa a forgalmat a végpontokra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="fef1f-247">A DNS-rekordokkal és a tartomány-hozzárendeléssel kapcsolatos további információkért lásd: [tartományok leképezése Traffic Managerokkal](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="fef1f-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="fef1f-248">Az Azure-végponthoz létre kell hoznia egy altartományt, amelyet a felhasználók használhatnak a webalkalmazás eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="fef1f-249">Ebben az oktatóanyagban használhatja a **app.Northwind.com**-t, de a saját tartománya alapján kell testreszabnia ezt az értéket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="fef1f-250">Létre kell hoznia egy altartományt is az Azure Stack hub-végponthoz tartozó rekorddal.</span><span class="sxs-lookup"><span data-stu-id="fef1f-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="fef1f-251">Használhatja a **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="fef1f-252">Egyéni tartomány konfigurálása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fef1f-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="fef1f-253">Adja hozzá a **app.Northwind.com** hostname-t az Azure-webalkalmazáshoz [egy CNAME hozzárendelésével Azure app Servicehoz](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="fef1f-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="fef1f-254">Egyéni tartományok konfigurálása Azure Stack központban</span><span class="sxs-lookup"><span data-stu-id="fef1f-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="fef1f-255">Adja hozzá a **azurestack.Northwind.com** hostname-t az Azure stack hub webalkalmazáshoz [egy olyan rekord hozzárendelésével, Azure app Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="fef1f-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="fef1f-256">Használja az App Service alkalmazás internetre irányítható IP-címét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="fef1f-257">Adja hozzá a **app.Northwind.com** hostname-t az Azure stack hub webalkalmazáshoz [egy CNAME-fájl Azure app Servicehoz való leképezésével](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="fef1f-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="fef1f-258">Használja az előző lépésben (1) konfigurált állomásnevet a CNAME célhelyként.</span><span class="sxs-lookup"><span data-stu-id="fef1f-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="fef1f-259">Az SSL-tanúsítványok konfigurálása a Felhőbeli skálázáshoz</span><span class="sxs-lookup"><span data-stu-id="fef1f-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="fef1f-260">Fontos, hogy a webalkalmazás által gyűjtött bizalmas adatok biztonságban legyenek az SQL Database-ben és azokon való átvitel során.</span><span class="sxs-lookup"><span data-stu-id="fef1f-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="fef1f-261">Az Azure-és Azure Stack hub-webalkalmazásokat az összes bejövő forgalom SSL-tanúsítványainak használatára konfigurálja.</span><span class="sxs-lookup"><span data-stu-id="fef1f-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="fef1f-262">SSL hozzáadása az Azure-hoz és Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="fef1f-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-263">SSL hozzáadása az Azure-hoz:</span><span class="sxs-lookup"><span data-stu-id="fef1f-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="fef1f-264">Győződjön meg arról, hogy a kapott SSL-tanúsítvány érvényes a létrehozott altartományhoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="fef1f-265">(A helyettesítő tanúsítványok használata rendben van.)</span><span class="sxs-lookup"><span data-stu-id="fef1f-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="fef1f-266">Az Azure-ban kövesse a **webalkalmazás előkészítése** és az SSL- **tanúsítvány** kötése szakaszát a [meglévő egyéni SSL-tanúsítvány kötése az Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) cikkhez című cikkben található utasításokat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="fef1f-267">Az **SSL-típusként**válassza a **SNI-alapú SSL** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="fef1f-268">A HTTPS-portra irányuló összes forgalom átirányítása.</span><span class="sxs-lookup"><span data-stu-id="fef1f-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="fef1f-269">Kövesse a [meglévő egyéni SSL-tanúsítvány kötése az Azure-Web Apps cikk HTTPS-alapú hitelesítésének](/azure/app-service/app-service-web-tutorial-custom-ssl) **megkövetelése** című szakaszának utasításait.</span><span class="sxs-lookup"><span data-stu-id="fef1f-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="fef1f-270">SSL hozzáadása Azure Stack hubhoz:</span><span class="sxs-lookup"><span data-stu-id="fef1f-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="fef1f-271">Ismételje meg az Azure-ban használt 1-3-es lépést.</span><span class="sxs-lookup"><span data-stu-id="fef1f-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="fef1f-272">A webalkalmazás konfigurálása és üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="fef1f-272">Configure and deploy the web app</span></span>

<span data-ttu-id="fef1f-273">Konfigurálja az telemetria a helyes Application Insights példányra, és konfigurálja a webalkalmazásokat a megfelelő kapcsolati karakterláncokkal.</span><span class="sxs-lookup"><span data-stu-id="fef1f-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="fef1f-274">További információ a Application Insightsről: [Mi az Application Insights?](/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="fef1f-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="fef1f-275">Application Insights hozzáadása</span><span class="sxs-lookup"><span data-stu-id="fef1f-275">Add Application Insights</span></span>

1. <span data-ttu-id="fef1f-276">Nyissa meg a webalkalmazást a Microsoft Visual Studióban.</span><span class="sxs-lookup"><span data-stu-id="fef1f-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="fef1f-277">[Adja hozzá Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) a projekthez, hogy továbbítsa a telemetria, amelyet a Application Insights használ a riasztások létrehozásához, amikor a webes forgalom növekszik vagy csökken.</span><span class="sxs-lookup"><span data-stu-id="fef1f-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="fef1f-278">Dinamikus kapcsolatok karakterláncának konfigurálása</span><span class="sxs-lookup"><span data-stu-id="fef1f-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="fef1f-279">A webalkalmazás minden példánya egy másik módszert fog használni az SQL-adatbázishoz való kapcsolódáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="fef1f-280">Az Azure-beli alkalmazás a SQL Server VM magánhálózati IP-címét, a Azure Stack hub alkalmazás pedig a SQL Server VM nyilvános IP-címét használja.</span><span class="sxs-lookup"><span data-stu-id="fef1f-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="fef1f-281">Egy Azure Stack hub-beli integrált rendszeren a nyilvános IP-cím nem lehet internetre irányítható.</span><span class="sxs-lookup"><span data-stu-id="fef1f-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="fef1f-282">Egy ASDK a nyilvános IP-cím nem a ASDK kívülre irányítható.</span><span class="sxs-lookup"><span data-stu-id="fef1f-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="fef1f-283">App Service környezeti változók használatával más kapcsolódási karakterláncot adhat át az alkalmazás minden egyes példányához.</span><span class="sxs-lookup"><span data-stu-id="fef1f-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="fef1f-284">Nyissa meg az alkalmazást a Visual Studióban.</span><span class="sxs-lookup"><span data-stu-id="fef1f-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="fef1f-285">Nyissa meg a Startup.cs, és keresse meg a következő kódrészletet:</span><span class="sxs-lookup"><span data-stu-id="fef1f-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="fef1f-286">Cserélje le az előző kódú blokkot a következő kódra, amely a *appsettings.js* fájljában definiált kapcsolatok karakterláncot használja:</span><span class="sxs-lookup"><span data-stu-id="fef1f-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="fef1f-287">App Service alkalmazás beállításainak konfigurálása</span><span class="sxs-lookup"><span data-stu-id="fef1f-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="fef1f-288">Hozzon létre a kapcsolatok karakterláncait az Azure-hoz és Azure Stack hub-hoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="fef1f-289">A karakterláncoknak azonosnak kell lenniük, kivéve a használt IP-címeket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="fef1f-290">Az Azure-ban és Azure Stack hub-ban adja hozzá a megfelelő kapcsolati karakterláncot [alkalmazás-beállításként](/azure/app-service/web-sites-configure) a webalkalmazásban, a `SQLCONNSTR\_` név előtagként használva.</span><span class="sxs-lookup"><span data-stu-id="fef1f-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="fef1f-291">**Mentse** a webalkalmazás beállításait, és indítsa újra az alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="fef1f-292">Automatikus skálázás engedélyezése a globális Azure-ban</span><span class="sxs-lookup"><span data-stu-id="fef1f-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="fef1f-293">Ha App Service környezetben hozza létre a webalkalmazást, az egy példánnyal kezdődik.</span><span class="sxs-lookup"><span data-stu-id="fef1f-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="fef1f-294">A példányok hozzáadásával automatikusan kibővítheti az alkalmazást, így több számítási erőforrást is biztosíthat az alkalmazáshoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="fef1f-295">Hasonlóképpen, automatikusan méretezheti és csökkentheti az alkalmazás által igényelt példányok számát.</span><span class="sxs-lookup"><span data-stu-id="fef1f-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="fef1f-296">Meg kell adnia egy App Service tervet a méretezési és méretezési beállítások konfigurálásához.</span><span class="sxs-lookup"><span data-stu-id="fef1f-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="fef1f-297">Ha nem rendelkezik csomaggal, hozzon létre egyet a következő lépések megkezdése előtt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="fef1f-298">Automatikus kibővítés engedélyezése</span><span class="sxs-lookup"><span data-stu-id="fef1f-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="fef1f-299">Az Azure-ban keresse meg a kibővíteni kívánt webhelyek App Service tervét, majd válassza a **kibővítés (App Service terv)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Kibővíthető Azure App Service](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="fef1f-301">Válassza az **autoskálázás engedélyezése**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-301">Select **Enable autoscale**.</span></span>

    ![Az autoskálázás engedélyezése Azure App Service](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="fef1f-303">Adja meg az **autoskálázási beállítás nevének**nevét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="fef1f-304">Az **alapértelmezett** automatikus skálázási szabálynál válassza a **skála mérőszám alapján**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="fef1f-305">A **példányokra vonatkozó korlátokat** állítsa **minimumra: 1**, **maximum: 10**, **alapértelmezett érték: 1**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Az autoskálázás konfigurálása Azure App Service](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="fef1f-307">Válassza **a + szabály hozzáadása**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="fef1f-308">A **metrika forrása**mezőben válassza ki az **aktuális erőforrás**elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="fef1f-309">A szabályhoz a következő feltételeket és műveleteket használhatja.</span><span class="sxs-lookup"><span data-stu-id="fef1f-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="fef1f-310">Feltételek</span><span class="sxs-lookup"><span data-stu-id="fef1f-310">Criteria</span></span>

1. <span data-ttu-id="fef1f-311">Az **idő összesítése** területen válassza az **átlag**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="fef1f-312">A **metrika neve**területen válassza a **CPU százalék**elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="fef1f-313">Az **operátor**területen válassza a **nagyobb, mint**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="fef1f-314">Állítsa a **küszöbértéket** **50**-re.</span><span class="sxs-lookup"><span data-stu-id="fef1f-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="fef1f-315">Állítsa az **időtartamot** **10**értékre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="fef1f-316">Művelet</span><span class="sxs-lookup"><span data-stu-id="fef1f-316">Action</span></span>

1. <span data-ttu-id="fef1f-317">A **művelet**területen válassza **a számának növekedése a**következővel:.</span><span class="sxs-lookup"><span data-stu-id="fef1f-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="fef1f-318">Állítsa a **példányszámot** a **2**értékre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="fef1f-319">Állítsa a **lehűlni** a **5-öt**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="fef1f-320">Válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-320">Select **Add**.</span></span>

5. <span data-ttu-id="fef1f-321">Válassza a **+ szabály hozzáadása**elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="fef1f-322">A **metrika forrása**mezőben válassza ki az **aktuális erőforrás elemet.**</span><span class="sxs-lookup"><span data-stu-id="fef1f-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="fef1f-323">Az aktuális erőforrás tartalmazni fogja a App Service terv nevét/GUID azonosítóját, és az **Erőforrás típusa** és az **erőforrás** legördülő listája nem lesz elérhető.</span><span class="sxs-lookup"><span data-stu-id="fef1f-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="fef1f-324">Automatikus skálázás engedélyezése</span><span class="sxs-lookup"><span data-stu-id="fef1f-324">Enable automatic scale in</span></span>

<span data-ttu-id="fef1f-325">A forgalom csökkenése esetén az Azure-webalkalmazás automatikusan csökkentheti az aktív példányok számát a költségek csökkentése érdekében.</span><span class="sxs-lookup"><span data-stu-id="fef1f-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="fef1f-326">Ez a művelet kevésbé agresszív, mint a felskálázás, és csökkenti az alkalmazás felhasználóira gyakorolt hatást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="fef1f-327">Lépjen az **alapértelmezett** Felskálázási feltételre, majd válassza a **+ szabály hozzáadása**elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="fef1f-328">A szabályhoz a következő feltételeket és műveleteket használhatja.</span><span class="sxs-lookup"><span data-stu-id="fef1f-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="fef1f-329">Feltételek</span><span class="sxs-lookup"><span data-stu-id="fef1f-329">Criteria</span></span>

1. <span data-ttu-id="fef1f-330">Az **idő összesítése** területen válassza az **átlag**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="fef1f-331">A **metrika neve**területen válassza a **CPU százalék**elemet.</span><span class="sxs-lookup"><span data-stu-id="fef1f-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="fef1f-332">Az **operátor**területen válassza a **kisebb, mint**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="fef1f-333">Állítsa a **küszöbértéket** **30**-ra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="fef1f-334">Állítsa az **időtartamot** **10**értékre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="fef1f-335">Művelet</span><span class="sxs-lookup"><span data-stu-id="fef1f-335">Action</span></span>

1. <span data-ttu-id="fef1f-336">A **művelet**területen válassza **a szám csökkentése a**következővel:.</span><span class="sxs-lookup"><span data-stu-id="fef1f-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="fef1f-337">Állítsa a **példányszámot** **1-re**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="fef1f-338">Állítsa a **lehűlni** a **5-öt**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="fef1f-339">Válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="fef1f-340">Traffic Manager profil létrehozása és a Felhőbeli skálázás konfigurálása</span><span class="sxs-lookup"><span data-stu-id="fef1f-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="fef1f-341">Hozzon létre egy Traffic Manager-profilt az Azure-ban, majd konfigurálja a végpontokat a Felhőbeli skálázás engedélyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="fef1f-342">Traffic Manager profil létrehozása</span><span class="sxs-lookup"><span data-stu-id="fef1f-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="fef1f-343">Válassza az **Erőforrás létrehozása** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="fef1f-344">Válassza a **Hálózat** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-344">Select **Networking**.</span></span>
3. <span data-ttu-id="fef1f-345">Válassza ki **Traffic Manager profilt** , és konfigurálja a következő beállításokat:</span><span class="sxs-lookup"><span data-stu-id="fef1f-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="fef1f-346">A **név**mezőben adja meg a profil nevét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="fef1f-347">Ennek a **névnek egyedinek kell** lennie az trafficmanager.net zónában, és egy új DNS-név (például northwindstore.trafficmanager.net) létrehozására szolgál.</span><span class="sxs-lookup"><span data-stu-id="fef1f-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="fef1f-348">Az **útválasztási módszer**beállításnál válassza ki a **súlyozott**értéket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="fef1f-349">Az **előfizetés**mezőben válassza ki azt az előfizetést, amelyben a profilt létre szeretné hozni.</span><span class="sxs-lookup"><span data-stu-id="fef1f-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="fef1f-350">Az **erőforráscsoport**területen hozzon létre egy új erőforráscsoportot ehhez a profilhoz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="fef1f-351">Az **Erőforráscsoport helye** területen válassza ki az erőforráscsoport helyét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="fef1f-352">Ez a beállítás az erőforráscsoport helyére vonatkozik, és nincs hatással a globálisan telepített Traffic Manager-profilra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="fef1f-353">Kattintson a **Létrehozás** gombra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-353">Select **Create**.</span></span>

    ![Traffic Manager profil létrehozása](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="fef1f-355">Ha a Traffic Manager-profil globális telepítése befejeződött, az a (z) alatt létrehozott erőforráscsoport erőforrásainak listájában jelenik meg.</span><span class="sxs-lookup"><span data-stu-id="fef1f-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="fef1f-356">Traffic Manager-végpontok hozzáadása</span><span class="sxs-lookup"><span data-stu-id="fef1f-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="fef1f-357">Keresse meg a létrehozott Traffic Manager profilt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="fef1f-358">Ha a profilhoz tartozó erőforráscsoporthoz navigál, válassza ki a profilt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="fef1f-359">**Traffic Manager profilban**a **Beállítások**területen válassza a **végpontok**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="fef1f-360">Válassza a **Hozzáadás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-360">Select **Add**.</span></span>

4. <span data-ttu-id="fef1f-361">A **végpont hozzáadása**területen használja az Azure stack hub következő beállításait:</span><span class="sxs-lookup"><span data-stu-id="fef1f-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="fef1f-362">A **Típus mezőben**válassza a **külső végpont**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="fef1f-363">Adja meg a végpont **nevét** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="fef1f-364">A **teljes tartománynév (FQDN) vagy az IP-** cím mezőben adja meg az Azure stack hub-webalkalmazás külső URL-címét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="fef1f-365">A **súlyozáshoz**tartsa meg az alapértelmezett értéket, **1**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="fef1f-366">Ez a súlyozás azt eredményezi, hogy az adott végpontra irányuló forgalom kifogástalan állapotú lesz.</span><span class="sxs-lookup"><span data-stu-id="fef1f-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="fef1f-367">Hagyja **Letiltva a Hozzáadás másként** jelölést.</span><span class="sxs-lookup"><span data-stu-id="fef1f-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="fef1f-368">Az Azure Stack hub-végpont mentéséhez kattintson **az OK gombra** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="fef1f-369">Ezután konfigurálja az Azure-végpontot.</span><span class="sxs-lookup"><span data-stu-id="fef1f-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="fef1f-370">**Traffic Manager profil**lapon válassza a **végpontok**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="fef1f-371">Válassza a **+ Hozzáadás**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-371">Select **+Add**.</span></span>
3. <span data-ttu-id="fef1f-372">A **végpont hozzáadása**lehetőségnél használja a következő beállításokat az Azure-hoz:</span><span class="sxs-lookup"><span data-stu-id="fef1f-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="fef1f-373">A **Típus mezőben**válassza az **Azure-végpont**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="fef1f-374">Adja meg a végpont **nevét** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="fef1f-375">A **cél erőforrástípus mezőben**válassza a **app Service**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="fef1f-376">A **cél erőforrásnál**válassza az **app Service kiválasztása** lehetőséget az azonos előfizetésben lévő Web Apps listájának megtekintéséhez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="fef1f-377">Az **Erőforrás** panelen válassza ki az első végpontként hozzáadni kívánt alkalmazásszolgáltatást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="fef1f-378">A **súlyozáshoz**válassza a **2**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="fef1f-379">Ez a beállítás azt eredményezi, hogy a végponthoz tartozó összes forgalom nem kifogástalan állapotú, vagy ha van olyan szabály/riasztás, amely az indításkor átirányítja a forgalmat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="fef1f-380">Hagyja **Letiltva a Hozzáadás másként** jelölést.</span><span class="sxs-lookup"><span data-stu-id="fef1f-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="fef1f-381">Az Azure-végpont mentéséhez kattintson **az OK gombra** .</span><span class="sxs-lookup"><span data-stu-id="fef1f-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="fef1f-382">Mindkét végpont konfigurálása után **Traffic Manager profilban** szerepelnek a **végpontok**kiválasztása után.</span><span class="sxs-lookup"><span data-stu-id="fef1f-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="fef1f-383">A következő képernyőfelvételen szereplő példa két végpontot mutat be, amelyek mindegyike állapota és konfigurációs adatai szerepelnek.</span><span class="sxs-lookup"><span data-stu-id="fef1f-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Traffic Manager profilban található végpontok](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="fef1f-385">Application Insights figyelésének és riasztásának beállítása</span><span class="sxs-lookup"><span data-stu-id="fef1f-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="fef1f-386">Az Azure Application Insights segítségével figyelheti az alkalmazást, és riasztásokat küldhet a konfigurált feltételek alapján.</span><span class="sxs-lookup"><span data-stu-id="fef1f-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="fef1f-387">Néhány példa: az alkalmazás nem érhető el, hibákba ütközik, vagy teljesítménnyel kapcsolatos problémákat mutat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="fef1f-388">A riasztások létrehozásához Application Insights metrikákat kell használnia.</span><span class="sxs-lookup"><span data-stu-id="fef1f-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="fef1f-389">Amikor ezek a riasztások aktiválódnak, a webalkalmazás példánya automatikusan átvált Azure Stack hub-ról az Azure-ra a vertikális felskálázáshoz, majd visszahelyezi a Azure Stack hub-ra a méretezéshez.</span><span class="sxs-lookup"><span data-stu-id="fef1f-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="fef1f-390">Riasztás létrehozása mérőszámokból</span><span class="sxs-lookup"><span data-stu-id="fef1f-390">Create an alert from metrics</span></span>

<span data-ttu-id="fef1f-391">Nyissa meg az oktatóanyaghoz tartozó erőforráscsoportot, majd válassza ki a Application Insights példányt a **Application Insights**megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="fef1f-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="fef1f-393">Ennek a nézetnek a használatával kibővíthető riasztást és egy méretezési riasztást hozhat létre.</span><span class="sxs-lookup"><span data-stu-id="fef1f-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="fef1f-394">A kibővíthető riasztás létrehozása</span><span class="sxs-lookup"><span data-stu-id="fef1f-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="fef1f-395">A **Konfigurálás**területen válassza a **riasztások (klasszikus)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="fef1f-396">Válassza a **metrikus riasztás hozzáadása (klasszikus)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="fef1f-397">A **szabály hozzáadása**területen adja meg a következő beállításokat:</span><span class="sxs-lookup"><span data-stu-id="fef1f-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="fef1f-398">A **név**mezőben adja meg az **Azure-felhőbe való betörést**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="fef1f-399">A **Leírás** megadása nem kötelező.</span><span class="sxs-lookup"><span data-stu-id="fef1f-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="fef1f-400">A **forrás**  >  **riasztás**területen válassza a **metrikák**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="fef1f-401">A **feltételek**területen válassza ki az előfizetését, a Traffic Manager profiljához tartozó erőforráscsoportot, valamint az erőforrás Traffic Manager profiljának nevét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="fef1f-402">A **metrika**mezőben válassza a **kérelmek aránya**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="fef1f-403">A **feltétel**beállításnál válassza a **nagyobb, mint**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="fef1f-404">A **küszöbérték**mezőbe írja be a **2**értéket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="fef1f-405">Az **időtartam**mezőben válassza **az utolsó 5 perc**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="fef1f-406">Az **értesítés**alatt:</span><span class="sxs-lookup"><span data-stu-id="fef1f-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="fef1f-407">Jelölje be az **e-mail-tulajdonosok, a közreműködők és az olvasók**jelölőnégyzetét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="fef1f-408">Adja meg az e-mail-címét **további rendszergazdai e-mailek (ek)** számára.</span><span class="sxs-lookup"><span data-stu-id="fef1f-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="fef1f-409">A menüsávon válassza a **Mentés**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="fef1f-410">A méretezési riasztás létrehozása</span><span class="sxs-lookup"><span data-stu-id="fef1f-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="fef1f-411">A **Konfigurálás**területen válassza a **riasztások (klasszikus)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="fef1f-412">Válassza a **metrikus riasztás hozzáadása (klasszikus)** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="fef1f-413">A **szabály hozzáadása**területen adja meg a következő beállításokat:</span><span class="sxs-lookup"><span data-stu-id="fef1f-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="fef1f-414">A **név**mezőbe írja be a **Méretezés vissza Azure stack hubhoz**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="fef1f-415">A **Leírás** megadása nem kötelező.</span><span class="sxs-lookup"><span data-stu-id="fef1f-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="fef1f-416">A **forrás**  >  **riasztás**területen válassza a **metrikák**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="fef1f-417">A **feltételek**területen válassza ki az előfizetését, a Traffic Manager profiljához tartozó erőforráscsoportot, valamint az erőforrás Traffic Manager profiljának nevét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="fef1f-418">A **metrika**mezőben válassza a **kérelmek aránya**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="fef1f-419">A **feltétel**beállításnál válassza a **kisebb, mint**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="fef1f-420">A **küszöbérték**mezőbe írja be a **2**értéket.</span><span class="sxs-lookup"><span data-stu-id="fef1f-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="fef1f-421">Az **időtartam**mezőben válassza **az utolsó 5 perc**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="fef1f-422">Az **értesítés**alatt:</span><span class="sxs-lookup"><span data-stu-id="fef1f-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="fef1f-423">Jelölje be az **e-mail-tulajdonosok, a közreműködők és az olvasók**jelölőnégyzetét.</span><span class="sxs-lookup"><span data-stu-id="fef1f-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="fef1f-424">Adja meg az e-mail-címét **további rendszergazdai e-mailek (ek)** számára.</span><span class="sxs-lookup"><span data-stu-id="fef1f-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="fef1f-425">A menüsávon válassza a **Mentés**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="fef1f-426">A következő képernyőképen a kibővíthető és méretezhető riasztások láthatók.</span><span class="sxs-lookup"><span data-stu-id="fef1f-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Application Insights riasztások (klasszikus)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fef1f-428">Az Azure és a Azure Stack hub közötti forgalom átirányítása</span><span class="sxs-lookup"><span data-stu-id="fef1f-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-429">Konfigurálhatja a webalkalmazás-forgalom manuális vagy automatikus átváltását az Azure és a Azure Stack hub között.</span><span class="sxs-lookup"><span data-stu-id="fef1f-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fef1f-430">Manuális váltás konfigurálása az Azure és a Azure Stack hub között</span><span class="sxs-lookup"><span data-stu-id="fef1f-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-431">Ha a webhely eléri a konfigurált küszöbértékeket, riasztást kap.</span><span class="sxs-lookup"><span data-stu-id="fef1f-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="fef1f-432">A következő lépésekkel manuálisan irányíthatja át a forgalmat az Azure-ba.</span><span class="sxs-lookup"><span data-stu-id="fef1f-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="fef1f-433">A Azure Portal válassza ki a Traffic Manager profilt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Azure Portal Traffic Manager végpontok](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="fef1f-435">Válassza a **végpontok**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="fef1f-436">Válassza ki az **Azure-végpontot**.</span><span class="sxs-lookup"><span data-stu-id="fef1f-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="fef1f-437">Az **állapot**területen válassza az **engedélyezve**lehetőséget, majd kattintson a **Mentés**gombra.</span><span class="sxs-lookup"><span data-stu-id="fef1f-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Azure-végpont engedélyezése Azure Portal](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="fef1f-439">A Traffic Manager profilhoz tartozó **végpontokon** válassza a **külső végpont**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="fef1f-440">Az **állapot**területen válassza a **Letiltva**, majd a **Mentés**lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="fef1f-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Azure Stack hub-végpont letiltása Azure Portal](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="fef1f-442">A végpontok konfigurálása után az alkalmazások forgalma az Azure Stack hub-webalkalmazás helyett az Azure kibővíthető webalkalmazásra kerül.</span><span class="sxs-lookup"><span data-stu-id="fef1f-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Az Azure webalkalmazás-forgalomban megváltoztatott végpontok](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="fef1f-444">Ha vissza szeretné fordítani a folyamatot Azure Stack hubhoz, a következő lépésekkel hajthatja végre az előző lépéseket:</span><span class="sxs-lookup"><span data-stu-id="fef1f-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="fef1f-445">Engedélyezze az Azure Stack hub-végpontot.</span><span class="sxs-lookup"><span data-stu-id="fef1f-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="fef1f-446">Tiltsa le az Azure-végpontot.</span><span class="sxs-lookup"><span data-stu-id="fef1f-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="fef1f-447">Az Azure és a Azure Stack hub közötti automatikus váltás konfigurálása</span><span class="sxs-lookup"><span data-stu-id="fef1f-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="fef1f-448">Application Insights figyelést is használhat, ha az alkalmazás a Azure Functions által biztosított [kiszolgáló](https://azure.microsoft.com/overview/serverless-computing/) nélküli környezetben fut.</span><span class="sxs-lookup"><span data-stu-id="fef1f-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="fef1f-449">Ebben az esetben beállíthatja, hogy a Application Insights egy olyan webhook használatára, amely meghívja a Function alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="fef1f-450">Ez az alkalmazás automatikusan engedélyezi vagy letiltja a végpontot egy riasztásra válaszul.</span><span class="sxs-lookup"><span data-stu-id="fef1f-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="fef1f-451">Az automatikus forgalmi váltás konfigurálásához kövesse az alábbi lépéseket útmutatóként.</span><span class="sxs-lookup"><span data-stu-id="fef1f-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="fef1f-452">Hozzon létre egy Azure Function-alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="fef1f-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="fef1f-453">Hozzon létre egy HTTP által aktivált függvényt.</span><span class="sxs-lookup"><span data-stu-id="fef1f-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="fef1f-454">Importálja a Resource Manager, a Web Apps és a Traffic Manager Azure SDK-kat.</span><span class="sxs-lookup"><span data-stu-id="fef1f-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="fef1f-455">Kód fejlesztése a következőre:</span><span class="sxs-lookup"><span data-stu-id="fef1f-455">Develop code to:</span></span>

   - <span data-ttu-id="fef1f-456">Hitelesítse magát az Azure-előfizetésében.</span><span class="sxs-lookup"><span data-stu-id="fef1f-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="fef1f-457">Olyan paramétert használjon, amely a Traffic Manager végpontokat az Azure-ba vagy Azure Stack hub-ra irányuló közvetlen forgalomra vált.</span><span class="sxs-lookup"><span data-stu-id="fef1f-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="fef1f-458">Mentse a kódot, és adja hozzá a Function app URL-címét a megfelelő paraméterekkel a Application Insights riasztási szabály beállításainak **webhook** szakaszához.</span><span class="sxs-lookup"><span data-stu-id="fef1f-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="fef1f-459">A rendszer automatikusan átirányítja a forgalmat, amikor egy Application Insights riasztás következik be.</span><span class="sxs-lookup"><span data-stu-id="fef1f-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="fef1f-460">További lépések</span><span class="sxs-lookup"><span data-stu-id="fef1f-460">Next steps</span></span>

- <span data-ttu-id="fef1f-461">Az Azure Cloud Patterns szolgáltatással kapcsolatos további információkért lásd: [Felhőbeli tervezési minták](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="fef1f-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
