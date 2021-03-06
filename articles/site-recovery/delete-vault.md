---
title: Ta bort ett Azure Site Recovery-valv
description: Lär dig hur du tar bort ett Recovery Services valv konfigurerat för Azure Site Recovery
author: rajani-janaki-ram
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 11/05/2019
ms.author: rajanaki
ms.openlocfilehash: 0e409ffdedbac822aedf48833f2dd85f8e04afa2
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75894970"
---
# <a name="delete-a-site-recovery-services-vault"></a>Ta bort ett Site Recovery Services-valv

I den här artikeln beskrivs hur du tar bort ett Recovery Services valv för Site Recovery. Om du vill ta bort ett valv som används i Azure Backup, se [ta bort ett säkerhets kopierings valv i Azure](../backup/backup-azure-delete-vault.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]


## <a name="before-you-start"></a>Innan du börjar

Innan du kan ta bort ett valv måste du ta bort registrerade servrar och objekt i valvet. Vad du behöver ta bort beror på de scenarier för replikering som du har distribuerat. 


## <a name="delete-a-vault-azure-vm-to-azure"></a>Ta bort ett valv – virtuell Azure-dator till Azure

1. Följ [dessa anvisningar](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-azure-vm-azure-to-azure) för att ta bort alla skyddade virtuella datorer.
2. Ta sedan bort valvet.

## <a name="delete-a-vault-vmware-vm-to-azure"></a>Ta bort ett valv – VMware VM till Azure

1. Följ [dessa anvisningar](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) för att ta bort alla skyddade virtuella datorer.
2. Följ [dessa steg](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) om du vill ta bort alla principer för replikering.
3. Ta bort referenser till vCenter med [de här stegen](vmware-azure-manage-vcenter.md#delete-a-vcenter-server).
4. Följ [dessa anvisningar](vmware-azure-manage-configuration-server.md#delete-or-unregister-a-configuration-server) om du vill inaktivera en konfigurations Server.
5. Ta sedan bort valvet.


## <a name="delete-a-vault-hyper-v-vm-with-vmm-to-azure"></a>Ta bort ett valv – virtuell Hyper-V-dator (med VMM) till Azure

1. Följ [dessa steg](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-replicating-to-azure-using-the-system-center-vmm-to-azure-scenario) om du vill ta bort virtuella Hyper-V-datorer som hanteras av System Center VMM.
2. Ta bort associationen och ta bort alla principer för replikering. Gör detta i valvet > **Site Recovery infrastruktur** > **för System Center VMM** > -**replikeringsprinciper**.
3. Följ [dessa steg](site-recovery-manage-registration-and-protection.md#unregister-a-vmm-server) om du vill avregistrera en ansluten VMM-Server.
4. Ta sedan bort valvet.

## <a name="delete-a-vault-hyper-v-vm-to-azure"></a>Ta bort ett valv – virtuell Hyper-V-dator till Azure

1. Följ [dessa steg](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-hyper-v-virtual-machine-hyper-v-to-azure) om du vill ta bort alla skyddade virtuella datorer.
2. Ta bort associationen och ta bort alla principer för replikering. Gör detta i valvet > **Site Recovery infrastruktur** > **för Hyper-V-platser** > **.**
3. Följ [dessa instruktioner](site-recovery-manage-registration-and-protection.md#unregister-a-hyper-v-host-in-a-hyper-v-site) om du vill avregistrera en Hyper-V-värd.
4. Ta bort Hyper-V-platsen.
5. Ta sedan bort valvet.


## <a name="use-powershell-to-force-delete-the-vault"></a>Använd PowerShell för att framtvinga borttagning av valvet 

> [!Important]
> Om du testar produkten och inte bekymrar dig om data förlust använder du metoden Force Delete för att snabbt ta bort valvet och alla dess beroenden.
> PowerShell-kommandot tar bort allt innehåll i valvet och går **inte att ångra**.

Om du vill ta bort Site Recovery valvet även om det finns skyddade objekt, använder du följande kommandon:

    Connect-AzAccount

    Select-AzSubscription -SubscriptionName "XXXXX"

    $vault = Get-AzRecoveryServicesVault -Name "vaultname"

    Remove-AzRecoveryServicesVault -Vault $vault

Läs mer om [Get-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesvault)och [Remove-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault).
