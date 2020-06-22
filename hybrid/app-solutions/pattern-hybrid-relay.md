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
# <a name="hybrid-relay-pattern"></a>Hibrid továbbítási minta

Megtudhatja, hogyan csatlakozhat a tűzfalak által védett peremhálózati erőforrásokhoz vagy eszközökhöz a hibrid továbbítási minta és a Azure Relay használatával.

## <a name="context-and-problem"></a>Kontextus és probléma

Az Edge-eszközök gyakran vállalati tűzfal vagy NAT-eszköz mögött találhatók. Bár biztonságosak, előfordulhat, hogy nem tudnak kommunikálni a nyilvános felhővel vagy a peremhálózati eszközökkel más vállalati hálózatokon. A nyilvános felhőben lévő felhasználók számára bizonyos portok és funkciók biztonságos módon történő közzététele szükséges lehet.

## <a name="solution"></a>Megoldás

A hibrid továbbítási minta Azure Relay használatával hoz létre egy WebSockets-alagutat két végpont között, amelyek nem tudnak közvetlenül kommunikálni. Azok az eszközök, amelyek nem a helyszínen vannak, de csatlakozniuk kell egy helyszíni végponthoz, csatlakozni fognak a nyilvános felhőben lévő végponthoz. Ez a végpont átirányítja a forgalmat az előre meghatározott útvonalakon egy biztonságos csatornán keresztül. A helyszíni környezetben található végpont fogadja a forgalmat, és átirányítja a megfelelő helyre.

![hibrid továbbító minta megoldás architektúrája](media/pattern-hybrid-relay/solution-architecture.png)

A hibrid továbbítási minta működése:

1. Az eszköz egy előre meghatározott porton csatlakozik a virtuális géphez (VM) az Azure-ban.
2. A rendszer továbbítja a forgalmat a Azure Relay az Azure-ban.
3. Azure Stack hub-beli virtuális gép, amely már régóta élt kapcsolattal rendelkezik a Azure Relay, fogadja a forgalmat, és továbbítja azt a célhelyre.
4. A helyszíni szolgáltatás vagy végpont dolgozza fel a kérelmet.

## <a name="components"></a>Összetevők

Ez a megoldás a következő összetevőket használja:

| Réteg | Összetevő | Description |
|----------|-----------|-------------|
| Azure | Azure VM | Az Azure-beli virtuális gépek nyilvánosan elérhető végpontot biztosítanak a helyszíni erőforráshoz. |
| | Azure Relay | Az [Azure Relay](/azure/azure-relay/) biztosítja az alagút és az Azure-beli virtuális gép és a Azure stack hub virtuális gép közötti kapcsolat fenntartásához szükséges infrastruktúrát.|
| Azure Stack hub | Compute | Az Azure Stack hub-alapú virtuális gép a hibrid továbbító alagút kiszolgálóoldali részét biztosítja. |
| | Storage | Azure Stack hubhoz üzembe helyezett AK-motor-fürt méretezhető, rugalmas motort biztosít az Face API tároló futtatásához.|

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A megoldás megvalósításának eldöntése során vegye figyelembe a következő szempontokat:

### <a name="scalability"></a>Méretezhetőség

Ez a minta csak a 1:1-es port-hozzárendelések használatát teszi lehetővé az ügyfélen és a kiszolgálón. Ha például az 80-as port az Azure-végpont egyik szolgáltatásának bújtatása, akkor nem használható másik szolgáltatáshoz. A portok megfeleltetéseit ennek megfelelően kell tervezni. A Azure Relay és a virtuális gépeket megfelelő méretezéssel kell ellátni a forgalom kezeléséhez.

### <a name="availability"></a>Rendelkezésre állás

Ezek az alagutak és kapcsolatok nem redundánsak. A magas rendelkezésre állás biztosítása érdekében érdemes lehet végrehajtani a hibakódot. Egy másik lehetőség, hogy a terheléselosztó mögött Azure Relay csatlakoztatott virtuális gépek készletére van szükség.

### <a name="manageability"></a>Kezelhetőség

Ez a megoldás számos eszközre és helyszínre képes, így nem lehet megszerezni a megoldást. Az Azure IoT szolgáltatásai automatikusan online állapotba helyezhetik az új telephelyeket és eszközöket, és naprakészen tarthatják őket.

### <a name="security"></a>Biztonság

Ez a minta lehetővé teszi, hogy korlátlan hozzáférést biztosítson egy belső eszköz portjához a szélétől. Érdemes lehet hitelesítési mechanizmust hozzáadni a szolgáltatáshoz a belső eszközön vagy a hibrid továbbító végpontja előtt.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- Ez a minta Azure Relay használ. További információ: [Azure Relay dokumentáció](/azure/azure-relay/).
- Az ajánlott eljárásokkal kapcsolatos további információkért és a további kérdésekre adott válaszokért tekintse meg a [hibrid alkalmazások kialakításával kapcsolatos szempontokat](overview-app-design-considerations.md) .
- A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .

Ha készen áll a megoldás tesztelésére, folytassa a [hibrid továbbító megoldás telepítési útmutatóját](https://aka.ms/hybridrelaydeployment). A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.