---
title: Översikt över Recovery Services-valv
description: En översikt och jämförelse mellan Recovery Services valv och Azure Backup valv.
ms.topic: conceptual
ms.date: 08/10/2018
ms.openlocfilehash: a0dacd82b7cf4258c0147bbaf9dc39ee6fc0fa25
ms.sourcegitcommit: acc558d79d665c8d6a5f9e1689211da623ded90a
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82597966"
---
# <a name="recovery-services-vaults-overview"></a>Översikt över Recovery Services-valv

I den här artikeln beskrivs funktionerna i ett Recovery Services-valv. Ett Recovery Services-valv är en lagrings enhet i Azure som huserar data. Data är vanligt vis kopior av data eller konfigurations information för virtuella datorer (VM), arbets belastningar, servrar eller arbets stationer. Du kan använda Recovery Services valv för att lagra säkerhets kopierings data för olika Azure-tjänster, till exempel virtuella IaaS-datorer (Linux eller Windows) och Azure SQL-databaser. Recovery Services valv stöder System Center DPM, Windows Server, Azure Backup Server med mera. Med Recovery Services-valv är det enkelt att organisera dina säkerhetskopierade data samtidigt som du minimerar hanteringskostnaden.

I en Azure-prenumeration kan du skapa upp till 500 Recovery Services valv per prenumeration per region.

## <a name="comparing-recovery-services-vaults-and-backup-vaults"></a>Jämföra Recovery Services valv och säkerhets kopierings valv

Om du fortfarande har säkerhets kopierings valv uppgraderas de automatiskt till Recovery Services valv. I november 2017 har alla säkerhets kopierings valv uppgraderats till Recovery Services valv.

Recovery Services-valv baseras på Azure Resource Manager modellen i Azure, medan säkerhets kopierings valv baserades på Azure Service Manager-modellen. När du uppgraderar ett säkerhets kopierings valv till ett Recovery Services-valv förblir säkerhets kopierings data intakta under och efter uppgraderings processen. Recovery Services-valv tillhandahåller funktioner som inte är tillgängliga för säkerhets kopierings valv, till exempel:

- **Förbättrade funktioner för att skydda säkerhets kopierings data**: med Recovery Services-valv tillhandahåller Azure Backup säkerhetsfunktioner för säkerhets kopiering av molnet. Säkerhetsfunktionerna garanterar att du kan skydda dina säkerhets kopior och återställa data på ett säkert sätt, även om produktions-och säkerhets kopierings servrar komprometteras. [Läs mer](backup-azure-security-feature.md)

- **Central övervakning för din hybrid IT-miljö**: med Recovery Services valv kan du bara övervaka dina [virtuella Azure IaaS-datorer](backup-azure-manage-vms.md) utan även dina [lokala till gångar](backup-azure-manage-windows-server.md#manage-backup-items) från en central Portal. [Läs mer](https://azure.microsoft.com/blog/alerting-and-monitoring-for-azure-backup)

- **Rollbaserad Access Control (RBAC)**: RBAC ger detaljerad åtkomst hanterings kontroll i Azure. [Azure tillhandahåller olika inbyggda roller](../role-based-access-control/built-in-roles.md)och Azure Backup har tre [inbyggda roller för att hantera återställnings punkter](backup-rbac-rs-vault.md). Recovery Services-valv är kompatibla med RBAC, vilket begränsar säkerhets kopiering och återställning av åtkomst till den definierade uppsättningen användar roller. [Läs mer](backup-rbac-rs-vault.md)

- **Skydda alla konfigurationer av Azure Virtual Machines**: Recovery Services-valv skyddar Resource Manager-baserade virtuella datorer, inklusive Premium diskar, Managed disks och krypterade virtuella datorer. Genom att uppgradera ett säkerhets kopierings valv till ett Recovery Services valv får du möjlighet att uppgradera Service Manager-baserade virtuella datorer till Resource Manager-baserade virtuella datorer. När du uppgraderar valvet kan du behålla dina Service Manager-baserade VM-återställnings punkter och konfigurera skydd för de uppgraderade (Resource Manager-aktiverade) virtuella datorerna. [Läs mer](https://azure.microsoft.com/blog/azure-backup-recovery-services-vault-ga)

- **Omedelbar återställning för virtuella IaaS-datorer**: med hjälp av Recovery Services valv kan du återställa filer och mappar från en IaaS-VM utan att återställa hela den virtuella datorn, vilket möjliggör snabbare återställnings tider. Omedelbar återställning för virtuella IaaS-datorer är tillgängligt för virtuella Windows-och Linux-datorer. [Läs mer](backup-instant-restore-capability.md)

## <a name="storage-settings-in-the-recovery-services-vault"></a>Lagrings inställningar i Recovery Services valvet

Ett Recovery Services-valv är en entitet som lagrar säkerhets kopior och återställnings punkter som skapats med tiden. Recovery Services valvet innehåller också de säkerhets kopierings principer som är associerade med de skyddade virtuella datorerna.

Azure Backup hanterar automatiskt lagring för valvet. Se hur [lagrings inställningarna kan ändras](https://docs.microsoft.com/azure/backup/backup-create-rs-vault#set-storage-redundancy).

Mer information om redundans finns i de här artiklarna om [geo](https://docs.microsoft.com/azure/storage/common/storage-redundancy-grs) -och [lokal](https://docs.microsoft.com/azure/storage/common/storage-redundancy-lrs) redundans.

## <a name="managing-your-recovery-services-vaults-in-the-portal"></a>Hantera dina Recovery Services-valv i portalen

Det är enkelt att skapa och hantera Recovery Services valv i Azure Portal eftersom säkerhets kopierings tjänsten integreras i andra Azure-tjänster. Den här integrationen innebär att du kan skapa eller hantera ett Recovery Services valv *i kontexten för mål tjänsten*. Om du till exempel vill visa återställnings punkter för en virtuell dator väljer du den virtuella datorn och klickar på **säkerhetskopiera** i menyn åtgärder.

![Recovery Services valv information VM](./media/backup-azure-recovery-services-vault-overview/rs-vault-in-context-vm.png)

Om den virtuella datorn inte har någon säkerhets kopia kommer du att uppmanas att konfigurera säkerhets kopieringen. Om säkerhets kopiering har kon figurer ATS visas säkerhets kopierings information om den virtuella datorn, inklusive en lista över återställnings punkter.  

![Virtuell dator för Recovery Services-valv](./media/backup-azure-recovery-services-vault-overview/vm-recovery-point-list.png)

I det tidigare exemplet är **ContosoVM** namnet på den virtuella datorn. **ContosoVM – demovault** är namnet på Recovery Services valvet. Du behöver inte komma ihåg namnet på Recovery Services valvet som lagrar återställnings punkterna. du kan komma åt den här informationen från den virtuella datorn.  

Om ett Recovery Services valv skyddar flera servrar kan det vara mer logiskt att titta på Recovery Services-valvet. Du kan söka efter alla Recovery Services valv i prenumerationen och välja ett från listan.

Följande avsnitt innehåller länkar till artiklar som förklarar hur du använder ett Recovery Services valv i varje typ av aktivitet.

> [!NOTE]
> Det går inte att skapa Recovery Services valv med samma namn om det har tagits bort inom 24 timmar. Använd ett annat resurs namn eller Välj en annan resurs grupp eller försök igen om 24 timmar.

### <a name="back-up-data"></a>Säkerhetskopiera data

- [Säkerhetskopiera en virtuell Azure-dator](backup-azure-vms-first-look-arm.md)
- [Säkerhetskopiera Windows Server eller Windows Workstation](backup-try-azure-backup-in-10-mins.md)
- [Säkerhetskopiera DPM-arbetsbelastningar till Azure](backup-azure-dpm-introduction.md)
- [Förbereda säkerhets kopiering av arbets belastningar med Azure Backup Server](backup-azure-microsoft-azure-backup.md)

### <a name="manage-recovery-points"></a>Hantera återställnings punkter

- [Hantera säkerhetskopior av virtuella Azure-datorer](backup-azure-manage-vms.md)
- [Hantera filer och mappar](backup-azure-manage-windows-server.md)

### <a name="restore-data-from-the-vault"></a>Återställa data från valvet

- [Återställa enskilda filer från en virtuell Azure-dator](backup-azure-restore-files-from-vm.md)
- [Återställa en virtuell Azure-dator](backup-azure-arm-restore-vms.md)

### <a name="secure-the-vault"></a>Skydda valvet

- [Skydda moln säkerhets kopierings data i Recovery Services valv](backup-azure-security-feature.md)

## <a name="azure-advisor"></a>Azure Advisor

[Azure Advisor](https://docs.microsoft.com/azure/advisor/) är en anpassad moln konsult som hjälper dig att optimera användningen av Azure. Den analyserar din Azure-användning och ger rekommendationer som hjälper dig att optimera och skydda dina distributioner. Det ger rekommendationer i fyra kategorier: hög tillgänglighet, säkerhet, prestanda och kostnad.

Azure Advisor ger [rekommendationer](https://docs.microsoft.com/azure/advisor/advisor-high-availability-recommendations#protect-your-virtual-machine-data-from-accidental-deletion) för virtuella datorer som inte har säkerhetskopierats, så att du aldrig har säkerhetskopierat viktiga virtuella datorer. Du kan också styra rekommendationerna genom att försätta dem i vilo läge.  Du kan klicka på rekommendationen och aktivera säkerhets kopiering på virtuella datorer på en rad genom att ange valvet (där säkerhets kopiorna ska lagras) och säkerhets kopierings policyn (schema för säkerhets kopior och kvarhållning av säkerhets kopior).

![Azure Advisor](./media/backup-azure-recovery-services-vault-overview/azure-advisor.png)

## <a name="next-steps"></a>Nästa steg

Använd följande artiklar för att:</br>
[Säkerhetskopiera en virtuell IaaS-dator](backup-azure-arm-vms-prepare.md)</br>
[Säkerhetskopiera en Azure Backup Server](backup-azure-microsoft-azure-backup.md)</br>
[Säkerhetskopiera en Windows Server](backup-windows-with-mars-agent.md)
