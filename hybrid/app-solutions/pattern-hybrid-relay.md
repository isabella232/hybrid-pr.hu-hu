---
title: Hibrid továbbítási minta az Azure-ban és Azure Stack hub-ban
description: Az Azure-ban és Azure Stack hub-ban található hibrid továbbítási minta használatával csatlakozhat a tűzfalak által védett peremhálózati erőforrásokhoz.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 03b20a20a04f620c977fb20e1ea26f5982e42721
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911145"
---
# <a name="hybrid-relay-pattern"></a><span data-ttu-id="928a4-103">Hibrid továbbítási minta</span><span class="sxs-lookup"><span data-stu-id="928a4-103">Hybrid relay pattern</span></span>

<span data-ttu-id="928a4-104">Megtudhatja, hogyan csatlakozhat a tűzfalak által védett peremhálózati erőforrásokhoz vagy eszközökhöz a hibrid továbbítási minta és a Azure Relay használatával.</span><span class="sxs-lookup"><span data-stu-id="928a4-104">Learn how to connect to edge resources or devices protected by firewalls using the hybrid relay pattern and Azure Relay.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="928a4-105">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="928a4-105">Context and problem</span></span>

<span data-ttu-id="928a4-106">Az Edge-eszközök gyakran vállalati tűzfal vagy NAT-eszköz mögött találhatók.</span><span class="sxs-lookup"><span data-stu-id="928a4-106">Edge devices are often behind a corporate firewall or NAT device.</span></span> <span data-ttu-id="928a4-107">Bár biztonságosak, előfordulhat, hogy nem tudnak kommunikálni a nyilvános felhővel vagy a peremhálózati eszközökkel más vállalati hálózatokon.</span><span class="sxs-lookup"><span data-stu-id="928a4-107">Although they're secure, they may be unable to communicate with the public cloud or edge devices on other corporate networks.</span></span> <span data-ttu-id="928a4-108">A nyilvános felhőben lévő felhasználók számára bizonyos portok és funkciók biztonságos módon történő közzététele szükséges lehet.</span><span class="sxs-lookup"><span data-stu-id="928a4-108">It may be necessary to expose certain ports and functionality to users in the public cloud in a secure manner.</span></span>

## <a name="solution"></a><span data-ttu-id="928a4-109">Megoldás</span><span class="sxs-lookup"><span data-stu-id="928a4-109">Solution</span></span>

<span data-ttu-id="928a4-110">A hibrid továbbítási minta Azure Relay használatával hoz létre egy WebSockets-alagutat két végpont között, amelyek nem tudnak közvetlenül kommunikálni.</span><span class="sxs-lookup"><span data-stu-id="928a4-110">The hybrid relay pattern uses Azure Relay to establish a WebSockets tunnel between two endpoints that can't directly communicate.</span></span> <span data-ttu-id="928a4-111">Azok az eszközök, amelyek nem a helyszínen vannak, de csatlakozniuk kell egy helyszíni végponthoz, csatlakozni fognak a nyilvános felhőben lévő végponthoz.</span><span class="sxs-lookup"><span data-stu-id="928a4-111">Devices that aren't on-premises but need to connect to an on-premises endpoint will connect to an endpoint in the public cloud.</span></span> <span data-ttu-id="928a4-112">Ez a végpont átirányítja a forgalmat az előre meghatározott útvonalakon egy biztonságos csatornán keresztül.</span><span class="sxs-lookup"><span data-stu-id="928a4-112">This endpoint will redirect the traffic on predefined routes over a secure channel.</span></span> <span data-ttu-id="928a4-113">A helyszíni környezetben található végpont fogadja a forgalmat, és átirányítja a megfelelő helyre.</span><span class="sxs-lookup"><span data-stu-id="928a4-113">An endpoint inside the on-premises environment receives the traffic and routes it to the correct destination.</span></span>

![hibrid továbbító minta megoldás architektúrája](media/pattern-hybrid-relay/solution-architecture.png)

<span data-ttu-id="928a4-115">A hibrid továbbítási minta működése:</span><span class="sxs-lookup"><span data-stu-id="928a4-115">Here's how the hybrid relay pattern works:</span></span>

