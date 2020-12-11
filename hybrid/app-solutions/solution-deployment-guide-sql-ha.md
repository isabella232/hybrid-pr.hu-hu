---
title: SQL Server 2016 rendelkezésre állási csoport üzembe helyezése az Azure-ban és Azure Stack hub-ban
description: Megtudhatja, hogyan helyezhet üzembe egy SQL Server 2016 rendelkezésre állási csoportot az Azure-ban és Azure Stack hub-ban.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0f857515a44ece7f967ade3dee8f493481709851
ms.sourcegitcommit: c890f2c5e5e5f9f93c921f02dd1a6ca5026d5289
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/10/2020
ms.locfileid: "97091802"
---
# <a name="deploy-a-sql-server-2016-availability-group-across-two-azure-stack-hub-environments"></a><span data-ttu-id="fa605-103">SQL Server 2016 rendelkezésre állási csoport üzembe helyezése két Azure Stack hub-környezetben</span><span class="sxs-lookup"><span data-stu-id="fa605-103">Deploy a SQL Server 2016 availability group across two Azure Stack Hub environments</span></span>

<span data-ttu-id="fa605-104">Ez a cikk végigvezeti egy olyan alapszintű magas rendelkezésre állású (HA) SQL Server 2016 Enterprise-fürt automatikus üzembe helyezésén, amely egy aszinkron vész-helyreállítási (DR) hellyel rendelkezik két Azure Stack hub-környezetben.</span><span class="sxs-lookup"><span data-stu-id="fa605-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="fa605-105">Ha többet szeretne megtudni a SQL Server 2016 és a magas rendelkezésre állásról, tekintse meg az [Always On rendelkezésre állási csoportok: magas rendelkezésre állású és vész-helyreállítási megoldást](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="fa605-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="fa605-106">Ebben a megoldásban egy példaként szolgáló környezetet fog kiépíteni a következőkre:</span><span class="sxs-lookup"><span data-stu-id="fa605-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="fa605-107">Üzembe helyezés koordinálása két Azure Stack hub között.</span><span class="sxs-lookup"><span data-stu-id="fa605-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="fa605-108">A Docker használatával csökkentheti a függőségi problémákat az Azure API-profilokkal.</span><span class="sxs-lookup"><span data-stu-id="fa605-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="fa605-109">Helyezzen üzembe egy alapszintű, magasan elérhető SQL Server 2016 Enterprise-fürtöt vész-helyreállítási hellyel.</span><span class="sxs-lookup"><span data-stu-id="fa605-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="fa605-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="fa605-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="fa605-111">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="fa605-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="fa605-112">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="fa605-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="fa605-113">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="fa605-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="fa605-114">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="fa605-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="fa605-115">SQL Server 2016 architektúrája</span><span class="sxs-lookup"><span data-stu-id="fa605-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 SQL HA Azure Stack hub](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="fa605-117">Az SQL Server 2016 előfeltételei</span><span class="sxs-lookup"><span data-stu-id="fa605-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="fa605-118">Két csatlakoztatott Azure Stack hub integrált rendszer (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="fa605-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="fa605-119">Ez a központi telepítés nem működik a Azure Stack Development Kiton (ASDK).</span><span class="sxs-lookup"><span data-stu-id="fa605-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="fa605-120">Ha többet szeretne megtudni az Azure Stack hub-ról, tekintse meg a [Azure stack áttekintését](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="fa605-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="fa605-121">Bérlői előfizetés az egyes Azure Stack hubokon.</span><span class="sxs-lookup"><span data-stu-id="fa605-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="fa605-122">**Jegyezze fel minden egyes Azure Stack hub előfizetési AZONOSÍTÓját és Azure Resource Manager végpontját.**</span><span class="sxs-lookup"><span data-stu-id="fa605-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="fa605-123">Egy Azure Active Directory (Azure AD) egyszerű szolgáltatásnév, amely jogosult a bérlői előfizetésre az egyes Azure Stack hubokon.</span><span class="sxs-lookup"><span data-stu-id="fa605-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="fa605-124">Előfordulhat, hogy két egyszerű szolgáltatást kell létrehoznia, ha az Azure Stack hubok különböző Azure AD-bérlők között vannak telepítve.</span><span class="sxs-lookup"><span data-stu-id="fa605-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="fa605-125">Ha meg szeretné tudni, hogyan hozhat létre egyszerű szolgáltatásnevet Azure Stack hubhoz, tekintse meg az [egyszerű szolgáltatás létrehozása az alkalmazások Azure stack hub-erőforrásokhoz való hozzáférésének biztosítása érdekében](/azure-stack/user/azure-stack-create-service-principals)című témakört.</span><span class="sxs-lookup"><span data-stu-id="fa605-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="fa605-126">**Jegyezze fel az egyes egyszerű szolgáltatások alkalmazás-AZONOSÍTÓját, az ügyfél titkos kulcsát és a bérlő nevét (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="fa605-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="fa605-127">SQL Server 2016 Enterprise konzorciális az egyes Azure Stack hub-piactéren.</span><span class="sxs-lookup"><span data-stu-id="fa605-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="fa605-128">További információ a Marketplace Syndication szolgáltatásról: [Marketplace-elemek letöltése Azure stack hubhoz](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="fa605-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="fa605-129">**Győződjön meg arról, hogy a szervezet rendelkezik a megfelelő SQL-licenccel.**</span><span class="sxs-lookup"><span data-stu-id="fa605-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="fa605-130">A helyi gépre telepített [Windows Docker](https://docs.docker.com/docker-for-windows/) .</span><span class="sxs-lookup"><span data-stu-id="fa605-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="fa605-131">A Docker-rendszerkép beszerzése</span><span class="sxs-lookup"><span data-stu-id="fa605-131">Get the Docker image</span></span>

<span data-ttu-id="fa605-132">Az egyes központi telepítésekhez tartozó Docker-rendszerképek megszüntetik a Azure PowerShell különböző verziói közötti függőségi problémákat.</span><span class="sxs-lookup"><span data-stu-id="fa605-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="fa605-133">Győződjön meg arról, hogy a Windows rendszerhez készült Docker Windows-tárolókat használ.</span><span class="sxs-lookup"><span data-stu-id="fa605-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="fa605-134">Futtassa a következő parancsfájlt egy rendszergazda jogú parancssorban a Docker-tároló üzembe helyezési parancsfájlokkal való lekéréséhez.</span><span class="sxs-lookup"><span data-stu-id="fa605-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="fa605-135">A rendelkezésre állási csoport üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="fa605-135">Deploy the availability group</span></span>

1. <span data-ttu-id="fa605-136">A tároló rendszerképének leállítása után indítsa el a rendszerképet.</span><span class="sxs-lookup"><span data-stu-id="fa605-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="fa605-137">A tároló elindítása után egy emelt szintű PowerShell-terminált kap a tárolóban.</span><span class="sxs-lookup"><span data-stu-id="fa605-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="fa605-138">Módosítsa a címtárakat az üzembe helyezési parancsfájl eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="fa605-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="fa605-139">Futtassa az üzemelő példányt.</span><span class="sxs-lookup"><span data-stu-id="fa605-139">Run the deployment.</span></span> <span data-ttu-id="fa605-140">Adja meg a hitelesítő adatokat és az erőforrások nevét, ahol szükséges.</span><span class="sxs-lookup"><span data-stu-id="fa605-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="fa605-141">HA az a Azure Stack központot jelöli, ahol a HA-fürtöt telepíteni fogja.</span><span class="sxs-lookup"><span data-stu-id="fa605-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="fa605-142">DR arra a Azure Stack hubhoz utal, ahol a DR-fürtöt telepíti.</span><span class="sxs-lookup"><span data-stu-id="fa605-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
      -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
      -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
      -AADTenantName_HA "hatenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_HA "haresourcegroupname" `
      -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
      -AzureStackSubscriptionId_HA "haSubscriptionId" `
      -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
      -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
      -AADTenantName_DR "drtenantname.onmicrosoft.com" `
      -AzureStackResourceGroup_DR "drresourcegroupname" `
      -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
      -AzureStackSubscriptionId_DR "drSubscriptionId"
      ```

4. <span data-ttu-id="fa605-143">Írja be a `Y` NuGet-szolgáltató telepítésének engedélyezését, amely a telepítendő "2018-03-01-Hybrid" modulok indítását fogja elindítani.</span><span class="sxs-lookup"><span data-stu-id="fa605-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="fa605-144">Várjon, amíg az erőforrás üzembe helyezése befejeződik.</span><span class="sxs-lookup"><span data-stu-id="fa605-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="fa605-145">Miután a DR erőforrás üzembe helyezése befejeződött, lépjen ki a tárolóból.</span><span class="sxs-lookup"><span data-stu-id="fa605-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="fa605-146">Vizsgálja meg az üzemelő példányt az egyes Azure Stack hub-portálon található erőforrások megtekintésével.</span><span class="sxs-lookup"><span data-stu-id="fa605-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="fa605-147">Kapcsolódjon az egyik SQL-példányhoz a HA-környezetben, és vizsgálja meg a rendelkezésre állási csoportot SQL Server Management Studio (SSMS) használatával.</span><span class="sxs-lookup"><span data-stu-id="fa605-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="fa605-149">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="fa605-149">Next steps</span></span>

- <span data-ttu-id="fa605-150">SQL Server Management Studio használatával manuálisan hajthatja végre a feladatátvételt a fürtön.</span><span class="sxs-lookup"><span data-stu-id="fa605-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="fa605-151">Lásd: az [Always On rendelkezésre állási csoport kényszerített manuális feladatátvételének végrehajtása (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="fa605-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="fa605-152">További információ a hibrid felhőalapú alkalmazásokról.</span><span class="sxs-lookup"><span data-stu-id="fa605-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="fa605-153">Tekintse meg a [hibrid felhőalapú megoldásokat.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="fa605-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="fa605-154">Használja saját adatait, vagy módosítsa a kódot erre a mintára a [githubon](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="fa605-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
