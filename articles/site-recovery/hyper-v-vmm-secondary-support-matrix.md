---
title: Stöd för matris – återställning av Hyper-V till en sekundär VMM-plats med Azure Site Recovery
description: Sammanfattar stöd för replikering av virtuella Hyper-V-datorer i VMM-moln till en sekundär plats med Azure Site Recovery.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/06/2019
ms.author: raynew
ms.openlocfilehash: 1126a85ed22ee17879767a93ca75dc76dd04b747
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74132958"
---
# <a name="support-matrix-for-disaster-recovery-of-hyper-v-vms-to-a-secondary-site"></a>Supportmatris för haveriberedskap av virtuella Hyper-V-datorer till en sekundär webbplats

I den här artikeln sammanfattas vad som stöds när du använder tjänsten [Azure Site Recovery](site-recovery-overview.md) för att replikera virtuella Hyper-V-datorer som hanteras i System Center Virtual Machine Manager (VMM)-moln till en sekundär plats. Om du vill replikera virtuella Hyper-V-datorer till Azure läser du [denna support mat ris](hyper-v-azure-support-matrix.md).

> [!NOTE]
> Du kan bara replikera till en sekundär plats när dina Hyper-V-värdar hanteras i VMM-moln.


## <a name="host-servers"></a>Värd servrar

**Operativsystem** | **Information**
--- | ---
Windows Server 2012 R2 | Servrarna måste köra de senaste uppdateringarna.
Windows Server 2016 |  VMM 2016-moln med en blandning av Windows Server 2016-och 2012 R2-värdar stöds inte för närvarande.<br/><br/> Distributioner som uppgraderas från System Center 2012 R2 VMM 2012 R2 till System Center 2016 stöds inte för närvarande.


## <a name="replicated-vm-support"></a>Stöd för replikerad virtuell dator

I följande tabell sammanfattas stöd för operativ system för datorer som replikeras med Site Recovery. Alla arbets belastningar kan köras på det operativ system som stöds.

**Windows-version** | **Hyper-V (med VMM)**
--- | ---
Windows Server 2016 | Alla gäst operativ system [som stöds av Hyper-V](https://docs.microsoft.com/windows-server/virtualization/hyper-v/Supported-Windows-guest-operating-systems-for-Hyper-V-on-Windows) på Windows Server 2016 
Windows Server 2012 R2 | Alla gäst operativ system [som stöds av Hyper-V](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn792027%28v%3dws.11%29) på Windows Server 2012 R2

## <a name="linux-machine-storage"></a>Linux-dator lagring

Endast Linux-datorer med följande lagring kan replikeras:

- Fil system (EXT3, ETX4, ReiserFS, XFS).
- Multipath program vara-enhets mappning.
- Volym hanteraren (LVM2).
- Fysiska servrar med HP CCISS Controller-lagring stöds inte.
- Fil systemet ReiserFS stöds endast på SUSE Linux Enterprise Server 11 SP3.

## <a name="network-configuration---hostguest-vm"></a>Nätverks konfiguration – värd/gäst-VM

**Konfiguration** | **Stöds**  
--- | --- 
Värd-NIC-teamning | Ja 
Värd-VLAN | Ja 
Värd-IPv4 | Ja 
Värd-IPv6 | Inga 
Gäst-VM – NIC Teaming | Inga
Gäst-VM – IPv4 | Ja
Gäst-VM-IPv6 | Inga
Gäst-VM – Windows/Linux-statisk IP-adress | Ja
Gäst-VM-Multi-NIC | Ja


## <a name="storage"></a>Storage

### <a name="host-storage"></a>Värd lagring

**Lagring (värd)** | **Stöds**
--- | --- 
NFS | E.t.
SMB 3.0 |  Ja
SAN (ISCSI) | Ja
Multipath (MPIO) | Ja

### <a name="guest-or-physical-server-storage"></a>Gäst eller fysisk server lagring

**Konfiguration** | **Stöds**
--- | --- | 
VMDK |  E.t.
VHD/VHDX | Ja (upp till 16 diskar)
Generation 2 VM | Ja
Delad kluster disk | Inga
Krypterad disk | Inga
UEFI| E.t.
NFS | Inga
SMB 3.0 | Inga
RDM | E.t.
Disk > 1 TB | Ja
Volym med Striped disk > 1 TB<br/><br/> LVM | Ja
Lagringsutrymmen | Ja
Snabb Lägg till/ta bort disk | Inga
Uteslut disk | Ja
Multipath (MPIO) | Ja

## <a name="vaults"></a>Valv

**Åtgärd** | **Stöds**
--- | --- 
Flytta valv över resurs grupper (inom eller mellan prenumerationer) |  Inga
Flytta lagring, nätverk, virtuella Azure-datorer över resurs grupper (inom eller mellan prenumerationer) | Inga

## <a name="azure-site-recovery-provider"></a>Azure Site Recovery Provider

Providern samordnar kommunikationen mellan VMM-servrar. 

**Nya** | **Uppdateringar**
--- | --- 
5.1.19 ([tillgängligt från portalen](https://aka.ms/downloaddra) | [Senaste funktioner och korrigeringar](https://support.microsoft.com/kb/3155002)



## <a name="next-steps"></a>Nästa steg

[Replikera virtuella Hyper-V-datorer i VMM-moln till en sekundär plats](tutorial-vmm-to-vmm.md)

