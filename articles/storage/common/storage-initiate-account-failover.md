---
title: Initiera redundans för lagrings konto
titleSuffix: Azure Storage
description: Lär dig hur du startar ett konto vid fel i händelse av att den primära slut punkten för ditt lagrings konto blir otillgänglig. Redundansväxlingen uppdaterar den sekundära regionen för att bli den primära regionen för ditt lagrings konto.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 05/05/2020
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: 2d07195e28c964a540eafdfba94a958e6c9f6981
ms.sourcegitcommit: f57297af0ea729ab76081c98da2243d6b1f6fa63
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82871356"
---
# <a name="initiate-a-storage-account-failover"></a>Initiera redundans för lagrings konto

Om den primära slut punkten för ditt geo-redundanta lagrings konto inte är tillgänglig av någon anledning kan du initiera en redundansväxling av kontot. Vid en konto redundansväxling uppdateras den sekundära slut punkten till den primära slut punkten för ditt lagrings konto. När redundansväxlingen är klar kan klienter börja skriva till den nya primära regionen. Med Tvingad redundans kan du upprätthålla hög tillgänglighet för dina program.

Den här artikeln visar hur du startar en konto redundansväxling för ditt lagrings konto med hjälp av Azure Portal, PowerShell eller Azure CLI. Mer information om redundansväxling av konton finns i [haveri beredskap och redundans för lagrings konton](storage-disaster-recovery-guidance.md).

> [!WARNING]
> Ett konto växling vid fel resulterar vanligt vis i viss data förlust. Om du vill förstå konsekvenserna av en redundansväxling av ett konto och förbereda dig för data förlust, kan du läsa [förstå processen för redundans av konto](storage-disaster-recovery-guidance.md#understand-the-account-failover-process).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Krav

Innan du kan utföra en redundansväxling på ditt lagrings konto kontrollerar du att ditt lagrings konto har kon figurer ATS för geo-replikering. Ditt lagrings konto kan använda något av följande alternativ för redundans:

- Geo-redundant lagring (GRS) eller Geo-redundant lagring med Läs behörighet (RA-GRS)
- Geo-Zone-redundant lagring (GZRS) eller (Read-Access geo-Zone-redundant lagring) (RA-GZRS)

Mer information om Azure Storage redundans finns [Azure Storage redundans](storage-redundancy.md).

## <a name="initiate-the-failover"></a>Initiera redundansväxlingen

## <a name="portal"></a>[Portal](#tab/azure-portal)

Följ dessa steg om du vill starta en redundansväxling av ett konto från Azure Portal:

1. Navigera till ditt lagringskonto.
2. Under **Inställningar**väljer du **geo-replikering**. Följande bild visar geo-replikering och redundans status för ett lagrings konto.

    ![Skärm bild som visar geo-replikering och redundans status](media/storage-initiate-account-failover/portal-failover-prepare.png)

3. Kontrol lera att ditt lagrings konto har kon figurer ATS för Geo-redundant lagring (GRS) eller Geo-redundant lagring med Läs behörighet (RA-GRS). Om den inte är det väljer du **konfiguration** under **Inställningar** för att uppdatera ditt konto till Geo-redundant. 
4. Egenskapen **tid för senaste synkronisering** anger hur långt den sekundära ligger bakom från den primära. **Tid för senaste synkronisering** ger en uppskattning av omfattningen av data förlust som du kommer att stöta på När redundansväxlingen har slutförts.
5. Välj **Förbered för redundans**.
6. Läs igenom bekräftelse dialog rutan. När du är klar anger du **Ja** för att bekräfta och initiera redundansväxlingen.

    ![Skärm bild som visar bekräftelse dialog ruta för redundans av konto](media/storage-initiate-account-failover/portal-failover-confirm.png)

## <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Funktionen för redundans av konto är allmänt tillgänglig, men är fortfarande beroende av en Preview-modul för PowerShell. Om du vill använda PowerShell för att initiera en redundansväxling av ett konto måste du först installera AZ. Storage [1.1.1-Preview-](https://www.powershellgallery.com/packages/Az.Storage/1.1.1-preview) modulen. Följ de här stegen för att installera modulen:

1. Avinstallera tidigare installationer av Azure PowerShell:

    - Ta bort alla tidigare installationer av Azure PowerShell från Windows med hjälp av inställningen **appar & funktioner** under **Inställningar**.
    - Ta bort alla **Azure** - `%Program Files%\WindowsPowerShell\Modules`moduler från.

1. Kontrol lera att du har den senaste versionen av PowerShellGet installerad. Öppna ett Windows PowerShell-fönster och kör följande kommando för att installera den senaste versionen:

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

1. Stäng PowerShell-fönstret och öppna det igen när du har installerat PowerShellGet.

1. Installera den senaste versionen av Azure PowerShell:

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Installera en Azure Storage Preview-modul som stöder redundans av konton:

    ```powershell
    Install-Module Az.Storage –Repository PSGallery -RequiredVersion 1.1.1-preview –AllowPrerelease –AllowClobber –Force
    ```

Kör följande kommando för att starta en redundansväxling av ett konto från PowerShell:

```powershell
Invoke-AzStorageAccountFailover -ResourceGroupName <resource-group-name> -Name <account-name>
```

## <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Kör följande kommandon om du vill använda Azure CLI för att initiera en konto växling vid fel:

```azurecli
az storage account show \ --name accountName \ --expand geoReplicationStats
az storage account failover \ --name accountName
```

---

## <a name="important-implications-of-account-failover"></a>Viktiga följder vid redundansväxling av konto

När du initierar ett konto för redundans för ditt lagrings konto uppdateras DNS-posterna för den sekundära slut punkten så att den sekundära slut punkten blir den primära slut punkten. Se till att du förstår de potentiella konsekvenserna av ditt lagrings konto innan du påbörjar en redundansväxling.

Om du vill beräkna omfattningen av sannolik data förlust innan du påbörjar en redundansväxling, kontrollerar du den **senaste synkroniseringstid** - `Get-AzStorageAccount` egenskapen med PowerShell-cmdleten `-IncludeGeoReplicationStats` och inkluderar parametern. Kontrol lera sedan `GeoReplicationStats` egenskapen för ditt konto.

Efter redundansväxlingen konverteras lagrings konto typen automatiskt till lokalt redundant lagring (LRS) i den nya primära regionen. Du kan återaktivera Geo-redundant lagring (GRS) eller Geo-redundant lagring med Läs behörighet (RA-GRS) för kontot. Observera att konvertering från LRS till GRS eller RA-GRS ådrar sig en ytterligare kostnad. Mer information finns i [pris information om bandbredd](https://azure.microsoft.com/pricing/details/bandwidth/).

När du har aktiverat GRS för ditt lagrings konto börjar Microsoft replikera data i ditt konto till den nya sekundära regionen. Replikerings tiden beror på mängden data som replikeras.  

## <a name="next-steps"></a>Nästa steg

- [Haveri beredskap och redundans för lagrings konto](storage-disaster-recovery-guidance.md)
- [Använd GEO-redundans för att skapa program med hög tillgänglighet](geo-redundant-design.md)
- [Självstudie: Bygg ett program med hög tillgänglighet med Blob Storage](../blobs/storage-create-geo-redundant-storage.md)
