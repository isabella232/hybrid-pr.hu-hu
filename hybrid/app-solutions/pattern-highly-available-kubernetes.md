---
title: Magas rendelkezésre állású Kubernetes-minta az Azure és a Azure Stack Hub
description: Megtudhatja, hogyan biztosít magas rendelkezésre állást egy Kubernetes-fürtmegoldás az Azure és a Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: f8a733bcdab871695e552ec687d42e3ff4230490
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281312"
---
# <a name="high-availability-kubernetes-cluster-pattern"></a>Magas rendelkezésre állású Kubernetes-fürtminta

Ez a cikk azt ismerteti, hogyan tervezhető meg és működtethető egy magas rendelkezésre álló Kubernetes-alapú infrastruktúra az Azure Kubernetes Service -motorral a Azure Stack Hub. Ez a forgatókönyv gyakori olyan szervezeteknél, amelyek kritikus fontosságú számítási feladatokat futtatnak szigorúan korlátozott és szabályozott környezetekben. Szervezetek olyan területeken, mint a pénzügy, a védelem és a kormányzat.

## <a name="context-and-problem"></a>Kontextus és probléma

Számos szervezet fejleszt olyan natív felhőalapú megoldásokat, amelyek a legtechnológiásabb szolgáltatásokat és technológiákat, például a Kubernetes-t kihasználják. Bár az Azure a világ legtöbb régiójában biztosít adatközpontokat, néha vannak olyan peremhálózati felhasználási esetek és forgatókönyvek, ahol az üzleti szempontjából kritikus fontosságú alkalmazásoknak egy adott helyen kell futniuk. Megfontolandó szempontok:

- Hely bizalmasság
- Késés az alkalmazás és a helyszíni rendszerek között
- Sávszélesség-takarékosság
- Kapcsolatok
- Szabályozási vagy törvényi követelmények

Az Azure a Azure Stack Hub együtt a legtöbb ilyen probléma kezelésével foglalkozik. Az alábbiakban a kubernetes-et futtató kubernetes sikeres implementálható lehetőségeinek, döntéseinek és szempontjainak Azure Stack Hub ismertetjük.

## <a name="solution"></a>Megoldás

Ez a minta feltételezi, hogy szigorú korlátozásokkal kell foglalkoznunk. Az alkalmazásnak a helyszínen kell futnia, és minden személyes adat nem érhet el nyilvános felhőszolgáltatásokat. A monitorozási és egyéb nem személyes adatok elküldését és feldolgozását az Azure-ba lehet küldeni. Külső szolgáltatások, például nyilvános Container Registry mások is elérhetők, de szűrhetők egy tűzfalon vagy proxykiszolgálón keresztül.

Az itt látható mintaalkalmazás (a [Azure Kubernetes Service Workshop](/learn/modules/aks-workshop/)alapján) a Kubernetes natív megoldásait használja, amikor csak lehetséges. Ez a kialakítás a natív platformszolgáltatások használata helyett elkerüli a szállítói zárolást. Az alkalmazás például egy saját üzemeltetett MongoDB-adatbázis háttérszolgáltatást használ PaaS-szolgáltatás vagy külső adatbázis-szolgáltatás helyett.

[![Hibrid alkalmazásminta](media/pattern-highly-available-kubernetes/application-architecture.png)](media/pattern-highly-available-kubernetes/application-architecture.png#lightbox)

A fenti ábra a Kubernetesben futó mintaalkalmazás alkalmazásarchitektúráját mutatja be a Azure Stack Hub. Az alkalmazás több összetevőből áll, többek között a következőkből:

 1) Egy AKS Engine-alapú Kubernetes-fürt a Azure Stack Hub.
 2) [tanúsítványkezelő,](https://www.jetstack.io/cert-manager/)amely a Kubernetes tanúsítványkezelési eszközeinek egy csomagját biztosítja, amellyel automatikusan kér tanúsítványokat a Let's Encrypt szolgáltatástól.
 3) Egy Kubernetes-névtér, amely tartalmazza az előtér (ratings-web), az API (ratings-api) és az adatbázis (ratings-mongodb) alkalmazás-összetevőit.
 4) A bejövőforgalom-vezérlő, amely a HTTP/HTTPS-forgalmat a Kubernetes-fürtben található végpontokrairányító.

A mintaalkalmazás az alkalmazásarchitektúra szemléltetésére használható. Minden összetevő példa. Az architektúra csak egyetlen alkalmazástelepítést tartalmaz. A magas rendelkezésre állás (HA) eléréséhez legalább kétszer futtatjuk az üzembe helyezést két különböző Azure Stack Hub-példányon – ezek ugyanazon a helyen vagy két (vagy több) különböző helyen futnak:

![Infrastruktúra-architektúra](media/pattern-highly-available-kubernetes/aks-azure-architecture.png)

Az olyan Azure Container Registry, Azure Monitor stb. az Azure-ban Azure Stack Hub helyszínen üzemelnek. Ez a hibrid kialakítás védelmet nyújt a megoldásnak az egyetlen Azure Stack Hub kimaradása ellen.

## <a name="components"></a>Összetevők

A teljes architektúra a következő összetevőkből áll:

**Azure Stack Hub** az Azure bővítménye, amely számítási feladatokat futtathat helyszíni környezetben azáltal, hogy Azure-szolgáltatásokat biztosít az adatközpontban. További [információért Azure Stack Hub áttekintését.](/azure-stack/operator/azure-stack-overview)

