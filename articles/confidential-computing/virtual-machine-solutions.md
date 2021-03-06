---
title: Azure-lösningar för konfidentiell behandling på virtuella datorer
description: Lär dig mer om Azures lösningar för konfidentiell behandling på virtuella datorer.
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: JenCook
ms.openlocfilehash: e574ac33e5f7da814c4bd813fc1c083c7cb4c2c9
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82187893"
---
# <a name="solutions-on-azure-virtual-machines"></a>Lösningar på virtuella Azure-datorer

Den här artikeln beskriver hur du distribuerar virtuella datorer med Azure konfidentiella data bearbetning (VM) som kör Intel-processorer som backas upp av [Intel Software Guard-tillägget](https://software.intel.com/sgx) (Intel SGX). 

## <a name="azure-confidential-computing-vm-sizes"></a>VM-storlekar för Azure konfidentiella datorer

Virtuella Azure-datorer med konfidentiell användning är utformade för att skydda konfidentiellt och integriteten hos dina data och kod när de bearbetas i molnet 

[DCsv2-serien](../virtual-machines/dcv2-series.md) Virtuella datorer är den senaste och senaste konfidentiella data bearbetnings familjen. De här virtuella datorerna har stöd för ett större antal distributions funktioner, har 2x enklaven Page cache (EPC) och ett större urval av storlekar jämfört med våra virtuella datorer i DC-serien. VM [-seriens](../virtual-machines/sizes-previous-gen.md#preview-dc-series) virtuella datorer är för närvarande i för hands version och kommer att vara inaktuella och ingår inte i allmän tillgänglighet.

Börja distribuera en virtuell dator med DCsv2-serien via Microsofts kommersiella marknads plats genom att följa [snabb starts guiden](quick-create-marketplace.md).

### <a name="current-available-sizes-and-regions"></a>Aktuella tillgängliga storlekar och regioner

Om du vill hämta en lista över alla allmänt tillgängliga storlekar för konfidentiell beräkning av virtuella datorer i tillgängliga regioner och tillgänglighets zoner kör du följande kommando i [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest):

```azurecli-interactive
az vm list-skus 
    --size dc 
    --query "[?family=='standardDCSv2Family'].{name:name,locations:locationInfo[0].location,AZ_a:locationInfo[0].zones[0],AZ_b:locationInfo[0].zones[1],AZ_c:locationInfo[0].zones[2]}" 
    --all 
    --output table
```

Från och med april 2020 är dessa SKU: er tillgängliga i följande regioner och tillgänglighets zoner:

```output
Name              Locations      AZ_a
----------------  -------------  ------
Standard_DC8_v2   eastus         2
Standard_DC1s_v2  eastus         2
Standard_DC2s_v2  eastus         2
Standard_DC4s_v2  eastus         2
Standard_DC8_v2   CanadaCentral
Standard_DC1s_v2  CanadaCentral
Standard_DC2s_v2  CanadaCentral
Standard_DC4s_v2  CanadaCentral
Standard_DC8_v2   uksouth        3
Standard_DC1s_v2  uksouth        3
Standard_DC2s_v2  uksouth        3
Standard_DC4s_v2  uksouth        3
Standard_DC8_v2   CentralUSEUAP
Standard_DC1s_v2  CentralUSEUAP
Standard_DC2s_v2  CentralUSEUAP
Standard_DC4s_v2  CentralUSEUAP
```

Om du vill ha en mer detaljerad vy över storlekarna ovan kör du följande kommando:

```azurecli-interactive
az vm list-skus 
    --size dc 
    --query "[?family=='standardDCSv2Family']"
```

## <a name="deployment-considerations"></a>Distributionsöverväganden

Följ en snabb starts guide för att distribuera en virtuell dator med DCsv2-serien på mindre än 10 minuter. 

- **Azure-prenumeration** – om du vill distribuera en konfidentiell VM-instans bör du fundera över en prenumeration där du betalar per användning eller något annat köp alternativ. Om du använder ett [kostnads fritt Azure-konto](https://azure.microsoft.com/free/)har du inte någon kvot för lämplig mängd Azure-beräknings kärnor.

- **Priser och regional tillgänglighet** – hitta priserna för virtuella datorer med DCsv2-serien på [sidan prissättning för virtuella datorer](https://azure.microsoft.com/pricing/details/virtual-machines/linux/). Kontrol lera [vilka produkter som är tillgängliga i region](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) för tillgänglighet i Azure-regioner.


- **Kärnor-kvot** – du kan behöva öka kvoten för kärnor i din Azure-prenumeration från standardvärdet. Din prenumeration kan också begränsa antalet kärnor som du kan distribuera i vissa storlekar på virtuella datorer, inklusive DCsv2-serien. Om du vill begära en kvot ökning [öppnar du en support förfrågan online](https://docs.microsoft.com/azure/azure-portal/supportability/per-vm-quota-requests) utan kostnad. Observera att standard gränserna kan variera beroende på din prenumerations kategori.

  > [!NOTE]
  > Kontakta Azure-supporten om du har storskaliga kapacitets behov. Azure-kvoter är kredit gränser, inte kapacitets garantier. Oavsett din kvot debiteras du bara för kärnor som du använder.
  
- **Ändra storlek** – på grund av deras specialiserade maskin vara kan du bara ändra storlek på konfidentiella dator instanser inom samma storleks familj. Du kan till exempel bara ändra storlek på en virtuell dator i DCsv2-serien från en DCsv2-serie till en annan. Det går inte att ändra storlek på en icke-konfidentiell dator storlek till en konfidentiell data behandling.  

- **Avbildning** – för att tillhandahålla Intel-stöd för Software Guard-tillägget (Intel SGX) på konfidentiella beräknings instanser måste alla distributioner köras på generation 2-avbildningar. Azures konfidentiella data behandling har stöd för arbets belastningar som körs på Ubuntu 18,04 gen 2, Ubuntu 16,04 gen 2 och Windows Server 2016 gen 2. Läs om [stöd för virtuella datorer i generation 2 på Azure](../virtual-machines/linux/generation-2.md) för att lära dig mer om scenarier som stöds och som inte stöds. 

- **Lagring** – data diskar för virtuella datorer i Azure konfidentiell dator och våra tillfälliga OS-diskar finns på NVMe-diskar. Instanser stöder bara Premium SSD och Standard SSD diskar, inte Ultra SSD eller Standard HDD. Den virtuella datorns storlek **DC8_v2** stöder inte Premium Storage. 

- **Disk kryptering** – konfidentiella beräknings instanser stöder inte disk kryptering för tillfället. 

## <a name="high-availability-and-disaster-recovery-considerations"></a>Överväganden för hög tillgänglighet och haveri beredskap

När du använder virtuella datorer i Azure är det du som ansvarar för att implementera en lösning för hög tillgänglighet och haveri beredskap för att undvika avbrott. 

Azures konfidentiella data behandling har inte stöd för zon-redundans via Tillgänglighetszoner för tillfället. Använd [tillgänglighets uppsättningar](../virtual-machines/windows/manage-availability.md#configure-multiple-virtual-machines-in-an-availability-set-for-redundancy)för högsta tillgänglighet och redundans för konfidentiell dator användning. På grund av maskin varu begränsningar kan tillgänglighets uppsättningar för konfidentiella data bearbetnings instanser bara ha högst 10 uppdaterings domäner. 

## <a name="deploying-via-an-azure-resource-manager-template"></a>Distribuera via en Azure Resource Manager-mall 

Azure Resource Manager är Azures tjänst för distribution och hantering. Det tillhandahåller ett hanterings lager som gör att du kan skapa, uppdatera och ta bort resurser i din Azure-prenumeration. Du kan använda hanterings funktioner som åtkomst kontroll, lås och taggar för att skydda och organisera dina resurser efter distributionen.

Mer information om Azure Resource Manager mallar finns [malldistribution översikt](../azure-resource-manager/templates/overview.md).

Om du vill distribuera en virtuell dator med DCsv2-serien i en ARM-mall använder du den [virtuella dator resursen](../virtual-machines/windows/template-description.md). Du måste se till att du anger rätt egenskaper för **vmSize** och för din **imageReference**.

### <a name="vm-size"></a>Storlek på virtuell dator

Ange en av följande storlekar i ARM-mallen i den virtuella dator resursen. Den här strängen anges som **vmSize** i **Egenskaper**.

```json
  [
        "Standard_DC1s_v2",
        "Standard_DC2s_v2",
        "Standard_DC4s_v2",
        "Standard_DC8_v2"
      ],
```

### <a name="gen2-os-image"></a>Gen2 OS-avbildning

Under **Egenskaper**måste du också referera till en bild under **storageProfile**. Använd *endast en* av följande avbildningar för din **imageReference**.

```json
      "2016-datacenter-gensecond": {
        "offer": "WindowsServer",
        "publisher": "MicrosoftWindowsServer",
        "sku": "2016-datacenter-gensecond",
        "version": "latest"
      },
        "18_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "18_04-lts-gen2",
        "version": "latest"
      },
      "16_04-lts-gen2": {
        "offer": "UbuntuServer",
        "publisher": "Canonical",
        "sku": "16_04-lts-gen2",
        "version": "latest"
      }
```

## <a name="next-steps"></a>Efterföljande moment 

I den här artikeln har du lärt dig om de kvalifikationer och konfigurationer som behövs när du skapar en virtuell dator med konfidentiell dator användning. Nu kan du gå till Azure Marketplace för att distribuera en virtuell dator med DCsv2-serien.

> [!div class="nextstepaction"]
> [Distribuera en virtuell dator med DCsv2-serien på Azure Marketplace](quick-create-marketplace.md)