---
title: Földrajzilag elosztott alkalmazás mintája Azure Stack központban
description: Ismerje meg az Azure-t és Azure Stack hubot használó, földrajzilag elosztott alkalmazási mintát az intelligens peremhálózat számára.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 1f6243927390c7a520c2607c722664b2d31fc07f
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910879"
---
# <a name="geo-distributed-app-pattern"></a><span data-ttu-id="da15b-103">Földrajzilag elosztott alkalmazás mintája</span><span class="sxs-lookup"><span data-stu-id="da15b-103">Geo-distributed app pattern</span></span>

<span data-ttu-id="da15b-104">Megtudhatja, hogyan biztosíthat alkalmazás-végpontokat több régióban, és hogyan irányíthatja át a felhasználói forgalmat a hely és a megfelelőségi igények alapján.</span><span class="sxs-lookup"><span data-stu-id="da15b-104">Learn how to provide app endpoints across multiple regions and route user traffic based on location and compliance needs.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="da15b-105">Kontextus és probléma</span><span class="sxs-lookup"><span data-stu-id="da15b-105">Context and problem</span></span>

<span data-ttu-id="da15b-106">Azok a szervezetek, amelyek széles körű földrajzi területtel rendelkeznek, az adathozzáférés biztonságos és pontos elosztására és engedélyezésére törekednek, miközben a biztonság, a megfelelőség és a teljesítmény szükséges szintje felhasználónként, helyen és eszközön a határokon belül.</span><span class="sxs-lookup"><span data-stu-id="da15b-106">Organizations with wide-reaching geographies strive to securely and accurately distribute and enable access to data while ensuring required levels of security, compliance and performance per user, location, and device across borders.</span></span>

## <a name="solution"></a><span data-ttu-id="da15b-107">Megoldás</span><span class="sxs-lookup"><span data-stu-id="da15b-107">Solution</span></span>

<span data-ttu-id="da15b-108">A Azure Stack hub földrajzi forgalmának útválasztási mintája vagy földrajzilag elosztott alkalmazások lehetővé teszi, hogy a forgalom a különböző metrikák alapján meghatározott végpontokra legyen irányítva.</span><span class="sxs-lookup"><span data-stu-id="da15b-108">The Azure Stack Hub geographic traffic routing pattern, or geo-distributed apps, lets traffic be directed to specific endpoints based on various metrics.</span></span> <span data-ttu-id="da15b-109">A földrajzi alapú útválasztási és végponti konfigurációval rendelkező Traffic Managerek a regionális követelmények, a vállalati és a nemzetközi szabályozás, valamint az adatszükségletek alapján irányítják át a végpontokra irányuló forgalmat.</span><span class="sxs-lookup"><span data-stu-id="da15b-109">Creating a Traffic Manager with geographic-based routing and endpoint configuration routes traffic to endpoints based on regional requirements, corporate and international regulation, and data needs.</span></span>

