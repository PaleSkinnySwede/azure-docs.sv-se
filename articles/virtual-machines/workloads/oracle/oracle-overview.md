---
title: Oracle-lösningar på Microsoft Azure | Microsoft Docs
description: Lär dig mer om alternativ för att distribuera Oracle-program och-lösningar på Microsoft Azure, inklusive att köra helt i Azure-infrastrukturen eller med hjälp av anslutningar mellan moln med OCI (Oracle Cloud Infrastructure).
services: virtual-machines-linux
documentationcenter: ''
author: BorisB2015
tags: azure-resource-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 03/16/2020
ms.author: borisb
ms.openlocfilehash: c737189650d571fb62a770707e84ed15c5a37a57
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81870504"
---
# <a name="overview-of-oracle-applications-and-solutions-on-azure"></a>Översikt över Oracle-program och Azure-lösningar på Azure

Den här artikeln beskriver funktioner för att köra Oracle-lösningar med Azure-infrastruktur. Se även detaljerade introduktioner till tillgängliga [Oracle VM-avbildningar](oracle-vm-solutions.md) på Azure Marketplace och möjligheten att sammanföra [Azure med Oracle Cloud Infrastructure (OCI)](oracle-oci-overview.md).

## <a name="oracle-databases-on-azure-infrastructure"></a>Oracle-databaser på Azure-infrastruktur

Kör Oracle-databaser på Azure-infrastruktur med hjälp av Oracle Database på Oracle Linux avbildningar som är tillgängliga på Azure Marketplace:

* Oracle Database 12,1, 12,2 och 18,3 Enterprise Edition 

* Oracle Database 12,1, 12,2 och 18,3 Standard Edition 

Du kan också välja att konfigurera Oracle Database på en icke-Oracle Linux avbildning som är tillgänglig i Azure, basera en lösning på en anpassad avbildning som du skapar från grunden i Azure eller ladda upp en anpassad avbildning från din lokala miljö.

Du kan också konfigurera med flera anslutna diskar och förbättra databasens prestanda genom att installera ASM (automatisk lagrings hantering i Oracle).

## <a name="applications-on-oracle-linux-and-weblogic-server"></a>Program på Oracle Linux-och WebLogic-Server

Kör företags program i Azure på Oracle-operativsystem som stöds. Följande avbildningar är tillgängliga på Azure Marketplace:

* 12.1.2 för Oracle WebLogic-Server

* Oracle Linux med den hårda företags kärnan (UEK) 6,8, 6,9, 6,10, 7,3, 7,4, 7,5 och 7,6 

## <a name="high-availability-and-disaster-recovery-options"></a>Alternativ för hög tillgänglighet och haveri beredskap

* Konfigurera [Oracle data Guard](https://docs.oracle.com/cd/B19306_01/server.102/b14239/concepts.htm#g1049956), [aktiva data skydd med FSFO](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/dgbkr/index.html), [horisontell partitionering](https://docs.oracle.com/en/database/oracle/oracle-database/12.2/admin/sharding-overview.html) eller [gyllene grind](https://www.oracle.com/middleware/technologies/goldengate.html) i Azure-infrastrukturen tillsammans med [Tillgänglighetszoner](../../../availability-zones/az-overview.md) för hög tillgänglighet i regionen. Du kan också konfigurera dessa konfigurationer i flera Azure-regioner för ytterligare tillgänglighet och haveri beredskap.

* Använd [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md) för att dirigera och hantera haveri beredskap för dina Oracle Linux virtuella datorer i Azure och dina lokala eller fysiska servrar. 

* Aktivera Oracle Real Application Clusters (RAC) i Azure med hjälp av [Azure VMware-lösning](https://docs.microsoft.com/azure/vmware-cloudsimple/oracle-real-application-clusters/) eller [FlashGrid SkyCluster](https://www.flashgrid.io/oracle-rac-in-azure/).

## <a name="backup-oracle-workloads"></a>Säkerhetskopiera Oracle-arbetsbelastningar

* Säkerhetskopiera dina virtuella Oracle-datorer med [Azure Backup](https://docs.microsoft.com/azure/backup/backup-overview)

* Säkerhetskopiera dina Oracle Database med Oracle-RMAN och alternativt använda [Azure Blob-säkring](https://docs.microsoft.com/azure/storage/blobs/storage-how-to-mount-container-linux) för att montera ett [Azure-Blob Storage-konto med hög redudant](https://docs.microsoft.com/azure/storage/common/storage-redundancy) och skriv dina rman-säkerhetskopieringar till det för extra återhämtning.

## <a name="integration-of-azure-with-oci"></a>Integrering av Azure med OCI

Köra Oracle-program i Azure-infrastrukturen, anslutna till backend-databaser i Oracle Cloud Infrastructure (OCI). Den här lösningen använder följande funktioner: 

* **Nätverk över flera moln** – Använd den direkta sammanlänkningen som är tillgänglig mellan Azure ExpressRoute och Oracle FastConnect för att upprätta anslutningar med hög bandbredd, privat och låg latens mellan programmet och databas skiktet.
* **Integrerad identitet** – konfigurera federerade identiteter mellan Azure AD och Oracle IDCS för att skapa en enda identitets källa för lösningarna. Aktivera enkel inloggning för att hantera resurser i OCI och Azure.

### <a name="deploy-oracle-applications-on-azure"></a>Distribuera Oracle-program på Azure

Använd terraform-mallar för att konfigurera Azure-infrastrukturen och installera Oracle-program. 

> [!IMPORTANT]
> Oracle certifierar dessa program för att köras i Azure när du använder Azure/Oracle Cloud Interconnect-lösningen från maj 2020.

* E-Business Suite
* JD Edwards-EnterpriseOne
* PeopleSoft
* Oracle återförsäljarversion-program
* Ekonomisk hantering för Oracle Hyperion

Distribuera även anpassade program i Azure som ansluter till OCI och andra Azure-tjänster.

### <a name="set-up-oracle-databases-in-oci"></a>Konfigurera Oracle-databaser i OCI

Använd Oracle Database Cloud Services (autonom databas, RAC, Exadata, DBaaS, enskild nod) tillsammans med Oracle-program som körs i Azure. Läs mer om [OCI-databas alternativ](https://docs.cloud.oracle.com/iaas/Content/Database/Concepts/databaseoverview.htm). 
 

## <a name="licensing"></a>Licensiering

Distribution av Oracle-program i Azure baseras på en "ta med din egen licens"-modell. Vi förutsätter att du är korrekt licensierad till att använda Oracle-programvara och att du har ett aktuellt support avtal på plats med Oracle. Oracle har garanterat licens mobilitet från lokal plats till Azure. Se [vanliga frågor och svar om](https://www.oracle.com/cloud/technologies/oracle-azure-faq.html)Oracle – Azure.

## <a name="next-steps"></a>Nästa steg

* Lär dig mer om att distribuera [VM-avbildningar](oracle-vm-solutions.md) i Azure-infrastrukturen.

* Lär dig mer om att ansluta [Azure med OCI](oracle-oci-overview.md).

* Kolla in [Oracle i Azure Overview-sessionen](https://myignite.techcommunity.microsoft.com/sessions/82915) från antändning 2019. 
