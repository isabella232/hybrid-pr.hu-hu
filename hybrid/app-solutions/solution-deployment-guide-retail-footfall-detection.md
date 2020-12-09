---
title: AI-alapú lépés hangja-észlelési megoldás üzembe helyezése az Azure-ban és Azure Stack hub-ban
description: Megtudhatja, hogyan helyezhet üzembe egy AI-alapú lépés hangja-észlelési megoldást a látogatói forgalom elemzéséhez a kiskereskedelmi üzletekben az Azure és a Azure Stack hub használatával.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901490"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a><span data-ttu-id="cf6ac-103">AI-alapú lépés hangja-észlelési megoldás üzembe helyezése az Azure és Azure Stack hub használatával</span><span class="sxs-lookup"><span data-stu-id="cf6ac-103">Deploy an AI-based footfall detection solution using Azure and Azure Stack Hub</span></span>

<span data-ttu-id="cf6ac-104">Ez a cikk bemutatja, hogyan helyezhet üzembe olyan AI-alapú megoldást, amely az Azure, az Azure Stack hub és a Custom Vision AI fejlesztői csomag használatával bepillantást nyerhet a valós világbeli műveletekkel.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-104">This article describes how to deploy an AI-based solution that generates insights from real world actions by using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

<span data-ttu-id="cf6ac-105">Ebben a megoldásban a következőket sajátíthatja el:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-105">In this solution, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="cf6ac-106">Felhőbeli natív alkalmazáscsomag (CNAB-EK) üzembe helyezése az Edge-ben.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-106">Deploy Cloud Native Application Bundles (CNAB) at the edge.</span></span> 
> - <span data-ttu-id="cf6ac-107">Felhőbeli határokat felölelő alkalmazás üzembe helyezése.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-107">Deploy an app that spans cloud boundaries.</span></span>
> - <span data-ttu-id="cf6ac-108">Használja a Custom Vision AI dev Kit for következtetést a szélén.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-108">Use the Custom Vision AI Dev Kit for inference at the edge.</span></span>

