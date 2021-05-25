---
title: Hibrid minták és megoldáspéékok az Azure-hoz és Azure Stack Hub
description: Hibrid minták és megoldási példák áttekintése hibrid megoldások tanulására és építésre az Azure-ban és a Azure Stack Hub.
author: BryanLa
ms.topic: overview
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: 9f3f13c23bec31c5132c7e90294356b9463fd72b
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343858"
---
# <a name="hybrid-solution-patterns-and-examples-for-azure-and-azure-stack"></a>Hibrid megoldások mintái és példái az Azure-hoz és Azure Stack

A Microsoft egységes Azure-Azure Stack kínál az Azure-termékek és -megoldások számára. A Microsoft Azure Stack-család az Azure bővítménye.

## <a name="the-hybrid-cloud-and-hybrid-apps"></a>A hibrid felhő és a hibrid alkalmazások

Azure Stack a felhőalapú számítástechnika agilitását a helyszíni környezetben és a peremhálózaton is lehetővé teszi egy hibrid *felhő engedélyezésével.* Azure Stack Hub, Azure Stack HCI és Azure Stack Edge kiterjesztheti az Azure-t a felhőből a szuverén adatközpontjaira, fiókirodáira, mezőire stb. Ezzel a sokféle képességgel a következő lehetőségeket használhatja:

- A kód újbóli használata és a natív felhőalkalmazások konzisztens futtatása az Azure-ban és a helyszíni környezetekben.
- Hagyományos virtualizált számítási feladatok futtatása azure-szolgáltatásokkal való választható kapcsolatokkal.
- A megfelelőség fenntartása érdekében az adatokat a felhőbe továbbíthatja, vagy megtarthatja a szuverén adatközpontban.
- A hardveresen gyorsított gépi tanulás, tárolók vagy virtualizált számítási feladatok futtatása az intelligens peremhálózaton.

A felhőkre is áttűnő alkalmazásokat hibrid *alkalmazásoknak is nevezzük.* Hibrid felhőalkalmazásokat építhet az Azure-ban, és üzembe helyezheti őket a csatlakoztatott vagy leválasztott adatközpontban bárhol.

A hibrid alkalmazások forgatókönyvei nagyban eltérnek a fejlesztéshez rendelkezésre álló erőforrásoktól függően. Emellett olyan szempontokra is vonatkoznak, mint a földrajzi hely, a biztonság, az internet-hozzáférés stb. Bár az itt ismertetett megoldásminták és példák nem minden követelményt elégnek ki, irányelveket és példákat tartalmaznak a hibrid megoldások megvalósítása során történő feltárásra és újbóli használatra.

## <a name="solution-patterns"></a>Megoldásminták

A megoldásminták általános, ismételhető tervezési útmutatót tartalmaznak a valós ügyfélforgatókönyvek és -tapasztalatok alapján. A minta absztrakt, így különböző típusú forgatókönyvekre vagy vertikális iparágakra alkalmazható. Minden minta bemutatja a kontextust és a problémát, és áttekintést nyújt egy megoldási példáról. A példamegoldást a minta lehetséges implementációjaként szántuk.

Kétféle mintacikk létezik:

- Egyetlen minta: tervezési útmutatást nyújt egyetlen általános célú forgatókönyvhöz.
- Több minta: tervezési útmutatást nyújt több minta alkalmazásához. Ez a minta gyakran szükséges összetettebb forgatókönyvek vagy iparág-specifikus problémák megoldásához.

## <a name="solution-deployment-guides"></a>Megoldás-üzembehelyezési útmutatók

A részletes üzembehelyezés útmutatói segítséget nyújtanak egy példamegoldás telepítéséhez. Az útmutató egy további kódmintára is hivatkozhat, amely a GitHub-megoldás mintaadattárában [van tárolva.](https://github.com/Azure-Samples/azure-intelligent-edge-patterns)

## <a name="next-steps"></a>Következő lépések

- A teljes [termék- Azure Stack termék-](/azure-stack) és megoldás-családról további információt talál a termék- és megoldás-családról.
- Ismerje meg a használati útmutató "Minták" és "Megoldástelepítési útmutatók" szakaszát, hogy többet tudjon meg mindegyikről.
- A hibrid [alkalmazások tervezésének szempontjait](overview-app-design-considerations.md) ismertető cikkből áttekintheti a hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzembe helyezéséhez szükséges szoftverminőség alappillérét.
- [Állítson be egy fejlesztési környezetet a Azure Stack](/azure-stack/user/azure-stack-dev-start) és telepítse az első alkalmazását [a](/azure-stack/user/azure-stack-dev-start-deploy-app) Azure Stack.
