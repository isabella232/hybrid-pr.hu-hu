---
title: Hibrid minták és megoldási példák az Azure-ra és Azure Stack hub-ra
description: Az Azure-ban és Azure Stack hub-ban hibrid megoldások megismerésére és megoldására szolgáló hibrid minták és példák áttekintése.
author: BryanLa
ms.topic: overview
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 4f86e5ae4b8b9bd7693617b07419b67dfcf05dc1
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895312"
---
# <a name="hybrid-patterns-and-solution-examples-for-azure-and-azure-stack"></a>Hibrid minták és megoldási példák az Azure-ra és a Azure Stack

A Microsoft az Azure-t és Azure Stack termékeket és megoldásokat biztosít egyetlen egységes Azure-ökoszisztémaként. A Microsoft Azure Stack család az Azure kiterjesztése.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>Hibrid Felhőbeli és hibrid alkalmazások

A Azure Stack a felhő-számítástechnika rugalmasságát a helyszíni környezetbe és a peremre helyezi a *hibrid felhő* engedélyezése révén. Azure Stack hub, Azure Stack HCI és Azure Stack Edge kiterjeszti az Azure-t a felhőből a szuverén adatközpontokban, a fiókirodákban, a mezőkben és azon kívül. Ezzel a különböző funkciókkal a következőket teheti:

- Az Azure-ban és a helyszíni környezetekben konzisztens módon használhat programkódot, és a felhőben natív alkalmazásokat is futtathat.
- Hagyományos virtualizált számítási feladatok futtatása az Azure-szolgáltatásokhoz való opcionális csatlakozással.
- Vigye át az adatait a felhőbe, vagy tartsa meg a szuverén adatközpontban a megfelelőség fenntartása érdekében.
- Hardveres gyorsítású gépi tanulási, tárolós vagy virtualizált munkaterhelések futtatása az intelligens peremhálózat minden részén.

A felhőkre kiterjedő alkalmazásokat *hibrid alkalmazásoknak* is nevezzük. Hibrid felhőalapú alkalmazásokat hozhat létre az Azure-ban, és üzembe helyezheti azokat a csatlakoztatott vagy leválasztott adatközpontban bárhol.

A hibrid alkalmazás-forgatókönyvek nagy mértékben eltérnek a fejlesztéshez rendelkezésre álló erőforrásokkal. Olyan szempontokat is figyelembe kell venniük, mint például a földrajz, a biztonság, az Internet-hozzáférés és egyebek. Bár az itt ismertetett minták és megoldások nem minden követelményre vonatkoznak, útmutatást és példákat biztosítanak a hibrid megoldások megvalósítása során felderített és újbóli felhasználáshoz.

## <a name="design-patterns"></a>Tervezési minták

A tervezési minták kiselejtezett általánosított kialakítási útmutatást nyújtanak a valós felhasználói forgatókönyvek és a tapasztalatok alapján. A minta absztrakt, amely lehetővé teszi, hogy a különböző típusú forgatókönyvekre vagy vertikális iparágakra alkalmazható legyen. Minden minta dokumentálja a környezetet és a problémát, és áttekintést nyújt a megoldásról. A megoldás példaként a minta lehetséges megvalósítását jelenti.

A minták két típusa létezik:

- Egyetlen minta: tervezési útmutatót biztosít egyetlen általános célú forgatókönyvhöz.
- Multi-Pattern: tervezési útmutatást biztosít, ahol több minta alkalmazása is használható. Ez a minta gyakran szükséges összetettebb forgatókönyvek vagy iparági problémák megoldásához.

## <a name="solution-deployment-guides"></a>Megoldás-üzembehelyezési útmutatók

A lépésenkénti üzembe helyezési útmutatók segítséget nyújtanak a megoldások üzembe helyezésében. Az útmutató a GitHub [Solutions](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)-tárházban tárolt Companion Code-mintára is hivatkozhat.

## <a name="next-steps"></a>Következő lépések

- A termékek és megoldások teljes portfóliójának megismeréséhez tekintse meg a [Azure stack termékcsaládot és megoldásokat](/azure-stack) .
- Ismerkedjen meg a "Patterns" és a "megoldás központi telepítési útmutatók" fejezeteivel a TARTALOMJEGYZÉKben.
- További információ a [hibrid alkalmazások kialakításával kapcsolatos szempontokról](overview-app-design-considerations.md) : a hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez szükséges szoftverek minőségi pillérének áttekintése.
- [Hozzon létre egy fejlesztési környezetet a Azure stackon](/azure-stack/user/azure-stack-dev-start) , és helyezze [üzembe az első alkalmazást](/azure-stack/user/azure-stack-dev-start-deploy-app) Azure stackon.
