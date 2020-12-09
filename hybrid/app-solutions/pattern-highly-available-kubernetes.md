---
title: Magas rendelkezésre állású Kubernetes minta az Azure és Azure Stack hub használatával
description: Megtudhatja, hogyan biztosít magas rendelkezésre állást a Kubernetes-fürt számára az Azure és a Azure Stack hub használatával.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 454cc0a0531882b7a8ec050a461420ce13eebcfe
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911792"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Magas rendelkezésre állású Kubernetes-fürt mintája

Ez a cikk azt ismerteti, hogyan lehet az Azure Stack hub Azure Kubernetes Service (ak) motorjának használatával egy magasan elérhető Kubernetes-alapú infrastruktúrát feldolgozni és működtetni. Ez a forgatókönyv gyakori a kritikus fontosságú számítási feladatokkal rendelkező vállalatok számára, amelyek nagy mértékben korlátozott és szabályozott környezetekben vannak. Többek között pénzügyi, védelmi és kormányzati szervezetek.

## <a name="context-and-problem"></a>Kontextus és probléma

Számos szervezet olyan felhőalapú megoldásokat fejleszt, amelyek a legkorszerűbb szolgáltatásokat és technológiákat, például a Kubernetes használják. Bár az Azure a világ legtöbb régiójában biztosít adatközpontokat, időnként előfordulnak olyan Edge-használati esetek és forgatókönyvek, ahol az üzleti szempontból kritikus fontosságú alkalmazásokat egy adott helyen kell futtatni. A szempontok a következők:

- Hely érzékenysége
- Az alkalmazás és a helyszíni rendszerek közötti késés
- Sávszélesség-megőrzés
- Kapcsolatok
- Szabályozási vagy jogszabályi követelmények

Az Azure az Azure Stack hub szolgáltatással együtt a legtöbb ilyen problémával foglalkozik. Az alábbiakban részletesen ismertetjük az Azure Stack hub-on futó Kubernetes sikeres megvalósításának lehetőségeit, döntéseit és szempontjait.

## <a name="solution"></a>Megoldás

Ez a minta azt feltételezi, hogy a kényszerek szigorú készletét kell kezelni. Az alkalmazásnak a helyszínen kell futnia, és az összes személyes adattal nem szabad a nyilvános felhőalapú szolgáltatásokhoz jutnia. A monitorozási és egyéb nem személyes adatok az Azure-ba küldhetők, és ott is feldolgozhatók. Az olyan külső szolgáltatások, mint például a nyilvános Container Registry vagy mások is elérhetők, de tűzfalon vagy proxykiszolgálón keresztül is szűrhetők.

Az itt látható minta alkalmazás (az [Azure Kubernetes Service workshop](/learn/modules/aks-workshop/)alapján) úgy van kialakítva, hogy Kubernetes-natív megoldásokat használjon, amikor csak lehetséges. Ez a kialakítás elkerüli a szállítói zárolást a platform-natív szolgáltatások használata helyett. Az alkalmazás például egy saját üzemeltetésű MongoDB adatbázis-hátteret használ a Pásti szolgáltatás vagy külső adatbázis-szolgáltatás helyett.

[![Alkalmazás minta hibrid](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

Az előző ábrán az Azure Stack hub Kubernetes futó minta alkalmazás architektúrája látható. Az alkalmazás több összetevőből áll, beleértve a következőket:

 1) Egy AK-alapú motoron alapuló Kubernetes-fürt Azure Stack központban.
 2) tanúsítvány [-kezelő](https://www.jetstack.io/cert-manager/), amely a Kubernetes lévő tanúsítványkezelő eszközeit biztosítja a tanúsítványok automatikus igényléséhez a titkosításhoz.
 3) Egy Kubernetes-névtér, amely az előtér (Ratings-web), az API (Ratings-API) és az adatbázis (Ratings-mongodb) alkalmazás-összetevőit tartalmazza.
 4) A bejövő vezérlő, amely a HTTP/HTTPS-forgalmat a Kubernetes-fürtön belüli végpontokra irányítja.

A minta alkalmazás az alkalmazás architektúrájának szemléltetésére szolgál. Az összes összetevő példa. Az architektúra csak egyetlen alkalmazás-telepítést tartalmaz. Ha magas rendelkezésre állást szeretne elérni, legalább kétszer futtatjuk az üzembe helyezést két különböző Azure Stack hub-példányon – ezek futhatnak ugyanazon a helyen, vagy két (vagy több) különböző helyen:

