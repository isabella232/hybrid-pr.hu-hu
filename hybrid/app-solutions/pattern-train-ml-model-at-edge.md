---
title: A Machine learning-modell tanítása a peremhálózati mintán
description: Ismerje meg, hogyan végezheti el a gépi tanulási modellek betanítását az Azure-ral és Azure Stack hub-val.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: da1012cc8847f221de6bb540ba4e191e43920f41
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911102"
---
# <a name="train-machine-learning-model-at-the-edge-pattern"></a><span data-ttu-id="92c97-103">A Machine learning-modell tanítása a peremhálózati mintán</span><span class="sxs-lookup"><span data-stu-id="92c97-103">Train machine learning model at the edge pattern</span></span>

<span data-ttu-id="92c97-104">Hordozható gépi tanulási (ML) modellek készítése csak a helyszínen található adatokból.</span><span class="sxs-lookup"><span data-stu-id="92c97-104">Generate portable machine learning (ML) models from data that only exists on-premises.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="92c97-105">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="92c97-105">Context and problem</span></span>

<span data-ttu-id="92c97-106">Számos szervezet szeretné feloldani a helyszíni vagy örökölt adatokból származó elemzéseket az adatszakértők által megértett eszközök használatával.</span><span class="sxs-lookup"><span data-stu-id="92c97-106">Many organizations would like to unlock insights from their on-premises or legacy data using tools that their data scientists understand.</span></span> <span data-ttu-id="92c97-107">A [Azure Machine learning](/azure/machine-learning/) a felhőben natív eszközöket biztosít az ml-és mély tanulási modellek betanításához, finomhangolásához és üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="92c97-107">[Azure Machine Learning](/azure/machine-learning/) provides cloud-native tooling to train, tune, and deploy ML and deep learning models.</span></span>  

<span data-ttu-id="92c97-108">Bizonyos adatmennyiségek azonban túl nagy mennyiségű küldést biztosítanak a felhőnek, vagy szabályozási okokból nem küldhetők el a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="92c97-108">However, some data is too large send to the cloud or can't be sent to the cloud for regulatory reasons.</span></span> <span data-ttu-id="92c97-109">Ennek a mintának a használatával az adatszakértők a Azure Machine Learning használatával betanítják a modelleket a helyszíni adatok és a számítások használatával.</span><span class="sxs-lookup"><span data-stu-id="92c97-109">Using this pattern, data scientists can use Azure Machine Learning to train models using on-premises data and compute.</span></span>

## <a name="solution"></a><span data-ttu-id="92c97-110">Megoldás</span><span class="sxs-lookup"><span data-stu-id="92c97-110">Solution</span></span>

<span data-ttu-id="92c97-111">Az Edge-minta betanítása Azure Stack hub-on futó virtuális gépet (VM) használ.</span><span class="sxs-lookup"><span data-stu-id="92c97-111">The training at the edge pattern uses a virtual machine (VM) running on Azure Stack Hub.</span></span> <span data-ttu-id="92c97-112">A virtuális gép számítási célként van regisztrálva az Azure ML-ben, és csak a helyszínen elérhető adatokhoz férhet hozzá.</span><span class="sxs-lookup"><span data-stu-id="92c97-112">The VM is registered as a compute target in Azure ML, letting it access data only available on-premises.</span></span> <span data-ttu-id="92c97-113">Ebben az esetben a rendszer a Azure Stack hub blob Storage-tárolójában tárolja az adattárakat.</span><span class="sxs-lookup"><span data-stu-id="92c97-113">In this case, the data is stored in Azure Stack Hub's blob storage.</span></span>

<span data-ttu-id="92c97-114">A modell betanítása után regisztrálva van az Azure ML-ben, a tárolóban, és hozzáadva egy Azure Container Registryhoz az üzembe helyezéshez.</span><span class="sxs-lookup"><span data-stu-id="92c97-114">Once the model is trained, it's registered with Azure ML, containerized, and added to an Azure Container Registry for deployment.</span></span> <span data-ttu-id="92c97-115">A minta ezen iterációjában a Azure Stack hub betanítási virtuális gépnek elérhetőnek kell lennie a nyilvános interneten.</span><span class="sxs-lookup"><span data-stu-id="92c97-115">For this iteration of the pattern, the Azure Stack Hub training VM must be reachable over the public internet.</span></span>