**Azure Kubernetes Service motor (AKS Engine)** a felügyelt Kubernetes szolgáltatásajánlat, az Azure Kubernetes Service (AKS) mögötti motor, amely jelenleg az Azure-ban érhető el. Az Azure Stack Hub AKS Engine lehetővé teszi a teljes funkcionalitású, saját felügyelt Kubernetes-fürtök üzembe helyezését, méretezését és frissítését az Azure Stack Hub IaaS képességeivel. További információért olvassa el az [AKS Engine Overview (AKS-motor áttekintése)](https://github.com/Azure/aks-engine) témakört.

Az [Azure-beli](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#known-issues-and-limitations) AKS Engine és az AKS Engine on Azure Stack Hub.

**Az Azure Virtual Network (VNet)** biztosítja a Kubernetes-fürt infrastruktúráját üzemeltető Azure Stack Hub-fürtök hálózati infrastruktúráját Virtual Machines (virtuális gépek) számára.

**Azure Load Balancer** a Kubernetes API-végponthoz és az Nginx bejövő forgalomvezérlőhöz használható. A terheléselosztás a külső (például internetes) forgalmat egy adott szolgáltatást kínáló csomópontokra és virtuális gépekreirányítást biztosít.

**Azure Container Registry (ACR)** a fürtön üzembe helyezett privát Docker-rendszerképek és Helm-diagramok tárolására használható. Az AKS-motor Azure AD-identitással Container Registry hitelesítést a Container Registry használatával. A Kubernetes nem igényel ACR-t. Más tárolóregisztrálókat is használhat, például a Docker Hub.

**Az Azure Repos** a kód kezeléséhez használható verzióvezérlő eszközök készlete. Használhat további GitHub git-alapú adattárakat is. További [információért olvassa el az Azure Repos overview (Az Azure-adattárak](/azure/devops/repos/get-started/what-is-repos) áttekintése) témakört.

**Az Azure Pipelines** az Azure DevOps Services része, és automatizált buildeket, teszteket és üzembe helyezéseket futtat. Harmadik féltől származó CI-/CD-megoldásokat is használhat, például a Jenkinst. További [információért olvassa](/azure/devops/pipelines/get-started/what-is-azure-pipelines) el az Azure Pipeline áttekintését.

**Azure Monitor** gyűjti és tárolja a metrikákat és naplókat, beleértve az Azure-szolgáltatások platformmetrikákat a megoldásban és az alkalmazás-telemetriában. Ezeket az adatokat használhatja az alkalmazás monitorzához, riasztások és irányítópultok beállításához, valamint a hibák kiváltó okainak elemzéséhez. Azure Monitor a Kubernetesbe, hogy metrikákat gyűjtsön a vezérlőktől, csomópontoktól és tárolóktól, valamint a tárolónaplóktól és a főcsomópontok naplóitól. További [információért Azure Monitor az Áttekintés](/azure/azure-monitor/overview) oldalon.

**Azure Traffic Manager** egy DNS-alapú forgalom-terheléselosztási eszköz, amely lehetővé teszi, hogy optimálisan ossza el a forgalmat a szolgáltatások között a különböző Azure-régiókban vagy Azure Stack Hub üzemelő példányok között. Traffic Manager magas rendelkezésre állást és válaszképességet is biztosít. Az alkalmazásvégpontnak kívülről elérhetőnek kell lennie. Más helyszíni megoldások is elérhetők.

**A Kubernetes bejövő forgalomvezérlő** HTTP(S) útvonalakat biztosít a Kubernetes-fürt szolgáltatásaihoz. Erre a célra az Nginx vagy bármely megfelelő bejövő vezérlő használható.

**A Helm** egy csomagkezelő a Kubernetes üzembe helyezéséhez, amely különböző Kubernetes-objektumok , például üzemelő példányok, szolgáltatások, titkos kulcsok egyetlen "diagramba" csomagolását biztosítja. Közzéteheti, üzembe helyezheti, vezérelheti a verziókezelést, és frissíthet egy diagramobjektumot. Azure Container Registry a csomagolt Helm-diagramok tárolására használhatók adattárként.

## <a name="design-considerations"></a>Kialakítási szempontok

Ez a minta a cikk következő szakaszaiban részletesebben ismertet néhány magas szintű szempontot:

- Az alkalmazás natív Kubernetes-megoldásokat használ a szállítói zárolás elkerülése érdekében.
- Az alkalmazás mikroszolgáltatási architektúrát használ.
- Azure Stack Hub nincs szükség bejövő forgalomra, de engedélyezi a kimenő internetkapcsolatot.

Ezek az ajánlott eljárások a valós számítási feladatokra és forgatókönyvekre is érvényesek.

## <a name="scalability-considerations"></a>Méretezési szempontok

A skálázhatóság azért fontos, hogy a felhasználók konzisztens, megbízható és jól teljesítő hozzáférést biztosítanak az alkalmazáshoz.

A mintaforgatókönyv az alkalmazáskészlet több rétegének skálázhatóságát is lefedi. Az alábbi magas szintű áttekintés a különböző rétegekről:

| Architektúraszint | Befolyásolja | Hogyan? |
| --- | --- | ---
| Alkalmazás | Alkalmazás | Horizontális skálázás a podok/replikák/Container Instances* |
| Fürt | Kubernetes-fürt | A csomópontok száma (1 és 50 között), vm-SKU-sizes és Node Pools (AKS Engine on Azure Stack Hub jelenleg csak egyetlen csomópontkészletet támogat); az AKS Engine skálázandó parancsának használatával (manuális) |
| Infrastruktúra | Azure Stack Hub | Csomópontok, kapacitás és méretezési egységek száma egy adott Azure Stack Hub üzemelő példányon belül |

\* A Kubernetes automatikus horizontális podméretozója (HPA) használata; automatizált metrikaalapú skálázás vagy vertikális skálázás a tárolópéldányok (cpu/memória) méretezése által.

**Azure Stack Hub (infrastruktúraszint)**

A Azure Stack Hub infrastruktúra az alapja ennek a megvalósításnak, Azure Stack Hub az adatközpontban fizikai hardveren futnak. A központi hardver kiválasztásakor választania kell a processzor, a memóriasűrűség, a tárolókonfiguráció és a kiszolgálók száma alapján. Ha többet szeretne megtudni a Azure Stack Hub skálázhatóságáról, tekintse meg az alábbi forrásokat:

- [Kapacitástervezés a Azure Stack Hub áttekintése](/azure-stack/operator/azure-stack-capacity-planning-overview)
- [További méretezésiegység-csomópontok hozzáadása a Azure Stack Hub](/azure-stack/operator/azure-stack-add-scale-node)

**Kubernetes-fürt (fürtszint)**

Maga a Kubernetes-fürt az Azure (Stack) IaaS-összetevőire épül, beleértve a számítási, tárolási és hálózati erőforrásokat. A Kubernetes-megoldások között fő és munkavégző csomópontok is vannak, amelyek virtuális gépként vannak telepítve az Azure-ban (Azure Stack Hub).

- [A vezérlősík csomópontjai](/azure/aks/concepts-clusters-workloads#control-plane) (fő csomópontjai) biztosítják az alapvető Kubernetes-szolgáltatásokat és az alkalmazás számítási feladatainak vezényléseit.
- [A feldolgozó csomópontok](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) (feldolgozók) futtatják az alkalmazás számítási feladatait.

A kezdeti üzembe helyezéshez a virtuálisgép-méretek kiválasztásakor több szempontot is figyelembe kell venni:  

- **Költség** – A munkavégző csomópontok tervezésekor tartsa szem előtt a virtuális gépenkénti teljes költséget. Ha például az alkalmazás számítási feladatai korlátozott erőforrásokat igényelnek, érdemes kisebb méretű virtuális gépeket üzembe helyezni. Azure Stack Hub azure-hoz hasonló szolgáltatások számlázása általában használat alapján történik, ezért a Kubernetes-beli virtuális gépek megfelelő méretezése elengedhetetlen a fogyasztási költségek optimalizálásához. 

- **Méretezhetőség** – A fürt skálázhatósága a fő- és munkavégző csomópontok számának le- és felméretezésével, vagy további (jelenleg nem elérhető) csomópontkészletek hozzáadásával Azure Stack Hub érhető el. A fürt skálázása a Container Elemzések (Azure Monitor + Log Analytics) használatával gyűjtött teljesítményadatok alapján történik. 

    Ha az alkalmazásnak több (vagy kevesebb) erőforrásra van szüksége, horizontálisan (vagy horizontálisan) horizontálisan skálázhatja fel horizontálisan az aktuális csomópontokat (1 és 50 csomópont között). Ha több mint 50 csomópontra van szüksége, létrehozhat egy további fürtöt egy külön előfizetésben. A tényleges virtuális gépeket nem skálázhatja vertikálisan egy másik virtuálisgép-méretre a fürt ismételt üzembeása nélkül.

    A skálázás manuálisan történik a Kubernetes-fürt kezdeti üzembe helyezéséhez használt AKS-motor segítő virtuális gép használatával. További információ: [Kubernetes-fürtök skálázása](https://github.com/Azure/aks-engine/blob/master/docs/topics/scale.md)

- **Kvóták** – Vegye figyelembe [az](/azure-stack/operator/azure-stack-quota-types) AKS üzembe helyezésének megtervezésekor a saját Azure Stack Hub. Győződjön meg [arról, hogy](/azure-stack/operator/service-plan-offer-subscription-overview) minden előfizetés rendelkezik a megfelelő csomagokkal és beállított kvótákkal. Az előfizetésnek el kell fogadnia a horizontális felskálás során a fürtökhöz szükséges számítási, tárolási és egyéb szolgáltatásokat.

- **Alkalmazás számítási feladatai** – [](/azure/aks/concepts-clusters-workloads#nodes-and-node-pools) Tekintse meg a Fürtök és számítási feladatok fogalmait a Kubernetes alapfogalmai című Azure Kubernetes Service dokumentumban. Ez a cikk segít az alkalmazás számítási és memóriabeli igényeinek megfelelő virtuálisgép-méret hatókörének megszűkésében.  

**Alkalmazás (alkalmazásszint)**

Az alkalmazásrétegben a Kubernetes automatikus horizontális [podméretozóját (HPA) használjuk.](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) A HPA különböző metrikák (például a processzorkihasználtság) alapján növelheti vagy csökkentheti a replikák (pod/Container Instances) számát az üzemelő példányban.

Egy másik lehetőség a tárolópéldányok vertikális skálázható. Ez úgy valósítható meg, hogy módosítja a kért cpu és memória mennyiségét, és elérhetővé válik egy adott üzemelő példányhoz. További [információ: Erőforrások kezelése tárolókhoz](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) kubernetes.io a tárolókhoz.

## <a name="networking-and-connectivity-considerations"></a>Hálózattal és kapcsolattal kapcsolatos szempontok

A hálózat és a kapcsolatok a Kuberneteshez korábban említett három réteget is érintik a Azure Stack Hub. Az alábbi táblázat a rétegeket és az ezek által tartalmazott szolgáltatásokat mutatja be:

| Réteg | Befolyásolja | Mi? |
| --- | --- | ---
| Alkalmazás | Alkalmazás | Hogyan érhető el az alkalmazás? Elérhető lesz az interneten? |
| Fürt | Kubernetes-fürt | Kubernetes API, AKS-motorral kapcsolatos virtuális gép, Tároló-rendszerképek lehúzása (kigresszió), Monitorozási adatok és telemetria küldése (kigressz) |
| Infrastruktúra | Azure Stack Hub | A felügyeleti Azure Stack Hub, például a portál és a Azure Resource Manager akadálymentessége. |

**Alkalmazás**

Az alkalmazásréteg legfontosabb szempontja az, hogy az alkalmazás elérhető-e és elérhető-e az internetről. Kubernetes szempontjából az internet-elérhetőség azt jelenti, hogy egy üzembe helyezést vagy podot egy Kubernetes Service vagy egy bejövő forgalomvezérlő használatával tesz elérhetőre.

> [!NOTE]
> Javasoljuk, hogy a Kubernetes Services elérhetővé tegye a bejövő forgalom vezérlőit, mivel az előtere nyilvános IP-Azure Stack Hub 5-re van korlátozva. Ez a kialakítás a (LoadBalancer típusú) Kubernetes Services számát is 5-re korlátozza, ami túl kicsi sok üzemelő példányhoz. További információért olvassa el az [AKS Engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#limited-number-of-frontend-public-ips) dokumentációját.

Ha egy alkalmazást nyilvános IP-címmel Load Balancer vagy bejövő forgalomvezérlővel, az nem jelenti azt, hogy az alkalmazás mostantól elérhető az interneten keresztül. Lehetséges, hogy Azure Stack Hub ip-címe csak a helyi intraneten látható – nem minden nyilvános IP-cím valóban internetre néz.

Az előző blokk az alkalmazás bejövő forgalmát veszi figyelembe. Egy másik témakör, amely a Kubernetes sikeres üzembe helyezéséhez szükséges, a kimenő/kimenő forgalom. Íme néhány olyan eset, amelyek a forgalmi forgalmat igénylik:

- A DockerHubon vagy a dockerhubon tárolt tároló-rendszerképek Azure Container Registry
- Helm-diagramok leolvasása
- Alkalmazásadatok (Elemzések monitorozási adatok) kibocsátása

Egyes vállalati környezetekhez transzparens vagy nem transzparens _proxykiszolgálók_ használata lehet szükséges.  Ezek a kiszolgálók speciális konfigurációt igényelnek a fürt különböző összetevőin. Az AKS Engine dokumentációja számos részletet tartalmaz a hálózati proxyk elhelyezéséről. További információ: [AKS Engine and proxy servers (AKS-motor és proxykiszolgálók)](https://github.com/Azure/aks-engine/blob/master/docs/topics/proxy-servers.md)

Végül pedig a fürtök közötti forgalomnak kell áramlik a Azure Stack Hub között. A mintatelepítés különálló, különálló példányon futó Kubernetes-fürtökből Azure Stack Hub áll. A két adatbázis közötti forgalom , például a két adatbázis közötti replikációs forgalom a "külső forgalom". A külső forgalmat egy hely–hely VPN-en vagy egy nyilvános IP-címen keresztül kell Azure Stack Hub Kubernetes két virtuális Azure Stack Hub keresztül:

![fürtközi és fürtön belüli forgalom](media/pattern-highly-available-kubernetes/aks-inter-and-intra-cluster-traffic.png)

**Csoport**  

A Kubernetes-fürtnek nem feltétlenül kell elérhetőnek lennie az interneten keresztül. A releváns rész a fürtök működtetéséhez használt Kubernetes API, például a `kubectl` használatával. A Kubernetes API-végpontnak elérhetőnek kell lennie mindenki számára, aki a fürtöt üzemelteti, vagy alkalmazásokat és szolgáltatásokat helyez üzembe rajta. Erről a témakörről részletesebben az alábbi Üzembe [helyezési (CI/CD)](#deployment-cicd-considerations) szempontok című szakaszban, DevOps-szempontból olvashat.

A fürtök szintjén a bejövő forgalommal kapcsolatban is figyelembe kell venni néhány szempontot:

- Csomópontfrissítések (Ubuntu esetén)
- Monitorozási adatok (elküldve az Azure LogAnalyticsnek)
- Más, kimenő forgalmat igénylő ügynökök (az egyes üzembe helyezők környezetére jellemző)

Mielőtt üzembe helyezné a Kubernetes-fürtöt az AKS-motorral, tervezze meg a végső hálózatépítési tervet. Dedikált hálózati Virtual Network helyett hatékonyabb lehet egy fürtöt egy meglévő hálózatban üzembe helyezni. Használhatja például a meglévő, a saját környezetében már konfigurált, Azure Stack Hub VPN-kapcsolatot.

**Infrastruktúra**  

Az infrastruktúra a felügyeleti végpontok Azure Stack Hub jelenti. A végpontok közé tartoznak a bérlői és felügyeleti portálok, valamint a Azure Resource Manager és a bérlői végpontok. Ezek a végpontok szükségesek a Azure Stack Hub alapvető szolgáltatásainak működtetéséhez.

## <a name="data-and-storage-considerations"></a>Adatokkal és tárolással kapcsolatos szempontok

Az alkalmazás két példánya lesz üzembe stb. két különálló Kubernetes-fürtön, két Azure Stack Hub példányban. Ehhez a kialakításhoz meg kell fontolnunk, hogyan replikáljuk és szinkronizáljuk közöttük az adatokat.

Az Azure-ral beépített képességünk van arra, hogy a tárolót több régióba és zónába replikáljuk a felhőben. Jelenleg Azure Stack Hub nincs natív módszer a tárterület két különböző Azure Stack Hub-példányra való replikálása – két független felhőt alkotnak, amelyek nem kínálnak átfogó módszert a tárolók készletként való kezelésére. A különböző rendszereken futó alkalmazások rugalmasságának megtervezése Azure Stack Hub, hogy figyelembe vegye ezt a rugalmasságot az alkalmazástervezésben és az üzembe helyezésekben.

A legtöbb esetben nincs szükség tárreplikációra az AKS-ben üzembe helyezett rugalmas és magas rendelkezésre állékonyságú alkalmazásokhoz. Az alkalmazás kialakítása során azonban érdemes megfontolni a Azure Stack Hub tárolópéldányonkénti független tárterületet. Ha ez a kialakítás problémát vagy útlezárást jelent a megoldás Azure Stack Hub üzembe helyezésekor, a Microsoft partnereinek van néhány lehetséges megoldása, amelyek tárolási mellékleteket biztosítanak. Storage mellékletek egy tárolóreplikációs megoldást biztosítanak több Azure Stack Hubon és az Azure-ban. További információ: [Partnermegoldások.](#partner-solutions)

Az architektúránkban ezeket a rétegeket tekinthetők át:

**Konfigurálás**

A konfiguráció magában foglalja az Azure Stack Hub, az AKS-motor és maga a Kubernetes-fürt konfigurációját. A konfigurációt a lehető legnagyobb mértékben automatizálni kell, és kódként infrastruktúraként kell tárolni egy Git-alapú verziókezelő rendszerben, például az Azure DevOpsban vagy a GitHub. Ezek a beállítások nem szinkronizálhatók egyszerűen több üzemelő példány között. Ezért javasoljuk a konfiguráció külső tárolását és alkalmazását, valamint a DevOps-folyamat használatát.

**Alkalmazás**

Az alkalmazást egy Git-alapú adattárban kell tárolni. Új üzembe helyezés, alkalmazásváltozások vagy vészhelyreállítás esetén könnyen üzembe helyezhető az Azure Pipelines használatával.

**Adatok**

A legtöbb alkalmazástervben az adatok a legfontosabbak. Az alkalmazásadatoknak szinkronban kell maradniuk az alkalmazás különböző példányai között. Az adatoknak emellett biztonsági mentési és vészhelyreállítási stratégiára is szüksége van, ha kimaradás történik.

Ennek megvalósítása erősen függ a technológiai döntésektől. Íme néhány megoldási példa egy adatbázis magas rendelkezésre állékony módon való Azure Stack Hub:

- [2016-os SQL Server rendelkezésre állási csoport üzembe helyezése az Azure-ban és a Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-sql-ha)
- [Magas rendelkezésre álló MongoDB-megoldás üzembe helyezése az Azure-ban és Azure Stack Hub](/azure-stack/hybrid/solution-deployment-guide-mongodb-ha)

A több helyen található adatokkal való munka esetén megfontolandó szempontok a magas rendelkezésre állású és rugalmas megoldások még összetettebb szempontjai. Megfontolandó szempontok:

- Késés és hálózati kapcsolat a Azure Stack Hubs között.
- A szolgáltatások és engedélyek identitásának rendelkezésre állása. Minden Azure Stack Hub-példány integrálható egy külső könyvtárba. Az üzembe helyezés során választhat, hogy a Azure Active Directory (Azure AD) vagy a Active Directory összevonási szolgáltatások (AD FS) (ADFS) használja. Így előfordulhat, hogy egyetlen identitást használ, amely több független identitáspéldánysal Azure Stack Hub interakciót.

## <a name="business-continuity-and-disaster-recovery"></a>Folyamatos üzletmenet és vészhelyreállítás

Az üzletmenet-folytonosság és a vészhelyreállítás (BCDR) fontos témakör mind a Azure Stack Hub, mind az Azure-ban. A fő különbség az, hogy Azure Stack Hub operátornak kell kezelnie a teljes BCDR-folyamatot. Az Azure-ban a BCDR egyes részeit automatikusan a Microsoft kezeli.

A BCDR az előző szakaszban említett adat- és tárolási [szempontokat érinti:](#data-and-storage-considerations)

- Infrastruktúra/konfiguráció
- Alkalmazás rendelkezésre állása
- Alkalmazásadatok

Ahogy azt az előző szakaszban említettük, ezek a területek az Azure Stack Hub felelősei, és szervezetenként eltérőek lehetnek. Tervezze meg a BCDR-t az elérhető eszközök és folyamatok alapján.

**Infrastruktúra és konfiguráció**

Ez a szakasz a fizikai és logikai infrastruktúrát, valamint az infrastruktúra Azure Stack Hub. A rendszergazdai és a bérlői terekben található műveleteket fedi le.

Az Azure Stack Hub operátor (vagy rendszergazda) felelős a Azure Stack Hub karbantartásáért. Beleértve az olyan összetevőket is, mint a hálózat, a tárolás, az identitás és a jelen cikk hatókörét kívül esik más témakörök. Az egyes műveletek Azure Stack Hub további információért tekintse meg a következő forrásokat:

- [Adatok helyreállítása Azure Stack Hub a Infrastructure Backup szolgáltatással](/azure-stack/operator/azure-stack-backup-infrastructure-backup)
- [Biztonsági mentés engedélyezése Azure Stack Hub a felügyeleti portálról](/azure-stack/operator/azure-stack-backup-enable-backup-console)
- [Helyreállítás végzetes adatvesztés esetén](/azure-stack/operator/azure-stack-backup-recover-data)
- [Infrastructure Backup Service – ajánlott eljárások](/azure-stack/operator/azure-stack-backup-best-practices)

Azure Stack Hub az a platform és háló, amelyen a Kubernetes-alkalmazásokat üzembe fogja helyezni. A Kubernetes-alkalmazás alkalmazástulajdonosa a Azure Stack Hub felhasználója lesz, és hozzáférést kap a megoldáshoz szükséges alkalmazás-infrastruktúra üzembe helyezéséhez. Ebben az esetben az alkalmazás infrastruktúrája azt jelenti, hogy az AKS-motorral üzembe helyezett Kubernetes-fürt és az azt körülvevő szolgáltatások. Ezeket az összetevőket üzembe Azure Stack Hub, és egy Azure Stack Hub korlátozza. Győződjön meg arról, hogy a Kubernetes-alkalmazás tulajdonosa által elfogadott ajánlat elegendő kapacitással rendelkezik (Azure Stack Hub kvótákban kifejezve) a teljes megoldás üzembe helyezéséhez. Az előző szakaszban javasolt módon az alkalmazás üzembe helyezését automatizálni kell a kódként használt infrastruktúra és az olyan üzembe helyezési folyamatok használatával, mint az Azure DevOps Azure Pipelines.

További információ a Azure Stack Hub kvótákról: Azure Stack Hub [szolgáltatások, csomagok,](/azure-stack/operator/service-plan-offer-subscription-overview) ajánlatok és előfizetések áttekintése

Fontos, hogy biztonságosan mentse és tárolja az AKS-motor konfigurációját a kimenetekkel együtt. Ezek a fájlok a Kubernetes-fürt eléréséhez használt bizalmas adatokat tartalmaznak, ezért meg kell védeni őket a nem rendszergazdák számára való hozzáféréstől.

**Alkalmazás rendelkezésre állása**

Az alkalmazás nem támaszkodhat az üzembe helyezett példány biztonsági másolataira. Általános gyakorlatként az alkalmazást teljes mértékben az infrastruktúra, mint kód mintákat követi. Például az Azure DevOps Azure Pipelines használatával újra üzembe lehet ásni. A BCDR eljárásnak magában kell foglalja az alkalmazás ugyanazon vagy egy másik Kubernetes-fürtön való ismételt üzembeését.

**Alkalmazásadatok**

Az adatvesztés minimalizálása érdekében az alkalmazásadatok kritikus fontosságúak. Az előző szakaszban az alkalmazás két (vagy több) példánya közötti adat-replikálási és -szinkronizálási technikákat ismertettünk. Az adatok tárolására használt adatbázis-infrastruktúrától (MySQL, MongoDB, MSSQL stb.) függően különböző adatbázis-elérhetőségi és biztonsági mentési technikák közül választhat.

Az integritás elérésének ajánlott módjai a következő módszerek valamelyikének használata:
- Natív biztonsági mentési megoldás az adott adatbázishoz.
- Olyan biztonsági mentési megoldás, amely hivatalosan támogatja az alkalmazás által használt adatbázistípus biztonsági mentését és helyreállítását.

> [!IMPORTANT]
> Ne tárolja a biztonsági mentési adatokat ugyanazon a Azure Stack Hub példányon, ahol az alkalmazásadatok találhatók. A biztonsági másolatok teljes Azure Stack Hub a biztonsági másolatokat is veszélyeztetné.

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Az AKS Azure Stack Hub on keresztül üzembe helyezett kubernetes a Azure Stack Hub nem felügyelt szolgáltatás. Ez egy Kubernetes-fürt automatikus üzembe helyezése és konfigurálása az Azure infrastruktúra szolgáltatásként (IaaS) használatával. Így ugyanolyan rendelkezésre állást biztosít, mint a mögöttes infrastruktúra.

Azure Stack Hub infrastruktúra már ellenáll a hibáknak, és olyan képességeket biztosít, mint a rendelkezésre állási készletek, hogy több tartalék és frissítési tartomány között ossza el [az összetevőket.](/azure-stack/user/azure-stack-vm-considerations#high-availability) A mögöttes technológia (feladatátvételi fürtszolgáltatás) azonban továbbra is jár némi állásidővel az érintett fizikai kiszolgálón lévő virtuális gépeken hardverhiba esetén.

Az éles Kubernetes-fürtöt és a számítási feladatot két (vagy több) fürtön is üzembe kell helyezni. Ezeket a fürtöknek különböző helyeken vagy adatközpontokban kell üzemeltetni, és olyan technológiákat kell használniuk, mint a Azure Traffic Manager a felhasználók a fürt válaszidejéhez vagy földrajzi helyhez való útválasztására.

![Forgalom Traffic Manager szabályozása a hálózati forgalommal](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager.png)

Azok az ügyfelek, akik egyetlen Kubernetes-fürttel vannak, általában egy adott alkalmazás szolgáltatás IP-címéhez vagy DNS-nevéhez csatlakoznak. Többfürtű üzembe helyezés esetén az ügyfeleknek csatlakozniuk kell egy olyan Traffic Manager DNS-névhez, amely az egyes Kubernetes-fürtök szolgáltatásaira/bejövő nevére mutat.

![Helyszíni Traffic Manager való útválasztáshoz használt Traffic Manager használatával](media/pattern-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

> [!NOTE]
> Ez a minta (felügyelt) AKS-fürtök esetén is ajánlott eljárás [az Azure-ban.](/azure/aks/operator-best-practices-multi-region#plan-for-multiregion-deployment)

Magának az AKS-motorral üzembe helyezett Kubernetes-fürtnek legalább három fő csomópontból és két munkavégző csomópontból kell állnia.

## <a name="identity-and-security-considerations"></a>Identitással és biztonsággal kapcsolatos szempontok

Az identitás és a biztonság fontos témakörök. Különösen akkor, ha a megoldás független Azure Stack Hub is. A Kubernetes és az Azure (Azure Stack Hub is) külön mechanizmusokkal rendelkezik a szerepköralapú hozzáférés-vezérléshez (RBAC):

- Az Azure RBAC szabályozza az Azure-beli erőforrásokhoz (és Azure Stack Hub), beleértve az új Azure-erőforrások létrehozására vonatkozó képességet is. Az engedélyek felhasználókhoz, csoportokhoz vagy szolgáltatásnévhez rendelhetők. (A szolgáltatásnév az alkalmazások által használt biztonsági identitás.)
- A Kubernetes RBAC szabályozza a Kubernetes API-hoz való engedélyeket. A podok létrehozása és a podok listázása például olyan műveletek, amelyek az RBAC-n keresztül egy felhasználó számára engedélyezettek (vagy elutasíthatóak). Ha Kubernetes-engedélyeket szeretne hozzárendelni a felhasználókhoz, szerepköröket és szerepkörkötéseket kell létrehoznia.

**Azure Stack Hub és RBAC**

Azure Stack Hub identitásszolgáltató két lehetőséget kínál. A használt szolgáltató a környezettől és attól függ, hogy csatlakoztatott vagy leválasztott környezetben fut-e:

- Azure AD – csak csatlakoztatott környezetben használható.
- Az ADFS-Active Directory erdőbe – csatlakoztatott vagy leválasztott környezetben is használható.

Az identitásszolgáltató kezeli a felhasználókat és csoportokat, beleértve az erőforrások eléréséhez szükséges hitelesítést és engedélyezést. Hozzáférés adható az olyan Azure Stack Hub erőforrásokhoz, mint az előfizetések, az erőforráscsoportok és az egyes erőforrások, például virtuális gépek vagy terheléselosztások. A konzisztens hozzáférési modellhez érdemes ugyanazt a csoportot (közvetlen vagy beágyazott) használni az összes Azure Stack Hubhoz. Példa a konfigurációra:

![beágyazott aad-csoportok az Azure Stack Hubbal](media/pattern-highly-available-kubernetes/azure-stack-azure-ad-nested-groups.png)

A példa egy dedikált csoportot tartalmaz (AAD vagy ADFS használatával) egy adott célra. Például közreműködői engedélyeket kell biztosítani ahhoz az erőforráscsoporthoz, amely a Kubernetes-fürt infrastruktúráját tartalmazza egy adott Azure Stack Hub-példányon (itt "Seattle K8s Cluster Contributor"). Ezeket a csoportokat a rendszer ezután egy olyan általános csoportba ágyazja, amely az egyes csoportok "alcsoportját" Azure Stack Hub.

Mintafelhasználónk mostantól "Közreműködő" engedéllyel fog rendelkezni mindkét olyan erőforráscsoporthoz, amely a Kubernetes-infrastruktúra erőforrásainak teljes készletét tartalmazza. A felhasználó mindkét példányon hozzáférhet az erőforrásokhoz, Azure Stack Hub példányok azonos identitásszolgáltatóval osztoznak.

> [!IMPORTANT]
> Ezek az engedélyek csak Azure Stack Hub erőforrásokra és a rá telepített erőforrások némelyikére vannak hatással. Az ilyen szintű hozzáféréssel rendelkezik felhasználók sok kárt okozhatnak, de nem férhetnek hozzá a Kubernetes IaaS virtuális gépekhez és a Kubernetes API-hoz anélkül, hogy további hozzáférést adna a Kubernetes üzemelő példányához.

**Kubernetes-identitás és RBAC**

A Kubernetes-fürtök alapértelmezés szerint nem ugyanazt az identitásszolgáltatót használják, mint az Azure Stack Hub. A Kubernetes-fürtöt, a főkiszolgálót és a munkavégző csomópontokat üzemeltető virtuális gépek a fürt üzembe helyezésekor megadott SSH-kulcsot használják. Erre az SSH-kulcsra azért van szükség, hogy SSH használatával csatlakozzon ezekhez a csomópontokhoz.

A Kubernetes API-t (például a használatával elért) szolgáltatásfiókok is védik, beleértve az alapértelmezett `kubectl` "fürt-rendszergazdai" szolgáltatásfiókot is. A szolgáltatásfiók hitelesítő adatait a rendszer kezdetben a `.kube/config` Kubernetes főcsomópontjainak fájljában tárolja.

**Titkos kulcsok kezelése és alkalmazás hitelesítő adatai**

A titkos kulcsok, például a kapcsolati sztringek vagy az adatbázis hitelesítő adatainak tárolására több lehetőség is van, például:

- Azure Key Vault
- A Kubernetes titkos kódjai
- Harmadik féltől származó megoldások, például a HashiCorp Vault (Kubernetesen futó)

A titkos kódokat és hitelesítő adatokat ne tárolja egyszerű szövegként a konfigurációs fájlokban, alkalmazáskódban vagy szkriptek között. És ne tárolja őket verzióvezérlő rendszerben. Ehelyett az üzembe helyezés automatizálásának szükség szerint kell lekérni a titkos adatokat.

## <a name="patch-and-update"></a>Javítás és frissítés

A **javítási és frissítési (PNU)** folyamat a Azure Kubernetes Service részben automatizált. A Kubernetes-verziófrissítések manuálisan, a biztonsági frissítések alkalmazása pedig automatikusan történik. Ezek a frissítések lehetnek az operációs rendszer biztonsági javításai vagy a kernelfrissítések. Az AKS nem indítja újra automatikusan ezeket a Linux-csomópontokat a frissítési folyamat befejezéséhez. 

Az AKS Engine használatával üzembe helyezett Kubernetes Azure Stack Hub fürt PNU-folyamata nem Azure Stack Hub, és a fürtüzemeltető feladata. 

Az AKS-motor segít a két legfontosabb feladat elvégzésében:

- [Frissítés újabb Kubernetes-verzióra és operációsrendszer-rendszerkép alapverzióra](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version)
- [Csak az alap operációsrendszer-rendszerkép frissítése](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image)

Az újabb alap operációsrendszer-lemezképek az operációs rendszer legújabb biztonsági javításokat és kernelfrissítéseket tartalmazzák. 

A [felügyelet nélküli](https://wiki.debian.org/UnattendedUpgrades) frissítési mechanizmus automatikusan telepíti a kiadott biztonsági frissítéseket, mielőtt az operációs rendszer rendszerképének új alapverziója elérhetővé válik a Azure Stack Hub Marketplace-en. A felügyelet nélküli frissítés alapértelmezés szerint engedélyezve van, és automatikusan telepíti a biztonsági frissítéseket, de nem indítja újra a Kubernetes-fürtcsomópontokat. A csomópontok újraindítása automatizálható a nyílt forráskódú [ **K** Ubernetes **RE** boot **D** aemon (rendszerindítási D aemon)) használatával.](/azure/aks/node-updates-kured) A rendszer az újraindítást igénylő Linux-csomópontokat figyeli, majd automatikusan kezeli a futó podok és csomópont-újraindítási folyamat ütemezését.

## <a name="deployment-cicd-considerations"></a>Üzembe helyezési (CI/CD) szempontok

Az Azure és Azure Stack Hub ugyanezeket az Azure Resource Manager REST API-kat teszi elérhetővé. Ezek az API-k úgy vannak meg címzve, mint bármely más Azure-felhő (Azure, Azure China 21Vianet, Azure Government). Az API-verziók között különbségek lehetnek a felhők között, és a Azure Stack Hub csak a szolgáltatások egy részkészletét biztosítja. A felügyeleti végpont URI-ja az egyes felhőkben és az egyes példányok Azure Stack Hub.

A fent említett apró különbségeken kívül a rest Azure Resource Manager API-k konzisztens lehetőséget biztosítanak az Azure-ral és a Azure Stack Hub. Itt ugyanazok az eszközök használhatók, mint bármely más Azure-felhőben. Az Azure DevOps, például a Jenkins vagy a PowerShell használatával üzembe helyezhet és vezényelhet szolgáltatásokat a Azure Stack Hub.

**Megfontolások**

Az internetes környezetek üzembe helyezésének egyik Azure Stack Hub az internetes elérhetőség kérdése. Az internetes elérhetőség határozza meg, hogy a CI-/CD-feladatokhoz Microsoft által üzemeltetett vagy saját által üzemeltetett buildügynök legyen kiválasztva.

A saját üzemeltetett ügynökök futtathatók az Azure Stack Hub (IaaS virtuális gépként) vagy egy hálózati alhálózaton, amely hozzáfér a Azure Stack Hub. A [különbségekről további információt az Azure Pipelines-ügynökökben](/azure/devops/pipelines/agents/agents) olvashat.

Az alábbi rendszerkép segít eldönteni, hogy egy saját vagy a Microsoft által üzemeltetett buildügynökre van-e szüksége:

![Saját üzemeltetett buildügynökök – Igen vagy Nem](media/pattern-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)

- Elérhetők Azure Stack Hub felügyeleti végpontok az interneten keresztül?
  - Igen: Az Azure Pipelines és a Microsoft által üzemeltetett ügynökök segítségével csatlakozhatunk a Azure Stack Hub.
  - Nem: Olyan saját üzemeltetett ügynökökre van szükségünk, amelyek Azure Stack Hub a felügyeleti végpontjaihoz.
- Elérhető a Kubernetes-fürt az interneten keresztül?
  - Igen: Az Azure Pipelines és a Microsoft által üzemeltetett ügynökök használatával közvetlenül kommunikálhat a Kubernetes API-végponttal.
  - Nem: Olyan saját üzemeltetett ügynökökre van szükségünk, amelyek csatlakozni tudnak a Kubernetes-fürt API-végpontjaihoz.

Olyan forgatókönyvekben, Azure Stack Hub felügyeleti végpontok és a Kubernetes API elérhetők az interneten keresztül, a telepítés a Microsoft által üzemeltetett ügynököt is használhat. Az üzembe helyezés az alábbi alkalmazásarchitektúrát eredményezi:

[![A nyilvános architektúra áttekintése](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern.png#lightbox)

Ha a Azure Resource Manager végpontok, a Kubernetes API vagy mindkettő nem érhető el közvetlenül az interneten keresztül, a folyamat lépéseit egy saját által üzemeltetett buildügynökkel futtathatja. Ennek a kialakításnak kevesebb kapcsolatra van szüksége, és csak helyszíni hálózati kapcsolattal helyezhető üzembe Azure Resource Manager végpontokkal és a Kubernetes API-val:

[![A on-prem architektúra áttekintése](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png)](media/pattern-highly-available-kubernetes/aks-azure-stack-app-pattern-self-hosted.png#lightbox)

> [!NOTE]
> **Mi a helyzet a leválasztott forgatókönyvekkel?** Olyan forgatókönyvekben, Azure Stack Hub a Kubernetes vagy mindkettő nem használ internetkapcsolattal bíró felügyeleti végpontokat, továbbra is használhatja az Azure DevOpsot az üzemelő példányok számára. Használhat egy helyi ügynökkészletet (amely egy helyszínen vagy a Azure Stack Hub-on futó DevOps-ügynök), vagy egy teljesen helyi Azure DevOps Server helyszínen. A saját üzemeltetett ügynöknek csak kimenő HTTPS(TCP/443) internetkapcsolatra van szüksége.

A minta egy Kubernetes-fürtöt használhat (az AKS-motorral üzembe helyezett és vezényelt) minden Azure Stack Hub példányon. Tartalmaz egy alkalmazást, amely egy előtere, egy középső rétegbeli, háttérszolgáltatások (például MongoDB) és egy nginx-alapú bejövő forgalomvezérlő. A K8s-fürtön üzemeltetett adatbázis használata helyett használhatja a "külső adattárakat". Az adatbázis-beállítások közé tartozik a MySQL, SQL Server, vagy bármilyen, az IaaS-Azure Stack Hub kívül üzemeltetett adatbázis. Az ehhez hasonló konfigurációk nincsenek jelen a hatókörben.

## <a name="partner-solutions"></a>Partneri megoldások

Vannak olyan Microsoft-partnermegoldások, amelyek kibővítik a Azure Stack Hub. Ezeket a megoldásokat hasznosnak találta a Kubernetes-fürtökön futó alkalmazások üzembe helyezésekor.  

## <a name="storage-and-data-solutions"></a>Storage és adatmegoldások

Az [adatokkal és tárolással](#data-and-storage-considerations)kapcsolatos szempontokat Azure Stack Hub jelenleg nem rendelkezik natív megoldással a tároló több példány közötti replikálása érdekében. Az Azure-ral ellentétben a tárterület több régióra történő replikálása nem létezik. Ebben Azure Stack Hub példányok saját különálló felhők. A Microsoft partnereitől származó megoldások azonban elérhetők, amelyek lehetővé teszik a tárreplikációt a Azure Stack Hubsban és az Azure-ban. 

**SCALITY (SKÁLÁZÁS)**

[A skálázás](https://www.scality.com/) olyan webes tárhelyet biztosít, amely 2009 óta működtet digitális vállalkozásokat. A szoftveres TÁROLÁS, a Scality RING, amely a kereskedelmi x86-kiszolgálókat korlátlan tárolókészletet hoz létre bármilyen típusú adathoz (fájlhoz és objektumhoz), petabájtos méretekben.

**CLOUDIAN**

[A Cloudian](https://www.cloudian.com/) a korlátlan skálázható tárolással leegyszerűsíti a vállalati tárolást, amely egyetlen, könnyen felügyelt környezetbe konszolidálja a nagy mennyiségű adatkészletet.

## <a name="next-steps"></a>Következő lépések

További információ a cikkben bemutatott fogalmakról:

- [A felhők közötti skálázás és](pattern-cross-cloud-scale.md) [földrajzi eloszlású alkalmazásminták](pattern-geo-distributed.md) Azure Stack Hub.
- [Mikroszolgáltatás-architektúra a Azure Kubernetes Service (AKS) szolgáltatásban.](/azure/architecture/reference-architectures/microservices/aks)

Ha készen áll a megoldás példának tesztelésére, folytassa a magas rendelkezésre állású [Kubernetes-fürt](/azure/architecture/hybrid/deployments/solution-deployment-guide-highly-available-kubernetes)üzembe helyezési útmutatóját. Az üzembe helyezési útmutató lépésenként útmutatást nyújt az összetevőinek üzembe helyezéséhez és teszteléséhez.