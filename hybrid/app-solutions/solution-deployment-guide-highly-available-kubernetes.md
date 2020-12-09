---
title: Magasan elérhető Kubernetes-fürt üzembe helyezése Azure Stack központban
description: Ismerje meg, hogyan helyezhet üzembe egy Kubernetes-fürtöt a magas rendelkezésre álláshoz az Azure és az Azure Stack hub használatával.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911736"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Magas rendelkezésre állású Kubernetes-fürt üzembe helyezése Azure Stack központban

Ebből a cikkből megtudhatja, hogyan hozhat létre több Azure Stack hub-példányon, különböző fizikai helyen üzembe helyezett, magasan elérhető Kubernetes-fürtöt.

A megoldás telepítési útmutatójában a következőket sajátíthatja el:

> [!div class="checklist"]
> - Az AK-motor letöltése és előkészítése
> - Kapcsolódás az KABAi motor segítő virtuális géphez
> - Kubernetes-fürt üzembe helyezése
> - Kapcsolódás a Kubernetes-fürthöz
> - Azure-folyamatok összekötése a Kubernetes-fürttel
> - Figyelés konfigurálása
> - Alkalmazás üzembe helyezése
> - Alkalmazás méretezése
> - A Traffic Manager konfigurálása
> - Kubernetes frissítése
> - Kubernetes skálázása

