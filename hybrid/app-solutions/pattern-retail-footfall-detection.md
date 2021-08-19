---
title: A lábléc észlelési mintája az Azure és a Azure Stack Hub
description: Megtudhatja, hogyan használhatja az Azure-Azure Stack Hub a kiskereskedelmi adatforgalom elemzésére szolgáló AI-alapú forgalomészlelési megoldást.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 79fb39d418bed53ef6a78980fcd9188bdf6e57ae
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281278"
---
# <a name="footfall-detection-pattern"></a>A lábléc észlelési mintája

Ez a minta áttekintést nyújt a kiskereskedelmi üzletek látogatói forgalmának elemzésére szolgáló AI-alapú forgalomészlelési megoldás megvalósításáról. A megoldás valós műveletekből hoz létre elemzéseket az Azure, Azure Stack Hub és a Custom Vision AI Dev Kit használatával.

## <a name="context-and-problem"></a>Kontextus és probléma

A Contoso Stores szeretne betekintést nyerni arról, hogy az ügyfelek hogyan kapják meg az aktuális termékeket az áruházi elrendezéshez képest. Nem tudnak minden szakaszban munkatársakat bevetni, és nem hatékony, ha egy elemzőcsapat áttekinti egy üzlet teljes kamerafelvételeit. Emellett egyik üzletük sem rendelkezik elegendő sávszélességgel ahhoz, hogy az összes kamerájukból a felhőbe streameljük a videókat elemzés céljából.

A Contoso egy diszkrét, adatvédelmet felhasználóbarát módon szeretné meghatározni az ügyfelek demográfiai adatait, hűségét és az áruházi megjelenítésekkel és termékekkel kapcsolatos reakcióit.

## <a name="solution"></a>Megoldás

Ez a kiskereskedelmi elemzési minta rétegzett megközelítést használ a peremhálózati dedoktáláshoz. Az AI Dev Kit Custom Vision használatával a rendszer csak emberi arcokat használó képeket küld elemzésre egy privát Azure Stack Hub, amely Azure Cognitive Services. A rendszer anonimizált, összesített adatokat küld az Azure-nak az összes üzlet és vizualizáció összesítése érdekében a Power BI. A peremhálózat és a nyilvános felhő kombinálásával a Contoso kihasználhatja a modern AI-technológiát, miközben továbbra is megfelel a vállalati szabályzatnak, és tiszteletben tartja az ügyfelek adatvédelmét.