1. <span data-ttu-id="928a4-116">Az eszköz egy előre meghatározott porton csatlakozik a virtuális géphez (VM) az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="928a4-116">A device connects to the virtual machine (VM) in Azure, on a predefined port.</span></span>
2. <span data-ttu-id="928a4-117">A rendszer továbbítja a forgalmat a Azure Relay az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="928a4-117">Traffic is forwarded to the Azure Relay in Azure.</span></span>
3. <span data-ttu-id="928a4-118">Azure Stack hub-beli virtuális gép, amely már régóta élt kapcsolattal rendelkezik a Azure Relay, fogadja a forgalmat, és továbbítja azt a célhelyre.</span><span class="sxs-lookup"><span data-stu-id="928a4-118">The VM on Azure Stack Hub, which has already established a long-lived connection to the Azure Relay, receives the traffic and forwards it on to the destination.</span></span>
4. <span data-ttu-id="928a4-119">A helyszíni szolgáltatás vagy végpont dolgozza fel a kérelmet.</span><span class="sxs-lookup"><span data-stu-id="928a4-119">The on-premises service or endpoint processes the request.</span></span>

## <a name="components"></a><span data-ttu-id="928a4-120">Összetevők</span><span class="sxs-lookup"><span data-stu-id="928a4-120">Components</span></span>

<span data-ttu-id="928a4-121">Ez a megoldás a következő összetevőket használja:</span><span class="sxs-lookup"><span data-stu-id="928a4-121">This solution uses the following components:</span></span>

| <span data-ttu-id="928a4-122">Réteg</span><span class="sxs-lookup"><span data-stu-id="928a4-122">Layer</span></span> | <span data-ttu-id="928a4-123">Összetevő</span><span class="sxs-lookup"><span data-stu-id="928a4-123">Component</span></span> | <span data-ttu-id="928a4-124">Description</span><span class="sxs-lookup"><span data-stu-id="928a4-124">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="928a4-125">Azure</span><span class="sxs-lookup"><span data-stu-id="928a4-125">Azure</span></span> | <span data-ttu-id="928a4-126">Azure VM</span><span class="sxs-lookup"><span data-stu-id="928a4-126">Azure VM</span></span> | <span data-ttu-id="928a4-127">Az Azure-beli virtuális gépek nyilvánosan elérhető végpontot biztosítanak a helyszíni erőforráshoz.</span><span class="sxs-lookup"><span data-stu-id="928a4-127">An Azure VM provides a publicly accessible endpoint for the on-premises resource.</span></span> |
| | <span data-ttu-id="928a4-128">Azure Relay</span><span class="sxs-lookup"><span data-stu-id="928a4-128">Azure Relay</span></span> | <span data-ttu-id="928a4-129">Az [Azure Relay](/azure/azure-relay/) biztosítja az alagút és az Azure-beli virtuális gép és a Azure stack hub virtuális gép közötti kapcsolat fenntartásához szükséges infrastruktúrát.</span><span class="sxs-lookup"><span data-stu-id="928a4-129">An [Azure Relay](/azure/azure-relay/) provides the infrastructure for maintaining the tunnel and connection between the Azure VM and Azure Stack Hub VM.</span></span>|
| <span data-ttu-id="928a4-130">Azure Stack hub</span><span class="sxs-lookup"><span data-stu-id="928a4-130">Azure Stack Hub</span></span> | <span data-ttu-id="928a4-131">Compute</span><span class="sxs-lookup"><span data-stu-id="928a4-131">Compute</span></span> | <span data-ttu-id="928a4-132">Az Azure Stack hub-alapú virtuális gép a hibrid továbbító alagút kiszolgálóoldali részét biztosítja.</span><span class="sxs-lookup"><span data-stu-id="928a4-132">An Azure Stack Hub VM provides the server-side of the Hybrid Relay tunnel.</span></span> |
| | <span data-ttu-id="928a4-133">Storage</span><span class="sxs-lookup"><span data-stu-id="928a4-133">Storage</span></span> | <span data-ttu-id="928a4-134">Azure Stack hubhoz üzembe helyezett AK-motor-fürt méretezhető, rugalmas motort biztosít az Face API tároló futtatásához.</span><span class="sxs-lookup"><span data-stu-id="928a4-134">The AKS engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span>|

## <a name="issues-and-considerations"></a><span data-ttu-id="928a4-135">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="928a4-135">Issues and considerations</span></span>

<span data-ttu-id="928a4-136">A megoldás megvalósításának eldöntése során vegye figyelembe a következő szempontokat:</span><span class="sxs-lookup"><span data-stu-id="928a4-136">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="928a4-137">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="928a4-137">Scalability</span></span>