> [!Tip]  
> ![Hibrid oszlopok](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub az Azure kiterjesztése. Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.  
> 
> A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez. A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.

## <a name="prerequisites"></a>Előfeltételek

Az üzembe helyezési útmutató első lépéseinek megkezdése előtt győződjön meg arról, hogy:

- Tekintse át a [magas rendelkezésre állású Kubernetes-fürt mintáját](pattern-highly-available-kubernetes.md) ismertető cikket.
- Tekintse át a [Companion GitHub-tárház](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)tartalmát, amely a cikkben hivatkozott további eszközöket tartalmazza.
- Rendelkeznie kell egy olyan fiókkal, amely hozzáfér a [Azure stack hub felhasználói portálhoz](/azure-stack/user/azure-stack-use-portal), amely legalább ["közreműködői" engedélyekkel](/azure-stack/user/azure-stack-manage-permissions)rendelkezik.

## <a name="download-and-prepare-aks-engine"></a>AK-motor letöltése és előkészítése

Az AK-motor egy bináris fájl, amely bármely olyan Windows-vagy Linux-gazdagépről használható, amely elérheti az Azure Stack hub Azure Resource Manager-végpontokat. Ez az útmutató egy új Linux (vagy Windows) virtuális gép üzembe helyezését ismerteti Azure Stack hub-on. Később lesz használatban, amikor az AK-motor telepíti a Kubernetes-fürtöket.

> [!NOTE]
> Egy meglévő Windows vagy Linux rendszerű virtuális gép használatával üzembe helyezhet egy Kubernetes-fürtöt Azure Stack hub-ban az AK motor használatával.

Az AK-motor lépésenkénti folyamatát és követelményeit a következő dokumentálja:

* [Az AK-motor telepítése Linux rendszerű Azure stack hub-ban](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (vagy [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)használatával)

Az AK-motor egy segítő eszköz a (nem felügyelt) Kubernetes-fürtök üzembe helyezéséhez és üzemeltetéséhez (az Azure-ban és Azure Stack hub-ban).

A Azure Stack hub-ban található AK-motor részleteit és különbségeit itt találja:

* [Mi a Azure Stack hub AK-motorja?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [AK-motor Azure stack hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) -on (a githubon)

A minta környezet a Terraform segítségével automatizálja az KABAi motor virtuális gép üzembe helyezését. A [részleteket és kódokat a Companion GitHub-](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)tárházban találja.

Ennek a lépésnek az eredménye egy új erőforráscsoport a Azure Stack hub-on, amely az AK motor segítő virtuális gépet és a kapcsolódó erőforrásokat tartalmazza:

![KABAi motor virtuálisgép-erőforrásai Azure Stack központban](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Ha az AK-motort egy leválasztott gapped-környezetben kell üzembe helyeznie, tekintse át a [leválasztott Azure stack hub-példányokat](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) további információért.

A következő lépésben az újonnan üzembe helyezett AK-beli virtuális gépet használjuk egy Kubernetes-fürt üzembe helyezéséhez.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Kapcsolódás az KABAi motor segítő virtuális géphez

Először kapcsolódnia kell a korábban létrehozott KABAi motor segítő virtuális géphez.

A virtuális gépnek nyilvános IP-címmel kell rendelkeznie, és az SSH-n keresztül kell elérhetőnek lennie (a 22-es porton keresztül).

![Az AK-motor virtuálisgép-áttekintő lapja](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Az SSH-val Linux rendszerű virtuális gépekhez való kapcsolódáshoz használhatja a MobaXterm, a PuTTY vagy a PowerShell Windows 10-es eszközét.

```console
ssh <username>@<ipaddress>
```

A csatlakozás után futtassa a parancsot `aks-engine` . Az AK-motor és a Kubernetes verzióival kapcsolatos további tudnivalókért lépjen a [támogatott AK-hajtóművek verzióihoz](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) .

![AK-motor parancssori példája](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Kubernetes-fürt üzembe helyezése

Az AK-motor segítő virtuális gépe még nem hozott létre Kubernetes-fürtöt Azure Stack hub-on. A fürt létrehozása az az első művelet, amelyet az AK motor segítő virtuális gépe elvégez.

A részletes folyamat dokumentálása itt található:

* [Kubernetes-fürt üzembe helyezése az AK-motorral Azure Stack hub-on](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

A parancs végeredménye `aks-engine deploy` és az előző lépésekben szereplő előkészületek egy teljesen Kiemelt Kubernetes-fürt, amely az első Azure stack hub-példány bérlői területére lett telepítve. Maga a fürt olyan Azure IaaS-összetevőkből áll, mint például a virtuális gépek, a terheléselosztó, a virtuális hálózatok, a lemezek és így tovább.

![A fürt IaaS összetevői Azure Stack központ portál](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer (K8s API-végpont)
2) Munkavégző csomópontok (ügynök-készlet)
3) Főcsomópontok

A fürt most már működik, és a következő lépésben csatlakozni fog hozzá.

## <a name="connect-to-the-kubernetes-cluster"></a>Kapcsolódás a Kubernetes-fürthöz

Mostantól csatlakozhat a korábban létrehozott Kubernetes-fürthöz SSH-n keresztül (az üzembe helyezés részeként megadott SSH-kulccsal) vagy a `kubectl` (javasolt) használatával. A Kubernetes parancssori eszköz a `kubectl` Windows, Linux és MacOS rendszerekhez érhető el. [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/) Már előre telepítve és konfigurálva van a fürt fő csomópontjain.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kubectl végrehajtása a főcsomóponton](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

A főcsomópontot nem ajánlott felügyeleti feladatokhoz Jumpbox használni. A `kubectl` konfiguráció a `.kube/config` fő csomópont (ok) ban, valamint az AK-beli motor virtuális gépen található. A konfigurációt egy olyan felügyeleti gépre másolhatja, amely kapcsolódik a Kubernetes-fürthöz, és használja a `kubectl` parancsot. A `.kube/config` fájlt később is felhasználjuk a szolgáltatás Azure-folyamatokban való konfigurálásához.

> [!IMPORTANT]
> Tartsa biztonságban ezeket a fájlokat, mert ezek tartalmazzák a Kubernetes-fürthöz tartozó hitelesítő adatokat. A fájlhoz való hozzáféréssel rendelkező támadó elegendő információval rendelkezik ahhoz, hogy rendszergazdai hozzáférést szerezzen. A kezdeti fájl használatával végzett összes művelet `.kube/config` egy fürt-rendszergazda fiók használatával történik.

Most már kipróbálhat különböző parancsokat a használatával a `kubectl` fürt állapotának vizsgálatához.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> A Kubernetes saját,*szerepköralapú Access Control (RBAC)** modellel rendelkezik, amely lehetővé teszi a részletes szerepkör-definíciók és a szerepkör-kötések létrehozását. Ez a lehetőség a fürthöz való hozzáférés szabályozására szolgál a fürt rendszergazdai engedélyeinek kihasználása helyett.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Azure-folyamatok összekötése Kubernetes-fürtökkel

Az Azure-folyamatok az újonnan telepített Kubernetes-fürthöz való összekapcsolásához a Kube config ( `.kube/config` ) fájlra van szükség az előző lépésben leírtak szerint.

* Kapcsolódjon a Kubernetes-fürt egyik főcsomópontjaihoz.
* Másolja a `.kube/config` fájl tartalmát.
* Ugrás az Azure DevOps > a Project Settings > szolgáltatás kapcsolatai új "Kubernetes" szolgáltatás kapcsolatának létrehozásához (a KubeConfig használata hitelesítési módszerként)

> [!IMPORTANT]
> Az Azure-folyamatoknak (vagy a saját Build ügynökének) hozzáféréssel kell rendelkezniük a Kubernetes API-hoz. Ha van internetkapcsolat az Azure-folyamatokból az Azure Stack hub Kubernetes clusetr, akkor telepítenie kell egy saját üzemeltetésű Azure-folyamatokat.

Ha saját üzemeltetésű ügynököket helyez üzembe az Azure-folyamatokhoz, akkor a Azure Stack hub-on vagy egy olyan gépen is üzembe helyezhető, amely hálózati kapcsolattal rendelkezik az összes szükséges felügyeleti végponthoz. Itt találja a részleteket:

* Windows vagy [Linux](/azure/devops/pipelines/agents/v2-linux) [rendszerű](/azure/devops/pipelines/agents/v2-windows) [Azure-folyamatok ügynökei](/azure/devops/pipelines/agents/agents)

A minta [üzembe helyezés (CI/CD) szempontjai](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) című szakasz egy olyan döntési folyamatot tartalmaz, amely segít megismerni, hogy a Microsoft által üzemeltetett ügynököket vagy saját üzemeltetésű ügynököket használ-e:

[![döntési folyamat – saját üzemeltetésű ügynökök](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

Ebben a példában a topológia egy saját üzemeltetésű Build ügynököt tartalmaz minden Azure Stack hub-példányon. Az ügynök elérheti az Azure Stack hub felügyeleti végpontokat és a Kubernetes-fürt API-végpontokat.

[![csak kimenő forgalom](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Ez a kialakítás közös szabályozási követelménynek felel meg, amely csak az alkalmazás-megoldás kimenő kapcsolataira vonatkozik.

## <a name="configure-monitoring"></a>Figyelés konfigurálása

A tárolók [Azure monitor](/azure/azure-monitor/) használhatók a megoldásban lévő tárolók figyelésére. Ez a pont a Azure Stack hub-beli Kubernetes-fürtre Azure Monitor.

A Azure Monitor kétféleképpen engedélyezhető a fürtön. Mindkét módszernél szükség van egy Log Analytics munkaterület beállítására az Azure-ban.

* A [metódus](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) egy Helm-diagramot használ
* [Második módszer](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) az AK motor fürtjének specifikációjának részeként

A minta topológiában az "a metódus" használatos, amely lehetővé teszi a folyamat automatizálását, és a frissítések könnyebben telepíthetők.

A következő lépéshez szüksége lesz egy Azure LogAnalytics-munkaterületre (azonosító és kulcs), `Helm` (3. verzió) és a `kubectl` gépre.

A Helm egy Kubernetes-csomagkezelő, amely macOS, Windows és Linux rendszeren futtatott binárisként érhető el. Innen tölthető le: a [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm a parancshoz használt Kubernetes konfigurációs fájlra támaszkodik `kubectl` .

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Ez a parancs telepíti a Azure Monitor ügynököt a Kubernetes-fürtön:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

A Kubernetes-fürt Operations Management Suite-(OMS-) ügynöke a figyelési adatok küldését az Azure Log Analytics munkaterületére (kimenő HTTPS használatával). Mostantól a Azure Monitor segítségével mélyebb elemzéseket készíthet a Kubernetes-fürtökről Azure Stack hub-on. Ez a kialakítás hatékony módszert nyújt az alkalmazás fürtjével automatikusan üzembe helyezhető elemzések hatékonyságának bemutatására.

[![Azure Stack hub-fürtök az Azure monitorban](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Azure Monitor fürt részletei](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Ha Azure Monitor nem jelenít meg Azure Stack hub-információt, győződjön meg arról, hogy követte a [AzureMonitor-Containers megoldás Azure Loganalytics-munkaterülethez való hozzáadásának](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) lépéseit.

## <a name="deploy-the-application"></a>Az alkalmazás üzembe helyezése

A minta alkalmazás telepítése előtt egy másik lépés az Nginx-alapú beléptetési vezérlő konfigurálása a Kubernetes-fürtön. A bejövő adatok vezérlője 7. rétegbeli terheléselosztó, amely a gazdagép, az elérési út vagy a protokoll alapján irányítja át a forgalmat a fürtön. Nginx – a bejövő forgalom élén álló diagramként érhető el. Részletes utasításokért tekintse meg a [Helm chart GitHub-tárházat](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

A minta alkalmazás egy Helm-diagramként is be van csomagolva, mint az előző lépésben az [Azure monitoring Agent](#configure-monitoring) . Ezért egyszerű az alkalmazás üzembe helyezése a Kubernetes-fürtön. A [Helm-diagram fájljait a Companion GitHub-](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) tárházban találja

A minta alkalmazás egy háromrétegű alkalmazás, amely két Azure Stack hub-példányon található Kubernetes-fürtre van telepítve. Az alkalmazás MongoDB-adatbázist használ. További információk az adattípusok [és a tárolási megfontolások](pattern-highly-available-kubernetes.md#data-and-storage-considerations)több példány között replikált adatainak lekéréséről.

Az alkalmazáshoz tartozó Helm diagram üzembe helyezése után az alkalmazás mindhárom rétege az üzembe helyezések és az állapot-nyilvántartó készletek (az adatbázis esetében) egyetlen pod:

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

A szolgáltatások oldalon megtalálja az Nginx-alapú bejövő adatkezelőt és annak nyilvános IP-címét:

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

A "külső IP" cím az "Application Endpoint". Hogy a felhasználók hogyan csatlakoznak az alkalmazás megnyitásához, és a következő lépéshez tartozó végpontként is használhatók a [Traffic Manager konfigurálásához](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Az alkalmazás méretezése
Opcionálisan beállíthatja, hogy a [vízszintes hüvely automatikusan méretezhető](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) legyen bizonyos mérőszámok, például a processzor kihasználtsága alapján. A következő parancs egy horizontális Pod automéretezőt hoz létre, amely az értékelés – web telepítés által vezérelt hüvelyek 1 – 10 replikáját tárolja. A HPA megnöveli és csökkenti a replikák számát (az üzemelő példányon keresztül), hogy az átlagos CPU-kihasználtságot fenntartsa az 80%-os összes hüvelyben.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Az autoskálázás aktuális állapotát a futtatásával is megtekintheti:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>A Traffic Manager konfigurálása

Az alkalmazás két (vagy több) üzemelő példánya közötti adatforgalom elosztásához az [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)fogjuk használni. Az Azure Traffic Manager egy DNS-alapú forgalom terheléselosztó az Azure-ban.

> [!NOTE]
> Traffic Manager a DNS használatával irányítja az ügyfelek kérelmeit a legmegfelelőbb szolgáltatási végpontra a forgalom-útválasztási módszer és a végpontok állapota alapján.

Az Azure Traffic Manager használata helyett használhat más, a helyszínen üzemeltetett globális terheléselosztási megoldásokat is. A minta forgatókönyvben az Azure Traffic Manager használatával osztja el a forgalmat az alkalmazás két példánya között. Azure Stack hub-példányokon futhatnak ugyanazon vagy különböző helyszíneken:

![helyszíni Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

Az Azure-ban úgy konfiguráljuk a Traffic Manager, hogy az alkalmazás két különböző példányára mutasson:

[![TM-végpont profilja](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Amint látható, a két végpont az üzembe helyezett alkalmazás két példányára mutat az [előző szakaszból](#deploy-the-application).

Ezen a ponton:
- A rendszer létrehozta a Kubernetes-infrastruktúrát, beleértve a bejövő vezérlőt is.
- A fürtök két Azure Stack hub-példányon lettek telepítve.
- A figyelés konfigurálva van.
- Az Azure Traffic Manager a két Azure Stack hub-példány közötti adatforgalom terheléselosztását fogja elosztani.
- Ezen infrastruktúra tetején a minta háromrészes alkalmazás üzembe helyezése automatizált módon, Helm-diagramok használatával történt. 

A megoldásnak mostantól elérhetőnek kell lennie a felhasználók számára.

Az üzembe helyezés utáni működési megfontolások is megtalálhatók, amelyek a következő két szakaszban vannak lefoglalva.

## <a name="upgrade-kubernetes"></a>Kubernetes frissítése

A Kubernetes-fürt frissítésekor vegye figyelembe a következő témaköröket:

- A Kubernetes-fürtök frissítése egy 2. napos művelet, amely az AK-motor használatával végezhető el. További információ: Kubernetes- [fürt frissítése Azure stack hub-on](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Az AK-motorral frissítheti a fürtöket az újabb Kubernetes és az operációs rendszer rendszerképének verzióira. További információ: [új Kubernetes-verzióra való frissítés lépései](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- A kihelyezett csomópontokat csak az operációs rendszer újabb rendszerképének verziójára frissítheti. További információ: [csak az operációsrendszer-rendszerkép frissítésének lépései](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Az újabb alap operációsrendszer-lemezképek biztonsági és kernel-frissítéseket tartalmaznak. Az új Kubernetes-verziók és operációsrendszer-lemezképek rendelkezésre állásának figyelése a fürt operátorának feladata. Az operátornak az AK motorral kell megterveznie és végrehajtania ezeket a frissítéseket. Az alapoperációs rendszer lemezképeit le kell tölteni az Azure Stack hub piactérről az Azure Stack hub-kezelővel.

## <a name="scale-kubernetes"></a>Kubernetes skálázása

A Scale egy másik nap 2 művelet, amely az AK-motor használatával állítható össze.

A Scale parancs újrahasznosítja a fürt konfigurációs fájlját (apimodel.json) a kimeneti könyvtárban, egy új Azure Resource Manager-telepítés bemenete. Az AK-motor végrehajtja a skálázási műveletet egy adott ügynök-készleten. A skálázási művelet befejezésekor a (z)-os motor frissíti a fürt definícióját ugyanazon a apimodel.jsfájlon. A fürt definíciója az új csomópontok számának változását tükrözi a frissített, aktuális fürtkonfiguráció megjelenítéséhez.

- [Kubernetes-fürt méretezése Azure Stack hub-on](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Következő lépések

- További információ a [hibrid alkalmazások kialakításával kapcsolatos szempontokról](overview-app-design-considerations.md)
- Tekintse át és javasolja [a githubon a minta kódjának](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)tökéletesítését.