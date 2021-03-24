---
title: Lépés hangja észlelési minta az Azure és Azure Stack hub használatával
description: Ismerje meg, hogyan használható az Azure és a Azure Stack hub a kiskereskedelmi adatforgalom elemzésére szolgáló AI-alapú lépés hangja-észlelési megoldás megvalósításához.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 866557ec3af2337e9f034da84cf417675508563b
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895329"
---
# <a name="footfall-detection-pattern"></a>Lépés hangja észlelési minta

Ez a minta áttekintést nyújt egy mesterséges intelligencia-alapú lépés hangja-észlelési megoldás megvalósításáról a kiskereskedelmi üzletek látogatói forgalmának elemzéséhez. A megoldás az Azure-ban, Azure Stack hub-on és a Custom Vision AI fejlesztői csomagban bepillantást nyerhet a valós világbeli műveletekkel.

## <a name="context-and-problem"></a>Kontextus és probléma

A contoso-áruházak elemzéseket szeretnének kapni arról, hogy az ügyfelek Hogyan kapják meg aktuális termékeiket az áruház elrendezésével kapcsolatban. Nem helyezhetők el a személyzet minden szakaszba, és nem hatékony, hogy az elemzők egy csoportja áttekintse a teljes áruház kamerás felvételeit. Emellett a tárolók egyike sem rendelkezik elegendő sávszélességgel, hogy az összes kameráról a felhőbe továbbítsa a videót elemzés céljából.

A contoso szeretné megkeresni az ügyfelek demográfiai, lojalitási és adatkezelési lehetőségeit a kijelzők és termékek tárolására.

## <a name="solution"></a>Megoldás

Ez a kiskereskedelmi elemzési minta egy lépcsőzetes megközelítést használ a szélén való következtetéshez. A Custom Vision AI dev kit használatával csak az emberi arcokkal rendelkező képek lesznek ellátva az Azure Cognitive Services futtató privát Azure Stack hubhoz való elemzésre. Anonim, összesített adatokat küld az Azure-ba az összes áruházban és vizualizációban Power BI. A peremhálózat és a nyilvános felhő együttes használata lehetővé teszi, hogy a contoso kihasználja a modern AI-technológia előnyeit, miközben a vállalati szabályzatoknak való megfelelés és az ügyfelek magánéletének tiszteletben tartásával is rendelkezik.