![Földrajzilag elosztott minta](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a><span data-ttu-id="da15b-111">Összetevők</span><span class="sxs-lookup"><span data-stu-id="da15b-111">Components</span></span>

### <a name="outside-the-cloud"></a><span data-ttu-id="da15b-112">A felhőn kívül</span><span class="sxs-lookup"><span data-stu-id="da15b-112">Outside the cloud</span></span>

#### <a name="traffic-manager"></a><span data-ttu-id="da15b-113">Traffic Manager</span><span class="sxs-lookup"><span data-stu-id="da15b-113">Traffic Manager</span></span>

<span data-ttu-id="da15b-114">A diagramon Traffic Manager a nyilvános felhőn kívül található, de a helyi adatközpontban és a nyilvános felhőben is képesnek kell lennie a forgalom koordinálására.</span><span class="sxs-lookup"><span data-stu-id="da15b-114">In the diagram, Traffic Manager is located outside of the public cloud, but it needs to able to coordinate traffic in both the local datacenter and the public cloud.</span></span> <span data-ttu-id="da15b-115">A Balancer a földrajzi helyszínekre irányítja a forgalmat.</span><span class="sxs-lookup"><span data-stu-id="da15b-115">The balancer routes traffic to geographical locations.</span></span>

#### <a name="domain-name-system-dns"></a><span data-ttu-id="da15b-116">Tartománynévrendszer (DNS)</span><span class="sxs-lookup"><span data-stu-id="da15b-116">Domain Name System (DNS)</span></span>

<span data-ttu-id="da15b-117">A tartománynévrendszer vagy a DNS a webhely vagy szolgáltatás nevének az IP-címére való fordítására (vagy feloldására) felelős.</span><span class="sxs-lookup"><span data-stu-id="da15b-117">The Domain Name System, or DNS, is responsible for translating (or resolving) a website or service name to its IP address.</span></span>

### <a name="public-cloud"></a><span data-ttu-id="da15b-118">Nyilvános felhő</span><span class="sxs-lookup"><span data-stu-id="da15b-118">Public cloud</span></span>

#### <a name="cloud-endpoint"></a><span data-ttu-id="da15b-119">Felhőbeli végpont</span><span class="sxs-lookup"><span data-stu-id="da15b-119">Cloud Endpoint</span></span>

<span data-ttu-id="da15b-120">A nyilvános IP-címek használatával a bejövő forgalom átirányítható a Traffic Managerrel a nyilvános Cloud app Resources végpontra.</span><span class="sxs-lookup"><span data-stu-id="da15b-120">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>  

### <a name="local-clouds"></a><span data-ttu-id="da15b-121">Helyi felhők</span><span class="sxs-lookup"><span data-stu-id="da15b-121">Local clouds</span></span>

#### <a name="local-endpoint"></a><span data-ttu-id="da15b-122">Helyi végpont</span><span class="sxs-lookup"><span data-stu-id="da15b-122">Local endpoint</span></span>

<span data-ttu-id="da15b-123">A nyilvános IP-címek használatával a bejövő forgalom átirányítható a Traffic Managerrel a nyilvános Cloud app Resources végpontra.</span><span class="sxs-lookup"><span data-stu-id="da15b-123">Public IP addresses are used to route the incoming traffic through traffic manager to the public cloud app resources endpoint.</span></span>

## <a name="issues-and-considerations"></a><span data-ttu-id="da15b-124">Problémák és megfontolandó szempontok</span><span class="sxs-lookup"><span data-stu-id="da15b-124">Issues and considerations</span></span>

<span data-ttu-id="da15b-125">A minta megvalósítása során az alábbi pontokat vegye figyelembe:</span><span class="sxs-lookup"><span data-stu-id="da15b-125">Consider the following points when deciding how to implement this pattern:</span></span>

### <a name="scalability"></a><span data-ttu-id="da15b-126">Méretezhetőség</span><span class="sxs-lookup"><span data-stu-id="da15b-126">Scalability</span></span>

<span data-ttu-id="da15b-127">A minta a földrajzi forgalom útválasztását kezeli, és nem méretezhető, hogy megfeleljen a forgalom növekedésének.</span><span class="sxs-lookup"><span data-stu-id="da15b-127">The pattern handles geographical traffic routing rather than scaling to meet increases in traffic.</span></span> <span data-ttu-id="da15b-128">Ezt a mintát azonban más Azure-és helyszíni megoldásokkal is kombinálhatja.</span><span class="sxs-lookup"><span data-stu-id="da15b-128">However, you can combine this pattern with other Azure and on-premises solutions.</span></span> <span data-ttu-id="da15b-129">Ez a minta például a több felhőre kiterjedő skálázási mintával használható.</span><span class="sxs-lookup"><span data-stu-id="da15b-129">For example, this pattern can be used with the cross-cloud scaling Pattern.</span></span>

### <a name="availability"></a><span data-ttu-id="da15b-130">Rendelkezésre állás</span><span class="sxs-lookup"><span data-stu-id="da15b-130">Availability</span></span>

<span data-ttu-id="da15b-131">Győződjön meg arról, hogy a helyileg telepített alkalmazások magas rendelkezésre állásra vannak konfigurálva a helyszíni hardverkonfiguráció és a szoftverek központi telepítése révén.</span><span class="sxs-lookup"><span data-stu-id="da15b-131">Ensure locally deployed apps are configured for high-availability through on-premises hardware configuration and software deployment.</span></span>

### <a name="manageability"></a><span data-ttu-id="da15b-132">Kezelhetőség</span><span class="sxs-lookup"><span data-stu-id="da15b-132">Manageability</span></span>

<span data-ttu-id="da15b-133">A minta zökkenőmentes felügyeletet és ismerős felületet biztosít a környezetek között.</span><span class="sxs-lookup"><span data-stu-id="da15b-133">The pattern ensures seamless management and familiar interface between environments.</span></span>

## <a name="when-to-use-this-pattern"></a><span data-ttu-id="da15b-134">Mikor érdemes ezt a mintát használni?</span><span class="sxs-lookup"><span data-stu-id="da15b-134">When to use this pattern</span></span>

- <span data-ttu-id="da15b-135">A szervezetem olyan nemzetközi ágakat tartalmaz, amelyek egyéni regionális biztonsági és terjesztési házirendeket igényelnek.</span><span class="sxs-lookup"><span data-stu-id="da15b-135">My organization has international branches requiring custom regional security and distribution policies.</span></span>
- <span data-ttu-id="da15b-136">A szervezeti irodák mindegyike lekéri az alkalmazottakat, az üzleti és a létesítményi tevékenységeket, amelyek helyi szabályozással és időzónával kapcsolatos jelentési tevékenységet igényelnek.</span><span class="sxs-lookup"><span data-stu-id="da15b-136">Each of my organization's offices pulls employee, business, and facility data, requiring reporting activity per local regulations and time zone.</span></span>
- <span data-ttu-id="da15b-137">A nagy léptékű követelmények az alkalmazások horizontális felskálázásával hajthatók végre, több alkalmazás üzembe helyezése egyetlen régióban és régiókban a szélsőséges terhelési követelmények kezelése érdekében.</span><span class="sxs-lookup"><span data-stu-id="da15b-137">High-scale requirements can be met by horizontally scaling out apps, with multiple app deployments being made within a single region and across regions to handle extreme load requirements.</span></span>
- <span data-ttu-id="da15b-138">Az alkalmazások számára elérhetőnek kell lennie, és az ügyfelek kéréseire is reagálni kell, még az egyrégiós kimaradások esetében is.</span><span class="sxs-lookup"><span data-stu-id="da15b-138">The apps must be highly available and responsive to client requests even in single-region outages.</span></span>

## <a name="next-steps"></a><span data-ttu-id="da15b-139">További lépések</span><span class="sxs-lookup"><span data-stu-id="da15b-139">Next steps</span></span>

<span data-ttu-id="da15b-140">További információ a cikkben bemutatott témakörökről:</span><span class="sxs-lookup"><span data-stu-id="da15b-140">To learn more about topics introduced in this article:</span></span>

- <span data-ttu-id="da15b-141">A DNS-alapú forgalom terheléselosztó működésével kapcsolatos további információkért tekintse meg az [Azure Traffic Manager áttekintését](/azure/traffic-manager/traffic-manager-overview) .</span><span class="sxs-lookup"><span data-stu-id="da15b-141">See the [Azure Traffic Manager overview](/azure/traffic-manager/traffic-manager-overview) to learn more about how this DNS-based traffic load balancer works.</span></span>
- <span data-ttu-id="da15b-142">Az ajánlott eljárásokról és a további kérdésekre adott válaszokért lásd a [hibrid alkalmazások kialakításával kapcsolatos szempontokat](overview-app-design-considerations.md) .</span><span class="sxs-lookup"><span data-stu-id="da15b-142">See [Hybrid app design considerations](overview-app-design-considerations.md) to learn more about best practices and to get answers for any additional questions.</span></span>
- <span data-ttu-id="da15b-143">A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .</span><span class="sxs-lookup"><span data-stu-id="da15b-143">See the [Azure Stack family of products and solutions](/azure-stack) to learn more about the entire portfolio of products and solutions.</span></span>

<span data-ttu-id="da15b-144">Ha készen áll a megoldás tesztelésére, folytassa a [földrajzilag elosztott app Solution telepítési útmutatóval](solution-deployment-guide-geo-distributed.md).</span><span class="sxs-lookup"><span data-stu-id="da15b-144">When you're ready to test the solution example, continue with the [Geo-distributed app solution deployment guide](solution-deployment-guide-geo-distributed.md).</span></span> <span data-ttu-id="da15b-145">A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.</span><span class="sxs-lookup"><span data-stu-id="da15b-145">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span> <span data-ttu-id="da15b-146">Megtudhatja, hogyan irányíthatja át a forgalmat adott végpontokra, a földrajzilag elosztott alkalmazás mintáját használó különféle mérőszámok alapján.</span><span class="sxs-lookup"><span data-stu-id="da15b-146">You learn how to direct traffic to specific endpoints, based on various metrics using the geo-distributed app pattern.</span></span> <span data-ttu-id="da15b-147">A Traffic Manager-profilok földrajzi alapú útválasztási és végponti konfigurációval való létrehozása biztosítja az információk átirányítását a végpontok számára a regionális követelmények, a vállalati és a nemzetközi szabályozás, valamint az adatok igényei alapján.</span><span class="sxs-lookup"><span data-stu-id="da15b-147">Creating a Traffic Manager profile with geographic-based routing and endpoint configuration ensures information is routed to endpoints based on regional requirements, corporate and international regulation, and your data needs.</span></span>
