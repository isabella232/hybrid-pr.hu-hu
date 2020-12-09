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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="3b516-103">Magas rendelkezésre állású Kubernetes-fürt üzembe helyezése Azure Stack központban</span><span class="sxs-lookup"><span data-stu-id="3b516-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="3b516-104">Ebből a cikkből megtudhatja, hogyan hozhat létre több Azure Stack hub-példányon, különböző fizikai helyen üzembe helyezett, magasan elérhető Kubernetes-fürtöt.</span><span class="sxs-lookup"><span data-stu-id="3b516-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="3b516-105">A megoldás telepítési útmutatójában a következőket sajátíthatja el:</span><span class="sxs-lookup"><span data-stu-id="3b516-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3b516-106">Az AK-motor letöltése és előkészítése</span><span class="sxs-lookup"><span data-stu-id="3b516-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="3b516-107">Kapcsolódás az KABAi motor segítő virtuális géphez</span><span class="sxs-lookup"><span data-stu-id="3b516-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="3b516-108">Kubernetes-fürt üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3b516-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="3b516-109">Kapcsolódás a Kubernetes-fürthöz</span><span class="sxs-lookup"><span data-stu-id="3b516-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="3b516-110">Azure-folyamatok összekötése a Kubernetes-fürttel</span><span class="sxs-lookup"><span data-stu-id="3b516-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="3b516-111">Figyelés konfigurálása</span><span class="sxs-lookup"><span data-stu-id="3b516-111">Configure monitoring</span></span>
> - <span data-ttu-id="3b516-112">Alkalmazás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3b516-112">Deploy application</span></span>
> - <span data-ttu-id="3b516-113">Alkalmazás méretezése</span><span class="sxs-lookup"><span data-stu-id="3b516-113">Autoscale application</span></span>
> - <span data-ttu-id="3b516-114">A Traffic Manager konfigurálása</span><span class="sxs-lookup"><span data-stu-id="3b516-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="3b516-115">Kubernetes frissítése</span><span class="sxs-lookup"><span data-stu-id="3b516-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="3b516-116">Kubernetes skálázása</span><span class="sxs-lookup"><span data-stu-id="3b516-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="3b516-117">![Hibrid oszlopok](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3b516-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3b516-118">Microsoft Azure Stack hub az Azure kiterjesztése.</span><span class="sxs-lookup"><span data-stu-id="3b516-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3b516-119">Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="3b516-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3b516-120">A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez.</span><span class="sxs-lookup"><span data-stu-id="3b516-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3b516-121">A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.</span><span class="sxs-lookup"><span data-stu-id="3b516-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3b516-122">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="3b516-122">Prerequisites</span></span>