[![Lépés hangja-észlelési minta megoldás](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Íme egy összefoglaló a megoldás működéséről:

1. A Custom Vision AI dev Kit IoT Hubtól származó konfigurációt kap, amely telepíti a IoT Edge futtatókörnyezetet és egy ML modellt.
2. Ha a modell egy személyt lát, egy képet helyez el, és feltölti Azure Stack hub blob Storage-ba.
3. A blob szolgáltatás elindít egy Azure-függvényt Azure Stack hub-on.
4. Az Azure-függvény egy tárolót hív meg a Face API, hogy demográfiai és érzelem-adatokhoz kapjon képet.
5. Az adatküldés egy Azure Event Hubs-fürtbe történik.
6. A Event Hubs-fürt leküldi az adatStream Analyticsba.
7. Stream Analytics összesíti az adatokat, és leküldi Power BIre.

## <a name="components"></a>Összetevők

Ez a megoldás a következő összetevőket használja:

| Réteg | Összetevő | Leírás |
|----------|-----------|-------------|
| Tárolt hardver | [Custom Vision AI fejlesztői csomag](https://azure.github.io/Vision-AI-DevKit-Pages/) | Egy helyi ML-modell használatával biztosítja az áruházbeli szűrést, amely csak az elemzésre alkalmas személyek képét rögzíti. Biztonságos kiépítés és frissítés IoT Hubon keresztül.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Az Azure Event Hubs méretezhető platformot biztosít a Azure Stream Analyticshoz szépen integrálható, névtelenül tárolt adatmennyiségek betöltéséhez. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Az Azure Stream Analytics-feladatok összesítik a névtelen adatokat, és a vizualizációk 15 másodperces Windowsba csoportosítják azokat. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | A Power BI egy könnyen használható irányítópult-felületet biztosít a Azure Stream Analytics kimenetének megtekintéséhez. |
| Azure Stack Hub | [APP SERVICE](/azure-stack/operator/azure-stack-app-service-overview) | A App Service erőforrás-szolgáltató (RP) egy alapot biztosít a peremhálózati összetevők számára, beleértve a Web Apps/API-k és függvények üzemeltetési és felügyeleti funkcióit. |
| | Azure Kubernetes szolgáltatás [(ak) motorjának](https://github.com/Azure/aks-engine) fürtje | A Azure Stack hub-ba telepített AKS-Engine-fürttel rendelkező AK RP méretezhető, rugalmas motort biztosít az Face API tároló futtatásához. |
| | Azure Cognitive Services [Face API tárolók](/azure/cognitive-services/face/face-how-to-install-containers)| Az Azure Cognitive Services RP Face API tárolókkal biztosítja a demográfiai, érzelem-és egyedi látogatói észlelést a contoso magánhálózaton. |
| | Blob Storage | Az AI fejlesztői csomagból rögzített rendszerképek fel lesznek töltve az Azure Stack hub blob Storage-tárolóba. |
| | Azure Functions | Azure Stack hub-on futó Azure-függvény fogadja a blob Storage-ból érkező adatokat, és kezeli az interakciókat a Face API. A rendszer az Azure-ban található Event Hubs-fürtre küldi el a névtelenül való adatgyűjtést.<br><br>|

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A megoldás megvalósításának eldöntése során vegye figyelembe a következő szempontokat:

### <a name="scalability"></a>Méretezhetőség

Ha engedélyezni szeretné, hogy a megoldás több kamera és hely között is méretezhető legyen, meg kell győződnie arról, hogy az összes összetevő képes kezelni a megnövekedett terhelést. Előfordulhat, hogy olyan műveleteket kell végrehajtania, mint például:

- Növelje Stream Analytics folyamatos átviteli egységek számát.
- A Face API üzembe helyezésének felskálázása.
- Növelje a Event Hubs-fürt átviteli sebességét.
- Szélsőséges esetekben szükség lehet a Azure Functionsról a virtuális gépre való Migrálás.

### <a name="availability"></a>Rendelkezésre állás

Mivel ez a megoldás többrétegű, fontos, hogy meggondolja, hogyan kell kezelni a hálózati vagy áramkimaradásokat. Az üzleti igényektől függően érdemes lehet megvalósítani egy olyan mechanizmust, amely helyileg gyorsítótárazza a lemezképeket, majd a kapcsolat visszaadásakor továbbítja Azure Stack hubhoz. Ha a hely elég nagy, akkor lehet, hogy egy Data Box Edge üzembe helyezése a Face API tárolóval az adott helyen jobb megoldás lehet.

### <a name="manageability"></a>Kezelhetőség

Ez a megoldás számos eszközre és helyszínre képes, így nem lehet megszerezni a megoldást. Az [Azure IoT-szolgáltatásaival](/azure/iot-fundamentals/) automatikusan online állapotba helyezhetők az új telephelyek és eszközök, és naprakészen tarthatók.

### <a name="security"></a>Biztonság

Ez a megoldás rögzíti a vásárlói rendszerképeket, így a biztonság a legfontosabb szempont. Győződjön meg arról, hogy az összes Storage-fiók biztonságos a megfelelő hozzáférési házirendekkel, és hogy rendszeresen forgatni kívánja a kulcsokat. Győződjön meg arról, hogy a Storage-fiókok és a Event Hubs rendelkeznek a vállalati és kormányzati adatvédelmi szabályozásoknak megfelelő adatmegőrzési szabályzatokkal Győződjön meg arról is, hogy a felhasználói hozzáférési szintek rétegben vannak. A rétegek biztosítása biztosítja, hogy a felhasználók csak a szerepkörük számára szükséges adathoz férhessenek hozzá.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- Tekintse meg a lépés hangja-észlelési minta által kihasználható, [többszintes adatmintát](https://aka.ms/tiereddatadeploy).
- További információ az egyéni jövőkép használatáról: [Custom Vision AI fejlesztői csomag](https://azure.github.io/Vision-AI-DevKit-Pages/) . 

Ha készen áll a megoldás tesztelésére, folytassa a lépés hangja- [észlelés telepítési útmutatóját](solution-deployment-guide-retail-footfall-detection.md). A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.