> [!Tip]  
> <span data-ttu-id="cf6ac-109">![Hibrid oszlopok diagramja](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="cf6ac-109">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="cf6ac-110">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-110">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="cf6ac-111">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-111">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="cf6ac-112">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-112">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="cf6ac-113">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-113">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="cf6ac-114">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="cf6ac-114">Prerequisites</span></span>

<span data-ttu-id="cf6ac-115">Az üzembe helyezési útmutató első lépéseinek megkezdése előtt győződjön meg arról, hogy:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-115">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="cf6ac-116">Tekintse át a [lépés hangja-észlelési minta](pattern-retail-footfall-detection.md) témakört.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-116">Review the [Footfall detection pattern](pattern-retail-footfall-detection.md) topic.</span></span>
- <span data-ttu-id="cf6ac-117">Felhasználói hozzáférés beszerzése egy Azure Stack Development Kit (ASDK) vagy Azure Stack hub integrált rendszerpéldányhoz, a következővel:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-117">Obtain user access to an Azure Stack Development Kit (ASDK) or Azure Stack Hub integrated system instance, with:</span></span>
  - <span data-ttu-id="cf6ac-118">A [Azure stack hub erőforrás-szolgáltatón telepített Azure app Service](/azure-stack/operator/azure-stack-app-service-overview.md) .</span><span class="sxs-lookup"><span data-stu-id="cf6ac-118">The [Azure App Service on Azure Stack Hub resource provider](/azure-stack/operator/azure-stack-app-service-overview.md) installed.</span></span> <span data-ttu-id="cf6ac-119">Az Azure Stack hub-példányhoz a kezelőhöz való hozzáférésre van szükség, vagy a telepítéshez a rendszergazdával kell dolgoznia.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-119">You need operator access to your Azure Stack Hub instance, or work with your administrator to install.</span></span>
  - <span data-ttu-id="cf6ac-120">App Service-és tárolási kvótát biztosító ajánlat előfizetése.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-120">A subscription to an offer that provides App Service and Storage quota.</span></span> <span data-ttu-id="cf6ac-121">Ajánlat létrehozásához operátori hozzáférésre van szükség.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-121">You need operator access to create an offer.</span></span>
- <span data-ttu-id="cf6ac-122">Azure-előfizetéshez való hozzáférés beszerzése.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-122">Obtain access to an Azure subscription.</span></span>
  - <span data-ttu-id="cf6ac-123">Ha nem rendelkezik Azure-előfizetéssel, regisztráljon az [ingyenes próbaverziós fiókra](https://azure.microsoft.com/free/) , mielőtt megkezdené.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-123">If you don't have an Azure subscription, sign up for a [free trial account](https://azure.microsoft.com/free/) before you begin.</span></span>
- <span data-ttu-id="cf6ac-124">Hozzon létre két egyszerű szolgáltatásnevet a címtárban:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-124">Create two service principals in your directory:</span></span>
  - <span data-ttu-id="cf6ac-125">Egy beállítás az Azure-erőforrásokkal való használatra, az Azure-előfizetések hatókörében való hozzáféréssel.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-125">One set up for use with Azure resources, with access at the Azure subscription scope.</span></span>
  - <span data-ttu-id="cf6ac-126">Az egyik beállítása Azure Stack hub-erőforrásokkal való használatra, a Azure Stack hub-előfizetés hatókörében való hozzáféréssel.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-126">One set up for use with Azure Stack Hub resources, with access at the Azure Stack Hub subscription scope.</span></span>
  - <span data-ttu-id="cf6ac-127">Az egyszerű szolgáltatások létrehozásával és a hozzáférés engedélyezésével kapcsolatos további tudnivalókért lásd: [alkalmazás-identitás használata az erőforrásokhoz való hozzáféréshez](/azure-stack/operator/azure-stack-create-service-principals.md).</span><span class="sxs-lookup"><span data-stu-id="cf6ac-127">To learn more about creating service principals and authorizing access, see [Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md).</span></span> <span data-ttu-id="cf6ac-128">Ha szívesebben szeretné használni az Azure CLI-t, tekintse meg [Az Azure-szolgáltatás létrehozása az Azure CLI-vel](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true)című témakört.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-128">If you prefer to use Azure CLI, see [Create an Azure service principal with Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).</span></span>
- <span data-ttu-id="cf6ac-129">Azure Cognitive Services üzembe helyezése az Azure-ban vagy Azure Stack hub-ban.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-129">Deploy Azure Cognitive Services in Azure or Azure Stack Hub.</span></span>
  - <span data-ttu-id="cf6ac-130">Először is tájékozódjon [Cognitive Servicesról](https://azure.microsoft.com/services/cognitive-services/).</span><span class="sxs-lookup"><span data-stu-id="cf6ac-130">First, [learn more about Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).</span></span>
  - <span data-ttu-id="cf6ac-131">Ezután látogasson el az [Azure Cognitive Services üzembe helyezése Azure stack hubhoz](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) az Azure stack hub-beli Cognitive Services üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-131">Then visit [Deploy Azure Cognitive Services to Azure Stack Hub](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) to deploy Cognitive Services on Azure Stack Hub.</span></span> <span data-ttu-id="cf6ac-132">Először regisztrálnia kell az előzetes verzióhoz való hozzáféréshez.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-132">You first need to sign up for access to the preview.</span></span>
- <span data-ttu-id="cf6ac-133">A nem konfigurált Azure Custom Vision AI fejlesztői csomag klónozása vagy letöltése.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-133">Clone or download an unconfigured Azure Custom Vision AI Dev Kit.</span></span> <span data-ttu-id="cf6ac-134">Részletekért tekintse meg a következő témakört: [AI fejlesztői készlet](https://azure.github.io/Vision-AI-DevKit-Pages/).</span><span class="sxs-lookup"><span data-stu-id="cf6ac-134">For details, see the [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).</span></span>
- <span data-ttu-id="cf6ac-135">Regisztráljon Power BI fiókra.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-135">Sign up for a Power BI account.</span></span>
- <span data-ttu-id="cf6ac-136">Az Azure Cognitive Services Face API az előfizetési kulcs és a végpont URL-címe.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-136">An Azure Cognitive Services Face API subscription key and endpoint URL.</span></span> <span data-ttu-id="cf6ac-137">A [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) ingyenes próbaverzióját is elérheti.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-137">You can get both with the [Try Cognitive Services](https://azure.microsoft.com/try/cognitive-services/?api=face-api) free trial.</span></span> <span data-ttu-id="cf6ac-138">Vagy kövesse a [Cognitive Services fiók létrehozása](/azure/cognitive-services/cognitive-services-apis-create-account)című témakör utasításait.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-138">Or, follow the instructions in [Create a Cognitive Services account](/azure/cognitive-services/cognitive-services-apis-create-account).</span></span>
- <span data-ttu-id="cf6ac-139">Telepítse a következő fejlesztői erőforrásokat:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-139">Install the following development resources:</span></span>
  - [<span data-ttu-id="cf6ac-140">Azure CLI 2.0</span><span class="sxs-lookup"><span data-stu-id="cf6ac-140">Azure CLI 2.0</span></span>](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [<span data-ttu-id="cf6ac-141">Docker CE</span><span class="sxs-lookup"><span data-stu-id="cf6ac-141">Docker CE</span></span>](https://hub.docker.com/search/?type=edition&offering=community)
  - <span data-ttu-id="cf6ac-142">[Porter](https://porter.sh/).</span><span class="sxs-lookup"><span data-stu-id="cf6ac-142">[Porter](https://porter.sh/).</span></span> <span data-ttu-id="cf6ac-143">A Porter használatával a felhőalapú alkalmazásokat üzembe helyezheti az Ön számára biztosított CNAB-csomagbeli jegyzékfájlok használatával.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-143">You use Porter to deploy cloud apps using CNAB bundle manifests that are provided for you.</span></span>
  - [<span data-ttu-id="cf6ac-144">Visual Studio Code</span><span class="sxs-lookup"><span data-stu-id="cf6ac-144">Visual Studio Code</span></span>](https://code.visualstudio.com/)
  - [<span data-ttu-id="cf6ac-145">Azure IoT-eszközök a Visual Studio Code-hoz</span><span class="sxs-lookup"><span data-stu-id="cf6ac-145">Azure IoT Tools for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [<span data-ttu-id="cf6ac-146">Python-bővítmény a Visual Studio Code-hoz</span><span class="sxs-lookup"><span data-stu-id="cf6ac-146">Python extension for Visual Studio Code</span></span>](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [<span data-ttu-id="cf6ac-147">Python</span><span class="sxs-lookup"><span data-stu-id="cf6ac-147">Python</span></span>](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a><span data-ttu-id="cf6ac-148">A hibrid felhőalapú alkalmazás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="cf6ac-148">Deploy the hybrid cloud app</span></span>

<span data-ttu-id="cf6ac-149">Először a Porter CLI használatával hozzon létre egy hitelesítőadat-készletet, majd telepítse a Cloud alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-149">First you use the Porter CLI to generate a credential set, then deploy the cloud app.</span></span>  

1. <span data-ttu-id="cf6ac-150">A megoldás mintájának klónozása vagy letöltése innen: https://github.com/azure-samples/azure-intelligent-edge-patterns .</span><span class="sxs-lookup"><span data-stu-id="cf6ac-150">Clone or download the solution sample code from https://github.com/azure-samples/azure-intelligent-edge-patterns.</span></span> 

1. <span data-ttu-id="cf6ac-151">A Porter hitelesítő adatokat állít elő, amelyek automatizálják az alkalmazás üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-151">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="cf6ac-152">A hitelesítő adatok generálására szolgáló parancs futtatása előtt győződjön meg arról, hogy a következők állnak rendelkezésre:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-152">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="cf6ac-153">Az Azure-erőforrások elérésére szolgáló egyszerű szolgáltatás, beleértve az egyszerű szolgáltatásnév, a kulcs és a bérlői DNS-t.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-153">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="cf6ac-154">Az Azure-előfizetéshez tartozó előfizetés-azonosító.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-154">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="cf6ac-155">Egy egyszerű szolgáltatásnév, amely az Azure Stack hub erőforrásainak elérésére szolgál, beleértve az egyszerű szolgáltatás AZONOSÍTÓját, kulcsát és a bérlői DNS-t.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-155">A service principal for accessing Azure Stack Hub resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="cf6ac-156">Az Azure Stack hub-előfizetés előfizetés-azonosítója.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-156">The subscription ID for your Azure Stack Hub subscription.</span></span>
    - <span data-ttu-id="cf6ac-157">Az Azure Cognitive Services Face API kulcs-és erőforrás-végpont URL-címét.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-157">Your Azure Cognitive Services Face API key and resource endpoint URL.</span></span>

1. <span data-ttu-id="cf6ac-158">Futtassa a Porter hitelesítő adatok létrehozásának folyamatát, és kövesse az utasításokat:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-158">Run the Porter credential generation process and follow the prompts:</span></span>

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. <span data-ttu-id="cf6ac-159">A Porternek a futtatandó paramétereket is meg kell adnia.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-159">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="cf6ac-160">Hozzon létre egy paraméter szöveges fájlt, és adja meg a következő név/érték párokat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-160">Create a parameter text file and enter the following name/value pairs.</span></span> <span data-ttu-id="cf6ac-161">Ha segítségre van szüksége a szükséges értékekkel kapcsolatban, kérdezze meg Azure Stack hub-rendszergazdáját.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-161">Ask your Azure Stack Hub administrator if you need assistance with any of the required values.</span></span>

   > [!NOTE] 
   > <span data-ttu-id="cf6ac-162">Az `resource suffix` érték segítségével biztosítható, hogy az üzembe helyezés erőforrásai egyedi névvel rendelkezzenek az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-162">The `resource suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="cf6ac-163">Nem lehet hosszabb 8 karakternél, és csak egyedi betűkből és számokból állhat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-163">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   <span data-ttu-id="cf6ac-164">Mentse a szövegfájlt, és jegyezze fel az elérési útját.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-164">Save the text file and make a note of its path.</span></span>

1. <span data-ttu-id="cf6ac-165">Most már készen áll a hibrid felhőalapú alkalmazás üzembe helyezésére a Porter használatával.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-165">You're now ready to deploy the hybrid cloud app using Porter.</span></span> <span data-ttu-id="cf6ac-166">Futtassa az install parancsot, és figyelje meg, hogy az erőforrások üzembe helyezése az Azure-ban és Azure Stack hub:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-166">Run the install command and watch as resources are deployed to Azure and Azure Stack Hub:</span></span>

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. <span data-ttu-id="cf6ac-167">Az üzembe helyezés befejezése után jegyezze fel a következő értékeket:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-167">Once deployment is complete, make note of the following values:</span></span>
    - <span data-ttu-id="cf6ac-168">A kamera csatlakoztatási karakterlánca.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-168">The camera's connection string.</span></span>
    - <span data-ttu-id="cf6ac-169">A rendszerkép Storage-fiókjának kapcsolatainak karakterlánca.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-169">The image storage account connection string.</span></span>
    - <span data-ttu-id="cf6ac-170">Az erőforráscsoport neve.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-170">The resource group names.</span></span>

## <a name="prepare-the-custom-vision-ai-devkit"></a><span data-ttu-id="cf6ac-171">A Custom Vision AI-fejlesztői készlet előkészítése</span><span class="sxs-lookup"><span data-stu-id="cf6ac-171">Prepare the Custom Vision AI DevKit</span></span>

<span data-ttu-id="cf6ac-172">Ezután állítsa be a Custom Vision AI fejlesztői csomagot, ahogyan az a [jövőkép AI fejlesztői készlet](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/)rövid útmutatójában látható.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-172">Next, set up the Custom Vision AI Dev Kit as shown in the [Vision AI DevKit quickstart](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/).</span></span> <span data-ttu-id="cf6ac-173">A kamerát úgy is beállíthatja és tesztelheti, hogy az előző lépésben megadott kapcsolatok sztringjét használja.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-173">You also set up and test your camera, using the connection string provided in the previous step.</span></span>

## <a name="deploy-the-camera-app"></a><span data-ttu-id="cf6ac-174">A kamera alkalmazás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="cf6ac-174">Deploy the camera app</span></span>

<span data-ttu-id="cf6ac-175">A Porter CLI használatával hozzon létre egy hitelesítőadat-készletet, majd telepítse a kamera alkalmazást.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-175">Use the Porter CLI to generate a credential set, then deploy the camera app.</span></span>

1. <span data-ttu-id="cf6ac-176">A Porter hitelesítő adatokat állít elő, amelyek automatizálják az alkalmazás üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-176">Porter will generate a set of credentials that will automate deployment of the app.</span></span> <span data-ttu-id="cf6ac-177">A hitelesítő adatok generálására szolgáló parancs futtatása előtt győződjön meg arról, hogy a következők állnak rendelkezésre:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-177">Before running the credential generation command, be sure to have the following available:</span></span>

    - <span data-ttu-id="cf6ac-178">Az Azure-erőforrások elérésére szolgáló egyszerű szolgáltatás, beleértve az egyszerű szolgáltatásnév, a kulcs és a bérlői DNS-t.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-178">A service principal for accessing Azure resources, including the service principal ID, key, and tenant DNS.</span></span>
    - <span data-ttu-id="cf6ac-179">Az Azure-előfizetéshez tartozó előfizetés-azonosító.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-179">The subscription ID for your Azure subscription.</span></span>
    - <span data-ttu-id="cf6ac-180">A felhőalapú alkalmazás üzembe helyezésekor megadott rendszerkép-tárolási fiók kapcsolódási karakterlánca.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-180">The image storage account connection string provided when you deployed the cloud app.</span></span>

1. <span data-ttu-id="cf6ac-181">Futtassa a Porter hitelesítő adatok létrehozásának folyamatát, és kövesse az utasításokat:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-181">Run the Porter credential generation process and follow the prompts:</span></span>

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. <span data-ttu-id="cf6ac-182">A Porternek a futtatandó paramétereket is meg kell adnia.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-182">Porter also requires a set of parameters to run.</span></span> <span data-ttu-id="cf6ac-183">Hozzon létre egy paraméter szöveges fájlt, és adja meg a következő szöveget.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-183">Create a parameter text file and enter the following text.</span></span> <span data-ttu-id="cf6ac-184">Ha nem ismeri a szükséges értékeket, kérdezze meg Azure Stack hub-rendszergazdáját.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-184">Ask your Azure Stack Hub administrator if you don't know some of the required values.</span></span>

    > [!NOTE]
    > <span data-ttu-id="cf6ac-185">Az `deployment suffix` érték segítségével biztosítható, hogy az üzembe helyezés erőforrásai egyedi névvel rendelkezzenek az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-185">The `deployment suffix` value is used to ensure that your deployment's resources have unique names across Azure.</span></span> <span data-ttu-id="cf6ac-186">Nem lehet hosszabb 8 karakternél, és csak egyedi betűkből és számokból állhat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-186">It must be a unique string of letters and numbers, no longer than 8 characters.</span></span>

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    <span data-ttu-id="cf6ac-187">Mentse a szövegfájlt, és jegyezze fel az elérési útját.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-187">Save the text file and make a note of its path.</span></span>

4. <span data-ttu-id="cf6ac-188">Most már készen áll a kamera alkalmazás üzembe helyezésére a Porter használatával.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-188">You're now ready to deploy the camera app using Porter.</span></span> <span data-ttu-id="cf6ac-189">Futtassa az install parancsot, és figyelje meg, hogy a IoT Edge központi telepítés létrejött.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-189">Run the install command and watch as the IoT Edge deployment is created.</span></span>

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. <span data-ttu-id="cf6ac-190">Ellenőrizze, hogy a kamera üzembe helyezése befejeződött-e a kamera hírcsatornájának megtekintésével `https://<camera-ip>:3000/` , ahol `<camara-ip>` a a kamera IP-címe.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-190">Verify that the camera's deployment is complete by viewing the camera feed at `https://<camera-ip>:3000/`, where `<camara-ip>` is the camera IP address.</span></span> <span data-ttu-id="cf6ac-191">Ez a lépés akár 10 percet is igénybe vehet.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-191">This step may take up to 10 minutes.</span></span>

## <a name="configure-azure-stream-analytics"></a><span data-ttu-id="cf6ac-192">Azure Stream Analytics konfigurálása</span><span class="sxs-lookup"><span data-stu-id="cf6ac-192">Configure Azure Stream Analytics</span></span>

<span data-ttu-id="cf6ac-193">Most, hogy az adatok a kamerából Azure Stream Analyticsnek, manuálisan engedélyeznie kell, hogy kommunikáljon a Power BIval.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-193">Now that data is flowing to Azure Stream Analytics from the camera, we need to manually authorize it to communicate with Power BI.</span></span>

1. <span data-ttu-id="cf6ac-194">A Azure Portal nyissa meg az **összes erőforrást** és a *Process-lépés hangja \[ yoursuffix \]* -feladatot.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-194">From the Azure portal, open **All Resources**, and the *process-footfall\[yoursuffix\]* job.</span></span>

2. <span data-ttu-id="cf6ac-195">A Stream Analytics-feladat panel **Feladattopológia** szakaszában válassza a **Kimenetek** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-195">In the **Job Topology** section of the Stream Analytics job pane, select the **Outputs** option.</span></span>

3. <span data-ttu-id="cf6ac-196">Válassza ki a **forgalom** kimeneti kimenetét.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-196">Select the **traffic-output** output sink.</span></span>

4. <span data-ttu-id="cf6ac-197">Válassza az **Engedélyezés megújítása** lehetőséget, majd jelentkezzen be Power bi-fiókjába.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-197">Select **Renew authorization** and sign in to your Power BI account.</span></span>
  
    ![Engedélyezési kérés megújítása Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. <span data-ttu-id="cf6ac-199">Mentse a kimeneti beállításokat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-199">Save the output settings.</span></span>

6. <span data-ttu-id="cf6ac-200">Lépjen az **Áttekintés** panelre, és válassza az **Indítás** lehetőséget az adatok Power BIba való küldésének megkezdéséhez.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-200">Go to the **Overview** pane and select **Start** to start sending data to Power BI.</span></span>

7. <span data-ttu-id="cf6ac-201">Válassza a **Most** beállítást a feladatkimenet kezdési idejeként, majd válassza az **Indítás** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-201">Select **Now** for job output start time and select **Start**.</span></span> <span data-ttu-id="cf6ac-202">A feladat állapotát az értesítési sávban tekintheti meg.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-202">You can view the job status in the notification bar.</span></span>

## <a name="create-a-power-bi-dashboard"></a><span data-ttu-id="cf6ac-203">Power BI irányítópult létrehozása</span><span class="sxs-lookup"><span data-stu-id="cf6ac-203">Create a Power BI Dashboard</span></span>

1. <span data-ttu-id="cf6ac-204">Ha a feladatot sikeresen elvégezte, lépjen [Power bi](https://powerbi.com/) , és jelentkezzen be munkahelyi vagy iskolai fiókjával.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-204">Once the job succeeds, go to [Power BI](https://powerbi.com/) and sign in with your work or school account.</span></span> <span data-ttu-id="cf6ac-205">Ha az Stream Analytics feladatok lekérdezése az eredményeket jeleníti meg, a létrehozott *lépés hangja-adatkészlet* az **adatkészletek** lapon található.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-205">If the Stream Analytics job query is outputting results, the *footfall-dataset* dataset you created exists under the **Datasets** tab.</span></span>

2. <span data-ttu-id="cf6ac-206">A Power BI munkaterületen válassza a **+ Létrehozás** elemet a *lépés hangja Analysis* nevű új irányítópult létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-206">From your Power BI workspace, select **+ Create** to create a new dashboard named *Footfall Analysis.*</span></span>

3. <span data-ttu-id="cf6ac-207">Válassza a **Csempe felvétele** lehetőséget az ablak tetején.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-207">At the top of the window, select **Add tile**.</span></span> <span data-ttu-id="cf6ac-208">Ezután válassza az **Egyedi folyamatos átviteli adatok**, majd a **Tovább** lehetőséget.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-208">Then select **Custom Streaming Data** and **Next**.</span></span> <span data-ttu-id="cf6ac-209">Válassza ki a **lépés hangja** az **adatkészletek alatt.**</span><span class="sxs-lookup"><span data-stu-id="cf6ac-209">Choose the **footfall-dataset** under **Your Datasets**.</span></span> <span data-ttu-id="cf6ac-210">Válassza a **kártya** lehetőséget a **vizualizáció típusa** legördülő listából, és adja hozzá a **kor** **mezőt a mezőkhöz**.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-210">Select **Card** from the **Visualization type** dropdown, and add **age** to **Fields**.</span></span> <span data-ttu-id="cf6ac-211">Kattintson a **Tovább** gombra, és nevezze el a csempét, majd kattintson az **Alkalmaz** elemre a csempe létrehozásához.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-211">Select **Next** to enter a name for the tile, and then select **Apply** to create the tile.</span></span>

4. <span data-ttu-id="cf6ac-212">Igény szerint további mezőket és kártyákat is hozzáadhat.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-212">You can add additional fields and cards as desired.</span></span>

## <a name="test-your-solution"></a><span data-ttu-id="cf6ac-213">A megoldás tesztelése</span><span class="sxs-lookup"><span data-stu-id="cf6ac-213">Test Your Solution</span></span>

<span data-ttu-id="cf6ac-214">Figyelje meg, hogy a Power BIban létrehozott kártyákban lévő adatváltozások Hogyan változnak meg a kamera előtt.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-214">Observe how the data in the cards you created in Power BI changes as different people walk in front of the camera.</span></span> <span data-ttu-id="cf6ac-215">A következtetések a rögzítés után akár 20 másodpercet is igénybe vehetnek.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-215">Inferences may take up to 20 seconds to appear once recorded.</span></span>

## <a name="remove-your-solution"></a><span data-ttu-id="cf6ac-216">Megoldás eltávolítása</span><span class="sxs-lookup"><span data-stu-id="cf6ac-216">Remove Your Solution</span></span>

<span data-ttu-id="cf6ac-217">Ha el szeretné távolítani a megoldást, futtassa a következő parancsokat a Porter használatával, ugyanazokkal a paraméterekkel, amelyeket az üzembe helyezéshez hozott létre:</span><span class="sxs-lookup"><span data-stu-id="cf6ac-217">If you'd like to remove your solution, run the following commands using Porter, using the same parameter files that you created for deployment:</span></span>

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a><span data-ttu-id="cf6ac-218">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="cf6ac-218">Next steps</span></span>

- <span data-ttu-id="cf6ac-219">További információ a [hibrid alkalmazások kialakításával kapcsolatos szempontokról](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="cf6ac-219">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="cf6ac-220">Tekintse át és javasolja [a githubon a minta kódjának](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis)tökéletesítését.</span><span class="sxs-lookup"><span data-stu-id="cf6ac-220">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).</span></span>
