---
title: Felhők közötti méretezési minta a Azure Stack Hub
description: Megtudhatja, hogyan építhet ki skálázható, felhők közötti alkalmazást az Azure-ban, és hogyan Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 90e0c177b5eaee4d223b4613e0b2ddf385fa799c
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281261"
---
# <a name="cross-cloud-scaling-pattern"></a>Felhők közötti méretezési minta

Erőforrások automatikus hozzáadása egy meglévő alkalmazáshoz a terhelés növekedése esetén.

## <a name="context-and-problem"></a>Kontextus és probléma

Az alkalmazás nem tudja növelni a kapacitást, hogy megfeleljen a kereslet váratlan növekedésének. A skálázhatóság hiánya miatt a felhasználók nem érik el az alkalmazást a használati csúcsidőszakban. Az alkalmazás rögzített számú felhasználót képes kiszolgálásra.

A globális vállalatoknak biztonságos, megbízható és elérhető felhőalapú alkalmazásokra van szükségük. A növekvő igényeknek való megfelelő igényeknek való megfelelő infrastruktúra használata kritikus fontosságú. A vállalatok nehezen egyensúlyba hozják a költségeket és a karbantartást az üzleti adatok biztonságával, tárolásával és valós idejű rendelkezésre állásával.

Előfordulhat, hogy nem tudja futtatni az alkalmazást a nyilvános felhőben. Előfordulhat azonban, hogy a vállalat számára nem megvalósítható a helyszíni környezetben szükséges kapacitás fenntartása az alkalmazás iránti kiugróan magas igények kezeléséhez. Ezzel a mintával használhatja a nyilvános felhő rugalmasságát a helyszíni megoldással.

## <a name="solution"></a>Megoldás

A felhők közötti méretezési minta kiterjeszti a helyi felhőben található alkalmazásokat nyilvános felhőbeli erőforrásokkal. A mintát a kereslet növekedése vagy csökkenése váltja ki, és erőforrásokat ad hozzá vagy távolít el a felhőben. Ezek az erőforrások redundanciát, gyors rendelkezésre állást és földrajzi megfelelő útválasztást biztosítanak.

![Felhők közötti méretezési minta](media/pattern-cross-cloud-scale/cross-cloud-scaling.png)

> [!NOTE]
> Ez a minta csak az alkalmazás állapot nélküli összetevőire vonatkozik.

## <a name="components"></a>Összetevők

A felhők közötti méretezési minta az alábbi összetevőkből áll.

### <a name="outside-the-cloud"></a>A felhőn kívül

#### <a name="traffic-manager"></a>Traffic Manager

Az ábrán ez a nyilvános felhőcsoporton kívül található, de képesnek kell lennie koordinálni a forgalmat a helyi adatközpontban és a nyilvános felhőben is. A balancer magas rendelkezésre állást biztosít az alkalmazások számára a végpontok monitorozásával és szükség esetén a feladatátvétel újraelosztásával.

#### <a name="domain-name-system-dns"></a>Tartománynévrendszer (DNS)

A tartománynévrendszer vagy a DNS felelős egy webhely vagy szolgáltatás nevének AZ IP-címére való lefordításáért (vagy feloldásáért).

### <a name="cloud"></a>Felhőbeli

#### <a name="hosted-build-server"></a>Üzemeltetett buildkiszolgáló

A build-folyamat üzemeltetési környezete.

#### <a name="app-resources"></a>Alkalmazás-erőforrások

Az alkalmazás-erőforrásoknak képesnek kell lennie horizontális le- és felskálákra, például virtuálisgép-méretezési készletekre és tárolókra.

#### <a name="custom-domain-name"></a>Egyéni tartománynév

Használjon egyéni tartománynevet az útválasztási kérelmekhez.

#### <a name="public-ip-addresses"></a>Nyilvános IP-címek

A nyilvános IP-címek a bejövő forgalmat a Traffic Manageren keresztül a nyilvános felhőalkalmazás erőforrás-végpontjára irányítják.  

### <a name="local-cloud"></a>Helyi felhő

#### <a name="hosted-build-server"></a>Üzemeltetett buildkiszolgáló

A build-folyamat üzemeltetési környezete.

#### <a name="app-resources"></a>Alkalmazás-erőforrások

Az alkalmazás-erőforrásoknak szükségük van a horizontális le- és felskálás képességre, például a virtuálisgép-méretezési készletekre és a tárolókra.

#### <a name="custom-domain-name"></a>Egyéni tartománynév

Használjon egyéni tartománynevet az útválasztási kérelmekhez.

#### <a name="public-ip-addresses"></a>Nyilvános IP-címek

A nyilvános IP-címek a bejövő forgalmat a Traffic Manageren keresztül a nyilvános felhőalkalmazás erőforrás-végpontjára irányítják.

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A minta megvalósítása során az alábbi pontokat vegye figyelembe:

### <a name="scalability"></a>Méretezhetőség

A felhők közötti skálázás fő összetevője az igény szerinti skálázás képessége. A skálázásnak a nyilvános és a helyi felhőalapú infrastruktúra között kell történnie, és igény szerint konzisztens, megbízható szolgáltatást kell nyújtania.

### <a name="availability"></a>Rendelkezésre állás

Győződjön meg arról, hogy a helyileg telepített alkalmazások magas rendelkezésre állásra vannak konfigurálva a helyszíni hardverkonfiguráción és a szoftverek telepítésén keresztül.

### <a name="manageability"></a>Kezelhetőség

A felhők közötti minta zökkenőmentes felügyeletet és ismerős felületet biztosít a környezetek között.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja a következő mintát a következő helyzetekben:

- Ha váratlan igényekkel vagy időszakos igényekkel kell növelnie az alkalmazás kapacitását.
- Ha nem szeretne olyan erőforrásokba befektetni, amelyek csak csúcsidőszakban lesznek használva. A használt használatért kell fizetnie.

Ez a minta nem ajánlott, ha:

- A megoldáshoz az interneten keresztül csatlakozó felhasználókra van szükség.
- A vállalata helyi szabályozásokkal rendelkezik, amelyek megkövetelik, hogy az eredeti kapcsolat egy helyszíni hívásból származtatva jöjjön létre.
- A hálózatban rendszeres szűk keresztmetszetek vannak, amelyek korlátozzák a skálázás teljesítményét.
- A környezet nem csatlakozik az internethez, és nem tudja elérni a nyilvános felhőt.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- A [DNS-Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) terheléselosztási rendszerével kapcsolatos további információkért tekintse meg az Azure Traffic Manager áttekintését.
- Az [ajánlott eljárásokról](overview-app-design-considerations.md) és a további kérdésekre adott válaszokért tekintse meg a Hibridalkalmazás-kialakítási szempontokat.
- A Azure Stack [termékek és](/azure-stack) megoldások teljes portfólióját további információért tekintse meg a termék- és megoldás-családról.

Ha készen áll a megoldás példának tesztelésére, folytassa a többfelhős skálázás megoldás üzembe helyezési [útmutatóját.](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling) Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez. Megtudhatja, hogyan hozhat létre egy többfelhős megoldást, amely manuálisan aktivált folyamatot biztosít a Azure Stack Hub üzemeltetett webalkalmazásról egy Azure-ban üzemeltetett webalkalmazásra való váltáshoz. Azt is megtudhatja, hogyan használhatja az automatikus skálázást a Traffic Managerrel, és hogyan biztosíthatja a rugalmas és méretezhető felhőbeli segédprogramokat terhelés esetén.