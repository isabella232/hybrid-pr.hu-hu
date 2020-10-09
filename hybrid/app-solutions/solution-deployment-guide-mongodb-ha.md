---
title: Magasan elérhető MongoDB-megoldás üzembe helyezése az Azure-ban és Azure Stack hub
description: Ismerje meg, hogyan helyezhet üzembe egy magasan elérhető MongoDB-megoldást az Azure-ban és Azure Stack hub-ban
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852507"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="ff110-103">Magasan elérhető MongoDB-megoldás üzembe helyezése az Azure-ban és Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="ff110-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="ff110-104">Ez a cikk végigvezeti egy olyan alapszintű magas rendelkezésre állású (HA) MongoDB-fürt automatikus üzembe helyezésén, amely egy vész-helyreállítási (DR) hellyel rendelkezik két Azure Stack hub-környezetben.</span><span class="sxs-lookup"><span data-stu-id="ff110-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="ff110-105">További információ a MongoDB és a magas rendelkezésre állásról: [replikakészlet tagjai](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="ff110-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="ff110-106">Ebben a megoldásban a következőhöz hozzon létre egy mintavételi környezetet:</span><span class="sxs-lookup"><span data-stu-id="ff110-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="ff110-107">Üzembe helyezés koordinálása két Azure Stack hub között.</span><span class="sxs-lookup"><span data-stu-id="ff110-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="ff110-108">A Docker használatával csökkentheti a függőségi problémákat az Azure API-profilokkal.</span><span class="sxs-lookup"><span data-stu-id="ff110-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="ff110-109">Helyezzen üzembe egy alapszintű, magasan elérhető MongoDB-fürtöt vész-helyreállítási hellyel.</span><span class="sxs-lookup"><span data-stu-id="ff110-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="ff110-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="ff110-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="ff110-111">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="ff110-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="ff110-112">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="ff110-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="ff110-113">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="ff110-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="ff110-114">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="ff110-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="ff110-115">A MongoDB architektúrája Azure Stack hub-vel</span><span class="sxs-lookup"><span data-stu-id="ff110-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![magasan elérhető MongoDB architektúra Azure Stack hub-ban](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="ff110-117">A MongoDB előfeltételei Azure Stack hubhoz</span><span class="sxs-lookup"><span data-stu-id="ff110-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="ff110-118">Két csatlakoztatott Azure Stack hub integrált rendszer (Azure Stack hub).</span><span class="sxs-lookup"><span data-stu-id="ff110-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="ff110-119">Ez a központi telepítés nem működik a Azure Stack Development Kiton (ASDK).</span><span class="sxs-lookup"><span data-stu-id="ff110-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="ff110-120">További információ az Azure Stack hub-ról: [Mi az Azure stack hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="ff110-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="ff110-121">Bérlői előfizetés az egyes Azure Stack hubokon.</span><span class="sxs-lookup"><span data-stu-id="ff110-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="ff110-122">**Jegyezze fel minden egyes Azure Stack hub előfizetési AZONOSÍTÓját és Azure Resource Manager végpontját.**</span><span class="sxs-lookup"><span data-stu-id="ff110-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="ff110-123">Egy Azure Active Directory (Azure AD) egyszerű szolgáltatásnév, amely jogosult a bérlői előfizetésre az egyes Azure Stack hubokon.</span><span class="sxs-lookup"><span data-stu-id="ff110-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="ff110-124">Előfordulhat, hogy két egyszerű szolgáltatást kell létrehoznia, ha az Azure Stack hubok különböző Azure AD-bérlők között vannak telepítve.</span><span class="sxs-lookup"><span data-stu-id="ff110-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="ff110-125">Ha meg szeretné tudni, hogyan hozhat létre egyszerű szolgáltatásnevet Azure Stack hubhoz, tekintse meg [az alkalmazás-identitás használata Azure stack hub-erőforrások eléréséhez](/azure-stack/user/azure-stack-create-service-principals)című témakört.</span><span class="sxs-lookup"><span data-stu-id="ff110-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="ff110-126">**Jegyezze fel az egyes egyszerű szolgáltatások alkalmazás-AZONOSÍTÓját, az ügyfél titkos kulcsát és a bérlő nevét (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="ff110-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="ff110-127">Ubuntu 16,04 szindikált minden Azure Stack hub piactéren.</span><span class="sxs-lookup"><span data-stu-id="ff110-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="ff110-128">További információ a Marketplace Syndication szolgáltatásról: [Marketplace-elemek letöltése Azure stack hubhoz](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="ff110-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="ff110-129">A helyi gépre telepített [Windows Docker](https://docs.docker.com/docker-for-windows/) .</span><span class="sxs-lookup"><span data-stu-id="ff110-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="ff110-130">A Docker-rendszerkép beszerzése</span><span class="sxs-lookup"><span data-stu-id="ff110-130">Get the Docker image</span></span>

<span data-ttu-id="ff110-131">Az egyes központi telepítésekhez tartozó Docker-rendszerképek megszüntetik a Azure PowerShell különböző verziói közötti függőségi problémákat.</span><span class="sxs-lookup"><span data-stu-id="ff110-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="ff110-132">Győződjön meg arról, hogy a Windows rendszerhez készült Docker Windows-tárolókat használ.</span><span class="sxs-lookup"><span data-stu-id="ff110-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="ff110-133">Futtassa a következő parancsot egy rendszergazda jogú parancssorban a Docker-tároló üzembe helyezési parancsfájlokkal való lekéréséhez.</span><span class="sxs-lookup"><span data-stu-id="ff110-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="ff110-134">A fürtök üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="ff110-134">Deploy the clusters</span></span>

1. <span data-ttu-id="ff110-135">A tároló rendszerképének leállítása után indítsa el a rendszerképet.</span><span class="sxs-lookup"><span data-stu-id="ff110-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="ff110-136">A tároló elindítása után egy emelt szintű PowerShell-terminált kap a tárolóban.</span><span class="sxs-lookup"><span data-stu-id="ff110-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="ff110-137">Módosítsa a címtárakat az üzembe helyezési parancsfájl eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="ff110-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="ff110-138">Futtassa az üzemelő példányt.</span><span class="sxs-lookup"><span data-stu-id="ff110-138">Run the deployment.</span></span> <span data-ttu-id="ff110-139">Adja meg a hitelesítő adatokat és az erőforrások nevét, ahol szükséges.</span><span class="sxs-lookup"><span data-stu-id="ff110-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="ff110-140">HA az a Azure Stack központot jelöli, ahol a HA-fürtöt telepíteni fogja.</span><span class="sxs-lookup"><span data-stu-id="ff110-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="ff110-141">DR arra a Azure Stack hubhoz utal, ahol a DR-fürtöt telepíti.</span><span class="sxs-lookup"><span data-stu-id="ff110-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="ff110-142">Írja be a `Y` NuGet-szolgáltató telepítésének engedélyezését, amely a telepítendő "2018-03-01-Hybrid" modulok indítását fogja elindítani.</span><span class="sxs-lookup"><span data-stu-id="ff110-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="ff110-143">Először a HA-erőforrások lesznek telepítve.</span><span class="sxs-lookup"><span data-stu-id="ff110-143">The HA resources will deploy first.</span></span> <span data-ttu-id="ff110-144">Figyelje a központi telepítést, és várjon, amíg befejeződik.</span><span class="sxs-lookup"><span data-stu-id="ff110-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="ff110-145">Ha az üzenet arról tájékoztatja, hogy a HA a telepítés befejeződött, akkor az üzembe helyezett erőforrások megtekintéséhez tekintse meg a HA Azure Stack hub portálját.</span><span class="sxs-lookup"><span data-stu-id="ff110-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="ff110-146">Folytassa a DR-erőforrások üzembe helyezésével, és döntse el, hogy szeretné-e engedélyezni a DR Azure Stack hub Jump Box használatát a fürttel való kommunikációhoz.</span><span class="sxs-lookup"><span data-stu-id="ff110-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="ff110-147">Várjon, amíg befejeződik a DR erőforrás üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="ff110-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="ff110-148">Miután a DR erőforrás üzembe helyezése befejeződött, lépjen ki a tárolóból.</span><span class="sxs-lookup"><span data-stu-id="ff110-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="ff110-149">További lépések</span><span class="sxs-lookup"><span data-stu-id="ff110-149">Next steps</span></span>

- <span data-ttu-id="ff110-150">Ha a DR Azure Stack hubhoz engedélyezte a Jump Box virtuális gépet, akkor SSH-n keresztül kapcsolódhat, és a Mongo parancssori felületének telepítésével használhatja a MongoDB-fürtöt.</span><span class="sxs-lookup"><span data-stu-id="ff110-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="ff110-151">Ha többet szeretne megtudni a MongoDB-mel való interakcióról, tekintse meg [a Mongo-rendszerhéjat](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="ff110-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="ff110-152">A hibrid felhőalapú alkalmazásokkal kapcsolatos további információkért lásd: [hibrid felhőalapú megoldások.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="ff110-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="ff110-153">Módosítsa a minta kódját a [githubon](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="ff110-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>