<span data-ttu-id="928a4-138">Ez a minta csak a 1:1-es port-hozzárendelések használatát teszi lehetővé az ügyfélen és a kiszolgálón.</span><span class="sxs-lookup"><span data-stu-id="928a4-138">This pattern only allows for 1:1 port mappings on the client and server.</span></span> <span data-ttu-id="928a4-139">Ha például az 80-as port az Azure-végpont egyik szolgáltatásának bújtatása, akkor nem használható másik szolgáltatáshoz.</span><span class="sxs-lookup"><span data-stu-id="928a4-139">For example, if port 80 is tunneled for one service on the Azure endpoint, it can't be used for another service.</span></span> <span data-ttu-id="928a4-140">A portok megfeleltetéseit ennek megfelelően kell tervezni.</span><span class="sxs-lookup"><span data-stu-id="928a4-140">Port mappings should be planned accordingly.</span></span> <span data-ttu-id="928a4-141">A Azure Relay és a virtuális gépeket megfelelő méretezéssel kell ellátni a forgalom kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="928a4-141">The Azure Relay and VMs should be appropriately scaled to handle traffic.</span></span>

### <a name="availability"></a><span data-ttu-id="928a4-142">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="928a4-142">Availability</span></span>

<span data-ttu-id="928a4-143">Ezek az alagutak és kapcsolatok nem redundánsak.</span><span class="sxs-lookup"><span data-stu-id="928a4-143">These tunnels and connections aren't redundant.</span></span> <span data-ttu-id="928a4-144">A magas rendelkezésre állás biztosítása érdekében érdemes lehet végrehajtani a hibakódot.</span><span class="sxs-lookup"><span data-stu-id="928a4-144">To ensure high-availability, you may want to implement error checking code.</span></span> <span data-ttu-id="928a4-145">Egy másik lehetőség, hogy a terheléselosztó mögött Azure Relay csatlakoztatott virtuális gépek készletére van szükség.</span><span class="sxs-lookup"><span data-stu-id="928a4-145">Another option is to have a pool of Azure Relay-connected VMs behind a load balancer.</span></span>

### <a name="manageability"></a><span data-ttu-id="928a4-146">Kezelhetőség</span><span class="sxs-lookup"><span data-stu-id="928a4-146">Manageability</span></span>

<span data-ttu-id="928a4-147">Ez a megoldás számos eszközre és helyszínre képes, így nem lehet megszerezni a megoldást.</span><span class="sxs-lookup"><span data-stu-id="928a4-147">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="928a4-148">Az Azure IoT szolgáltatásai automatikusan online állapotba helyezhetik az új telephelyeket és eszközöket, és naprakészen tarthatják őket.</span><span class="sxs-lookup"><span data-stu-id="928a4-148">Azure's IoT services can automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="928a4-149">Biztonság</span><span class="sxs-lookup"><span data-stu-id="928a4-149">Security</span></span>

<span data-ttu-id="928a4-150">Ez a minta lehetővé teszi, hogy korlátlan hozzáférést biztosítson egy belső eszköz portjához a szélétől.</span><span class="sxs-lookup"><span data-stu-id="928a4-150">This pattern as shown allows for unfettered access to a port on an internal device from the edge.</span></span> <span data-ttu-id="928a4-151">Érdemes lehet hitelesítési mechanizmust hozzáadni a szolgáltatáshoz a belső eszközön vagy a hibrid továbbító végpontja előtt.</span><span class="sxs-lookup"><span data-stu-id="928a4-151">Consider adding an authentication mechanism to the service on the internal device, or in front of the hybrid relay endpoint.</span></span>

## <a name="next-steps"></a><span data-ttu-id="928a4-152">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="928a4-152">Next steps</span></span>

<span data-ttu-id="928a4-153">További információ a cikkben bemutatott témakörökről:</span><span class="sxs-lookup"><span data-stu-id="928a4-153">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="928a4-154">Ez a minta Azure Relay használ.</span><span class="sxs-lookup"><span data-stu-id="928a4-154">This pattern uses Azure Relay.</span></span> <span data-ttu-id="928a4-155">További információ: [Azure Relay dokumentáció](/azure/azure-relay/).</span><span class="sxs-lookup"><span data-stu-id="928a4-155">For more information, see the [Azure Relay documentation](/azure/azure-relay/).</span></span>
- <span data-ttu-id="928a4-156">Az ajánlott eljárásokkal kapcsolatos további információkért és a további kérdésekre adott válaszokért tekintse meg a [hibrid alkalmazások kialakításával kapcsolatos szempontokat](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="928a4-156">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and get answers to any additional questions.</span></span>
- <span data-ttu-id="928a4-157">A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="928a4-157">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="928a4-158">Ha készen áll a megoldás tesztelésére, folytassa a [hibrid továbbító megoldás telepítési útmutatóját](https://aka.ms/hybridrelaydeployment).</span><span class="sxs-lookup"><span data-stu-id="928a4-158">When you're ready to test the solution example, continue with the [Hybrid relay solution deployment guide](https://aka.ms/hybridrelaydeployment).</span></span> <span data-ttu-id="928a4-159">A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="928a4-159">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>