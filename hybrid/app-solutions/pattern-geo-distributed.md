---
title: Földrajzi eloszlású alkalmazásminta a Azure Stack Hub
description: Ismerje meg az Azure és az Azure használatával az intelligens peremhálózathoz használt földrajzi alapú alkalmazásminta Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod2019
ms.openlocfilehash: 3c839d9bf3b6c3e1ff50cc695fd5f1a1127793d2
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281227"
---
# <a name="geo-distributed-app-pattern"></a>Földrajzi eloszlású alkalmazásminta

Megtudhatja, hogyan biztosíthat alkalmazásvégpontokat több régióban, és hogyan irányítható a felhasználói forgalom a hely és a megfelelőségi igények alapján.

## <a name="context-and-problem"></a>Kontextus és probléma

A széles körű földrajzi helyekkel rendelkezik szervezetek igyekeznek biztonságosan és pontosan elosztani és engedélyezni az adatokhoz való hozzáférést, ugyanakkor biztosítaniuk kell a szükséges biztonsági, megfelelőségi és teljesítményszinteket felhasználónként, helyszínen és eszközönként a határokon keresztül.

## <a name="solution"></a>Megoldás

A Azure Stack Hub forgalom-útválasztási minta vagy földrajzi elosztott alkalmazások lehetővé teszi, hogy a forgalom különböző metrikák alapján meghatározott végpontokra legyen irányítva. A Traffic Manager útválasztással és végpont-konfigurációval való létrehozása a regionális követelmények, a vállalati és nemzetközi szabályozás és az adatigények alapján a végpontokrairányja a forgalmat.

![Földrajzi eloszlású minta](media/pattern-geo-distributed/geo-distribution.png)

## <a name="components"></a>Összetevők

### <a name="outside-the-cloud"></a>A felhőn kívül

#### <a name="traffic-manager"></a>Traffic Manager

A diagramon Traffic Manager a nyilvános felhőn kívül található, de képesnek kell lennie koordinálni a forgalmat a helyi adatközpontban és a nyilvános felhőben is. A balancer a forgalmat földrajzi helyekre forgalmaz.

#### <a name="domain-name-system-dns"></a>Tartománynévrendszer (DNS)

A tartománynévrendszer vagy a DNS felelős egy webhely vagy szolgáltatás nevének AZ IP-címére való lefordításáért (vagy feloldásáért).

### <a name="public-cloud"></a>Nyilvános felhő

#### <a name="cloud-endpoint"></a>Felhőbeli végpont

A nyilvános IP-címek a bejövő forgalmat a Traffic Manageren keresztül a nyilvános felhőalkalmazás erőforrás-végpontjára irányítják.  

### <a name="local-clouds"></a>Helyi felhők

#### <a name="local-endpoint"></a>Helyi végpont

A nyilvános IP-címek a bejövő forgalmat a Traffic Manageren keresztül a nyilvános felhőalkalmazás erőforrás-végpontjára irányítják.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

### <a name="scalability"></a>Méretezhetőség

A minta nem a forgalom növekedése miatt, hanem a földrajzi forgalom útválasztását kezeli a skálázás helyett. Ezt a mintát azonban kombinálhatja más Azure- és helyszíni megoldásokkal is. Ez a minta használható például a felhők közötti méretezési mintával.

### <a name="availability"></a>Rendelkezésre állás

Győződjön meg arról, hogy a helyileg telepített alkalmazások magas rendelkezésre állásra vannak konfigurálva a helyszíni hardverkonfiguráción és a szoftverek telepítésén keresztül.

### <a name="manageability"></a>Kezelhetőség

A minta zökkenőmentes felügyeletet és ismerős felületet biztosít a környezetek között.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

- A szervezetem nemzetközi ágakkal rendelkezik, amelyekhez egyéni regionális biztonsági és terjesztési szabályzatokra van szükség.
- A szervezet minden irodája lekérte az alkalmazottak, az üzleti és a létesítmény adatait, és a jelentéskészítési tevékenységet helyi szabályozások és időzóna szerint követeli meg.
- A nagy léptékű követelmények az alkalmazások horizontális horizontális felskálával teljesülnek, és a szélsőséges terhelési követelmények kezeléséhez több alkalmazástelepítést kell végrehajtani egyetlen régión belül és régiók között.
- Az alkalmazásoknak magas rendelkezésre állékonyságúnak kell lennie, és képesnek kell lennie reagálni az ügyfélkérésre még egy régiós kimaradások esetén is.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- A [DNS-Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) terheléselosztási rendszerével kapcsolatos további információkért tekintse meg az Azure Traffic Manager áttekintését.
- Az [ajánlott eljárásokkal](overview-app-design-considerations.md) kapcsolatos további információkért és a további kérdésekre adott válaszokért tekintse meg a hibrid alkalmazástervezési szempontokat.
- A teljes [termék- Azure Stack termék-](/azure-stack) és megoldás-családról további információt talál a termék- és megoldás-családról.

Ha készen áll a megoldási példa tesztelésére, folytassa a földrajzi elosztású alkalmazásmegoldás üzembe helyezési [útmutatóját.](/azure/architecture/hybrid/deployments/solution-deployment-guide-geo-distributed) Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez. Megtanulja, hogyan irányítható a forgalom adott végpontokra különböző metrikák alapján a földrajzi elosztott alkalmazásminta használatával. Ha földrajzi Traffic Manager útválasztással és végpontkonfigurációval hoz létre egy profilt, azzal biztosítható, hogy az információk a regionális követelmények, a vállalati és a nemzetközi szabályozás, valamint az adatokra vonatkozó igények alapján a végpontokra vannak irányítva.