[![A lábléc-észlelési minta megoldása](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)

Az összefoglalás a megoldás működését összegezve látható:

1. Az Custom Vision AI Dev Kit kap egy konfigurációt a IoT Hub-tól, amely telepíti az IoT Edge Runtime-t és egy ML modellt.
2. Ha a modell egy személyt lát, akkor képet vesz, és feltölti Azure Stack Hub blobtárolóba.
3. A blobszolgáltatás aktivál egy Azure-függvényt a Azure Stack Hub.
4. Az Azure-függvény egy Face API-val hív meg egy tárolót, hogy demográfiai és érzelemadatokat kapjunk a képről.
5. A rendszer anonimizálja az adatokat, és elküldi őket egy Azure Event Hubs fürtnek.
6. A Event Hubs fürt lek pushed az adatokat a Stream Analytics.
7. Stream Analytics összesíti az adatokat, és lekullja őket a Power BI.

## <a name="components"></a>Összetevők

Ez a megoldás a következő összetevőket használja:

| Réteg | Összetevő | Leírás |
|----------|-----------|-------------|
| Áruházbeli hardver | [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) | Áruházbeli szűrést biztosít egy helyi ML, amely csak személyek képeit rögzíti elemzés céljából. Biztonságosan kiépítve és frissítve a IoT Hub.<br><br>|
| Azure | [Azure Event Hubs](/azure/event-hubs/) | Azure Event Hubs olyan skálázható platformot biztosít az anonimizált adatokhoz, amelyek tökéletesen integrálhatók a Azure Stream Analytics. |
|  | [Azure Stream Analytics](/azure/stream-analytics/) | Egy Azure Stream Analytics feladat összesíti az anonimizált adatokat, és 15 másodperces ablakba csoportosíti őket a vizualizációhoz. |
|  | [Microsoft Power BI](https://powerbi.microsoft.com/) | Power BI egyszerűen használható irányítópult-felületet biztosít a jelentések kimenetének Azure Stream Analytics. |
| Azure Stack Hub | [APP SERVICE](/azure-stack/operator/azure-stack-app-service-overview) | A App Service erőforrás-szolgáltató (RP) egy alapként használható a peremhálózati összetevők számára, beleértve a webalkalmazások/API-k és a Functions üzemeltetési és felügyeleti funkcióit. |
| | Azure Kubernetes Service [(AKS) motorfürt](https://github.com/Azure/aks-engine) | Az AKS RP AKS-Engine fürtben üzembe helyezett Azure Stack Hub méretezhető, rugalmas motort biztosít a Face API-tároló futtatásához. |
| | Azure Cognitive Services Face [API-tárolók használata](/azure/cognitive-services/face/face-how-to-install-containers)| A Azure Cognitive Services RP a Face API-tárolók használatával demográfiai, érzelem- és egyedi látogatóészlelést biztosít a Contoso magánhálózatán. |
| | Blob Storage | Az AI Dev Kitből rögzített képek fel vannak töltve Azure Stack Hub blobtárolóba. |
| | Azure Functions | A szolgáltatáson futó Azure-Azure Stack Hub fogad bemenetet a Blob Storage-ból, és kezeli a Face API-val való interakciókat. Anonimizált adatokat bocsát ki egy Event Hubs Azure-ban található fürtbe.<br><br>|

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A megoldás megvalósításakor vegye figyelembe a következő pontokat:

### <a name="scalability"></a>Méretezhetőség

Ahhoz, hogy ez a megoldás több kamerára és helyre is méretezhető legyen, meg kell győződni arról, hogy az összes összetevő képes kezelni a megnövekedett terhelést. Előfordulhat, hogy a következő műveleteket kell majd eltennünk:

- Növelje a streamelési egységek Stream Analytics számát.
- Skálázja fel horizontálisan a Face API üzembe helyezését.
- Növelje a Event Hubs átviteli sebességét.
- Szélsőséges esetben szükség lehet a Azure Functions virtuális gépre való áttelepítésre.

### <a name="availability"></a>Rendelkezésre állás

Mivel ez a megoldás rétegzett, fontos át gondolni, hogyan kell kezelni a hálózat- vagy áramkimaradásokat. Az üzleti igényektől függően érdemes lehet olyan mechanizmust megvalósítani, amely helyileg gyorsítótárazza a rendszerképeket, majd továbbítja a Azure Stack Hub amikor a kapcsolat visszatér. Ha a hely elég nagy, a Data Box Edge-ben a Face API-tárolóval erre a helyre való üzembe helyezése jobb megoldás lehet.

### <a name="manageability"></a>Kezelhetőség

Ez a megoldás több eszközre és helyre is kihathat, ami problémás lehet. [Az Azure IoT-szolgáltatásaival](/azure/iot-fundamentals/) automatikusan online állapotba hozhatja az új helyeket és eszközöket, és naprakészen tarthatja őket.

### <a name="security"></a>Biztonság

Ez a megoldás rögzíti az ügyfelek rendszerképét, és kiemelten fontos szempont a biztonság. Győződjön meg arról, hogy az összes tárfiókot a megfelelő hozzáférési szabályzatok biztosítják, és rendszeresen váltogatja a kulcsokat. Győződjön meg arról, hogy a tárfiókok Event Hubs a vállalati és kormányzati adatvédelmi előírásoknak megfelelő adatmegőrzési szabályzatokkal. Ügyeljen arra is, hogy a felhasználói hozzáférési szintek rétege legyen. A rétegezés biztosítja, hogy a felhasználók csak a szerepkörükhöz szükséges adatokhoz férnek hozzá.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- Lásd: [Rétegzett adatok minta,](https://aka.ms/tiereddatadeploy)amelyet a láblécészlelési minta kihasznál.
- A [custom vision Custom Vision az AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) használatával kapcsolatos további információkért tekintse meg a Custom Vision AI Dev Kitet. 

Ha készen áll a megoldási példa tesztelésére, folytassa a Következő üzembe [helyezési útmutatóval:](/azure/architecture/hybrid/deployments/solution-deployment-guide-retail-footfall-detection). Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez.