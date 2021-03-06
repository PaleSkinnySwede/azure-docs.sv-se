---
title: Montera ett virtuellt fil system på en pool – Azure Batch | Microsoft Docs
description: Lär dig hur du monterar ett virtuellt fil system i en batch-pool.
ms.topic: article
ms.date: 08/13/2019
ms.author: labrenne
ms.openlocfilehash: 703b65f0a1571659d7be479776dd8fdf02d86731
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82117037"
---
# <a name="mount-a-virtual-file-system-on-a-batch-pool"></a>Montera ett virtuellt fil system i en batch-pool

Azure Batch stöder nu montering av moln lagring eller ett externt fil system på Windows-eller Linux-datornoder i dina batch-pooler. När en Compute-nod ansluter till en pool monteras det virtuella fil systemet och behandlas som en lokal enhet på den noden. Du kan montera fil system som Azure Files, Azure Blob Storage, NFS (Network File System), inklusive ett [AVERT vFXT-cache](../avere-vfxt/avere-vfxt-overview.md)eller CIFS (common Internet File System).

I den här artikeln får du lära dig hur du monterar ett virtuellt fil system på en pool av datornoder med hjälp av [batch Management-biblioteket för .net](https://docs.microsoft.com/dotnet/api/overview/azure/batch?view=azure-dotnet).

> [!NOTE]
> Det finns stöd för att montera ett virtuellt fil system på batch-pooler som skapats på eller efter 2019-08-19. Batch-pooler som skapats före 2019-08-19 har inte stöd för den här funktionen.
> 
> API: erna för att montera fil system på en Compute-nod är en del av [batch .net](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch?view=azure-dotnet) -biblioteket.

## <a name="benefits-of-mounting-on-a-pool"></a>Fördelar med att montera i en pool

Genom att montera fil systemet till poolen, i stället för att låta aktiviteter Hämta egna data från en stor data uppsättning, gör det enklare och mer effektivt för aktiviteter att komma åt nödvändiga data.

Överväg ett scenario med flera aktiviteter som kräver åtkomst till en gemensam uppsättning data, t. ex. rendering av en film. Varje uppgift återger en eller flera bild rutor i taget från scenens filer. Genom att montera en enhet som innehåller scen filerna är det enklare att beräkna noder för att komma åt delade data. Dessutom kan det underliggande fil systemet väljas och skalas oberoende av varandra baserat på prestanda och skala (data flöde och IOPS) som krävs för antalet datornoder som används för att komma åt data samtidigt. Till exempel kan en [aver-vFXT](../avere-vfxt/avere-vfxt-overview.md) distribuerad in-memory cache användas för att stödja stora bildskärms skalor med tusentals samtidiga rendering-noder, komma åt käll data som finns lokalt. För data som redan finns i molnbaserad Blob Storage kan [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md) också användas för att montera dessa data som ett lokalt fil system. Blobfuse är bara tillgängligt på Linux-noder, men [Azure Files](https://azure.microsoft.com/blog/a-new-era-for-azure-files-bigger-faster-better/) innehåller ett liknande arbets flöde och är tillgängligt på både Windows och Linux.

## <a name="mount-a-virtual-file-system-on-a-pool"></a>Montera ett virtuellt fil system på en pool  

Genom att montera ett virtuellt fil system på en pool gör du fil systemet tillgängligt för varje Compute-nod i poolen. Fil systemet konfigureras antingen när en Compute-nod ansluter till en pool, eller när noden startas om eller avbildningas om.

Skapa ett `MountConfiguration` objekt för att montera ett fil system på en pool. Välj det objekt som passar ditt virtuella fil system: `AzureBlobFileSystemConfiguration`, `AzureFileShareConfiguration` `NfsMountConfiguration`, eller `CifsMountConfiguration`.

Alla monterings konfigurations objekt behöver följande bas parametrar. Vissa Mount-konfigurationer har parametrar som är speciella för det fil system som används, som beskrivs mer ingående i kod exemplen.

- **Konto namn eller källa**: om du vill montera en virtuell fil resurs behöver du namnet på lagrings kontot eller dess källa.
- **Relativ monterings Sök väg eller källa**: platsen för det fil system som är monterat på Compute-noden, `fsmounts` i förhållande till standard katalogen som `AZ_BATCH_NODE_MOUNTS_DIR`är tillgänglig på noden via. Den exakta platsen varierar beroende på vilket operativ system som används på noden. Till exempel mappas den fysiska platsen på en Ubuntu-nod till `mnt\batch\tasks\fsmounts`, och på en CentOS-nod mappas den `mnt\resources\batch\tasks\fsmounts`till.
- **Monterings alternativ eller alternativ för blobfuse**: de här alternativen beskriver vissa parametrar för att montera ett fil system.

När `MountConfiguration` objektet har skapats tilldelar du det till `MountConfigurationList` egenskapen när du skapar poolen. Fil systemet monteras antingen när en nod ansluter till en pool eller när noden startas om eller avbildningar.

När fil systemet är monterat skapas en miljö variabel `AZ_BATCH_NODE_MOUNTS_DIR` som pekar på platsen för de monterade fil systemen samt loggfiler, vilket är användbart för fel sökning och fel sökning. Loggfilerna beskrivs mer ingående i avsnittet [diagnosticera monterings fel](#diagnose-mount-errors) .  

> [!IMPORTANT]
> Det maximala antalet monterade fil system i en pool är 10. Mer information och andra begränsningar finns i [batch-tjänstens kvoter och begränsningar](batch-quota-limit.md#other-limits) .

## <a name="examples"></a>Exempel

Följande kod exempel visar hur du monterar en rad olika fil resurser i en pool med datornoder.

### <a name="azure-files-share"></a>Azure Files resurs

Azure Files är standard erbjudandet för Azure Cloud File System. Mer information om hur du hämtar någon av parametrarna i monterings kods exemplet finns i [använda en Azure Files-resurs](../storage/files/storage-how-to-use-files-windows.md).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureFileShareConfiguration = new AzureFileShareConfiguration
            {
                AccountName = "AccountName",
                AzureFileUrl = "AzureFileShareUrl",
                AccountKey = "StorageAccountKey",
                RelativeMountPath = "RelativeMountPath",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,sec=ntlmssp"
            },
        }
    }
}
```

### <a name="azure-blob-file-system"></a>Azure Blob File System

Ett annat alternativ är att använda Azure Blob Storage via [blobfuse](../storage/blobs/storage-how-to-mount-container-linux.md). Att montera ett BLOB-filsystem kräver `AccountKey` ett `SasKey` eller för ditt lagrings konto. Information om hur du hämtar dessa nycklar finns i [Hantera åtkomst nycklar för lagrings konton](../storage/common/storage-account-keys-manage.md)eller [använda signaturer för delad åtkomst (SAS)](../storage/common/storage-dotnet-shared-access-signature-part-1.md). Mer information om hur du använder blobfuse finns i blobfuse [Felsöka vanliga frågor och svar](https://github.com/Azure/azure-storage-fuse/wiki/3.-Troubleshoot-FAQ). Kör uppgiften som **administratör**för att få standard åtkomst till den blobfuse monterade katalogen. Blobfuse monterar katalogen i användar utrymmet och när poolen skapas monteras den som rot. I Linux är alla **Administratörs** aktiviteter rot. Alla alternativ för den säkra modulen beskrivs på [referens sidan för säkring](https://manpages.ubuntu.com/manpages/xenial/man8/mount.fuse.8.html).

Förutom fel söknings guiden är GitHub-problem i blobfuse-lagringsplatsen ett bra sätt att kontrol lera aktuella blobfuse-problem och lösningar. Mer information finns i [blobfuse-problem](https://github.com/Azure/azure-storage-fuse/issues).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            AzureBlobFileSystemConfiguration = new AzureBlobFileSystemConfiguration
            {
                AccountName = "StorageAccountName",
                ContainerName = "containerName",
                AccountKey = "StorageAccountKey",
                SasKey = "",
                RelativeMountPath = "RelativeMountPath",
                BlobfuseOptions = "-o attr_timeout=240 -o entry_timeout=240 -o negative_timeout=120 "
            },
        }
    }
}
```

### <a name="network-file-system"></a>Network File System

NFS (Network File System) kan också monteras till pool-noder så att traditionella fil system enkelt kan nås av Azure Batch noder. Detta kan vara en enda NFS-server som distribueras i molnet, eller en lokal NFS-server som nås via ett virtuellt nätverk. Du kan också dra nytta av den [vFXT](../avere-vfxt/avere-vfxt-overview.md) -distribuerade cache-lösningen i minnet, som ger sömlös anslutning till lokal lagring, läsning av data på begäran till cacheminnet och ger hög prestanda och skalbarhet till molnbaserade datornoder.

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            NfsMountConfiguration = new NFSMountConfiguration
            {
                Source = "source",
                RelativeMountPath = "RelativeMountPath",
                MountOptions = "options ver=1.0"
            },
        }
    }
}
```

### <a name="common-internet-file-system"></a>Common Internet File System

Vanliga Internet fil system (CIFS) kan också monteras till pool-noder så att traditionella fil system enkelt kan nås av Azure Batch noder. CIFS är ett fildelnings protokoll som ger en öppen och plattforms oberoende mekanism för att begära filer och tjänster för nätverks servrar. CIFS baseras på den förbättrade versionen av Microsofts SMB-protokoll (Server Message Block) för delning av Internet och intranät och används för att montera externa fil system på Windows-noder. Mer information om SMB finns i [fil server och SMB](https://docs.microsoft.com/windows-server/storage/file-server/file-server-smb-overview).

```csharp
new PoolAddParameter
{
    Id = poolId,
    MountConfiguration = new[]
    {
        new MountConfiguration
        {
            CifsMountConfiguration = new CIFSMountConfiguration
            {
                Username = "StorageAccountName",
                RelativeMountPath = "cifsmountpoint",
                Source = "source",
                Password = "StorageAccountKey",
                MountOptions = "-o vers=3.0,dir_mode=0777,file_mode=0777,serverino"
            },
        }
    }
}
```

## <a name="diagnose-mount-errors"></a>Diagnostisera monterings fel

Om en monterings konfiguration Miss lyckas kommer Compute-noden i poolen att Miss lyckas och nodens tillstånd blir oanvändbar. Om du vill diagnostisera ett monterings konfigurations fel [`ComputeNodeError`](https://docs.microsoft.com/rest/api/batchservice/computenode/get#computenodeerror) kan du granska egenskapen för information om felet.

Om du vill hämta loggfilerna för fel sökning använder du [OutputFiles](batch-task-output-files.md) för att ladda `*.log` upp filerna. `*.log` Filerna innehåller information om fil systemets montering på `AZ_BATCH_NODE_MOUNTS_DIR` platsen. Monterings logg filen har formatet: `<type>-<mountDirOrDrive>.log` för varje montering. Till exempel har en `cifs` Mount i en monterings katalog `test` med namnet en monterings logg fil med `cifs-test.log`namnet:.

## <a name="supported-skus"></a>SKU: er som stöds

| Utgivare | Erbjudande | SKU | Azure Files resurs | Blobfuse | NFS-montering | CIFS-montering |
|---|---|---|---|---|---|---|
| batch | rendering – centos73 | Render | :heavy_check_mark: <br>Obs: kompatibel med CentOS 7,7</br>| :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Canonical | UbuntuServer | 16,04-LTS, 18,04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 8| :heavy_check_mark: | röntgen | :heavy_check_mark: | :heavy_check_mark: |
| Credativ | Debian | 9 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Microsoft-Ads | Linux-data-science-VM | linuxdsvm | :heavy_check_mark: <br>Obs: kompatibel med CentOS 7,4. </br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Microsoft – Azure-Batch | CentOS-container | 7,6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Microsoft – Azure-Batch | CentOS-container-RDMA | 7.4 | :heavy_check_mark: <br>Obs: stöder A_8-eller 9-lagring</br> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Microsoft – Azure-Batch | Ubuntu-Server-container | 16.04-LTS | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Microsoft-dsvm | Linux-data-science-VM-Ubuntu | linuxdsvmubuntu | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS | 7,6 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| OpenLogic | CentOS – HPC | 7,4, 7,3, 7,1 | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Oracle | Oracle – Linux | 7,6 | röntgen | röntgen | röntgen | röntgen |
| Windows | WindowsServer | 2012, 2016, 2019 | :heavy_check_mark: | röntgen | röntgen | röntgen |

## <a name="next-steps"></a>Nästa steg

- Läs mer om hur du monterar en Azure Files-resurs med [Windows](../storage/files/storage-how-to-use-files-windows.md) eller [Linux](../storage/files/storage-how-to-use-files-linux.md).
- Lär dig mer om att använda och montera [blobfuse](https://github.com/Azure/azure-storage-fuse) -virtuella fil system.
- Se [Översikt över Network File System](https://docs.microsoft.com/windows-server/storage/nfs/nfs-overview) om du vill lära dig mer om NFS och dess program.
- Se [Översikt över Microsoft SMB-protokoll och CIFS-protokoll](https://docs.microsoft.com/windows/desktop/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) för att lära dig mer om CIFS.
