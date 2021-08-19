---
title: Felhők közötti skálázás (helyszíni adatok) minta a Azure Stack Hub
description: Ismerje meg, hogyan lehet skálázható, több felhőre átfedő alkalmazást összeépíteni, amely az Azure-ban és az Azure-ban Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281244"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Felhők közötti skálázás (helyszíni adatok) minta

Megtudhatja, hogyan építhet ki az Azure-t és az Azure-t Azure Stack Hub. Ez a minta azt is bemutatja, hogyan használható egyetlen helyszíni adatforrás a megfelelőséghez.

## <a name="context-and-problem"></a>Kontextus és probléma

Számos szervezet gyűjt és tárol nagy mennyiségű bizalmas ügyféladatot. Gyakran előfordul, hogy a vállalati szabályozások vagy kormányzati szabályzatok miatt nem tárolnak bizalmas adatokat a nyilvános felhőben. Ezek a szervezetek a nyilvános felhő méretezhetőségét is szeretnék kihasználni. A nyilvános felhő képes kezelni a forgalom szezonális csúcsát, így az ügyfelek pontosan a szükséges hardverért fizetnek, amikor szükségük van rá.

## <a name="solution"></a>Megoldás

A megoldás kihasználja a magánfelhő megfelelőségi előnyeit, és kombinálja őket a nyilvános felhő méretezhetőségével. Az Azure és Azure Stack Hub hibrid felhő konzisztens élményt nyújt a fejlesztőknek. Ez a konzisztencia lehetővé teszi, hogy a nyilvános felhőben és a helyszíni környezetekben is alkalmazza a készségeit.

A megoldás üzembe helyezési útmutatója lehetővé teszi egy azonos webalkalmazás nyilvános és magánfelhőben való üzembe helyezését. A magánfelhőben üzemeltetett, nem internetes átirányítható hálózatokat is elérheti. A rendszer figyeli a webalkalmazások terhelését. A forgalom jelentős növekedése esetén a program ÚGY módosítja a DNS-rekordokat, hogy átirányítsa a forgalmat a nyilvános felhőbe. Ha a forgalom már nem jelentős, a DNS-rekordok frissülnek, hogy visszairányják a forgalmat a magánfelhőbe.

