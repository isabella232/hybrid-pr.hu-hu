---
title: Több felhőre kiterjedő méretezési minta az Azure Stack központban
description: Megtudhatja, hogyan hozhat létre méretezhető, többfelhős alkalmazást az Azure-ban és Azure Stack hub-on.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: a830f96e97c347cbbcc09a1b17f4836ecb6eb3e6
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911131"
---
# <a name="cross-cloud-scaling-pattern"></a><span data-ttu-id="0b30b-103">Több felhőre kiterjedő skálázási minta</span><span class="sxs-lookup"><span data-stu-id="0b30b-103">Cross-cloud scaling pattern</span></span>

<span data-ttu-id="0b30b-104">Erőforrások automatikus hozzáadása egy meglévő alkalmazáshoz a terhelés növekedésének kielégítése érdekében.</span><span class="sxs-lookup"><span data-stu-id="0b30b-104">Automatically add resources to an existing app to accommodate an increase in load.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="0b30b-105">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="0b30b-105">Context and problem</span></span>

<span data-ttu-id="0b30b-106">Az alkalmazás nem növelheti a kapacitást, hogy megfeleljen az igény váratlan növekedésének.</span><span class="sxs-lookup"><span data-stu-id="0b30b-106">Your app can't increase capacity to meet unexpected increases in demand.</span></span> <span data-ttu-id="0b30b-107">A skálázhatóság hiánya azt eredményezi, hogy a felhasználók nem érik el az alkalmazást a csúcsérték-használat idején.</span><span class="sxs-lookup"><span data-stu-id="0b30b-107">This lack of scalability results in users not reaching the app during peak usage times.</span></span> <span data-ttu-id="0b30b-108">Az alkalmazás meghatározott számú felhasználót tud kiszolgálni.</span><span class="sxs-lookup"><span data-stu-id="0b30b-108">The app can service a fixed number of users.</span></span>

<span data-ttu-id="0b30b-109">A globális vállalatok biztonságos, megbízható és elérhető felhőalapú alkalmazásokat igényelnek.</span><span class="sxs-lookup"><span data-stu-id="0b30b-109">Global enterprises require secure, reliable, and available cloud-based apps.</span></span> <span data-ttu-id="0b30b-110">A találkozó az igény szerint növekszik, és a megfelelő infrastruktúra használatával támogatja az igényeket.</span><span class="sxs-lookup"><span data-stu-id="0b30b-110">Meeting increases in demand and using the right infrastructure to support that demand is critical.</span></span> <span data-ttu-id="0b30b-111">A vállalatok az üzleti adatbiztonsággal, a tárolással és a valós idejű rendelkezésre állással kapcsolatos költségek és karbantartás egyensúlyával küzdenek.</span><span class="sxs-lookup"><span data-stu-id="0b30b-111">Businesses struggle to balance costs and maintenance with business data security, storage, and real-time availability.</span></span>

<span data-ttu-id="0b30b-112">Előfordulhat, hogy nem tudja futtatni az alkalmazást a nyilvános felhőben.</span><span class="sxs-lookup"><span data-stu-id="0b30b-112">You may not be able to run your app in the public cloud.</span></span> <span data-ttu-id="0b30b-113">Előfordulhat azonban, hogy nem valósítható meg gazdaságosan a vállalat számára, hogy fenntartsa a helyszíni környezetben szükséges kapacitást, hogy kezelni tudja az alkalmazás iránti igényt.</span><span class="sxs-lookup"><span data-stu-id="0b30b-113">However, it may not be economically feasible for the business to maintain the capacity required in their on-premises environment to handle spikes in demand for the app.</span></span> <span data-ttu-id="0b30b-114">Ezzel a mintával a nyilvános felhő rugalmasságát a helyszíni megoldással is használhatja.</span><span class="sxs-lookup"><span data-stu-id="0b30b-114">With this pattern, you can use the elasticity of the public cloud with your on-premises solution.</span></span>

## <a name="solution"></a><span data-ttu-id="0b30b-115">Megoldás</span><span class="sxs-lookup"><span data-stu-id="0b30b-115">Solution</span></span>