<span data-ttu-id="92c97-116">[![ML modell betanítása a peremhálózati architektúrában](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="92c97-116">[![Train ML model at the edge architecture](media/pattern-train-ml-model-at-edge/solution-architecture.png)](media/pattern-train-ml-model-at-edge/solution-architecture.png)</span></span>

<span data-ttu-id="92c97-117">A minta működése:</span><span class="sxs-lookup"><span data-stu-id="92c97-117">Here's how the pattern works:</span></span>

1. <span data-ttu-id="92c97-118">Az Azure Stack hub virtuális gép üzembe helyezése és számítási célként való regisztrálása az Azure ML-ben történik.</span><span class="sxs-lookup"><span data-stu-id="92c97-118">The Azure Stack Hub VM is deployed and registered as a compute target with Azure ML.</span></span>
2. <span data-ttu-id="92c97-119">Az Azure ML-ben olyan kísérlet jön létre, amely a Azure Stack hub virtuális gépet számítási célként használja.</span><span class="sxs-lookup"><span data-stu-id="92c97-119">An experiment is created in Azure ML that uses the Azure Stack Hub VM as a compute target.</span></span>
3. <span data-ttu-id="92c97-120">A modell betanítása után regisztrálva és tárolóban van.</span><span class="sxs-lookup"><span data-stu-id="92c97-120">Once the model is trained, it's registered and containerized.</span></span>
4. <span data-ttu-id="92c97-121">A modell mostantól a helyszínen vagy a felhőben is üzembe helyezhető.</span><span class="sxs-lookup"><span data-stu-id="92c97-121">The model can now be deployed to locations that are either on-premises or in the cloud.</span></span>

## <a name="components"></a><span data-ttu-id="92c97-122">Összetevők</span><span class="sxs-lookup"><span data-stu-id="92c97-122">Components</span></span>

<span data-ttu-id="92c97-123">Ez a megoldás a következő összetevőket használja:</span><span class="sxs-lookup"><span data-stu-id="92c97-123">This solution uses the following components:</span></span>

| <span data-ttu-id="92c97-124">Réteg</span><span class="sxs-lookup"><span data-stu-id="92c97-124">Layer</span></span> | <span data-ttu-id="92c97-125">Összetevő</span><span class="sxs-lookup"><span data-stu-id="92c97-125">Component</span></span> | <span data-ttu-id="92c97-126">Description</span><span class="sxs-lookup"><span data-stu-id="92c97-126">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="92c97-127">Azure</span><span class="sxs-lookup"><span data-stu-id="92c97-127">Azure</span></span> | <span data-ttu-id="92c97-128">Azure Machine Learning</span><span class="sxs-lookup"><span data-stu-id="92c97-128">Azure Machine Learning</span></span> | <span data-ttu-id="92c97-129">[Azure Machine learning](/azure/machine-learning/) összehangolja a ml modell betanítását.</span><span class="sxs-lookup"><span data-stu-id="92c97-129">[Azure Machine Learning](/azure/machine-learning/) orchestrates the training of the ML model.</span></span> |
| | <span data-ttu-id="92c97-130">Azure Container Registry</span><span class="sxs-lookup"><span data-stu-id="92c97-130">Azure Container Registry</span></span> | <span data-ttu-id="92c97-131">Az Azure ML becsomagolja a modellt egy tárolóba, és tárolja [Azure Container Registry](/azure/container-registry/) a telepítéshez.</span><span class="sxs-lookup"><span data-stu-id="92c97-131">Azure ML packages the model into a container and stores it in an [Azure Container Registry](/azure/container-registry/) for deployment.</span></span>|
| <span data-ttu-id="92c97-132">Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="92c97-132">Azure Stack Hub</span></span> | <span data-ttu-id="92c97-133">App Service</span><span class="sxs-lookup"><span data-stu-id="92c97-133">App Service</span></span> | <span data-ttu-id="92c97-134">A [Azure stack hub és a app Service](/azure-stack/operator/azure-stack-app-service-overview) biztosítja az összetevők alapját az Edge-ben.</span><span class="sxs-lookup"><span data-stu-id="92c97-134">[Azure Stack Hub with App Service](/azure-stack/operator/azure-stack-app-service-overview) provides the base for the components at the edge.</span></span> |
| | <span data-ttu-id="92c97-135">Compute</span><span class="sxs-lookup"><span data-stu-id="92c97-135">Compute</span></span> | <span data-ttu-id="92c97-136">Az Ubuntut és a Docker-t futtató Azure Stack hub-beli virtuális gép a ML modell betanítására szolgál.</span><span class="sxs-lookup"><span data-stu-id="92c97-136">An Azure Stack Hub VM running Ubuntu with Docker is used to train the ML model.</span></span> |
| | <span data-ttu-id="92c97-137">Storage</span><span class="sxs-lookup"><span data-stu-id="92c97-137">Storage</span></span> | <span data-ttu-id="92c97-138">A magánhálózati adattárak Azure Stack hub blob Storage-ban is tárolhatók.</span><span class="sxs-lookup"><span data-stu-id="92c97-138">Private data can be hosted in Azure Stack Hub blob storage.</span></span> |

## <a name="issues-and-considerations"></a><span data-ttu-id="92c97-139">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="92c97-139">Issues and considerations</span></span>

<span data-ttu-id="92c97-140">A megoldás megvalósításának eldöntése során vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="92c97-140">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="92c97-141">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="92c97-141">Scalability</span></span>

<span data-ttu-id="92c97-142">A megoldás méretezésének engedélyezéséhez létre kell hoznia egy megfelelő méretű virtuális gépet a Azure Stack hub-on a betanításhoz.</span><span class="sxs-lookup"><span data-stu-id="92c97-142">To enable this solution to scale, you'll need to create an appropriately sized VM on Azure Stack Hub for training.</span></span>

### <a name="availability"></a><span data-ttu-id="92c97-143">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="92c97-143">Availability</span></span>

<span data-ttu-id="92c97-144">Győződjön meg arról, hogy a betanítási szkriptek és a Azure Stack hub virtuális gép hozzáfér a betanításhoz használt helyszíni adatszolgáltatásokhoz.</span><span class="sxs-lookup"><span data-stu-id="92c97-144">Ensure that the training scripts and Azure Stack Hub VM have access to the on-premises data used for training.</span></span>

### <a name="manageability"></a><span data-ttu-id="92c97-145">Kezelhetőség</span><span class="sxs-lookup"><span data-stu-id="92c97-145">Manageability</span></span>

<span data-ttu-id="92c97-146">Győződjön meg arról, hogy a modellek és kísérletek megfelelően vannak regisztrálva, verziószámozással és címkézve, hogy elkerülje a modell üzembe helyezése során felmerülő zavart.</span><span class="sxs-lookup"><span data-stu-id="92c97-146">Ensure that models and experiments are appropriately registered, versioned, and tagged to avoid confusion during model deployment.</span></span>

### <a name="security"></a><span data-ttu-id="92c97-147">Biztonság</span><span class="sxs-lookup"><span data-stu-id="92c97-147">Security</span></span>

<span data-ttu-id="92c97-148">Ez a minta lehetővé teszi, hogy az Azure ML hozzáférhessen a lehetséges bizalmas adatokhoz a helyszínen.</span><span class="sxs-lookup"><span data-stu-id="92c97-148">This pattern lets Azure ML access possible sensitive data on-premises.</span></span> <span data-ttu-id="92c97-149">Győződjön meg arról, hogy az SSH-ba Azure Stack hub virtuális géphez használt fiók erős jelszóval és betanítási szkriptekkel nem őrzi meg az adatok felhőbe való feltöltését</span><span class="sxs-lookup"><span data-stu-id="92c97-149">Ensure the account used to SSH into Azure Stack Hub VM has a strong password and training scripts don't preserve or upload data to the cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="92c97-150">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="92c97-150">Next steps</span></span>

<span data-ttu-id="92c97-151">További információ a cikkben bemutatott témakörökről:</span><span class="sxs-lookup"><span data-stu-id="92c97-151">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="92c97-152">A ML és a kapcsolódó témakörök áttekintéséhez tekintse meg a [Azure Machine learning dokumentációját](/azure/machine-learning) .</span><span class="sxs-lookup"><span data-stu-id="92c97-152">See the [Azure Machine Learning documentation](/azure/machine-learning) for an overview of ML and related topics.</span></span>
- <span data-ttu-id="92c97-153">Tekintse meg a [Azure Container Registry](/azure/container-registry/) a lemezképek létrehozásához, tárolásához és kezeléséhez a tárolók üzembe helyezéséhez című témakört.</span><span class="sxs-lookup"><span data-stu-id="92c97-153">See [Azure Container Registry](/azure/container-registry/) to learn how to build, store, and manage images for container deployments.</span></span>
- <span data-ttu-id="92c97-154">Az erőforrás-szolgáltatóval és a telepítésével kapcsolatos további tudnivalókért tekintse [meg a app Service on Azure stack hub](/azure-stack/operator/azure-stack-app-service-overview) című témakört.</span><span class="sxs-lookup"><span data-stu-id="92c97-154">Refer to [App Service on Azure Stack Hub](/azure-stack/operator/azure-stack-app-service-overview) to learn more about the resource provider and how to deploy.</span></span>
- <span data-ttu-id="92c97-155">Tekintse meg a [hibrid alkalmazások kialakításával kapcsolatos szempontokat](overview-app-design-considerations.md) az ajánlott eljárásokkal kapcsolatos további információkért és a további kérdések megválaszolásához.</span><span class="sxs-lookup"><span data-stu-id="92c97-155">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get any additional questions answered.</span></span>
- <span data-ttu-id="92c97-156">A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="92c97-156">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="92c97-157">Ha készen áll a megoldás tesztelésére, folytassa az [éles üzembe helyezési útmutatóban található "ml" betanítási modellel](https://aka.ms/edgetrainingdeploy).</span><span class="sxs-lookup"><span data-stu-id="92c97-157">When you're ready to test the solution example, continue with the [Train ML model at the edge deployment guide](https://aka.ms/edgetrainingdeploy).</span></span> <span data-ttu-id="92c97-158">A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="92c97-158">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>