[![Felhők közötti skálázás a saját adatmintával](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Összetevők

Ez a megoldás a következő összetevőket használja:

| Réteg | Összetevő | Leírás |
|----------|-----------|-------------|
| Azure | Azure App Service | [Azure App Service](/azure/app-service/) lehetővé teszi webalkalmazások, RESTful API-alkalmazások és webalkalmazások Azure Functions. Mindezt egy ön által választott programozási nyelven, infrastruktúrakezelés nélkül. |
| | Azure Virtual Network| [Az Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) az Azure-beli magánhálózatok alapvető építőeleme. A VNet lehetővé teszi, hogy több Azure-erőforrástípus, például a virtuális gépek (VM) biztonságosan kommunikáljanak egymással, az internettel és a helyszíni hálózatokkal. A megoldás további hálózati összetevők használatát is bemutatja:<br>– alkalmazás- és átjáró-alhálózatok.<br>– helyi helyszíni hálózati átjáró.<br>– egy virtuális hálózati átjáró, amely hely–hely VPN Gateway-kapcsolatként működik.<br>– egy nyilvános IP-cím.<br>– pont–hely VPN-kapcsolat.<br>– Azure DNS DNS-tartományok üzemeltetése és a névfeloldás biztosítása. |
| | Azure Traffic Manager | [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview) egy DNS-alapú forgalom-terheléselosztási rendszer. Lehetővé teszi a felhasználói forgalom elosztásának szabályozását a különböző adatközpontokban a szolgáltatásvégpontokkal. |
| | Azure Application Insights | [Az Elemzések](/azure/azure-monitor/app/app-insights-overview) alkalmazásteljesítmény-kezelési szolgáltatás olyan webfejlesztők számára, akik több platformon építik fel és kezelik az alkalmazásokat.|
| | Azure Functions | [Azure Functions](/azure/azure-functions/) lehetővé teszi a kód kiszolgáló nélküli környezetben való végrehajtását anélkül, hogy először létre kell hoznia egy virtuális gépet, vagy közzé kell tennie egy webalkalmazást. |
| | Automatikus méretezés az Azure-ban | [Az automatikus skálázás](/azure/azure-monitor/platform/autoscale-overview) a virtuális gépek, Cloud Services és webalkalmazások beépített funkciója. A funkció lehetővé teszi, hogy az alkalmazások a lehető legjobban teljesítsen, amikor az igények megváltoznak. Az alkalmazások alkalmazkodnak a kiugró adatforgalomhoz, és értesítik, ha a metrikák változnak, és szükség szerint skáláznak. |
| Azure Stack Hub | IaaS-számítás | Azure Stack Hub lehetővé teszi, hogy ugyanazt az alkalmazásmodellt, önkiszolgáló portált és az Azure által engedélyezett API-kat használja. Azure Stack Hub IaaS nyílt forráskódú technológiák széles körét teszi lehetővé a konzisztens hibrid felhőalapú környezetek számára. A példamegoldás például egy Windows kiszolgálói virtuális gépet SQL Server virtuális géphez.|
| | Azure App Service | Az Azure-webalkalmazáshoz Azure App Service megoldás Azure Stack Hub [a](/azure-stack/operator/azure-stack-app-service-overview) webalkalmazást. |
| | Hálózatkezelés | A Azure Stack Hub Virtual Network pontosan úgy működik, mint az Azure Virtual Network. Számos azonos hálózati összetevőt használ, beleértve az egyéni állomásneveket is.
| Azure DevOps Services | Regisztráció | Gyorsan beállíthatja a folyamatos integrációt a buildhez, a teszteléshez és az üzembe helyezéshez. További információ: [Regisztráció, bejelentkezés az Azure DevOpsba.](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops) |
| | Azure Pipelines | Az [Azure Pipelines használata](/azure/devops/pipelines/agents/agents?view=azure-devops) folyamatos integrációhoz/folyamatos teljesítéshez. Az Azure Pipelines lehetővé teszi az üzemeltetett build- és kiadási ügynökök és definíciók kezelését. |
| | Kódtár | Több kódtárat is kihasználhat a fejlesztési folyamat leegyszerűsítéséhez. Meglévő kódtárak használata a GitHub, a Bitbucket, Dropbox, OneDrive és az Azure Repos szolgáltatásban. |

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

A megoldás megvalósításakor vegye figyelembe a következő pontokat:

### <a name="scalability"></a>Méretezhetőség

Az Azure Azure Stack Hub és a szolgáltatások egyedileg illeszkednek napjaink globálisan elosztott vállalkozásának igényeihez.

#### <a name="hybrid-cloud-without-the-hassle"></a>Hibrid felhő probléma nélkül

A Microsoft a helyszíni eszközök páratlan integrációját kínálja a Azure Stack Hub és az Azure-ral egyetlen egységes megoldásban. Ez az integráció kiküszöböli a többpontos megoldások és a felhőszolgáltatók vegyes kezelésével járó gondokat. A felhők közötti skálázásnak az Azure csupán néhány kattintással elérhető. Egyszerűen csatlakoztassa a Azure Stack Hub azure-hoz a felhőalapú adatlokozással, és az adatok és az alkalmazások szükség esetén elérhetők lesznek az Azure-ban.

- Nincs szükség másodlagos DR-hely építésére és karbantartására.
- Időt és pénzt takaríthat meg, ha megszünteti a szalagos biztonsági mentést, és akár 99 éves biztonsági mentési adatokat is ment az Azure-ban.
- A futó Hyper-V, Fizikai (előzetes verzió) és VMware (előzetes verzió) számítási feladatok egyszerűen mihetők az Azure-ba a felhő gazdasági és rugalmasságának kihasználása érdekében.
- A számítási feladatok befolyásolása nélkül futtathat nagy számítási igényű jelentéseket vagy elemzéseket a helyszíni adateszköz replikált példányán az Azure-ban.
- A felhőbe való úsítás és a helyszíni számítási feladatok futtatása az Azure-ban, szükség esetén nagyobb számítási sablonokkal. A Hybrid akkor biztosítja a szükséges teljesítményt, amikor szüksége van rá.
- Hozzon létre többrétegű fejlesztési környezeteket néhány kattintással az Azure-ban – akár az éles éles adatokat is replikálhatja a fejlesztési/tesztelési környezetbe, hogy közel valós idejű szinkronban tartsa őket.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>A felhők közötti skálázás gazdasági Azure Stack Hub

A felhőalapú kiesés fő előnye a gazdaságos megtakarítás. Csak akkor kell fizetnie a további erőforrásokért, ha az erőforrásokra igény van. Nincs több felesleges extra kapacitásra fordított kiadás, vagy a keresleti csúcsok és ingadozások előrejelzése.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>A felhőbe irányuló nagy terhelések csökkentése

A felhők közötti skálázás a feldolgozási terhek csökkentése érdekében használható. A terhelés úgy osztható el, hogy alapszintű alkalmazásokat a nyilvános felhőbe költöztet, és felszabadítja a helyi erőforrásokat az üzletileg kritikus alkalmazások számára. Egy alkalmazás alkalmazható a magánfelhőre, majd a nyilvános felhőbe való áttűnhet, ha az igényeknek megfelelően szükség van rá.

### <a name="availability"></a>Rendelkezésre állás

A globális üzembe helyezésnek saját kihívásai vannak, például a változó kapcsolatok és a régiókra vonatkozó különböző kormányzati szabályozások. A fejlesztők csak egy alkalmazást fejleszthet, majd különböző okokból különböző követelményekkel helyezhetik üzembe. Az alkalmazás üzembe helyezése a nyilvános Azure-felhőben, majd további példányok vagy összetevők helyi üzembe helyezése. Az Azure-ral kezelheti az összes példány közötti forgalmat.

### <a name="manageability"></a>Kezelhetőség

#### <a name="a-single-consistent-development-approach"></a>Egyetlen, konzisztens fejlesztési megközelítés

Az Azure Azure Stack Hub a teljes szervezetben egységes fejlesztői eszközöket használhat. Ez a konzisztencia megkönnyíti a folyamatos integráció és folyamatos fejlesztés (CI/CD) gyakorlatának implementációját. Az Azure-ban vagy az Azure-ban Azure Stack Hub alkalmazások és szolgáltatások felcserélhetők, és zökkenőmentesen futtathatók mindkét helyen.

A hibrid CI-/CD-folyamatok a következőben segíthetnek:

- Kezdeményezzen egy új buildet a kódtárban való kód véglegesítései alapján.
- Az újonnan létrehozott kód automatikus üzembe helyezése az Azure-ban a felhasználói elfogadás teszteléséhez.
- Miután a kód végzett a teszteléssel, automatikusan üzembe helyezheti Azure Stack Hub.

### <a name="a-single-consistent-identity-management-solution"></a>Egyetlen, konzisztens identitáskezelési megoldás

Azure Stack Hub az Azure Active Directory (Azure AD) és a Active Directory összevonási szolgáltatások (AD FS) (ADFS) szolgáltatásokkal is működik. Azure Stack Hub az Azure AD-val csatlakoztatott forgatókönyvekben. Az olyan környezetek esetében, amelyek nem csatlakoznak, az ADFS-t kapcsolat nélküli megoldásként használhatja. A szolgáltatásnévvel hozzáférést adhat az alkalmazásokhoz, így erőforrásokat helyezhet üzembe vagy konfigurálhet a Azure Resource Manager.

### <a name="security"></a>Biztonság

#### <a name="ensure-compliance-and-data-sovereignty"></a>A megfelelőség és az adatszuverenség biztosítása

Azure Stack Hub lehetővé teszi, hogy ugyanazt a szolgáltatást több országban is futtassa, mintha nyilvános felhőt használna. Ha ugyanazt az alkalmazást minden országban adatközpontokban telepíti, akkor teljesülnek az adatok elszuverentségre vonatkozó követelményei. Ez a funkció biztosítja, hogy a személyes adatok az egyes országok határain belül maradnak.

#### <a name="azure-stack-hub---security-posture"></a>Azure Stack Hub – biztonsági rendszer

A biztonság nem megfelelő, folyamatos karbantartási folyamat nélkül. Ezért a Microsoft egy olyan vezénylési motorba fektetett, amely zökkenőmentesen alkalmazza a javításokat és frissítéseket a teljes infrastruktúrára.

A hardvergyártói partnerekkel Azure Stack Hub partneri kapcsolatoknak köszönhetően a Microsoft ugyanezt a biztonsági rendszereket kiterjeszti az OEM-specifikus összetevőkre is, például a hardver életciklus-gazdaszámítógépére és a rajta futó szoftverekre. Ez a partneri kapcsolat Azure Stack Hub, hogy a teljes infrastruktúrában egységes, stabil biztonsági rendszer legyen. Az ügyfelek felépítik és biztonságossá is tehetik az alkalmazás számítási feladatait.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Szolgáltatásnév használata a PowerShell, a parancssori felület és a Azure Portal

Ha hozzáférést ad az erőforrásoknak egy szkripthez vagy alkalmazáshoz, állítson be egy identitást az alkalmazáshoz, és hitelesítse az alkalmazást a saját hitelesítő adataival. Ezt az identitást szolgáltatásnévnek is nevezik, és a következőt teszi lehetővé:

- Rendeljen olyan engedélyeket az alkalmazásidentitáshoz, amelyek eltérnek a saját engedélyeitől, és pontosan az alkalmazás igényeire vannak korlátozva.
- Tanúsítvány használata hitelesítéshez szkriptek felügyelet nélküli futtatásakor.

A szolgáltatásnév létrehozásával és a hitelesítő adatok tanúsítványának használatával kapcsolatos további információkért lásd: Alkalmazásidentitás használata [erőforrások eléréséhez.](/azure-stack/operator/azure-stack-create-service-principals)

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

- A szervezetem DevOps-megközelítést használ, vagy a közeljövőben tervez egyet.
- CI/CD-eljárásokat szeretnék megvalósítani a Azure Stack Hub és a nyilvános felhőben.
- Össze szeretném konszolidálni a CI-/CD-folyamatot a felhőben és a helyszíni környezetekben.
- Zökkenőmentesen szeretnék alkalmazásokat fejleszteni felhőalapú vagy helyszíni szolgáltatásokkal.
- Konzisztens fejlesztői készségeket szeretnék használni a felhőben és a helyszíni alkalmazásokban.
- Az Azure-t használom, de vannak fejlesztők, akik egy helyszíni felhőben Azure Stack Hub dolgoznak.
- A helyszíni alkalmazások igénycsúcsokat tapasztalnak a szezonális, ciklikus vagy kiszámíthatatlan ingadozások során.
- Helyszíni összetevőkkel is szeretnék együttműködni, és a felhővel szeretném zökkenőmentesen skálázni őket.
- Skálázhatóságot szeretnék a felhőben, de azt szeretném, hogy az alkalmazás a lehető legnagyobb mértékben fusson a helyszínen.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott témakörökről:

- Tekintse [meg az alkalmazások adatközpontok és](https://www.youtube.com/watch?v=2lw8zOpJTn0) nyilvános felhő közötti dinamikus skálázása témakört, amely áttekintést nyújt ennek a mintának a használatával.
- Az [ajánlott eljárásokról](overview-app-design-considerations.md) és az esetleg további kérdésekre adott válaszokért tekintse meg a hibrid alkalmazástervezési szempontokat.
- Ez a minta a Azure Stack, beleértve a Azure Stack Hub. A teljes [termék- Azure Stack termék-](/azure-stack) és megoldás-családról itt talál további információt.

Ha készen áll a megoldás példának tesztelésére, folytassa a felhők közötti skálázás [(helyszíni adatok)](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data)megoldás üzembe helyezési útmutatóját. Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez.