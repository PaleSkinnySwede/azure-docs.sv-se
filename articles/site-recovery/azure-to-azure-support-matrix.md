---
title: Support mat ris för haveri beredskap i Azure VM med Azure Site Recovery
description: Sammanfattar stöd för haveri beredskap för virtuella Azure-datorer till en sekundär region med Azure Site Recovery.
ms.topic: article
ms.date: 01/10/2020
ms.author: raynew
ms.openlocfilehash: 2fc2a32c47991b9b3615417dfb8f50ca3e7c988f
ms.sourcegitcommit: 999ccaf74347605e32505cbcfd6121163560a4ae
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2020
ms.locfileid: "82983507"
---
# <a name="support-matrix-for-azure-vm-disaster-recovery-between-azure-regions"></a>Support mat ris för haveri beredskap för virtuella Azure-datorer mellan Azure-regioner

Den här artikeln sammanfattar support och krav för haveri beredskap för virtuella Azure-datorer från en Azure-region till en annan med hjälp av tjänsten [Azure Site Recovery](site-recovery-overview.md) .


## <a name="deployment-method-support"></a>Stöd för distributions metoder

**Distribution** |  **Support**
--- | ---
**Azure Portal** | Stöds.
**PowerShell** | Stöds. [Läs mer](azure-to-azure-powershell.md)
**REST-API** | Stöds.
**CLI** | Stöds för närvarande inte


## <a name="resource-support"></a>Resursstöd

**Resurs åtgärd** | **Information**
--- | ---
**Flytta valv över resurs grupper** | Stöds inte
**Flytta beräknings-/lagrings-/nätverks resurser över resurs grupper** | Stöds inte.<br/><br/> Om du flyttar en virtuell dator eller tillhör ande komponenter, till exempel lagring/nätverk när den virtuella datorn har repliker ATS, måste du inaktivera och sedan återaktivera replikering för den virtuella datorn.
**Replikera virtuella Azure-datorer från en prenumeration till en annan för haveri beredskap** | Stöds inom samma Azure Active Directory-klient.
**Migrera virtuella datorer mellan regioner inom de geografiska kluster som stöds (inom och över prenumerationer)** | Stöds inom samma Azure Active Directory-klient.
**Migrera virtuella datorer inom samma region** | Stöds inte.

## <a name="region-support"></a>Stöd för regioner

Du kan replikera och återställa virtuella datorer mellan två regioner i samma geografiska kluster. Geografiska kluster definieras för att hålla data latens och suveränitet i åtanke.


**Geografiskt kluster** | **Azure-regioner**
-- | --
Vägnar | Östra Kanada, centrala Kanada, södra centrala USA, västra centrala USA, östra USA, östra USA 2, västra USA, västra USA 2, centrala USA, norra centrala USA
Europa | Storbritannien, västra, Storbritannien, södra, Europa, Västeuropa, västra Europa, västra Sydafrika, norra Afrika, östra Norge, västra Norge
Asien | Södra Indien, centrala Indien, västra Indien, Sydostasien, Asien, östra, Östra Japan, västra Japan, centrala Korea, centrala Korea, södra Korea
Australien    | Australien, östra, Australien, sydöstra, Australien, centrala, Australien, centrala 2
Azure Government    | US GOV Virginia, US GOV Iowa, US GOV Arizona, US GOV Texas, USA DOD öst, USA DOD Central
Tyskland    | Tyskland, centrala, Tyskland nordöstra
Kina | Kina, östra, Kina, norra, Kina North2, Kina östra
Begränsade regioner som är reserverade för haveri beredskap i landet |Tyskland, norra reserverad för Tyskland, västra centrala, Schweiz, västra reserverad för Schweiz, norra, Frankrike, södra reserverade för centrala Frankrike, Förenade Arabemiraten Central restricted för Förenade Arabemiraten Nord-kunder

>[!NOTE]
>
> - För **södra Brasilien**kan du replikera och redundansväxla till dessa regioner: södra centrala USA, västra centrala USA, östra USA, östra USA 2, västra USA, västra USA 2 och norra centrala USA.
> - Södra Brasilien kan endast användas som käll region från vilka virtuella datorer kan replikeras med hjälp av Site Recovery. Den kan inte fungera som mål region. Detta beror på fördröjnings problem på grund av geografiska avstånd. Observera att om du växlar över från Brasilien, södra som käll region till ett mål, stöds failback till södra Brasilien från mål regionen.
> - Du kan arbeta i regioner som du har lämplig åtkomst till.
> - Om den region där du vill skapa ett valv inte visas kontrollerar du att prenumerationen har åtkomst till att skapa resurser i den regionen.
> - Om du inte kan se en region i ett geografiskt kluster när du aktiverar replikering, se till att din prenumeration har behörighet att skapa virtuella datorer i den regionen.



## <a name="cache-storage"></a>Cachelagring

I den här tabellen sammanfattas stödet för cache Storage-kontot som används av Site Recovery under replikeringen.

**Inställning** | **Support** | **Information**
--- | --- | ---
Generell användning v2-lagrings konton (frekvent och låg frekvent nivå) | Stöds | Användning av GPv2 rekommenderas inte eftersom transaktionskostnader för v2 är betydligt högre än v1-lagrings konton.
Premium Storage | Stöds inte | Standard lagrings konton används för cachelagring för att hjälpa till att optimera kostnaderna.
Azure Storage brand väggar för virtuella nätverk  | Stöds | Om du använder en brand vägg som är aktive rad för cache-lagring eller mål lagrings konto, se till att du [tillåter betrodda Microsoft-tjänster](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions).<br></br>Se också till att du tillåter åtkomst till minst ett undernät för det virtuella käll nätverket.


## <a name="replicated-machine-operating-systems"></a>Replikerade dator operativ system

Site Recovery stöder replikering av virtuella Azure-datorer som kör operativ systemen som anges i det här avsnittet. Observera att om en redan replikerande dator sedan uppgraderas (eller nedgraderas) till en annan större kernel måste du inaktivera replikering och återaktivera replikering efter uppgraderingen.

### <a name="windows"></a>Windows


**Operativsystem** | **Information**
--- | ---
Windows Server 2019 | Stöds för Server Core, server med Skriv bords miljö.
Windows Server 2016  | Server Core som stöds, server med Skriv bords miljö.
Windows Server 2012 R2 | Stöds.
Windows Server 2012 | Stöds.
Windows Server 2008 R2 med SP1/SP2 | Stöds.<br/><br/> Från version [9,30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) av mobilitets tjänst tillägget för virtuella Azure-datorer måste du installera en Windows [servicing stack Update-uppdatering (självbetjänings)](https://support.microsoft.com/help/4490628) och [SHA-2](https://support.microsoft.com/help/4474419) på datorer som kör Windows Server 2008 R2 SP1/SP2.  SHA-1 stöds inte från september 2019 och om SHA-2-kod signering inte är aktiverat installeras/uppgraderas inte som förväntat i agent tillägget. Läs mer om [SHA-2-uppgradering och krav](https://aka.ms/SHA-2KB).
Windows 10 (x64) | Stöds.
Windows 8,1 (x64) | Stöds.
Windows 8 (x64) | Stöds.
Windows 7 (x64) med SP1 och senare | Från version [9,30](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) av mobilitets tjänst tillägget för virtuella Azure-datorer måste du installera en Windows [servicing stack Update-uppdatering (självbetjänings)](https://support.microsoft.com/help/4490628) och [SHA-2](https://support.microsoft.com/help/4474419) på datorer som kör Windows 7 med SP1.  SHA-1 stöds inte från september 2019 och om SHA-2-kod signering inte är aktiverat installeras/uppgraderas inte som förväntat i agent tillägget. Läs mer om [SHA-2-uppgradering och krav](https://aka.ms/SHA-2KB).



#### <a name="linux"></a>Linux

**Operativsystem** | **Information**
--- | ---
Red Hat Enterprise Linux | 6,7, 6,8, 6,9, 6,10, 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6,[7,7](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery), [8,0](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery), 8,1
CentOS | 6,5, 6,6, 6,7, 6,8, 6,9, 6,10, 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6, 7,7, 8,0, 8,1
Ubuntu 14,04 LTS-Server | [Kernel-versioner som stöds](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Ubuntu 16,04 LTS-Server | [Kernel-version som stöds](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)<br/><br/> Ubuntu-servrar som använder lösenordsbaserad autentisering och loggar in och Cloud-Init-paketet för att konfigurera virtuella datorer i molnet kan ha lösenordsbaserad inloggning inaktive rad vid redundansväxling (beroende på cloudinit-konfigurationen). Lösenordsbaserade inloggningar kan återaktiveras på den virtuella datorn genom att återställa lösen ordet från support > fel sökning > Inställningar-menyn (av den misslyckade virtuella datorn i Azure Portal.
Ubuntu 18,04 LTS-Server | [Kernel-version som stöds](#supported-ubuntu-kernel-versions-for-azure-virtual-machines)
Debian 7 | [Kernel-versioner som stöds](#supported-debian-kernel-versions-for-azure-virtual-machines)
Debian 8 | [Kernel-versioner som stöds](#supported-debian-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 12 | SP1, SP2, SP3, SP4. [(Kernel-versioner som stöds)](#supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 15 | 15 och 15 SP1. [(Kernel-versioner som stöds)](#supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines)
SUSE Linux Enterprise Server 11 | INSTALLERAS<br/><br/> Uppgradering av replikering av datorer från SP3 till SP4 stöds inte. Om en replikerad dator har uppgraderats måste du inaktivera replikering och återaktivera replikering efter uppgraderingen.
SUSE Linux Enterprise Server 11 | SP4
Oracle Linux | 6,4, 6,5, 6,6, 6,7, 6,8, 6,9, 6,10, 7,0, 7,1, 7,2, 7,3, 7,4, 7,5, 7,6, [7,7](https://support.microsoft.com/en-us/help/4531426/update-rollup-42-for-azure-site-recovery) <br/><br/> Köra Red Hat-kompatibel kernel eller Enterprise kernel release 3, 4 & 5 (UEK3, UEK4, UEK5)


#### <a name="supported-ubuntu-kernel-versions-for-azure-virtual-machines"></a>Ubuntu kernel-versioner som stöds för Azure Virtual Machines

**Frisläpp** | **Mobilitets tjänst version** | **Kernelversion** |
--- | --- | --- |
14,04 LTS | 9,32| 3.13.0-24-genered to 3.13.0-170-Generic,<br/>3.16.0-25-genered to 3.16.0-77-Generic,<br/>3.19.0-18-genered till 3.19.0-80-genered,<br/>4.2.0-18-genered to 4.2.0-42-Generic,<br/>4.4.0-21-genered to 4.4.0-148-genered,<br/>4.15.0 – 1023 – Azure till 4.15.0 – 1045 – Azure |
14,04 LTS | 9,31 | 3.13.0-24-genered to 3.13.0-170-Generic,<br/>3.16.0-25-genered to 3.16.0-77-Generic,<br/>3.19.0-18-genered till 3.19.0-80-genered,<br/>4.2.0-18-genered to 4.2.0-42-Generic,<br/>4.4.0-21-genered to 4.4.0-148-genered,<br/>4.15.0 – 1023 – Azure till 4.15.0 – 1045 – Azure |
14,04 LTS | 9,30 | 3.13.0-24-genered to 3.13.0-170-Generic,<br/>3.16.0-25-genered to 3.16.0-77-Generic,<br/>3.19.0-18-genered till 3.19.0-80-genered,<br/>4.2.0-18-genered to 4.2.0-42-Generic,<br/>4.4.0-21-genered to 4.4.0-148-genered,<br/>4.15.0 – 1023 – Azure till 4.15.0 – 1045 – Azure |
14,04 LTS | 9,29 | 3.13.0-24-genered to 3.13.0-170-Generic,<br/>3.16.0-25-genered to 3.16.0-77-Generic,<br/>3.19.0-18-genered till 3.19.0-80-genered,<br/>4.2.0-18-genered to 4.2.0-42-Generic,<br/>4.4.0-21-genered to 4.4.0-148-genered,<br/>4.15.0 – 1023 – Azure till 4.15.0 – 1045 – Azure |
|||
16,04 LTS | 9,32 | 4.4.0-21-genered to 4.4.0-171-Generic,<br/>4.8.0-34-Generic to 4.8.0-58-Generic,<br/>4.10.0-14-generic to 4.10.0-42-Generic,<br/>4.11.0-13-genered to 4.11.0-14-generic,<br/>4.13.0-16-genered to 4.13.0-45-genered,<br/>4.15.0-13-genered to 4.15.0-74-generic<br/>4.11.0-1009 – Azure till 4.11.0-1016-Azure,<br/>4.13.0-1005 – Azure till 4.13.0 – 1018 – Azure <br/>4.15.0-1012 – Azure till 4.15.0-1066-Azure|
16,04 LTS | 9,31 | 4.4.0 – 21-genered to 4.4.0-170-Generic,<br/>4.8.0-34-Generic to 4.8.0-58-Generic,<br/>4.10.0-14-generic to 4.10.0-42-Generic,<br/>4.11.0-13-genered to 4.11.0-14-generic,<br/>4.13.0-16-genered to 4.13.0-45-genered,<br/>4.15.0-13-Generic to 4.15.0-72-generic<br/>4.11.0-1009 – Azure till 4.11.0-1016-Azure,<br/>4.13.0-1005 – Azure till 4.13.0 – 1018 – Azure <br/>4.15.0-1012 – Azure till 4.15.0-1063 – Azure|
16,04 LTS | [9,30](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) | 4.4.0-21-Generic to 4.4.0-166-Generic,<br/>4.8.0-34-Generic to 4.8.0-58-Generic,<br/>4.10.0-14-generic to 4.10.0-42-Generic,<br/>4.11.0-13-genered to 4.11.0-14-generic,<br/>4.13.0-16-genered to 4.13.0-45-genered,<br/>4.15.0-13-genered to 4.15.0-66-generic<br/>4.11.0-1009 – Azure till 4.11.0-1016-Azure,<br/>4.13.0-1005 – Azure till 4.13.0 – 1018 – Azure <br/>4.15.0-1012 – Azure till 4.15.0-1061 – Azure|
16,04 LTS | 9,29 | 4.4.0 – 21-genered to 4.4.0-164-Generic,<br/>4.8.0-34-Generic to 4.8.0-58-Generic,<br/>4.10.0-14-generic to 4.10.0-42-Generic,<br/>4.11.0-13-genered to 4.11.0-14-generic,<br/>4.13.0-16-genered to 4.13.0-45-genered,<br/>4.15.0-13-genered to 4.15.0-64-genered<br/>4.11.0-1009 – Azure till 4.11.0-1016-Azure,<br/>4.13.0-1005 – Azure till 4.13.0 – 1018 – Azure <br/>4.15.0-1012 – Azure till 4.15.0-1059 – Azure|
|||
18,04 LTS | 9,32| 4.15.0-20-Generic to 4.15.0-74-generic </br> 4.18.0-13-Generic to 4.18.0-25-genered </br> 5.0.0-15-genered to 5.0.0-37-generisk </br> 5.3.0-19-Generic to 5.3.0-24-genered </br> 4.15.0-1009 – Azure till 4.15.0-1037 – Azure </br> 4.18.0-1006 – Azure till 4.18.0 – 1025 – Azure </br> 5.0.0-1012 – Azure till 5.0.0-1028 – Azure </br> 5.3.0-1007-Azure till 5.3.0-1009 – Azure|
18,04 LTS | 9,31| 4.15.0-20-Generic to 4.15.0-72-generic </br> 4.18.0-13-Generic to 4.18.0-25-genered </br> 5.0.0-15-genered to 5.0.0-37-generisk </br> 5.3.0-19-Generic to 5.3.0-24-genered </br> 4.15.0-1009 – Azure till 4.15.0-1037 – Azure </br> 4.18.0-1006 – Azure till 4.18.0 – 1025 – Azure </br> 5.0.0-1012 – Azure till 5.0.0-1025 – Azure </br> 5.3.0-1007 – Azure|
18,04 LTS | [9,30](https://support.microsoft.com/help/4531426/update-rollup-42-for-azure-site-recovery) | 4.15.0-20-Generic to 4.15.0-66-generic </br> 4.18.0-13-Generic to 4.18.0-25-genered </br> 5.0.0-15-Generic to 5.0.0-32-genered </br> 4.15.0-1009 – Azure till 4.15.0-1037 – Azure </br> 4.18.0-1006 – Azure till 4.18.0 – 1025 – Azure </br> 5.0.0-1012 – Azure till 5.0.0-1023 – Azure|
18,04 LTS | [9,29](https://support.microsoft.com/help/4528026/update-rollup-41-for-azure-site-recovery) | 4.15.0-20-Generic to 4.15.0-64-genered </br> 4.18.0-13-Generic to 4.18.0-25-genered </br> 5.0.0-15-genered to 5.0.0-29-generic </br> 4.15.0-1009 – Azure till 4.15.0-1037 – Azure </br> 4.18.0-1006 – Azure till 4.18.0 – 1025 – Azure </br> 5.0.0-1012 – Azure till 5.0.0-1020 – Azure|


#### <a name="supported-debian-kernel-versions-for-azure-virtual-machines"></a>Debian Kernel-versioner som stöds för Azure Virtual Machines

**Frisläpp** | **Mobilitets tjänst version** | **Kernelversion** |
--- | --- | --- |
Debian 7 | 9.28,9.29,9.30,9.31 | 3.2.0 – 4-amd64 till 3.2.0-6-amd64, 3.16.0 -0. bpo. 4-amd64 |
|||
Debian 8 | 9.29,9.30,9.31 | 3.16.0 – 4-amd64 till 3.16.0-10-amd64, 4.9.0 -0. bpo. 4-amd64 till 4.9.0 -0. bpo. 11-amd64 |
Debian 8 | 9,28 | 3.16.0 – 4-amd64 till 3.16.0-10-amd64, 4.9.0 -0. bpo. 4-amd64 till 4.9.0 -0. bpo. 9-amd64 |

#### <a name="supported-suse-linux-enterprise-server-12-kernel-versions-for-azure-virtual-machines"></a>Stöd för SUSE Linux Enterprise Server 12 kernel-versioner för Azure Virtual Machines

**Frisläpp** | **Mobilitets tjänst version** | **Kernelversion** |
--- | --- | --- |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9,32 | Alla aktie-och SUSE 12 SP1-, SP2-, SP3-och SP4-kärnor stöds.</br></br> 4.4.138-4,7-Azure till 4.4.180-4.31 – Azure</br>4.12.14-6.3 – Azure till 4.12.14-6.34 – Azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9,31 | Alla aktie-och SUSE 12 SP1-, SP2-, SP3-och SP4-kärnor stöds.</br></br> 4.4.138-4,7-Azure till 4.4.180-4.31 – Azure</br>4.12.14-6.3 – Azure till 4.12.14-6.29 – Azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9,30 | Alla aktie-och SUSE 12 SP1-, SP2-, SP3-och SP4-kärnor stöds.</br></br> 4.4.138-4,7-Azure till 4.4.180-4.31 – Azure</br>4.12.14-6.3 – Azure till 4.12.14-6.29 – Azure  |
SUSE Linux Enterprise Server 12 (SP1, SP2, SP3, SP4) | 9,29 | Alla aktie-och SUSE 12 SP1-, SP2-, SP3-och SP4-kärnor stöds.</br></br> 4.4.138-4,7-Azure till 4.4.180-4.31 – Azure</br>4.12.14-6.3 – Azure till 4.12.14-6.23 – Azure  |

#### <a name="supported-suse-linux-enterprise-server-15-kernel-versions-for-azure-virtual-machines"></a>SUSE Linux Enterprise Server 15 kernel-versioner som stöds för Azure Virtual Machines

**Frisläpp** | **Mobilitets tjänst version** | **Kernelversion** |
--- | --- | --- |
SUSE Linux Enterprise Server 15 och 15 SP1 | 9,32 | Som standard stöds alla [börs-och 15-kernels](https://www.suse.com/support/kb/doc/?id=000019587) .</br></br> 4.12.14 – 5.5 – Azure till 4.12.14 – 8.22 – Azure |


## <a name="replicated-machines---linux-file-systemguest-storage"></a>Replikerade datorer – Linux-filsystem/gäst lagring

* Fil system: ext3, ext4, XFS, BTRFS
* Volym hanterare: LVM2
* Program vara för flera sökvägar: enhets mappning


## <a name="replicated-machines---compute-settings"></a>Replikerade datorer – beräknings inställningar

**Inställning** | **Support** | **Information**
--- | --- | ---
Storlek | Valfri storlek på virtuella Azure-datorer med minst 2 processor kärnor och 1 GB RAM | Verifiera [storleken på virtuella Azure-datorer](../virtual-machines/windows/sizes.md).
Tillgänglighetsuppsättningar | Stöds | Om du aktiverar replikering för en virtuell Azure-dator med standard alternativen skapas en tillgänglighets uppsättning automatiskt, baserat på käll regions inställningarna. Du kan ändra de här inställningarna.
Tillgänglighetszoner | Stöds |
Hybrid användnings förmånen (hubb) | Stöds | Om den virtuella käll datorn har en nav-licens aktive rad, används även HUB-licensen i en redundanstest eller redundansväxling av den virtuella datorn.
Skalningsuppsättningar för virtuella datorer | Stöds inte |
Azure Gallery-bilder – Microsoft publicerat | Stöds | Stöds om den virtuella datorn körs på ett operativ system som stöds.
Azure Gallery-avbildningar – tredje part publicerad | Stöds | Stöds om den virtuella datorn körs på ett operativ system som stöds.
Anpassade avbildningar – tredje part publicerad | Stöds | Stöds om den virtuella datorn körs på ett operativ system som stöds.
Virtuella datorer som migrerats med Site Recovery | Stöds | Om en virtuell VMware-dator eller en fysisk dator har migrerats till Azure med hjälp av Site Recovery måste du avinstallera den äldre versionen av mobilitets tjänsten som körs på datorn och starta om datorn innan du replikerar den till en annan Azure-region.
RBAC-principer | Stöds inte | Principer för rollbaserad åtkomst kontroll (RBAC) för virtuella datorer replikeras inte till den virtuella redundansväxlingen i mål regionen.
Tillägg | Stöds inte | Tillägg replikeras inte till den virtuella redundansväxlingen i mål regionen. Den måste installeras manuellt efter redundansväxlingen.
Placerings grupper för närhet | Stöds inte | Virtuella datorer som finns inuti en närhets placerings grupp kan inte skyddas med hjälp av Site Recovery.


## <a name="replicated-machines---disk-actions"></a>Replikerade datorer – disk åtgärder

**Åtgärd** | **Information**
-- | ---
Ändra storlek på disk på replikerad virtuell dator | Stöds på den virtuella käll datorn före redundans. Du behöver inte inaktivera/återaktivera replikering.<br/><br/> Om du ändrar den virtuella käll datorn efter redundansväxlingen, fångas inte ändringarna.<br/><br/> Om du ändrar disk storleken på den virtuella Azure-datorn efter redundansväxlingen, registreras inte ändringarna av Site Recovery och återställningen görs till den ursprungliga virtuella dator storleken.
Lägga till en disk till en replikerad virtuell dator | Stöds

## <a name="replicated-machines---storage"></a>Replikerade datorer – lagring

Den här tabellen sammanfattade stödet för den virtuella Azure OS-disken, data disken och den temporära disken.

- Det är viktigt att Observera de virtuella datorernas disk gränser och mål för virtuella [Linux](../virtual-machines/linux/disk-scalability-targets.md) -och [Windows](../virtual-machines/windows/disk-scalability-targets.md) -datorer för att undvika prestanda problem.
- Om du distribuerar med standardinställningarna skapar Site Recovery automatiskt diskar och lagrings konton baserat på käll inställningarna.
- Om du anpassar, se till att du följer rikt linjerna.

**Komponent** | **Support** | **Information**
--- | --- | ---
Maximal storlek för OS-disk | 2048 GB | [Läs mer](../virtual-machines/windows/managed-disks-overview.md) om VM-diskar.
Tillfällig disk | Stöds inte | Den tillfälliga disken är alltid exkluderad från replikering.<br/><br/> Lagra inte beständiga data på den temporära disken. [Läs mer](../virtual-machines/windows/managed-disks-overview.md).
Maximal storlek för data disk | 8192 GB för Managed disks<br></br>4095 GB för ohanterade diskar|
Minsta storlek för data disk | Ingen begränsning för ohanterade diskar. 2 GB för hanterade diskar |
Högsta antal data diskar | Upp till 64, i enlighet med stöd för en speciell storlek på virtuell Azure-dator | [Läs mer](../virtual-machines/windows/sizes.md) om storlekar på virtuella datorer.
Ändrings takt för data disk | Högst 10 MBps per disk för Premium Storage. Högst 2 Mbit/s per disk för standard lagring. | Om genomsnitts data ändrings takten på disken kontinuerligt är högre än det högsta antalet kommer replikering inte att fångas upp.<br/><br/>  Men om det maximala värdet överskrids sporadisk, kan replikeringen fångas upp, men du kan se något fördröjda återställnings punkter.
Data disk-standard lagrings konto | Stöds |
Data disk – Premium Storage-konto | Stöds | Om en virtuell dator har diskar som sprids över Premium-och standard lagrings konton kan du välja ett annat mål lagrings konto för varje disk, så att du har samma lagrings konfiguration i mål regionen.
Hanterad disk – standard | Stöds i Azure-regioner där Azure Site Recovery stöds. |
Hanterad disk – Premium | Stöds i Azure-regioner där Azure Site Recovery stöds. |
Standard SSD | Stöds |
Redundans | LRS och GRS stöds.<br/><br/> ZRS stöds inte.
Kylning och frekvent lagring | Stöds inte | VM-diskar stöds inte på låg frekvent lagring
Lagringsutrymmen | Stöds |
Kryptering i rest (SSE) | Stöds | SSE är standardinställningen på lagrings konton.
Kryptering i vilo läge (CMK) | Stöds | Både program-och HSM-nycklar stöds för hanterade diskar
Azure Disk Encryption (ADE) för Windows OS | Stöds för virtuella datorer med hanterade diskar. | Virtuella datorer med ohanterade diskar stöds inte. <br/><br/> HSM-skyddade nycklar stöds inte. <br/><br/> Kryptering av enskilda volymer på en enskild disk stöds inte. |
Azure Disk Encryption (ADE) för Linux OS | Stöds för virtuella datorer med hanterade diskar. | Virtuella datorer med ohanterade diskar stöds inte. <br/><br/> HSM-skyddade nycklar stöds inte. <br/><br/> Kryptering av enskilda volymer på en enskild disk stöds inte. |
Frekvent tillägg    | Stöds | Att aktivera replikering för en datadisk som du lägger till i en replikerad virtuell Azure-dator stöds för virtuella datorer som använder hanterade diskar. <br/><br/> Det går bara att lägga till en hård disk i taget till en virtuell Azure-dator. Det finns inte stöd för att lägga till flera diskar parallellt. |
Snabb borttagnings disk    | Stöds inte | Om du tar bort datadisk på den virtuella datorn måste du inaktivera replikeringen och aktivera replikeringen igen för den virtuella datorn.
Uteslut disk | Hjälp. Du måste använda [PowerShell](azure-to-azure-exclude-disks.md) för att konfigurera. |    Temporära diskar undantas som standard.
Lagringsutrymmen direkt  | Stöds för kraschbaserade återställnings punkter. Programmets konsekventa återställnings punkter stöds inte. |
Skalbar fil Server  | Stöds för kraschbaserade återställnings punkter. Programmets konsekventa återställnings punkter stöds inte. |
DRBD | Diskar som ingår i en DRBD-installation stöds inte. |
LRS | Stöds |
GRS | Stöds |
RA-GRS | Stöds |
ZRS | Stöds inte |
Kylning och frekvent lagring | Stöds inte | Virtuella dator diskar stöds inte i låg frekvent lagring och snabb lagring
Azure Storage brand väggar för virtuella nätverk  | Stöds | Aktivera [Tillåt betrodda Microsoft-tjänster](https://docs.microsoft.com/azure/storage/common/storage-network-security#exceptions)om du begränsar åtkomsten till virtuella nätverk till lagrings konton.
Generell användning v2-lagrings konton (både frekvent och låg frekvent nivå) | Stöds | Transaktions kostnader ökar avsevärt jämfört med generell användning v1-lagrings konton
Generation 2 (UEFI-start) | Stöds

>[!IMPORTANT]
> Undvik prestanda problem genom att följa skalbarhet för virtuella dator diskar och prestanda mål för virtuella [Linux](../virtual-machines/linux/disk-scalability-targets.md) -eller [Windows](../virtual-machines/windows/disk-scalability-targets.md) -datorer. Om du använder standardinställningarna skapar Site Recovery de nödvändiga diskarna och lagrings kontona baserat på käll konfigurationen. Om du anpassar och väljer dina egna inställningar följer du diskens skalbarhet och prestanda mål för dina virtuella käll datorer.

## <a name="limits-and-data-change-rates"></a>Begränsningar och data ändrings takt

I följande tabell sammanfattas Site Recovery gränser.

- Dessa gränser baseras på våra tester, men det är självklart inte alla möjliga kombinationer av program I/O.
- De faktiska resultaten kan variera beroende på appens I/O-mix.
- Det finns två gränser att överväga, data omsättning per disk och data omsättning per virtuell dator.

**Lagrings mål** | **Genomsnittligt käll disk-I/O** |**Average Source Disk Data Churn** (Genomsnittlig dataomsättning för källdisk) | **Total käll disk data omsättning per dag**
---|---|---|---
Standard Storage | 8 kB    | 2 MB/s | 168 GB per disk
Premium P10- eller P15-disk | 8 kB    | 2 MB/s | 168 GB per disk
Premium P10- eller P15-disk | 16 kB | 4 MB/s |    336 GB per disk
Premium P10- eller P15-disk | 32 kB eller mer | 8 MB/s | 672 GB per disk
Premium P20-, P30-, P40- eller P50-disk | 8 kB    | 5 MB/s | 421 GB per disk
Premium P20-, P30-, P40- eller P50-disk | minst 16 kB |20 MB/s | 1684 GB per disk

## <a name="replicated-machines---networking"></a>Replikerade datorer – nätverk
**Inställning** | **Support** | **Information**
--- | --- | ---
NIC | Maximalt antal som stöds för en angiven storlek på virtuell Azure-dator | Nätverkskort skapas när den virtuella datorn skapas under redundansväxling.<br/><br/> Antalet nätverkskort på den virtuella redundansväxlingen är beroende av antalet nätverkskort på den virtuella käll datorn när replikering har Aktiver ATS. Om du lägger till eller tar bort ett nätverkskort efter att ha aktiverat replikering, påverkar det inte antalet nätverkskort på den replikerade virtuella datorn efter redundansväxlingen. Observera också att ordningen på nätverkskort efter redundansväxlingen inte garanterat är densamma som den ursprungliga ordningen.
Internet-lastbalanserare | Stöds | Koppla den förkonfigurerade belastningsutjämnaren med hjälp av ett Azure Automation-skript i en återställnings plan.
Intern belastningsutjämnare | Stöds | Koppla den förkonfigurerade belastningsutjämnaren med hjälp av ett Azure Automation-skript i en återställnings plan.
Offentlig IP-adress | Stöds | Koppla en befintlig offentlig IP-adress till NÄTVERKSKORTet. Du kan också skapa en offentlig IP-adress och associera den med NÄTVERKSKORTet med hjälp av ett Azure Automation-skript i en återställnings plan.
NSG på nätverkskort | Stöds | Koppla NSG till NÄTVERKSKORTet med hjälp av ett Azure Automation-skript i en återställnings plan.
NSG i undernät | Stöds | Koppla NSG till under nätet med hjälp av ett Azure Automation-skript i en återställnings plan.
Reserverad (statisk) IP-adress | Stöds | Om NÄTVERKSKORTet på den virtuella käll datorn har en statisk IP-adress, och mål under nätet har samma IP-adress, tilldelas den misslyckade över VM.<br/><br/> Om mål under nätet inte har samma IP-adress är en av de tillgängliga IP-adresserna i under nätet reserverad för den virtuella datorn.<br/><br/> Du kan också ange en fast IP-adress och undernät i**Inställningar** > för **replikerade objekt** > ,**beräknings-och nätverks** > **nätverks gränssnitt**.
Dynamisk IP-adress | Stöds | Om NÄTVERKSKORTet på källan har dynamisk IP-adressering, är NÄTVERKSKORTet på den misslyckade virtuella datorn också dynamiskt som standard.<br/><br/> Du kan ändra detta till en fast IP-adress om det behövs.
Flera IP-adresser | Stöds inte | När du växlar över en virtuell dator som har ett nätverkskort med flera IP-adresser behålls bara den primära IP-adressen för NÄTVERKSKORTet i käll regionen. Om du vill tilldela flera IP-adresser kan du lägga till virtuella datorer i en [återställnings plan](recovery-plan-overview.md) och bifoga ett skript för att tilldela ytterligare IP-adresser till planen, eller så kan du göra ändringen manuellt eller med ett skript efter redundansväxlingen.
Traffic Manager     | Stöds | Du kan förkonfigurera Traffic Manager så att trafiken dirigeras till slut punkten i käll regionen regelbundet och till slut punkten i mål regionen i händelse av redundans.
Azure DNS | Stöds |
Anpassad DNS    | Stöds |
Oautentiserad proxy | Stöds | [Läs mer](site-recovery-azure-to-azure-networking-guidance.md)
Autentiserad proxy | Stöds inte | Om den virtuella datorn använder en autentiserad proxy för utgående anslutningar kan den inte replikeras med hjälp av Azure Site Recovery.
VPN plats-till-plats-anslutning till lokal plats<br/><br/>(med eller utan ExpressRoute)| Stöds | Se till att UDR och NSG: er har kon figurer ATS på ett sådant sätt att den Site Recovery trafiken inte dirigeras till lokalt. [Läs mer](site-recovery-azure-to-azure-networking-guidance.md)
VNET-till-VNET-anslutning    | Stöds | [Läs mer](site-recovery-azure-to-azure-networking-guidance.md)
Tjänstslutpunkter för virtuellt nätverk | Stöds | Om du begränsar det virtuella nätverkets åtkomst till lagrings konton måste du se till att de betrodda Microsoft-tjänsterna har åtkomst till lagrings kontot.
Snabbare nätverk | Stöds | Accelererat nätverk måste vara aktiverat på den virtuella käll datorn. [Läs mer](azure-vm-disaster-recovery-with-accelerated-networking.md).



## <a name="next-steps"></a>Nästa steg
- Läs [nätverks vägledning](site-recovery-azure-to-azure-networking-guidance.md) för replikering av virtuella Azure-datorer.
- Distribuera haveri beredskap genom att [Replikera virtuella Azure-datorer](site-recovery-azure-to-azure.md).