![Infrastruktúra-architektúra](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Az olyan szolgáltatásokat, mint például a Azure Container Registry, a Azure Monitor és más szolgáltatások, az Azure-ban vagy a helyszínen található Azure Stack hub-on kívül futnak. Ez a hibrid kialakítás védi a megoldást egyetlen Azure Stack hub-példány KIMARADÁSÁVAL szemben.

## <a name="components"></a>Összetevők

A teljes architektúra a következő összetevőkből áll:

Az **Azure stack hub** az Azure egy bővítménye, amely a munkaterhelések helyszíni környezetben való futtatását teszi lehetővé az adatközpontban elérhető Azure-szolgáltatások biztosításával. További információért látogasson el a [Azure stack hub áttekintésére](/azure-stack/operator/azure-stack-overview) .

Az Azure **Kubernetes Service Engine (ak motor)** a felügyelt Kubernetes Service-ajánlat, az Azure Kubernetes Service (ak) mögötti motor, amely jelenleg az Azure-ban érhető el. Azure Stack hub esetében az AK motorja lehetővé teszi, hogy a Azure Stack hub IaaS képességeivel teljes funkcionalitású, önállóan felügyelt Kubernetes-fürtöket helyezzen üzembe, méretezhetin és frissítsen. További információt a következő témakörben talál: az [AK-motor áttekintése](https://github.com/Azure/aks-engine) .

Ismerje meg az [ismert problémákat és korlátozásokat, és](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) Ismerkedjen meg a Azure stack hub-beli, az Azure-beli és az AK-beli motorok közötti különbségekkel.

Az **Azure Virtual Network (VNet)** a hálózati infrastruktúra biztosítására szolgál az egyes Azure stack hubokon a Kubernetes-fürt infrastruktúráját üzemeltető Virtual Machines (VM) esetében.

**Azure Load Balancer** a Kubernetes API-végponthoz és az Nginx beáramló vezérlőhöz használható. A terheléselosztó külső (például internetes) forgalmat irányít át egy adott szolgáltatást biztosító csomópontokra és virtuális gépekre.

A **Azure Container Registry (ACR)** a fürtön üzembe helyezett privát Docker-rendszerképek és Helm-diagramok tárolására szolgál. Az AK-motor az Azure AD-identitás használatával tud hitelesíteni a Container Registry. A Kubernetes nem igényel ACR-t. Más tároló-nyilvántartásokat is használhat, például a Docker hub-t.

Az **Azure Repos** olyan verziókövetés-eszközök összessége, amelyek segítségével kezelheti a kódot. A GitHub vagy más git-alapú Tárházak is használhatók. További információért látogasson el az [Azure Repos áttekintése oldalra](/azure/devops/repos/get-started/what-is-repos) .

Az **Azure-folyamatok** az Azure DevOps-szolgáltatások részét képezik, és automatizált buildeket, teszteket és üzembe helyezéseket futtatnak. Harmadik féltől származó CI/CD-megoldásokat is használhat, például a Jenkinst. További információért látogasson el az [Azure-folyamatok áttekintésére](/azure/devops/pipelines/get-started/what-is-azure-pipelines) .

**Azure monitor** gyűjti és tárolja a mérőszámokat és naplókat, beleértve az Azure-szolgáltatásokra vonatkozó platform-mérőszámokat a megoldásban és az alkalmazás telemetria. Ezekkel az adatokkal figyelheti az alkalmazást, riasztásokat és irányítópultokat állíthat be, és a hibák kiváltó okának elemzését végezheti el. A Azure Monitor együttműködik a Kubernetes a vezérlők, a csomópontok és a tárolók, valamint a tároló-naplók és a fő csomópont-naplók metrikáinak összegyűjtéséhez. További információért [tekintse meg Azure monitor áttekintést](/azure/azure-monitor/overview) .

Az **azure Traffic Manager** egy DNS-alapú forgalom terheléselosztó, amely lehetővé teszi a forgalom optimális elosztását különböző Azure-régiókba vagy Azure stack hub-környezetek szolgáltatásaiba. A Traffic Manager magas rendelkezésre állást és válaszadást is biztosít. Az alkalmazás-végpontoknak elérhetőnek kell lenniük a kívülről. Más helyszíni megoldások is elérhetők.

A **Kubernetes bejövő** adatforgalom-vezérlő http (S) útvonalakat tesz elérhetővé egy Kubernetes-fürt szolgáltatásai számára. Erre a célra az Nginx vagy bármely alkalmas bejövő vezérlő használható.

A **Helm** egy Kubernetes üzembe helyezésére szolgáló csomagkezelő, amely lehetővé teszi különböző Kubernetes-objektumok, például üzembe helyezések, szolgáltatások, titkos kódok egyetlen "diagramba" való beépítését. Közzéteheti, telepítheti, vezérelheti a verziószámozást, és frissítheti a diagram objektumait. A Azure Container Registry használható tárházként a csomagolt Helm-diagramok tárolásához.

## <a name="design-considerations"></a>Kialakítási szempontok

Ez a minta néhány olyan magas szintű szempontot követ, amely részletesebben ismerteti a cikk következő részében ismertetett lépéseket:

- Az alkalmazás Kubernetes-natív megoldásokat használ a szállítók zárolásának elkerülése érdekében.
- Az alkalmazás egy Services architektúrát használ.
- Azure Stack hub nem igényel bejövő kapcsolatot, de engedélyezi a kimenő internetkapcsolatot.

Ezek a javasolt eljárások a valós munkaterhelésekre és forgatókönyvekre is érvényesek lesznek.

## <a name="scalability-considerations"></a>Méretezési szempontok

A méretezhetőség fontos, hogy a felhasználók konzisztens, megbízható és jól teljesítő hozzáférést nyújtsanak az alkalmazáshoz.

A minta forgatókönyv az alkalmazás-verem több rétegének méretezhetőségét fedi le. Az alábbiakban áttekintheti a különböző rétegeket:

| Architektúra szintje | Befolyásolja | Hogyan? |
| --- | --- | ---
| Alkalmazás | Alkalmazás | Vízszintes skálázás a hüvelyek/replikák/Container Instances * száma alapján |
| Fürt | Kubernetes-fürt | A csomópontok száma (1 és 50 között), VM-SKU-size és Node-készletek (az Azure Stack hub-on jelenleg csak egyetlen csomópontos készletet támogat); az KABAi motor méretezési parancsának használata (manuális) |
| Infrastruktúra | Azure Stack Hub | A csomópontok, a kapacitás és a skálázási egységek száma egy Azure Stack hub üzemelő példányon belül |

\* A Kubernetes horizontális Pod autoskálázás (HPA) használata; Automatikus metrika-alapú skálázás vagy vertikális skálázás a Container instances (CPU/memória) méretezésével.

**Azure Stack hub (infrastruktúra szintje)**

Az Azure Stack hub-infrastruktúra ennek a megvalósításnak az alapja, mivel Azure Stack hub fizikai hardveren fut egy adatközpontban. A hub hardver kiválasztásakor meg kell adnia a CPU-t, a memória sűrűségét, a tárolási konfigurációt és a kiszolgálók számát. Ha többet szeretne megtudni a Azure Stack hub méretezhetőségéről, tekintse meg a következő forrásokat:

- [Azure Stack hub kapacitásának tervezése – áttekintés](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [További skálázási egység csomópontjainak hozzáadása Azure Stack hub-ban](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-fürt (fürt szintje)**

Maga a Kubernetes-fürt áll, és az Azure (stack) IaaS-összetevőkre épül, beleértve a számítási, tárolási és hálózati erőforrásokat. A Kubernetes-megoldások fő-és munkavégző csomópontokat tartalmaznak, amelyek az Azure-ban (és Azure Stack hub-ban) virtuális gépekként vannak üzembe helyezve.

- A [vezérlési sík csomópontjai](/azure/aks/concepts-clusters-workloads#control-plane) (Master) biztosítják a legfontosabb Kubernetes szolgáltatásokat és az alkalmazások számítási feladatainak előkészítését.
- [Munkavégző csomópontok](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (feldolgozó) futtatják az alkalmazás számítási feladatait.

A virtuális gépek méretének kiválasztásakor a kezdeti üzembe helyezés több szempontot is figyelembe vesz:  

- **Költség** – a munkavégző csomópontok tervezésekor vegye figyelembe, hogy a virtuális gépen felmerülő teljes költség. Ha például az alkalmazás számítási feladatának korlátozott erőforrásokra van szüksége, érdemes kisebb méretű virtuális gépeket telepítenie. Az Azure Stack hub (például az Azure) általában fogyasztási alapon kerül számlázásra, ezért a virtuális gépek megfelelő méretezése a Kubernetes-szerepkörökhöz elengedhetetlen a használati költségek optimalizálásához. 

- **Méretezhetőség** – a fürt méretezhetősége a fő-és munkavégző csomópontok számának és a feldolgozói csomópontok számának skálázásával, vagy további Node-készletek hozzáadásával érhető el (a mai Azure stack hub-on nem érhetők el). A fürt méretezése a teljesítményadatok alapján, a tároló-elemzések (Azure Monitor + Log Analytics) használatával végezhető el. 

    Ha az alkalmazásnak több (vagy kevesebb) erőforrásra van szüksége, horizontálisan kibővítheti az aktuális csomópontokat (vagy a-ben) vízszintesen (1 és 50 csomópont között). Ha 50-nél több csomópontra van szüksége, hozzon létre egy további fürtöt egy külön előfizetésben. A tényleges virtuális gépeket nem lehet függőlegesen átméretezni egy másik virtuálisgép-méretre a fürt újbóli üzembe helyezése nélkül.

    A skálázást manuálisan, a Kubernetes-fürt üzembe helyezéséhez használt AK Engine Helper VM használatával végezheti el. További információ: Kubernetes- [fürtök skálázása](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Kvóták** – vegye figyelembe azokat a [kvótákat](/azure-stack/operator/azure-stack-quota-types) , amelyeket az AK-beli központi telepítés tervezésekor konfigurált a Azure stack hubhoz. Győződjön meg arról, hogy minden [előfizetés](/azure-stack/operator/service-plan-offer-subscription-overview) rendelkezik a megfelelő csomagokkal és a beállított kvótákkal. Az előfizetésnek el kell fogadnia a fürtökhöz szükséges számítási, tárolási és egyéb szolgáltatások mennyiségét.

- **Alkalmazás-munkaterhelések** – tekintse meg a [fürtök és a számítási feladatok fogalmait](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) az Azure Kubernetes-Kubernetes alapvető fogalmait ismertető témakörben. Ez a cikk segítséget nyújt a megfelelő virtuálisgép-méret hatókörében az alkalmazás számítási és memóriabeli igényei alapján.  

**Alkalmazás (alkalmazás szintje)**

Az alkalmazási rétegben a Kubernetes [horizontális Pod automéretezőt (hPa)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)használjuk. A HPA a különböző metrikák (például a CPU-kihasználtság) alapján növelheti vagy csökkentheti az üzemelő példányok számát (Pod/Container Instances).

Egy másik lehetőség a tároló-példányok függőleges méretezése. Ezt úgy teheti meg, hogy megváltoztatja a kért és elérhető CPU és memória mennyiségét egy adott telepítéshez. További információkért lásd: [tárolók erőforrásainak kezelése](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) a kubernetes.IO-on.

## <a name="networking-and-connectivity-considerations"></a>Hálózatkezelés és kapcsolódási megfontolások

A Hálózatkezelés és a kapcsolat is befolyásolja az Azure Stack hub Kubernetes korábban említett három réteget. A következő táblázat a rétegek és a bennük található szolgáltatásokat tartalmazza:

| Réteg | Befolyásolja | Mi? |
| --- | --- | ---
| Alkalmazás | Alkalmazás | Hogyan érhető el az alkalmazás? Elérhető lesz az Internet? |
| Fürt | Kubernetes-fürt | Kubernetes API, KABAi motor VM, tároló lemezképek húzása (kimenő), figyelési adatok küldése és telemetria (kimenő forgalom) |
| Infrastruktúra | Azure Stack Hub | Az Azure Stack hub felügyeleti végpontok, például a portál és a Azure Resource Manager végpontok hozzáférhetősége. |

**Alkalmazás**

Az alkalmazási réteg esetében a legfontosabb szempont, hogy az alkalmazás elérhető-e az internetről. A Kubernetes perspektívában az Internet kisegítő lehetőségek az üzemelő példányok vagy a hüvelyek Kubernetes szolgáltatással vagy bejövő vezérlővel való kihelyezését jelentik.

> [!NOTE]
> Azt javasoljuk, hogy a beáramlási vezérlők használatával tegye közzé a Kubernetes szolgáltatásait, mivel a Azure Stack hub-beli előtér nyilvános IP-címeinek száma legfeljebb 5. Ez a kialakítás azt is korlátozza, hogy a Kubernetes szolgáltatások száma (a terheléselosztó típussal együtt) 5, ami túl kicsi lesz a sok üzemelő példány esetében. További információért nyissa meg az [AK-motor dokumentációját](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) .

A nyilvános IP-címet használó alkalmazások Load Balancer vagy egy bejövő vezérlőn keresztül nem nessecarily azt jelenti, hogy az alkalmazás most már elérhető az interneten keresztül. Lehetséges, hogy Azure Stack hub olyan nyilvános IP-címmel rendelkezik, amely csak a helyi intraneten látható – nem minden nyilvános IP-cím valóban internetre irányul.

Az előző blokk az alkalmazás felé irányuló forgalmat veszi át. Egy másik témakör, amelyet a sikeres Kubernetes-telepítésnek figyelembe kell vennie kimenő/kimenő forgalom. Íme néhány olyan használati eset, amely a kimenő forgalmat igényli:

- DockerHub vagy Azure Container Registry tárolt tároló-lemezképek húzása
- Helm-diagramok beolvasása
- AdatApplication Insights-adatkibocsátás (vagy más figyelési adatmennyiség)

Előfordulhat, hogy egyes vállalati környezetek _transzparens_ vagy _nem transzparens_ proxykiszolgálók használatát igénylik. Ezeknek a kiszolgálóknak speciális konfigurációra van szükségük a fürt különböző összetevőin. Az AK-motor dokumentációja a hálózati proxyk befogadásának részletes ismertetését tartalmazza. További részletek: [AK-motor és proxykiszolgáló](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Végül a fürt közötti forgalomnak Azure Stack hub-példányok között kell haladnia. A minta üzembe helyezése különálló, Azure Stack hub-példányokon futó Kubernetes-fürtökből áll. A közöttük lévő forgalom, például a két adatbázis közötti replikációs forgalom "külső forgalom". A külső forgalmat helyek közötti VPN-vagy Azure Stack hub nyilvános IP-címekről kell irányítani a Kubernetes két Azure Stack hub-példányon való összekapcsolásához:

![csomópontok közötti és belüli forgalom](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Csoport**  

A Kubernetes-fürtnek nem feltétlenül kell elérhetőnek lennie az interneten keresztül. Az érintett rész a fürt üzemeltetéséhez használt Kubernetes API, például a használatával `kubectl` . A Kubernetes API-végpontnak elérhetőnek kell lennie mindenki számára, aki üzemelteti a fürtöt, vagy alkalmazásokat és szolgáltatásokat helyez üzembe. Ezt a témakört részletesebben az [üzembe helyezési (CI/CD) szempontokat](#deployment-cicd-considerations) ismertető szakasz DevOps tárgyalja.

A fürt szintjén a kimenő forgalomról is van néhány szempont:

- Csomópont-frissítések (Ubuntu)
- Figyelési adatszolgáltatások (az Azure LogAnalytics-ben elküldve)
- Egyéb, kimenő forgalmat igénylő ügynökök (az egyes telepítők környezete esetében)

Mielőtt üzembe helyezi a Kubernetes-fürtöt az AK Engine használatával, tervezze meg a végső hálózatkezelési tervet. A dedikált Virtual Network létrehozása helyett hatékonyabb lehet a fürt üzembe helyezése egy meglévő hálózatban. Használhat például egy meglévő, a Azure Stack hub-környezetben már konfigurált, két hálózat közötti pont-pont típusú VPN-kapcsolat használatát.

**Infrastruktúra**  

Az infrastruktúra az Azure Stack hub felügyeleti végpontok elérésére utal. A végpontok közé tartoznak a bérlői és a felügyeleti portálok, valamint a Azure Resource Manager felügyeleti és bérlői végpontok. Ezek a végpontok a Azure Stack hub és az alapvető szolgáltatásainak üzemeltetéséhez szükségesek.

## <a name="data-and-storage-considerations"></a>Az adatkezeléssel és a tárolással kapcsolatos megfontolások

Az alkalmazás két példánya lesz üzembe helyezve két különálló Kubernetes-fürtön két Azure Stack hub-példányon. Ennek a kialakításnak meg kell fontolnia, hogyan replikálja és szinkronizálja az adatokat közöttük.

Az Azure-ban beépített képességgel rendelkezünk a tárterület több régióban és zónán belüli replikálásához a felhőben. Jelenleg Azure Stack hub-ban nincsenek natív módon replikálni a tárterületet két különböző Azure Stack hub-példány között – két független felhőt alkotnak, amelyek nincsenek átfogó módon kezelve a készletként. Az Azure Stack hub-on futó alkalmazások rugalmasságának megtervezése arra kényszeríti, hogy az alkalmazás megtervezésében és üzembe helyezésében vegye figyelembe ezt a függetlenséget.

A legtöbb esetben a tárolók replikálására nem lesz szükség az AK-on üzembe helyezett rugalmas és nagy rendelkezésre állású alkalmazásokhoz. Az alkalmazás megtervezése során azonban Azure Stack hub-példányon kell megfontolnia a független tárterületet. Ha ez a kialakítás a megoldás Azure Stack hub-on való üzembe helyezésére vonatkozó aggályos vagy országúti blokk, a Microsoft partnereinek lehetséges megoldásai vannak, amelyek tárolási mellékleteket biztosítanak. A tárolási mellékletek tárolási replikációs megoldást biztosítanak több Azure Stack hub és az Azure között. További információ: [partneri megoldások](#partner-solutions).

Architektúránk esetében a következő rétegeket vették figyelembe:

**Konfigurálás**

A konfiguráció magában foglalja az Azure Stack hub, az AK motor és a Kubernetes-fürt konfigurációját. A konfigurációnak a lehető legszélesebb körűen kell lennie, és a git-alapú verziókövető rendszer, például az Azure DevOps vagy a GitHub esetében az infrastruktúra-kódként kell tárolnia. Ezeket a beállításokat nem lehet egyszerűen szinkronizálni több üzemelő példány között. Ezért javasoljuk, hogy a konfigurációt kívülről és a DevOps folyamat használatával tárolja és alkalmazza.

**Alkalmazás**

Az alkalmazást git-alapú tárházban kell tárolni. Ha új központi telepítésre kerül sor, az alkalmazás változásai vagy a vész-helyreállítási folyamat egyszerűen üzembe helyezhető az Azure-folyamatok használatával.

**Adatok**

Az alkalmazások többsége a legfontosabb szempont. Az alkalmazásadatok szinkronban kell maradnak az alkalmazás különböző példányai között. Az adatvesztés miatt a biztonsági mentési és vész-helyreállítási stratégia is szükséges.

Ennek a kialakításnak a megvalósítása nagy mértékben függ a technológiai lehetőségektől. Íme néhány példa arra, hogyan implementálhat egy adatbázist a Azure Stack hub-ban található, magasan elérhető módon:

- [SQL Server 2016 rendelkezésre állási csoport üzembe helyezése az Azure-ban és Azure Stack hub-ban](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Magasan elérhető MongoDB-megoldás üzembe helyezése az Azure-ban és Azure Stack hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

A több helyen tárolt adatkezelési megfontolások még összetettebbek a nagyfokú rendelkezésre állású és rugalmas megoldás érdekében. Megfontolandó szempontok:

- Azure Stack hubok közötti késés és hálózati kapcsolat.
- A szolgáltatások és engedélyek identitásának rendelkezésre állása. Minden Azure Stack hub-példány egy külső címtárral integrálódik. Az üzembe helyezés során a Azure Active Directory (Azure AD) vagy a Active Directory összevonási szolgáltatások (AD FS) (ADFS) használatát választja. Ilyen esetben egyetlen identitást használhat, amely több független Azure Stack hub-példánnyal is képes együttműködni.

## <a name="business-continuity-and-disaster-recovery"></a>Folyamatos üzletmenet és vészhelyreállítás

Az üzletmenet-folytonosság és a vész-helyreállítás (BCDR) fontos témakör a Azure Stack hub és az Azure között. A fő különbség az, hogy Azure Stack központban az operátornak a teljes BCDR folyamatot kell kezelnie. Az Azure-ban a BCDR egyes részeit a Microsoft automatikusan kezeli.

A BCDR az előző szakaszban említett, az [adatkezelési és a tárolási szempontokat](#data-and-storage-considerations)is érinti:

- Infrastruktúra/konfiguráció
- Alkalmazás rendelkezésre állása
- Alkalmazásadatok

Az előző szakaszban említettek szerint ezek a területek a Azure Stack hub-kezelő feladata, és a szervezetek között változhatnak. Tervezze meg a BCDR az elérhető eszközök és folyamatok alapján.

**Infrastruktúra és konfiguráció**

Ez a szakasz az Azure Stack hub fizikai és logikai infrastruktúráját és konfigurációját ismerteti. A rendszergazda és a bérlői terek műveleteit fedi le.

Az Azure Stack hub operátor (vagy a rendszergazda) felelős az Azure Stack hub-példányok fenntartásáért. Olyan összetevőket is beleértve, mint például a hálózat, a tárolás, az identitás és más, a jelen cikk hatálya alá eső témakörök. Ha többet szeretne megtudni az Azure Stack hub-műveletek jellemzőiről, tekintse meg a következő forrásokat:

- [Azure Stack hub adatainak helyreállítása a Infrastructure Backup szolgáltatással](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Biztonsági mentés engedélyezése Azure Stack hub számára a felügyeleti portálról](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Helyreállítás végzetes adatvesztés esetén](/azure-stack/operator/azure-stack-backup-recover-data)
- [Infrastructure Backup Service – ajánlott eljárások](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack hub az a platform és háló, amelyen a Kubernetes-alkalmazásokat telepíteni fogja. A Kubernetes alkalmazás tulajdonosa a Azure Stack hub felhasználója lesz, és hozzáférést biztosít a megoldáshoz szükséges alkalmazás-infrastruktúra telepítéséhez. Az alkalmazás-infrastruktúra ebben az esetben a Kubernetes-fürtöt jelenti, amely az AK motorral és a környező szolgáltatásokkal van üzembe helyezve. Ezek az összetevők Azure Stack hubhoz lesznek telepítve, amelyet egy Azure Stack hub-ajánlat korlátoz. Győződjön meg arról, hogy a Kubernetes alkalmazás tulajdonosa által elfogadott ajánlat elegendő kapacitással rendelkezik (Azure Stack hub-kvótákban kifejezve) a teljes megoldás üzembe helyezéséhez. Ahogy azt az előző szakaszban is javasolta, az alkalmazás központi telepítésének automatizáltnak kell lennie az infrastruktúra-kód és az üzembe helyezési folyamatok (például az Azure DevOps Azure-folyamatok) használatával.

Az Azure Stack hub-ajánlatokkal és-kvótákkal kapcsolatos további információkért lásd: [Azure stack hub-szolgáltatások, csomagok, ajánlatok és előfizetések áttekintése](/azure-stack/operator/service-plan-offer-subscription-overview)

Fontos, hogy biztonságosan mentse és tárolja az AK-motor konfigurációját, beleértve a kimeneteit is. Ezek a fájlok a Kubernetes-fürt eléréséhez használt bizalmas információkat tartalmaznak, ezért védeni kell őket a nem rendszergazdák számára.

**Alkalmazás rendelkezésre állása**

Az alkalmazásnak nem szabad felhasználnia egy telepített példány biztonsági mentését. Általános gyakorlatként telepítse újra az alkalmazást az infrastruktúra-programkódok szerinti mintázatok után. Például telepítse újra az Azure DevOps Azure-folyamatokat. A BCDR eljárásnak magában kell foglalnia az alkalmazás újratelepítését ugyanarra vagy egy másik Kubernetes-fürtre.

**Alkalmazásadatok**

Az alkalmazásadatok az adatvesztés minimalizálásának kritikus részét képezik. Az előző szakaszban ismertetjük az alkalmazás két (vagy több) példánya közötti replikálási és szinkronizálási technikákat. Az adatok tárolásához használt adatbázis-infrastruktúrától (MySQL, MongoDB, MSSQL vagy más) függően különböző adatbázis-rendelkezésre állási és biztonsági mentési technikák választhatók ki.

Az integritás elérésének ajánlott módja a következők használata:
- Natív biztonsági mentési megoldás az adott adatbázishoz.
- Olyan biztonsági mentési megoldás, amely hivatalosan támogatja az alkalmazás által használt adatbázis-típus biztonsági mentését és helyreállítását.

> [!IMPORTANT]
> Ne tárolja a biztonsági mentési adatait ugyanarra a Azure Stack hub-példányra, ahol az alkalmazás adatai találhatók. Az Azure Stack hub-példány teljes kimaradása szintén veszélyezteti a biztonsági mentéseket.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az AK-n keresztül üzembe helyezett Azure Stack hub Kubernetes nem felügyelt szolgáltatás. Ez egy Kubernetes-fürt automatikus üzembe helyezése és konfigurálása az Azure infrastruktúra-szolgáltatás (IaaS) használatával. Ennek megfelelően a mögöttes infrastruktúrával megegyező rendelkezésre állást biztosít.

Azure Stack hub-infrastruktúra már rugalmas a hibákhoz, és olyan képességeket biztosít, mint például a rendelkezésre állási csoportok, amelyek lehetővé teszi az összetevők több [hiba és frissítési tartomány](/azure-stack/user/azure-stack-vm-considerations#high-availability)közötti elosztását Az alapul szolgáló technológia (feladatátvételi fürtszolgáltatás) azonban továbbra is leállást okoz a virtuális gépeken egy érintett fizikai kiszolgálón, ha hardverhiba van.

Célszerű üzembe helyezni az üzemi Kubernetes-fürtöt, valamint a munkaterhelést két (vagy több) fürtön. Ezeket a fürtöket különböző helyeken vagy adatközpontokban kell üzemeltetni, és olyan technológiákat kell használni, mint az Azure Traffic Manager a felhasználók a fürt válaszideje vagy földrajz alapján történő továbbítására.

![A Traffic Manager használata a forgalmi folyamatok vezérléséhez](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Azok az ügyfelek, akik egyetlen Kubernetes-fürttel rendelkeznek, általában egy adott alkalmazás szolgáltatás IP-címéhez vagy DNS-nevéhez csatlakoznak. Több fürtből álló telepítés esetén az ügyfeleknek csatlakozniuk kell egy Traffic Manager DNS-névhez, amely az egyes Kubernetes-fürtök szolgáltatásaira/beléptetésére mutat.

![Traffic Manager használata a helyszíni fürtre való átirányításhoz](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Ez a minta az [Azure-beli (felügyelt) AK-fürtök esetében is ajánlott eljárás](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment).

Maga a Kubernetes-fürt, amely az KABAi motoron keresztül lett üzembe helyezve, legalább három fő csomópontból és két feldolgozó csomópontból állhat.

## <a name="identity-and-security-considerations"></a>Identitás-és biztonsági megfontolások

Az identitás és a biztonság fontos témakörök. Különösen akkor, ha a megoldás független Azure Stack hub-példányokra terjed ki. A Kubernetes és az Azure (beleértve az Azure Stack hubot is) különböző mechanizmusokkal rendelkezik a szerepköralapú hozzáférés-vezérléshez (RBAC):

- Az Azure RBAC az Azure-ban (és Azure Stack hub-ban) lévő erőforrásokhoz való hozzáférést szabályozza, beleértve az új Azure-erőforrások létrehozásának lehetőségét is. Engedélyek rendelhetők felhasználókhoz, csoportokhoz vagy egyszerű szolgáltatásokhoz. (Az egyszerű szolgáltatásnév az alkalmazások által használt biztonsági identitás.)
- A Kubernetes RBAC a Kubernetes API-ra irányítja az engedélyeket. A hüvelyek és a hüvelyek listázása például olyan műveletek, amelyek a RBAC használatával engedélyezhetők (vagy megtagadhatják) a felhasználók számára. Ahhoz, hogy Kubernetes engedélyeket rendeljen a felhasználókhoz, létre kell hoznia a szerepköröket és a szerepkör-kötéseket.

**Azure Stack hub identitás-és RBAC**

Azure Stack hub két identitás-szolgáltatói lehetőséget biztosít. A használt szolgáltató a környezettől függ, attól függetlenül, hogy csatlakoztatott vagy leválasztott környezetben fut-e:

- Az Azure AD-t csak csatlakoztatott környezetben lehet használni.
- Az ADFS-t egy hagyományos Active Directory erdőhöz is használhatja, amely csatlakoztatott vagy leválasztott környezetben is használható.

Az identitás-szolgáltató kezeli a felhasználókat és csoportokat, beleértve a hitelesítést és az erőforrások elérésének engedélyezését. Hozzáférés adható Azure Stack hub-erőforrásokhoz, például előfizetésekhez, erőforráscsoportokhoz és egyéni erőforrásokhoz, például virtuális gépekhez vagy terheléselosztóhoz. Ha konzisztens hozzáférési modellt szeretne használni, érdemes megfontolnia, hogy ugyanazt a csoportot használja (közvetlen vagy beágyazott) az összes Azure Stack-hubhoz. Íme egy konfigurációs példa:

![beágyazott HRE-csoportok az Azure stack hub-vel](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

A példa egy dedikált csoportot tartalmaz (HRE vagy ADFS használatával) egy adott célra. Ha például a Kubernetes-fürt infrastruktúráját tartalmazó erőforráscsoporthoz közreműködői engedélyeket szeretne adni egy adott Azure Stack hub-példányon (itt "Seattle K8s cluster közreműködő"). Ezeket a csoportokat ezután egy teljes csoportba ágyazzák be, amely az egyes Azure Stack hubok "alcsoportait" tartalmazza.

A minta felhasználónk immár "közreműködői" engedélyekkel rendelkezik mindkét olyan erőforrás-csoporthoz, amely a Kubernetes-infrastruktúra teljes készletét tartalmazza. A felhasználó Azure Stack hub-példányokon is hozzáférhet az erőforrásokhoz, mert a példányok ugyanazzal az identitás-szolgáltatóval rendelkeznek.

> [!IMPORTANT]
> Ezek az engedélyek csak Azure Stack központot és a rajta üzembe helyezett erőforrásokat érintik. Az ilyen szintű hozzáféréssel rendelkező felhasználók nagy károkat okozhatnak, de nem férhetnek hozzá a Kubernetes IaaS virtuális gépekhez és a Kubernetes API-hoz anélkül, hogy további hozzáférésre lenne szükség a Kubernetes-telepítéshez.

**Kubernetes-identitás és RBAC**

A Kubernetes-fürt alapértelmezés szerint nem ugyanazt az identitás-szolgáltatót használja, mint a Azure Stack hub. A Kubernetes-fürtöt, a főkiszolgálót és a feldolgozó csomópontokat futtató virtuális gépek a fürt központi telepítése során megadott SSH-kulcsot használják. Ez az SSH-kulcs szükséges ahhoz, hogy SSH-kapcsolaton keresztül csatlakozhasson ezekhez a csomópontokhoz.

A Kubernetes API-t (például a használatával érhető el `kubectl` ) a szolgáltatásfiók is védi, beleértve az alapértelmezett "Fürtfelügyelő" szolgáltatásfiókot is. A szolgáltatásfiók hitelesítő adatait kezdetben a `.kube/config` Kubernetes főcsomópontjain található fájlban tárolja a rendszer.

**Titkok kezelése és alkalmazás hitelesítő adatai**

A titkokat, például a kapcsolódási karakterláncokat vagy az adatbázis hitelesítő adatait több lehetőség is tárolhatja, többek között:

- Azure Key Vault
- A Kubernetes titkos kódjai
- külső megoldások, például a HashiCorp-tároló (Kubernetes-on futó)

Ne tárolja a titkos kulcsokat vagy a hitelesítő adatokat a konfigurációs fájlokban, az alkalmazás kódjában vagy a parancsfájlokon belül. És ne tárolja azokat a verziókövetés rendszerében. Ehelyett a központi telepítési automatizálásnak szükség szerint le kell kérnie a titkokat.

## <a name="patch-and-update"></a>Javítás és frissítés

Az Azure Kubernetes szolgáltatásban a **patch és a Update (PNU)** folyamat részben automatizált. A Kubernetes-verziók frissítése manuálisan történik, a biztonsági frissítéseket pedig automatikusan alkalmazza a rendszer. Ezek a frissítések operációsrendszer-biztonsági javításokat vagy kernel-frissítéseket is tartalmazhatnak. Az AK nem újraindítja automatikusan ezeket a Linux-csomópontokat a frissítési folyamat befejezéséhez. 

A Azure Stack hub alhálózatán keresztül üzembe helyezett Kubernetes-fürtök PNU folyamata nem felügyelt, és a fürt operátorának feladata. 

Az AK-motor segíti a két legfontosabb feladatot:

- [Frissítés egy újabb Kubernetes és az alap operációs rendszer rendszerképének verziójára](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Csak az alap operációs rendszer rendszerképének frissítése](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Az újabb alap operációsrendszer-lemezképek tartalmazzák a legújabb operációsrendszer-biztonsági javításokat és a kernel frissítéseit. 

A [felügyelet nélküli frissítési](https://wiki.debian.org/UnattendedUpgrades) mechanizmus automatikusan telepíti azokat a biztonsági frissítéseket, amelyeket a rendszer az új alap operációsrendszer-rendszerkép verziója előtt szabadít fel az Azure stack hub piactéren. A felügyelet nélküli frissítés alapértelmezés szerint engedélyezve van, és automatikusan telepíti a biztonsági frissítéseket, de nem indítják újra a Kubernetes-fürt csomópontjait. A csomópontok újraindítása automatizálható a nyílt forráskódú [ **K** Ubernetes újraindítási **RE** **D** Aemon (kured)](/azure/aks/node-updates-kured)használatával. Kured az újraindítást igénylő Linux-csomópontokra, majd automatikusan kezeli a futó hüvelyek és a csomópont-újraindítási folyamat újraütemezését.

## <a name="deployment-cicd-considerations"></a>Üzembe helyezési (CI/CD) megfontolások

Az Azure és a Azure Stack hub ugyanazokat a Azure Resource Manager REST API-kat teszi elérhetővé. Ezek az API-k minden más Azure-felhőhöz (Azure, Azure China 21Vianet, Azure Government) vannak címezve. Lehetnek eltérések a felhők közötti API-verziók között, és a Azure Stack hub csak a szolgáltatások egy részét biztosítja. A felügyeleti végpont URI-ja különbözik az egyes felhőknek és az Azure Stack hub minden példányának.

A fentiekben említett enyhe eltérések mellett Azure Resource Manager REST API-k egységes módszert biztosítanak az Azure és az Azure Stack hub szolgáltatással való kommunikációra. Ugyanazokat az eszközöket használhatja itt, mint bármely más Azure-felhőben. Az Azure DevOps, például a Jenkins vagy a PowerShell használatával a szolgáltatások üzembe helyezése és összehangolása Azure Stack hubhoz.

**Megfontolások**

Az Azure Stack hub központi telepítésének egyik fő különbsége az internetes hozzáférhetőség kérdése. Az Internet hozzáférhetősége határozza meg, hogy ki kell-e választania egy Microsoft által üzemeltetett vagy saját üzemeltetésű Build ügynököt a CI/CD-feladatokhoz.

A saját üzemeltetésű ügynök a Azure Stack hub (IaaS VM) vagy egy olyan hálózati alhálózat felett futhat, amely hozzáfér Azure Stack hubhoz. A különbségekről további információt az [Azure-folyamatok ügynököknél](/azure/devops/pipelines/agents/agents) olvashat.

Az alábbi képen eldöntheti, hogy szüksége van-e egy saját üzemeltetésű vagy egy Microsoft által üzemeltetett Build-ügynökre:

![Saját üzemeltetésű Build-ügynökök igen vagy nem](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Elérhetők az Azure Stack hub felügyeleti végpontok az interneten keresztül?
  - Igen: az Azure-folyamatokat Microsoft által üzemeltetett ügynökökkel is használhatjuk Azure Stack hubhoz való kapcsolódáshoz.
  - Nem: szükség van olyan önkiszolgáló ügynökökre, amelyek csatlakozhatnak Azure Stack hub felügyeleti végpontokhoz.
- Elérhető a Kubernetes-fürt az interneten keresztül?
  - Igen: az Azure-folyamatokat Microsoft által üzemeltetett ügynökökkel is használhatja a Kubernetes API-végponttal való közvetlen interakcióhoz.
  - Nem: szükség van olyan önkiszolgáló ügynökökre, amelyek csatlakozhatnak a Kubernetes-fürt API-végponthoz.

Olyan esetekben, amikor az Azure Stack hub felügyeleti végpontok és a Kubernetes API az interneten keresztül elérhető, a központi telepítés Microsoft által üzemeltetett ügynököt használhat. Ez az üzembe helyezés az alkalmazás architektúráját fogja eredményezni a következőképpen:

[![Nyilvános architektúra – áttekintés](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Ha az Azure Resource Manager-végpontok, a Kubernetes API vagy mindkettő nem érhető el közvetlenül az interneten keresztül, a folyamat lépéseinek futtatásához kihasználhatjuk a saját üzemeltetésű Build ügynököt. Ennek a kialakításnak kevesebb kapcsolatra van szüksége, és csak helyszíni hálózati kapcsolattal helyezhető üzembe Azure Resource Manager végpontokhoz és a Kubernetes API-hoz:

[![Helyszíni architektúra áttekintése](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Mi a helyzet a leválasztott forgatókönyvekkel?** Olyan helyzetekben, ahol Azure Stack hub vagy Kubernetes, vagy mindkettő nem rendelkezik internetre irányuló felügyeleti végpontokkal, továbbra is használhatja az Azure DevOps-t az üzemelő példányokhoz. Használhat saját üzemeltetésű ügynököt is (amely a helyszínen vagy a Azure Stack hub-on futtatott DevOps-ügynök) vagy egy teljesen helyi Azure DevOps Server a helyszínen. A saját üzemeltetésű ügynöknek csak kimenő HTTPS (TCP/443) internetkapcsolatra van szüksége.

A minta egy Kubernetes-fürtöt is használhat (az AK-motorral üzembe helyezett és összehangolt) minden Azure Stack hub-példányon. Egy olyan alkalmazást tartalmaz, amely egy frontendből, egy közepes szintű háttér-szolgáltatásokból (például MongoDB) és egy Nginx-alapú bejövő adatkezelőből áll. A K8s-fürtön üzemeltetett adatbázis helyett a "külső adattárak" is kihasználható. Az adatbázis-beállítások közé tartozik a MySQL, a SQL Server vagy az Azure Stack hub vagy a IaaS szolgáltatáson kívül üzemeltetett adatbázis. Az ehhez hasonló konfigurációk nem tartoznak ide.

## <a name="partner-solutions"></a>Partneri megoldások

Vannak olyan Microsoft partneri megoldások, amelyek kiterjeszthetik Azure Stack hub képességeit. Ezek a megoldások a Kubernetes-fürtökön futó alkalmazások központi telepítése során hasznosak.  

## <a name="storage-and-data-solutions"></a>Tárolási és adatkezelési megoldások

Az [adatkezelési és tárolási megfontolásokban](#data-and-storage-considerations)leírtak szerint Azure stack hub jelenleg nem rendelkezik natív megoldással a tárterület több példányra való replikálásához. Az Azure-tól eltérően a tárterület több régióban való replikálásának lehetősége nem létezik. Azure Stack hub-ban minden példány a saját különböző felhője. A megoldások azonban olyan Microsoft-partnerektől érhetők el, amelyek lehetővé teszik a tárolók replikálását Azure Stack hubok és az Azure között. 

**SKÁLÁZÁS**

A [skálázhatóság](https://www.scality.com/) olyan webes tárterületet biztosít, amely a 2009-as óta digitális üzleti vállalkozásokat üzemeltet. A skálázási kör, a szoftveresen definiált tárolás, a hagyományos x86-kiszolgálók korlátlan tárolási készletbe kapcsolása bármilyen típusú adattípushoz – fájl és objektum – a petabyte skálán.

**CLOUDIAN**

A [Cloudian](https://www.cloudian.com/) leegyszerűsíti a nagyvállalati tárterületet, és korlátlanul méretezhető tárterületet biztosít, amely egyetlen, könnyen felügyelt környezetbe összevonja a nagyméretű adatkészleteket.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott fogalmakról:

- [Több felhőre kiterjedő méretezési](pattern-cross-cloud-scale.md) és [földrajzilag elosztott alkalmazási minták](pattern-geo-distributed.md) Azure stack központban.
- [A Services architektúra az Azure Kubernetes szolgáltatásban (ak)](/azure/architecture/reference-architectures/microservices/aks).

Ha készen áll a megoldás tesztelésére, folytassa a [magas rendelkezésre állású Kubernetes-fürt telepítési útmutatóját](solution-deployment-guide-highly-available-kubernetes.md). A telepítési útmutató részletes útmutatást nyújt az összetevők üzembe helyezéséhez és teszteléséhez.