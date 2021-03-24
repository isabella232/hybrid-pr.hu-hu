---
title: Hibrid felhőalapú identitás konfigurálása az Azure-hoz és Azure Stack hub-alkalmazásokhoz
description: Ismerje meg, hogyan konfigurálhatja az Azure és Azure Stack hub-alkalmazások hibrid felhőalapú identitását.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895346"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Hibrid felhőalapú identitás konfigurálása az Azure-hoz és Azure Stack hub-alkalmazásokhoz

Megtudhatja, hogyan konfigurálhat hibrid Felhőbeli identitást Azure-és Azure Stack hub-alkalmazásaihoz.

Két lehetősége van arra, hogy hozzáférjen az alkalmazásaihoz a globális Azure-ban és Azure Stack hub-ban is.

 * Ha Azure Stack hub folyamatos internetkapcsolattal rendelkezik, használhatja a Azure Active Directory (Azure AD).
 * Ha Azure Stack hub le van választva az internetről, használhatja az Azure Directory összevont szolgáltatások (AD FS) szolgáltatást.

Az egyszerű szolgáltatásokkal hozzáférést biztosíthat a Azure Stack hub-alkalmazásokhoz az Azure Stack hub Azure Resource Manager használatával történő üzembe helyezéshez vagy konfiguráláshoz.

Ebben a megoldásban egy példaként szolgáló környezetet fog kiépíteni a következőkre:

> [!div class="checklist"]
> - Hibrid identitás létrehozása a globális Azure-ban és Azure Stack hub-ban
> - Token lekérése az Azure Stack hub API eléréséhez.

Ennek a megoldásnak a lépéseihez Azure Stack hub-kezelő engedélyekkel kell rendelkeznie.

> [!Tip]  
> ![Hibrid oszlopok diagramja](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack hub az Azure kiterjesztése. Azure Stack hub a felhő-számítástechnika rugalmasságát és innovációját a helyszíni környezetbe helyezi, így az egyetlen hibrid felhő, amely lehetővé teszi a hibrid alkalmazások bárhol történő létrehozását és üzembe helyezését.  
> 
> A [hibrid alkalmazások kialakításával kapcsolatos megfontolások](overview-app-design-considerations.md) a szoftverek minőségének (elhelyezés, skálázhatóság, rendelkezésre állás, rugalmasság, kezelhetőség és biztonság) pilléreit tekintik át hibrid alkalmazások tervezéséhez, üzembe helyezéséhez és üzemeltetéséhez. A kialakítási szempontok segítik a hibrid alkalmazások kialakításának optimalizálását, ami minimalizálja az éles környezetekben felmerülő kihívásokat.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Egyszerű szolgáltatásnév létrehozása az Azure AD-hez a portálon

Ha Azure Stack hub-t az Azure AD-ben telepítette az identitás-tárolóként, az Azure-hoz hasonlóan az egyszerű szolgáltatásokat is létrehozhatja. Az [erőforrások eléréséhez használt alkalmazás-identitással](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) megtudhatja, hogyan hajthatja végre a lépéseket a portálon. A Kezdés előtt győződjön meg arról, hogy rendelkezik a [szükséges Azure ad-engedélyekkel](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Egyszerű szolgáltatásnév létrehozása AD FShoz a PowerShell használatával

Ha AD FS használatával telepített Azure Stack hubot, a PowerShell segítségével létrehozhat egy egyszerű szolgáltatásnevet, hozzárendelhet egy szerepkört a hozzáféréshez, és bejelentkezhet a PowerShellből az identitás használatával. Az [erőforrások eléréséhez használt alkalmazás-identitással](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) megtudhatja, hogyan hajthatja végre a szükséges lépéseket a PowerShell használatával.

## <a name="using-the-azure-stack-hub-api"></a>Az Azure Stack hub API használata

Az [Azure stack hub API](/azure-stack/user/azure-stack-rest-api-use)  -megoldás végigvezeti a token lekérésének folyamatán az Azure stack hub API eléréséhez.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Kapcsolódás Azure Stack hubhoz a PowerShell használatával

Az [Azure stack hub PowerShell-](/azure-stack/operator/azure-stack-powershell-install) lel való üzembe helyezéséhez szükséges gyors útmutató végigvezeti a Azure PowerShell telepítésének és a Azure stack hub-telepítéshez való kapcsolódásnak a lépésein.

### <a name="prerequisites"></a>Előfeltételek

Egy, az Azure AD-hez csatlakoztatott Azure Stack hub-telepítésre van szükség, amelyhez hozzá tud férni. Ha nem rendelkezik Azure Stack hub-telepítéssel, a következő utasításokat követve állíthatja be a [Azure stack Development Kitt (ASDK)](/azure-stack/asdk/asdk-install).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Kapcsolódás Azure Stack hubhoz kód használatával

Ha kódot használ a Azure Stack hubhoz való kapcsolódáshoz, használja a Azure Resource Manager endpoints API-t az Azure Stack hub-telepítéshez tartozó hitelesítési és gráf végpontok beszerzéséhez. Ezután végezze el a hitelesítést a REST-kérelmek használatával. A [githubon](https://github.com/shriramnat/HybridARMApplication)megtalálhatja a minta ügyfélalkalmazás alkalmazást.

>[!Note]
>Ha a választott nyelvhez készült Azure SDK támogatja az Azure API-profilokat, előfordulhat, hogy az SDK nem működik együtt Azure Stack hub-val. Az Azure API-profilokkal kapcsolatos további tudnivalókért tekintse meg az [API-verzió profiljainak kezelése](/azure-stack/user/azure-stack-version-profiles) című cikket.

## <a name="next-steps"></a>Következő lépések

- Ha többet szeretne megtudni arról, hogyan kezeli az identitást Azure Stack központban, tekintse meg a [Azure stack hub Identity Architecture](/azure-stack/operator/azure-stack-identity-architecture)című témakört.
- Az Azure Cloud Patterns szolgáltatással kapcsolatos további információkért lásd: [Felhőbeli tervezési minták](/azure/architecture/patterns).