<span data-ttu-id="0b30b-116">A többfelhős méretezési minta kibővít egy helyi felhőben található alkalmazást, amely nyilvános Felhőbeli erőforrásokkal rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="0b30b-116">The cross-cloud scaling pattern extends an app located in a local cloud with public cloud resources.</span></span> <span data-ttu-id="0b30b-117">A mintát az igény növelésével vagy csökkentésével, illetve a felhőben lévő erőforrások hozzáadásával vagy eltávolításával indítja el a rendszer.</span><span class="sxs-lookup"><span data-stu-id="0b30b-117">The pattern is triggered by an increase or decrease in demand, and respectively adds or removes resources in the cloud.</span></span> <span data-ttu-id="0b30b-118">Ezek az erőforrások redundanciát, gyors rendelkezésre állást és földrajzilag megfelelő útválasztást biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="0b30b-118">These resources provide redundancy, rapid availability, and geo-compliant routing.</span></span>

![Több felhőre kiterjedő skálázási minta](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> <span data-ttu-id="0b30b-120">Ez a minta csak az alkalmazás állapot nélküli összetevőire vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="0b30b-120">This pattern applies only to stateless components of your app.</span></span>

## <a name="components"></a><span data-ttu-id="0b30b-121">Összetevők</span><span class="sxs-lookup"><span data-stu-id="0b30b-121">Components</span></span>

<span data-ttu-id="0b30b-122">A több felhőre kiterjedő skálázási minta a következő összetevőkből áll.</span><span class="sxs-lookup"><span data-stu-id="0b30b-122">The cross-cloud scaling pattern consists of the following components.</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="0b30b-123">A felhőn kívül</span><span class="sxs-lookup"><span data-stu-id="0b30b-123">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="0b30b-124">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="0b30b-124">Traffic Manager</span></span>

<span data-ttu-id="0b30b-125">A diagramon ez a nyilvános felhőn kívül esik, de a helyi adatközpontban és a nyilvános felhőben is képesnek kell lennie a forgalom koordinálására.</span><span class="sxs-lookup"><span data-stu-id="0b30b-125">In the diagram, this is located outside of the public cloud group, but it would need to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="0b30b-126">A Balancer magas rendelkezésre állást biztosít az alkalmazás számára a végpontok figyelésével, valamint szükség esetén a feladatátvételi újraelosztás biztosításával.</span><span class="sxs-lookup"><span data-stu-id="0b30b-126">The balancer delivers high availability for app by monitoring endpoints and providing failover redistribution when required.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="0b30b-127">Tartománynévrendszer (DNS)</span><span class="sxs-lookup"><span data-stu-id="0b30b-127">Domain Name System (DNS)</span></span>

<span data-ttu-id="0b30b-128">A tartománynévrendszer vagy a DNS a webhely vagy szolgáltatás nevének az IP-címére való fordítására (vagy feloldására) felelős.</span><span class="sxs-lookup"><span data-stu-id="0b30b-128">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="cloud"></a><span data-ttu-id="0b30b-129">Felhő</span><span class="sxs-lookup"><span data-stu-id="0b30b-129">Cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="0b30b-130">Üzemeltetett Build kiszolgáló</span><span class="sxs-lookup"><span data-stu-id="0b30b-130">Hosted build server</span></span>

<span data-ttu-id="0b30b-131">Környezet a build-folyamat üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="0b30b-131">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="0b30b-132">Alkalmazás-erőforrások</span><span class="sxs-lookup"><span data-stu-id="0b30b-132">App resources</span></span>

<span data-ttu-id="0b30b-133">Az alkalmazás erőforrásainak képesnek kell lenniük a méretezésre és a vertikális felskálázásra, például a virtuálisgép-méretezési csoportokra és a tárolók méretezésére.</span><span class="sxs-lookup"><span data-stu-id="0b30b-133">The app resources need to be able to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="0b30b-134">Egyéni tartománynév</span><span class="sxs-lookup"><span data-stu-id="0b30b-134">Custom domain name</span></span>

<span data-ttu-id="0b30b-135">Használjon egyéni tartománynevet a glob útválasztási kérelmekhez.</span><span class="sxs-lookup"><span data-stu-id="0b30b-135">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="0b30b-136">Nyilvános IP-címek</span><span class="sxs-lookup"><span data-stu-id="0b30b-136">Public IP addresses</span></span>

<span data-ttu-id="0b30b-137">A nyilvános IP-címek használatával a bejövő forgalom átirányítható a Traffic Managerrel a nyilvános Cloud app Resources végpontra.</span><span class="sxs-lookup"><span data-stu-id="0b30b-137">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-cloud"></a><span data-ttu-id="0b30b-138">Helyi felhő</span><span class="sxs-lookup"><span data-stu-id="0b30b-138">Local cloud</span></span>

#### <a name="hosted-build-server"></a><span data-ttu-id="0b30b-139">Üzemeltetett Build kiszolgáló</span><span class="sxs-lookup"><span data-stu-id="0b30b-139">Hosted build server</span></span>

<span data-ttu-id="0b30b-140">Környezet a build-folyamat üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="0b30b-140">An environment for hosting your build pipeline.</span></span>

#### <a name="app-resources"></a><span data-ttu-id="0b30b-141">Alkalmazás-erőforrások</span><span class="sxs-lookup"><span data-stu-id="0b30b-141">App resources</span></span>

<span data-ttu-id="0b30b-142">Az alkalmazás erőforrásainak képesnek kell lenniük a méretezésre és a vertikális felskálázásra, például a virtuálisgép-méretezési csoportokra és a tárolók tárolására.</span><span class="sxs-lookup"><span data-stu-id="0b30b-142">The app resources need the ability to scale in and scale out, like virtual machine scale sets and Containers.</span></span>

#### <a name="custom-domain-name"></a><span data-ttu-id="0b30b-143">Egyéni tartománynév</span><span class="sxs-lookup"><span data-stu-id="0b30b-143">Custom domain name</span></span>

<span data-ttu-id="0b30b-144">Használjon egyéni tartománynevet a glob útválasztási kérelmekhez.</span><span class="sxs-lookup"><span data-stu-id="0b30b-144">Use a custom domain name for routing requests glob.</span></span>

#### <a name="public-ip-addresses"></a><span data-ttu-id="0b30b-145">Nyilvános IP-címek</span><span class="sxs-lookup"><span data-stu-id="0b30b-145">Public IP addresses</span></span>

<span data-ttu-id="0b30b-146">A nyilvános IP-címek használatával a bejövő forgalom átirányítható a Traffic Managerrel a nyilvános Cloud app Resources végpontra.</span><span class="sxs-lookup"><span data-stu-id="0b30b-146">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="0b30b-147">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="0b30b-147">Issues and considerations</span></span>

<span data-ttu-id="0b30b-148">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="0b30b-148">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="0b30b-149">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="0b30b-149">Scalability</span></span>

<span data-ttu-id="0b30b-150">A többfelhős méretezés fő összetevője az igény szerinti skálázás lehetősége.</span><span class="sxs-lookup"><span data-stu-id="0b30b-150">The key component of cross-cloud scaling is the ability to deliver on-demand scaling.</span></span> <span data-ttu-id="0b30b-151">A skálázásnak a nyilvános és a helyi felhőalapú infrastruktúra között kell történnie, és igény szerint egységes, megbízható szolgáltatást kell biztosítania.</span><span class="sxs-lookup"><span data-stu-id="0b30b-151">Scaling must happen between public and local cloud infrastructure and provide a consistent, reliable service per the demand.</span></span>

### <a name="availability"></a><span data-ttu-id="0b30b-152">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="0b30b-152">Availability</span></span>

<span data-ttu-id="0b30b-153">Győződjön meg arról, hogy a helyileg telepített alkalmazások magas rendelkezésre állásra vannak konfigurálva a helyszíni hardverkonfiguráció és a szoftverek központi telepítése révén.</span><span class="sxs-lookup"><span data-stu-id="0b30b-153">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="0b30b-154">Kezelhetőség</span><span class="sxs-lookup"><span data-stu-id="0b30b-154">Manageability</span></span>

<span data-ttu-id="0b30b-155">A Felhőbeli mintázat zökkenőmentes felügyeletet és ismerős felületet biztosít a környezetek között.</span><span class="sxs-lookup"><span data-stu-id="0b30b-155">The cross-cloud pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="0b30b-156">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="0b30b-156">When to use this pattern</span></span>

<span data-ttu-id="0b30b-157">Használja a következő mintát a következő helyzetekben:</span><span class="sxs-lookup"><span data-stu-id="0b30b-157">Use this pattern:</span></span>

- <span data-ttu-id="0b30b-158">Ha az alkalmazás kapacitását nem várt igényekkel vagy igény szerinti rendszeres igényekkel kell bővíteni.</span><span class="sxs-lookup"><span data-stu-id="0b30b-158">When you need to increase your app capacity with unexpected demands or periodic demands in demand.</span></span>
- <span data-ttu-id="0b30b-159">Ha nem szeretne olyan erőforrásba befektetni, amelyet csak a csúcsok során használunk.</span><span class="sxs-lookup"><span data-stu-id="0b30b-159">When you don't want to invest in resources that will only be used during peaks.</span></span> <span data-ttu-id="0b30b-160">A ténylegesen használt funkciókért kell fizetnie.</span><span class="sxs-lookup"><span data-stu-id="0b30b-160">Pay for what you use.</span></span>

<span data-ttu-id="0b30b-161">Ez a minta nem ajánlott a következő esetekben:</span><span class="sxs-lookup"><span data-stu-id="0b30b-161">This pattern isn't recommended when:</span></span>

- <span data-ttu-id="0b30b-162">A megoldáshoz az interneten keresztül csatlakozó felhasználók szükségesek.</span><span class="sxs-lookup"><span data-stu-id="0b30b-162">Your solution requires users connecting over the internet.</span></span>
- <span data-ttu-id="0b30b-163">Vállalata rendelkezik olyan helyi előírásokkal, amelyek megkövetelik, hogy a kezdeményező kapcsolódás egy helyszíni hívásból történjen.</span><span class="sxs-lookup"><span data-stu-id="0b30b-163">Your business has local regulations that require that the originating connection to come from an onsite call.</span></span>
- <span data-ttu-id="0b30b-164">A hálózat olyan rendszeres szűk keresztmetszeteket tapasztal, amelyek korlátozzák a skálázás teljesítményét.</span><span class="sxs-lookup"><span data-stu-id="0b30b-164">Your network experiences regular bottlenecks that would restrict the performance of the scaling.</span></span>
- <span data-ttu-id="0b30b-165">A környezet nem kapcsolódik az internethez, és nem éri el a nyilvános felhőt.</span><span class="sxs-lookup"><span data-stu-id="0b30b-165">Your environment is disconnected from the internet and can't reach the public cloud.</span></span>

## <a name="next-steps"></a><span data-ttu-id="0b30b-166">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="0b30b-166">Next steps</span></span>

<span data-ttu-id="0b30b-167">További információ a cikkben bemutatott témakörökről:</span><span class="sxs-lookup"><span data-stu-id="0b30b-167">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="0b30b-168">A DNS-alapú forgalom terheléselosztó működésével kapcsolatos további információkért tekintse meg az [Azure Traffic Manager áttekintését](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="0b30b-168">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="0b30b-169">Az ajánlott eljárásokról és a további kérdésekre adott válaszokért lásd a [hibrid alkalmazások kialakításával kapcsolatos szempontokat](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="0b30b-169">See [Hybrid application design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="0b30b-170">A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="0b30b-170">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="0b30b-171">Ha készen áll a megoldás tesztelésére, folytassa a [többfelhős méretezési megoldás telepítési útmutatóját](solution-deployment-guide-cross-cloud-scaling.md).</span><span class="sxs-lookup"><span data-stu-id="0b30b-171">When you're ready to test the solution example, continue with the [Cross-cloud scaling solution deployment guide](solution-deployment-guide-cross-cloud-scaling.md).</span></span> <span data-ttu-id="0b30b-172">A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="0b30b-172">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="0b30b-173">Megtudhatja, hogyan hozhat létre többfelhős megoldást egy olyan manuálisan aktivált folyamat megadására, amely egy Azure Stack hub által üzemeltetett webalkalmazásból egy Azure-ban üzemeltetett webalkalmazásra vált át.</span><span class="sxs-lookup"><span data-stu-id="0b30b-173">You learn how to create a cross-cloud solution to provide a manually triggered process for switching from an Azure Stack Hub hosted web app to an Azure hosted web app.</span></span> <span data-ttu-id="0b30b-174">Azt is megtudhatja, hogyan használhatja az automatikus skálázást a Traffic Manager használatával, rugalmas és méretezhető felhőalapú segédprogramot biztosítva a betöltés alatt.</span><span class="sxs-lookup"><span data-stu-id="0b30b-174">You also learn how to use autoscaling via traffic manager, ensuring flexible and scalable cloud utility when under load.</span></span>
