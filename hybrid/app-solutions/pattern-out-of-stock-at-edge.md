---
title: Készleten nem volt észlelve az Azure és a Azure Stack Edge
description: Megtudhatja, hogyan használhatja az Azure-t Azure Stack Edge a készleten nem használt észlelés megvalósításához.
author: BryanLa
ms.topic: article
ms.date: 05/24/2021
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 05/24/2021
ms.openlocfilehash: b25a6391c4e64fa7018031bac4fb7d098c56b529
ms.sourcegitcommit: cf2c4033d1b169f5b63980ce1865281366905e2e
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 05/25/2021
ms.locfileid: "110343875"
---
# <a name="out-of-stock-detection-at-the-edge-pattern"></a>Készleten nem található a peremhálózati minta

Ez a minta azt mutatja be, hogyan állapítható meg, hogy a polcokon nincs-e készleten lévő tétel egy Azure Stack Edge vagy Azure IoT Edge és hálózati kamerával.

## <a name="context-and-problem"></a>Kontextus és probléma

A fizikai kiskereskedelmi üzletek elveszítik az értékesítést, mert amikor az ügyfelek egy terméket keresnek, az nem található meg a polcon. Előfordulhat azonban, hogy az elem az üzlet háta mögött van, és nem lett újra készletre hozva. Az üzletek szeretnék hatékonyabban használni a személyzetet, és automatikusan értesítést kapni, ha az elemeket újra kell behozni.

## <a name="solution"></a>Megoldás

A megoldás példájában egy peremhálózati eszközt, például egy Azure Stack Edge használ az egyes áruházakban, amely hatékonyan dolgozza fel az áruházban található kamerák adatait. Ez az optimalizált kialakítás lehetővé teszi, hogy a tárolók csak a releváns eseményeket és rendszerképeket küldjék a felhőbe. A kialakítás sávszélességet és tárhelyet takarít meg, és gondoskodik az ügyfelek adatainak védelméről. Ahogy a képkockákat beolvassa az egyes kamerákból, az ML-modell feldolgozza a képet, és visszaadja a készletből származó területeket. A kép és a készleten nem található területek jelennek meg egy helyi webalkalmazásban. Az adatokat elküldheti egy Time Series Insights-környezetbe, hogy elemzéseket mutasson a Power BI.

![Készleten nem található a peremhálózati megoldás architektúrája](media/pattern-out-of-stock-at-edge/solution-architecture.png)

A megoldás működése a következő:

1. A képek egy hálózati kamerából vannak rögzítettek HTTP- vagy RTSP-kapcsolaton keresztül.
2. A rendszer átméretezi a képet, és elküldi a következtetési illesztőnek, amely kommunikál az ML-modellel annak megállapításához, hogy nincsenek-e készleten a képek.
3. Az ML-modell a készleten nem található területeket adja vissza.
4. A következtetési illesztőprogram feltölti a nyers képet egy blobba (ha meg van adva), és elküldi az eredményeket a modellből az Azure IoT Hub-nek és egy határolókeret-feldolgozónak az eszközön.
5. A határolókeret-feldolgozó határolókereteket ad a képhez, és gyorsítótárazza a kép elérési útját egy memória-adatbázisban.
6. A webalkalmazás lekérdezi a rendszerképeket, és a kapott sorrendben jeleníti meg őket.
7. A rendszer IoT Hub összesíti az üzeneteket a Time Series Insights.
8. Power BI egy interaktív jelentést jelenít meg a készleten lévő termékekről az idő alatt, a készleten lévő Time Series Insights.


## <a name="components"></a>Összetevők

Ez a megoldás a következő összetevőket használja:

| Réteg | Összetevő | Leírás |
|----------|-----------|-------------|
| Helyszíni hardver | Hálózati kamera | A képek deduktálása érdekében szükség van egy HTTP- vagy RTSP-hírcsatornával is elérhető hálózati kamerára. |
| Azure | Azure IoT Hub | [Azure IoT Hub](/azure/iot-hub/) a peremeszközök kiépítését és üzenetkezelését kezeli. |
|  | Azure Time Series Insights | [Azure Time Series Insights](/azure/time-series-insights/) a vizualizációhoz IoT Hub üzeneteket. |
|  | Power BI | [A Microsoft Power BI](https://powerbi.microsoft.com/) üzleti központú jelentéseket biztosít a készleten lévő eseményekről. Power BI egy könnyen használható irányítópult-felületet biztosít a jelentések kimenetének Azure Stream Analytics. |
| Azure Stack Edge vagy<br>Azure IoT Edge eszköz | Azure IoT Edge | [Azure IoT Edge](/azure/iot-edge/) a helyszíni tárolók futtatáskörnyezetét, és kezeli az eszközkezelést és a frissítéseket.|
| | Azure Project Brainwave | Egy virtuális Azure Stack Edge a [Project Brainwave](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) Field-Programmable kaputömbök (FPGA-k) használatával felgyorsítja az ML-következtetést.|

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A megoldás megvalósításakor vegye figyelembe a következő pontokat:

### <a name="scalability"></a>Méretezhetőség

A legtöbb gépi tanulási modell a megadott hardvertől függően csak bizonyos számú képkockát képes futtatni másodpercenként. Határozza meg a kamera(k) optimális mintaarányát annak érdekében, hogy a gépi tanulási folyamat ne legyen biztonságimentve. A különböző hardvertípusok különböző számú kamerát és képkocka-számot kezelnek.

### <a name="availability"></a>Rendelkezésre állás

Fontos megfontolni, hogy mi történhet, ha a peremeszköz kapcsolata megszakad. Gondolja át, hogy milyen adatok elvesznek a Time Series Insights és Power BI irányítópultról. A megadott példamegoldás nem magas rendelkezésre áll rendelkezésre áll.

### <a name="manageability"></a>Kezelhetőség

Ez a megoldás több eszközre és helyre is kihathat, ami problémás lehet. Az Azure IoT-szolgáltatásai automatikusan online állapotba hozzák az új helyeket és eszközöket, és naprakészen tartják őket. A megfelelő adatirányítási eljárásokat is követni kell.

### <a name="security"></a>Biztonság

Ez a minta bizalmas adatokat kezel. Ellenőrizze, hogy a kulcsok rendszeresen le vannak-e állítva, és hogy az Azure Storage-fiók és a helyi megosztások engedélyei megfelelően vannak-e beállítva.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:
- Ebben a mintában több IoT-hez kapcsolódó szolgáltatás is használatos, beleértve a [Azure IoT Edge](/azure/iot-edge/), [Azure IoT Hub](/azure/iot-hub/)és [Azure Time Series Insights.](/azure/time-series-insights/)
- A Microsoft Project Brainwave projekttel kapcsolatos további információkért tekintse meg a [blog](https://blogs.microsoft.com/ai/build-2018-project-brainwave/) bejelentését, és tekintse meg az [Azure Accelerated Machine Learning project brainwave videóját.](https://www.youtube.com/watch?v=DJfMobMjCX0)
- Az [ajánlott eljárásokról](overview-app-design-considerations.md) és a további kérdésekre adott válaszokért lásd: Hibridalkalmazás-kialakítási szempontok.
- A Azure Stack [termékek](/azure-stack) és megoldások teljes portfólióját további információért tekintse meg a termék- és megoldás-családról.

Ha készen áll a megoldási példa tesztelésére, folytassa az [Edge ML-következtetési](https://aka.ms/edgeinferencingdeploy)megoldás üzembe helyezési útmutatóját. Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez.