<span data-ttu-id="3b516-123">Az üzembe helyezési útmutató első lépéseinek megkezdése előtt győződjön meg arról, hogy:</span><span class="sxs-lookup"><span data-stu-id="3b516-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="3b516-124">Tekintse át a [magas rendelkezésre állású Kubernetes-fürt mintáját](pattern-highly-available-kubernetes.md) ismertető cikket.</span><span class="sxs-lookup"><span data-stu-id="3b516-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="3b516-125">Tekintse át a [Companion GitHub-tárház](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)tartalmát, amely a cikkben hivatkozott további eszközöket tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="3b516-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="3b516-126">Rendelkeznie kell egy olyan fiókkal, amely hozzáfér a [Azure stack hub felhasználói portálhoz](/azure-stack/user/azure-stack-use-portal), amely legalább ["közreműködői" engedélyekkel](/azure-stack/user/azure-stack-manage-permissions)rendelkezik.</span><span class="sxs-lookup"><span data-stu-id="3b516-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="3b516-127">AK-motor letöltése és előkészítése</span><span class="sxs-lookup"><span data-stu-id="3b516-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="3b516-128">Az AK-motor egy bináris fájl, amely bármely olyan Windows-vagy Linux-gazdagépről használható, amely elérheti az Azure Stack hub Azure Resource Manager-végpontokat.</span><span class="sxs-lookup"><span data-stu-id="3b516-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="3b516-129">Ez az útmutató egy új Linux (vagy Windows) virtuális gép üzembe helyezését ismerteti Azure Stack hub-on.</span><span class="sxs-lookup"><span data-stu-id="3b516-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="3b516-130">Később lesz használatban, amikor az AK-motor telepíti a Kubernetes-fürtöket.</span><span class="sxs-lookup"><span data-stu-id="3b516-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="3b516-131">Egy meglévő Windows vagy Linux rendszerű virtuális gép használatával üzembe helyezhet egy Kubernetes-fürtöt Azure Stack hub-ban az AK motor használatával.</span><span class="sxs-lookup"><span data-stu-id="3b516-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="3b516-132">Az AK-motor lépésenkénti folyamatát és követelményeit a következő dokumentálja:</span><span class="sxs-lookup"><span data-stu-id="3b516-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="3b516-133">[Az AK-motor telepítése Linux rendszerű Azure stack hub-ban](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (vagy [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows)használatával)</span><span class="sxs-lookup"><span data-stu-id="3b516-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="3b516-134">Az AK-motor egy segítő eszköz a (nem felügyelt) Kubernetes-fürtök üzembe helyezéséhez és üzemeltetéséhez (az Azure-ban és Azure Stack hub-ban).</span><span class="sxs-lookup"><span data-stu-id="3b516-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="3b516-135">A Azure Stack hub-ban található AK-motor részleteit és különbségeit itt találja:</span><span class="sxs-lookup"><span data-stu-id="3b516-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="3b516-136">Mi a Azure Stack hub AK-motorja?</span><span class="sxs-lookup"><span data-stu-id="3b516-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="3b516-137">[AK-motor Azure stack hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) -on (a githubon)</span><span class="sxs-lookup"><span data-stu-id="3b516-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="3b516-138">A minta környezet a Terraform segítségével automatizálja az KABAi motor virtuális gép üzembe helyezését.</span><span class="sxs-lookup"><span data-stu-id="3b516-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="3b516-139">A [részleteket és kódokat a Companion GitHub-](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md)tárházban találja.</span><span class="sxs-lookup"><span data-stu-id="3b516-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="3b516-140">Ennek a lépésnek az eredménye egy új erőforráscsoport a Azure Stack hub-on, amely az AK motor segítő virtuális gépet és a kapcsolódó erőforrásokat tartalmazza:</span><span class="sxs-lookup"><span data-stu-id="3b516-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![KABAi motor virtuálisgép-erőforrásai Azure Stack központban](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="3b516-142">Ha az AK-motort egy leválasztott gapped-környezetben kell üzembe helyeznie, tekintse át a [leválasztott Azure stack hub-példányokat](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) további információért.</span><span class="sxs-lookup"><span data-stu-id="3b516-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="3b516-143">A következő lépésben az újonnan üzembe helyezett AK-beli virtuális gépet használjuk egy Kubernetes-fürt üzembe helyezéséhez.</span><span class="sxs-lookup"><span data-stu-id="3b516-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="3b516-144">Kapcsolódás az KABAi motor segítő virtuális géphez</span><span class="sxs-lookup"><span data-stu-id="3b516-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="3b516-145">Először kapcsolódnia kell a korábban létrehozott KABAi motor segítő virtuális géphez.</span><span class="sxs-lookup"><span data-stu-id="3b516-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="3b516-146">A virtuális gépnek nyilvános IP-címmel kell rendelkeznie, és az SSH-n keresztül kell elérhetőnek lennie (a 22-es porton keresztül).</span><span class="sxs-lookup"><span data-stu-id="3b516-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Az AK-motor virtuálisgép-áttekintő lapja](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="3b516-148">Az SSH-val Linux rendszerű virtuális gépekhez való kapcsolódáshoz használhatja a MobaXterm, a PuTTY vagy a PowerShell Windows 10-es eszközét.</span><span class="sxs-lookup"><span data-stu-id="3b516-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="3b516-149">A csatlakozás után futtassa a parancsot `aks-engine` .</span><span class="sxs-lookup"><span data-stu-id="3b516-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="3b516-150">Az AK-motor és a Kubernetes verzióival kapcsolatos további tudnivalókért lépjen a [támogatott AK-hajtóművek verzióihoz](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) .</span><span class="sxs-lookup"><span data-stu-id="3b516-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![AK-motor parancssori példája](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="3b516-152">Kubernetes-fürt üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3b516-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="3b516-153">Az AK-motor segítő virtuális gépe még nem hozott létre Kubernetes-fürtöt Azure Stack hub-on.</span><span class="sxs-lookup"><span data-stu-id="3b516-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="3b516-154">A fürt létrehozása az az első művelet, amelyet az AK motor segítő virtuális gépe elvégez.</span><span class="sxs-lookup"><span data-stu-id="3b516-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="3b516-155">A részletes folyamat dokumentálása itt található:</span><span class="sxs-lookup"><span data-stu-id="3b516-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="3b516-156">Kubernetes-fürt üzembe helyezése az AK-motorral Azure Stack hub-on</span><span class="sxs-lookup"><span data-stu-id="3b516-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="3b516-157">A parancs végeredménye `aks-engine deploy` és az előző lépésekben szereplő előkészületek egy teljesen Kiemelt Kubernetes-fürt, amely az első Azure stack hub-példány bérlői területére lett telepítve.</span><span class="sxs-lookup"><span data-stu-id="3b516-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="3b516-158">Maga a fürt olyan Azure IaaS-összetevőkből áll, mint például a virtuális gépek, a terheléselosztó, a virtuális hálózatok, a lemezek és így tovább.</span><span class="sxs-lookup"><span data-stu-id="3b516-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![A fürt IaaS összetevői Azure Stack központ portál](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="3b516-160">Azure Load Balancer (K8s API-végpont)</span><span class="sxs-lookup"><span data-stu-id="3b516-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="3b516-161">Munkavégző csomópontok (ügynök-készlet)</span><span class="sxs-lookup"><span data-stu-id="3b516-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="3b516-162">Főcsomópontok</span><span class="sxs-lookup"><span data-stu-id="3b516-162">Master Nodes</span></span>

<span data-ttu-id="3b516-163">A fürt most már működik, és a következő lépésben csatlakozni fog hozzá.</span><span class="sxs-lookup"><span data-stu-id="3b516-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="3b516-164">Kapcsolódás a Kubernetes-fürthöz</span><span class="sxs-lookup"><span data-stu-id="3b516-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="3b516-165">Mostantól csatlakozhat a korábban létrehozott Kubernetes-fürthöz SSH-n keresztül (az üzembe helyezés részeként megadott SSH-kulccsal) vagy a `kubectl` (javasolt) használatával.</span><span class="sxs-lookup"><span data-stu-id="3b516-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="3b516-166">A Kubernetes parancssori eszköz a `kubectl` Windows, Linux és MacOS rendszerekhez érhető el. [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)</span><span class="sxs-lookup"><span data-stu-id="3b516-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="3b516-167">Már előre telepítve és konfigurálva van a fürt fő csomópontjain.</span><span class="sxs-lookup"><span data-stu-id="3b516-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Kubectl végrehajtása a főcsomóponton](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="3b516-169">A főcsomópontot nem ajánlott felügyeleti feladatokhoz Jumpbox használni.</span><span class="sxs-lookup"><span data-stu-id="3b516-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="3b516-170">A `kubectl` konfiguráció a `.kube/config` fő csomópont (ok) ban, valamint az AK-beli motor virtuális gépen található.</span><span class="sxs-lookup"><span data-stu-id="3b516-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="3b516-171">A konfigurációt egy olyan felügyeleti gépre másolhatja, amely kapcsolódik a Kubernetes-fürthöz, és használja a `kubectl` parancsot.</span><span class="sxs-lookup"><span data-stu-id="3b516-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="3b516-172">A `.kube/config` fájlt később is felhasználjuk a szolgáltatás Azure-folyamatokban való konfigurálásához.</span><span class="sxs-lookup"><span data-stu-id="3b516-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3b516-173">Tartsa biztonságban ezeket a fájlokat, mert ezek tartalmazzák a Kubernetes-fürthöz tartozó hitelesítő adatokat.</span><span class="sxs-lookup"><span data-stu-id="3b516-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="3b516-174">A fájlhoz való hozzáféréssel rendelkező támadó elegendő információval rendelkezik ahhoz, hogy rendszergazdai hozzáférést szerezzen.</span><span class="sxs-lookup"><span data-stu-id="3b516-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="3b516-175">A kezdeti fájl használatával végzett összes művelet `.kube/config` egy fürt-rendszergazda fiók használatával történik.</span><span class="sxs-lookup"><span data-stu-id="3b516-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="3b516-176">Most már kipróbálhat különböző parancsokat a használatával a `kubectl` fürt állapotának vizsgálatához.</span><span class="sxs-lookup"><span data-stu-id="3b516-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="3b516-177">A Kubernetes saját,*szerepköralapú Access Control (RBAC)*\* modellel rendelkezik, amely lehetővé teszi a részletes szerepkör-definíciók és a szerepkör-kötések létrehozását.</span><span class="sxs-lookup"><span data-stu-id="3b516-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="3b516-178">Ez a lehetőség a fürthöz való hozzáférés szabályozására szolgál a fürt rendszergazdai engedélyeinek kihasználása helyett.</span><span class="sxs-lookup"><span data-stu-id="3b516-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="3b516-179">Azure-folyamatok összekötése Kubernetes-fürtökkel</span><span class="sxs-lookup"><span data-stu-id="3b516-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="3b516-180">Az Azure-folyamatok az újonnan telepített Kubernetes-fürthöz való összekapcsolásához a Kube config ( `.kube/config` ) fájlra van szükség az előző lépésben leírtak szerint.</span><span class="sxs-lookup"><span data-stu-id="3b516-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="3b516-181">Kapcsolódjon a Kubernetes-fürt egyik főcsomópontjaihoz.</span><span class="sxs-lookup"><span data-stu-id="3b516-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="3b516-182">Másolja a `.kube/config` fájl tartalmát.</span><span class="sxs-lookup"><span data-stu-id="3b516-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="3b516-183">Ugrás az Azure DevOps > a Project Settings > szolgáltatás kapcsolatai új "Kubernetes" szolgáltatás kapcsolatának létrehozásához (a KubeConfig használata hitelesítési módszerként)</span><span class="sxs-lookup"><span data-stu-id="3b516-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3b516-184">Az Azure-folyamatoknak (vagy a saját Build ügynökének) hozzáféréssel kell rendelkezniük a Kubernetes API-hoz.</span><span class="sxs-lookup"><span data-stu-id="3b516-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="3b516-185">Ha van internetkapcsolat az Azure-folyamatokból az Azure Stack hub Kubernetes clusetr, akkor telepítenie kell egy saját üzemeltetésű Azure-folyamatokat.</span><span class="sxs-lookup"><span data-stu-id="3b516-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="3b516-186">Ha saját üzemeltetésű ügynököket helyez üzembe az Azure-folyamatokhoz, akkor a Azure Stack hub-on vagy egy olyan gépen is üzembe helyezhető, amely hálózati kapcsolattal rendelkezik az összes szükséges felügyeleti végponthoz.</span><span class="sxs-lookup"><span data-stu-id="3b516-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="3b516-187">Itt találja a részleteket:</span><span class="sxs-lookup"><span data-stu-id="3b516-187">See the details here:</span></span>

* <span data-ttu-id="3b516-188">Windows vagy [Linux](/azure/devops/pipelines/agents/v2-linux) [rendszerű](/azure/devops/pipelines/agents/v2-windows) [Azure-folyamatok ügynökei](/azure/devops/pipelines/agents/agents)</span><span class="sxs-lookup"><span data-stu-id="3b516-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="3b516-189">A minta [üzembe helyezés (CI/CD) szempontjai](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) című szakasz egy olyan döntési folyamatot tartalmaz, amely segít megismerni, hogy a Microsoft által üzemeltetett ügynököket vagy saját üzemeltetésű ügynököket használ-e:</span><span class="sxs-lookup"><span data-stu-id="3b516-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="3b516-190">[![döntési folyamat – saját üzemeltetésű ügynökök](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3b516-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="3b516-191">Ebben a példában a topológia egy saját üzemeltetésű Build ügynököt tartalmaz minden Azure Stack hub-példányon.</span><span class="sxs-lookup"><span data-stu-id="3b516-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="3b516-192">Az ügynök elérheti az Azure Stack hub felügyeleti végpontokat és a Kubernetes-fürt API-végpontokat.</span><span class="sxs-lookup"><span data-stu-id="3b516-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="3b516-193">[![csak kimenő forgalom](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3b516-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="3b516-194">Ez a kialakítás közös szabályozási követelménynek felel meg, amely csak az alkalmazás-megoldás kimenő kapcsolataira vonatkozik.</span><span class="sxs-lookup"><span data-stu-id="3b516-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="3b516-195">Figyelés konfigurálása</span><span class="sxs-lookup"><span data-stu-id="3b516-195">Configure monitoring</span></span>

<span data-ttu-id="3b516-196">A tárolók [Azure monitor](/azure/azure-monitor/) használhatók a megoldásban lévő tárolók figyelésére.</span><span class="sxs-lookup"><span data-stu-id="3b516-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="3b516-197">Ez a pont a Azure Stack hub-beli Kubernetes-fürtre Azure Monitor.</span><span class="sxs-lookup"><span data-stu-id="3b516-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="3b516-198">A Azure Monitor kétféleképpen engedélyezhető a fürtön.</span><span class="sxs-lookup"><span data-stu-id="3b516-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="3b516-199">Mindkét módszernél szükség van egy Log Analytics munkaterület beállítására az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3b516-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="3b516-200">A [metódus](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) egy Helm-diagramot használ</span><span class="sxs-lookup"><span data-stu-id="3b516-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="3b516-201">[Második módszer](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) az AK motor fürtjének specifikációjának részeként</span><span class="sxs-lookup"><span data-stu-id="3b516-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="3b516-202">A minta topológiában az "a metódus" használatos, amely lehetővé teszi a folyamat automatizálását, és a frissítések könnyebben telepíthetők.</span><span class="sxs-lookup"><span data-stu-id="3b516-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="3b516-203">A következő lépéshez szüksége lesz egy Azure LogAnalytics-munkaterületre (azonosító és kulcs), `Helm` (3. verzió) és a `kubectl` gépre.</span><span class="sxs-lookup"><span data-stu-id="3b516-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="3b516-204">A Helm egy Kubernetes-csomagkezelő, amely macOS, Windows és Linux rendszeren futtatott binárisként érhető el.</span><span class="sxs-lookup"><span data-stu-id="3b516-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="3b516-205">Innen tölthető le: a [Helm.sh](https://helm.sh/docs/intro/quickstart/) Helm a parancshoz használt Kubernetes konfigurációs fájlra támaszkodik `kubectl` .</span><span class="sxs-lookup"><span data-stu-id="3b516-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="3b516-206">Ez a parancs telepíti a Azure Monitor ügynököt a Kubernetes-fürtön:</span><span class="sxs-lookup"><span data-stu-id="3b516-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="3b516-207">A Kubernetes-fürt Operations Management Suite-(OMS-) ügynöke a figyelési adatok küldését az Azure Log Analytics munkaterületére (kimenő HTTPS használatával).</span><span class="sxs-lookup"><span data-stu-id="3b516-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="3b516-208">Mostantól a Azure Monitor segítségével mélyebb elemzéseket készíthet a Kubernetes-fürtökről Azure Stack hub-on.</span><span class="sxs-lookup"><span data-stu-id="3b516-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="3b516-209">Ez a kialakítás hatékony módszert nyújt az alkalmazás fürtjével automatikusan üzembe helyezhető elemzések hatékonyságának bemutatására.</span><span class="sxs-lookup"><span data-stu-id="3b516-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="3b516-210">[![Azure Stack hub-fürtök az Azure monitorban](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3b516-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="3b516-211">[![Azure Monitor fürt részletei](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3b516-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="3b516-212">Ha Azure Monitor nem jelenít meg Azure Stack hub-információt, győződjön meg arról, hogy követte a [AzureMonitor-Containers megoldás Azure Loganalytics-munkaterülethez való hozzáadásának](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) lépéseit.</span><span class="sxs-lookup"><span data-stu-id="3b516-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="3b516-213">Az alkalmazás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="3b516-213">Deploy the application</span></span>

<span data-ttu-id="3b516-214">A minta alkalmazás telepítése előtt egy másik lépés az Nginx-alapú beléptetési vezérlő konfigurálása a Kubernetes-fürtön.</span><span class="sxs-lookup"><span data-stu-id="3b516-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="3b516-215">A bejövő adatok vezérlője 7. rétegbeli terheléselosztó, amely a gazdagép, az elérési út vagy a protokoll alapján irányítja át a forgalmat a fürtön.</span><span class="sxs-lookup"><span data-stu-id="3b516-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="3b516-216">Nginx – a bejövő forgalom élén álló diagramként érhető el.</span><span class="sxs-lookup"><span data-stu-id="3b516-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="3b516-217">Részletes utasításokért tekintse meg a [Helm chart GitHub-tárházat](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="3b516-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="3b516-218">A minta alkalmazás egy Helm-diagramként is be van csomagolva, mint az előző lépésben az [Azure monitoring Agent](#configure-monitoring) .</span><span class="sxs-lookup"><span data-stu-id="3b516-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="3b516-219">Ezért egyszerű az alkalmazás üzembe helyezése a Kubernetes-fürtön.</span><span class="sxs-lookup"><span data-stu-id="3b516-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="3b516-220">A [Helm-diagram fájljait a Companion GitHub-](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm) tárházban találja</span><span class="sxs-lookup"><span data-stu-id="3b516-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="3b516-221">A minta alkalmazás egy háromrétegű alkalmazás, amely két Azure Stack hub-példányon található Kubernetes-fürtre van telepítve.</span><span class="sxs-lookup"><span data-stu-id="3b516-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="3b516-222">Az alkalmazás MongoDB-adatbázist használ.</span><span class="sxs-lookup"><span data-stu-id="3b516-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="3b516-223">További információk az adattípusok [és a tárolási megfontolások](pattern-highly-available-kubernetes.md#data-and-storage-considerations)több példány között replikált adatainak lekéréséről.</span><span class="sxs-lookup"><span data-stu-id="3b516-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="3b516-224">Az alkalmazáshoz tartozó Helm diagram üzembe helyezése után az alkalmazás mindhárom rétege az üzembe helyezések és az állapot-nyilvántartó készletek (az adatbázis esetében) egyetlen pod:</span><span class="sxs-lookup"><span data-stu-id="3b516-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="3b516-225">A szolgáltatások oldalon megtalálja az Nginx-alapú bejövő adatkezelőt és annak nyilvános IP-címét:</span><span class="sxs-lookup"><span data-stu-id="3b516-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="3b516-226">A "külső IP" cím az "Application Endpoint".</span><span class="sxs-lookup"><span data-stu-id="3b516-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="3b516-227">Hogy a felhasználók hogyan csatlakoznak az alkalmazás megnyitásához, és a következő lépéshez tartozó végpontként is használhatók a [Traffic Manager konfigurálásához](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="3b516-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="3b516-228">Az alkalmazás méretezése</span><span class="sxs-lookup"><span data-stu-id="3b516-228">Autoscale the application</span></span>
<span data-ttu-id="3b516-229">Opcionálisan beállíthatja, hogy a [vízszintes hüvely automatikusan méretezhető](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) legyen bizonyos mérőszámok, például a processzor kihasználtsága alapján.</span><span class="sxs-lookup"><span data-stu-id="3b516-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="3b516-230">A következő parancs egy horizontális Pod automéretezőt hoz létre, amely az értékelés – web telepítés által vezérelt hüvelyek 1 – 10 replikáját tárolja.</span><span class="sxs-lookup"><span data-stu-id="3b516-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="3b516-231">A HPA megnöveli és csökkenti a replikák számát (az üzemelő példányon keresztül), hogy az átlagos CPU-kihasználtságot fenntartsa az 80%-os összes hüvelyben.</span><span class="sxs-lookup"><span data-stu-id="3b516-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="3b516-232">Az autoskálázás aktuális állapotát a futtatásával is megtekintheti:</span><span class="sxs-lookup"><span data-stu-id="3b516-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="3b516-233">A Traffic Manager konfigurálása</span><span class="sxs-lookup"><span data-stu-id="3b516-233">Configure Traffic Manager</span></span>

<span data-ttu-id="3b516-234">Az alkalmazás két (vagy több) üzemelő példánya közötti adatforgalom elosztásához az [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview)fogjuk használni.</span><span class="sxs-lookup"><span data-stu-id="3b516-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="3b516-235">Az Azure Traffic Manager egy DNS-alapú forgalom terheléselosztó az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="3b516-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="3b516-236">Traffic Manager a DNS használatával irányítja az ügyfelek kérelmeit a legmegfelelőbb szolgáltatási végpontra a forgalom-útválasztási módszer és a végpontok állapota alapján.</span><span class="sxs-lookup"><span data-stu-id="3b516-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="3b516-237">Az Azure Traffic Manager használata helyett használhat más, a helyszínen üzemeltetett globális terheléselosztási megoldásokat is.</span><span class="sxs-lookup"><span data-stu-id="3b516-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="3b516-238">A minta forgatókönyvben az Azure Traffic Manager használatával osztja el a forgalmat az alkalmazás két példánya között.</span><span class="sxs-lookup"><span data-stu-id="3b516-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="3b516-239">Azure Stack hub-példányokon futhatnak ugyanazon vagy különböző helyszíneken:</span><span class="sxs-lookup"><span data-stu-id="3b516-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![helyszíni Traffic Manager](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="3b516-241">Az Azure-ban úgy konfiguráljuk a Traffic Manager, hogy az alkalmazás két különböző példányára mutasson:</span><span class="sxs-lookup"><span data-stu-id="3b516-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="3b516-242">[![TM-végpont profilja](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="3b516-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="3b516-243">Amint látható, a két végpont az üzembe helyezett alkalmazás két példányára mutat az [előző szakaszból](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="3b516-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="3b516-244">Ezen a ponton:</span><span class="sxs-lookup"><span data-stu-id="3b516-244">At this point:</span></span>
- <span data-ttu-id="3b516-245">A rendszer létrehozta a Kubernetes-infrastruktúrát, beleértve a bejövő vezérlőt is.</span><span class="sxs-lookup"><span data-stu-id="3b516-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="3b516-246">A fürtök két Azure Stack hub-példányon lettek telepítve.</span><span class="sxs-lookup"><span data-stu-id="3b516-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="3b516-247">A figyelés konfigurálva van.</span><span class="sxs-lookup"><span data-stu-id="3b516-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="3b516-248">Az Azure Traffic Manager a két Azure Stack hub-példány közötti adatforgalom terheléselosztását fogja elosztani.</span><span class="sxs-lookup"><span data-stu-id="3b516-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="3b516-249">Ezen infrastruktúra tetején a minta háromrészes alkalmazás üzembe helyezése automatizált módon, Helm-diagramok használatával történt.</span><span class="sxs-lookup"><span data-stu-id="3b516-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="3b516-250">A megoldásnak mostantól elérhetőnek kell lennie a felhasználók számára.</span><span class="sxs-lookup"><span data-stu-id="3b516-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="3b516-251">Az üzembe helyezés utáni működési megfontolások is megtalálhatók, amelyek a következő két szakaszban vannak lefoglalva.</span><span class="sxs-lookup"><span data-stu-id="3b516-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="3b516-252">Kubernetes frissítése</span><span class="sxs-lookup"><span data-stu-id="3b516-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="3b516-253">A Kubernetes-fürt frissítésekor vegye figyelembe a következő témaköröket:</span><span class="sxs-lookup"><span data-stu-id="3b516-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="3b516-254">A Kubernetes-fürtök frissítése egy 2. napos művelet, amely az AK-motor használatával végezhető el.</span><span class="sxs-lookup"><span data-stu-id="3b516-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="3b516-255">További információ: Kubernetes- [fürt frissítése Azure stack hub-on](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="3b516-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="3b516-256">Az AK-motorral frissítheti a fürtöket az újabb Kubernetes és az operációs rendszer rendszerképének verzióira.</span><span class="sxs-lookup"><span data-stu-id="3b516-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="3b516-257">További információ: [új Kubernetes-verzióra való frissítés lépései](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="3b516-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="3b516-258">A kihelyezett csomópontokat csak az operációs rendszer újabb rendszerképének verziójára frissítheti.</span><span class="sxs-lookup"><span data-stu-id="3b516-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="3b516-259">További információ: [csak az operációsrendszer-rendszerkép frissítésének lépései](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="3b516-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="3b516-260">Az újabb alap operációsrendszer-lemezképek biztonsági és kernel-frissítéseket tartalmaznak.</span><span class="sxs-lookup"><span data-stu-id="3b516-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="3b516-261">Az új Kubernetes-verziók és operációsrendszer-lemezképek rendelkezésre állásának figyelése a fürt operátorának feladata.</span><span class="sxs-lookup"><span data-stu-id="3b516-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="3b516-262">Az operátornak az AK motorral kell megterveznie és végrehajtania ezeket a frissítéseket.</span><span class="sxs-lookup"><span data-stu-id="3b516-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="3b516-263">Az alapoperációs rendszer lemezképeit le kell tölteni az Azure Stack hub piactérről az Azure Stack hub-kezelővel.</span><span class="sxs-lookup"><span data-stu-id="3b516-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="3b516-264">Kubernetes skálázása</span><span class="sxs-lookup"><span data-stu-id="3b516-264">Scale Kubernetes</span></span>

<span data-ttu-id="3b516-265">A Scale egy másik nap 2 művelet, amely az AK-motor használatával állítható össze.</span><span class="sxs-lookup"><span data-stu-id="3b516-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="3b516-266">A Scale parancs újrahasznosítja a fürt konfigurációs fájlját (apimodel.json) a kimeneti könyvtárban, egy új Azure Resource Manager-telepítés bemenete.</span><span class="sxs-lookup"><span data-stu-id="3b516-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="3b516-267">Az AK-motor végrehajtja a skálázási műveletet egy adott ügynök-készleten.</span><span class="sxs-lookup"><span data-stu-id="3b516-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="3b516-268">A skálázási művelet befejezésekor a (z)-os motor frissíti a fürt definícióját ugyanazon a apimodel.jsfájlon.</span><span class="sxs-lookup"><span data-stu-id="3b516-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="3b516-269">A fürt definíciója az új csomópontok számának változását tükrözi a frissített, aktuális fürtkonfiguráció megjelenítéséhez.</span><span class="sxs-lookup"><span data-stu-id="3b516-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="3b516-270">Kubernetes-fürt méretezése Azure Stack hub-on</span><span class="sxs-lookup"><span data-stu-id="3b516-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="3b516-271">Következő lépések</span><span class="sxs-lookup"><span data-stu-id="3b516-271">Next steps</span></span>

- <span data-ttu-id="3b516-272">További információ a [hibrid alkalmazások kialakításával kapcsolatos szempontokról](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="3b516-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="3b516-273">Tekintse át és javasolja [a githubon a minta kódjának](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub)tökéletesítését.</span><span class="sxs-lookup"><span data-stu-id="3b516-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>