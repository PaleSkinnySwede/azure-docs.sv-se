---
title: Kryptering på Server sidan av Azure Managed Disks – PowerShell
description: Azure Storage skyddar dina data genom att kryptera dem i vila innan du sparar dem i lagrings kluster. Du kan förlita dig på Microsoft-hanterade nycklar för kryptering av dina hanterade diskar, eller så kan du använda Kundhanterade nycklar för att hantera kryptering med dina egna nycklar.
author: roygara
ms.date: 04/21/2020
ms.topic: conceptual
ms.author: rogarana
ms.service: virtual-machines-windows
ms.subservice: disks
ms.openlocfilehash: 4b693ef1eaf7c8dd1f2fd95116c24392ee9a9454
ms.sourcegitcommit: 90d2d95f2ae972046b1cb13d9956d6668756a02e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/14/2020
ms.locfileid: "83402580"
---
# <a name="server-side-encryption-of-azure-managed-disks"></a>Kryptering på Server sidan av Azure Managed disks

Azure Managed disks krypterar automatiskt dina data som standard när de sparas i molnet. Server Side Encryption (SSE) skyddar dina data och hjälper dig att uppfylla organisationens säkerhets-och efterlevnads åtaganden.

Data i Azure Managed disks krypteras transparent med 256-bitars [AES-kryptering](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), en av de starkaste block chiffer som är tillgängliga och är FIPS 140-2-kompatibel. Mer information om de kryptografiska modulerna underliggande Azure Managed disks finns i [Cryptography-API: nästa generation](https://docs.microsoft.com/windows/desktop/seccng/cng-portal)

Kryptering påverkar inte prestandan hos hanterade diskar och det finns ingen ytterligare kostnad för krypteringen. 

> [!NOTE]
> Temporära diskar är inte hanterade diskar och krypteras inte av SSE; Mer information om temporära diskar finns i [Översikt över Managed disks: disk roller](managed-disks-overview.md#disk-roles).

## <a name="about-encryption-key-management"></a>Om hantering av krypterings nyckel

Du kan förlita dig på plattforms hanterade nycklar för kryptering av din hanterade disk, eller så kan du hantera kryptering med hjälp av dina egna nycklar. Om du väljer att hantera kryptering med dina egna nycklar kan du ange en *kundhanterad nyckel* som ska användas för att kryptera och dekryptera alla data i Managed disks. 

I följande avsnitt beskrivs de olika alternativen för nyckel hantering i större detalj.

## <a name="platform-managed-keys"></a>Plattforms hanterade nycklar

Som standard använder hanterade diskar plattforms hanterade krypterings nycklar. Från och med den 10 juni 2017 krypteras alla nya hanterade diskar, ögonblicks bilder, bilder och nya data som skrivs till befintliga hanterade diskar automatiskt i vila med plattforms hanterade nycklar.

## <a name="customer-managed-keys"></a>Kundhanterade nycklar

Du kan välja att hantera kryptering på nivån för varje hanterad disk med dina egna nycklar. Kryptering på Server sidan för Managed disks med Kundhanterade nycklar ger en integrerad upplevelse med Azure Key Vault. Du kan antingen importera [dina RSA-nycklar](../../key-vault/keys/hsm-protected-keys.md) till Key Vault eller generera nya RSA-nycklar i Azure Key Vault. 

Azure Managed disks hanterar kryptering och dekryptering på ett helt transparent sätt med hjälp av [Kuvert kryptering](../../storage/common/storage-client-side-encryption.md#encryption-and-decryption-via-the-envelope-technique). Den krypterar data med hjälp av en [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 256-baserad data krypterings nyckel (DEK), som i sin tur skyddas med hjälp av dina nycklar. Lagrings tjänsten genererar data krypterings nycklar och krypterar dem med Kundhanterade nycklar med hjälp av RSA-kryptering. Med kuvert krypteringen kan du rotera (ändra) dina nycklar regelbundet enligt dina efterlevnadsprinciper utan att de virtuella datorerna påverkas. När du roterar dina nycklar krypterar lagrings tjänsten om data krypterings nycklarna med de nya Kundhanterade nycklarna. 

Du måste bevilja åtkomst till hanterade diskar i Key Vault för att kunna använda dina nycklar för att kryptera och dekryptera DEK. Detta gör att du får fullständig kontroll över dina data och nycklar. Du kan när som helst inaktivera dina nycklar eller återkalla åtkomsten till hanterade diskar. Du kan också granska krypterings nyckel användningen med Azure Key Vault övervakning så att endast hanterade diskar eller andra betrodda Azure-tjänster får åtkomst till dina nycklar.

För Premium SSD, standard-SSD och standard-HDD: när du inaktiverar eller tar bort din nyckel stängs alla virtuella datorer med diskar som använder den nyckeln automatiskt. Efter detta kommer de virtuella datorerna inte att vara användbara om inte nyckeln är aktive rad igen eller om du tilldelar en ny nyckel.

När du inaktiverar eller tar bort en nyckel för Ultra disk stängs alla virtuella datorer med Ultra disks med hjälp av nyckeln inte automatiskt. När du har avallokerat och startat om de virtuella datorerna slutar diskarna att använda nyckeln och de virtuella datorerna kommer inte att vara online igen. Om du vill ta de virtuella datorerna online igen måste du tilldela en ny nyckel eller aktivera den befintliga nyckeln.

Följande diagram visar hur hanterade diskar använder Azure Active Directory och Azure Key Vault för att göra förfrågningar med hjälp av kundhanterad nyckel:

![Arbets flöde för hanterade diskar och Kundhanterade nycklar. En administratör skapar en Azure Key Vault, skapar en disk krypterings uppsättning och konfigurerar disk krypterings uppsättningen. Uppsättningen är kopplad till en virtuell dator, vilket gör att disken kan använda Azure AD för att autentisera](media/disk-storage-encryption/customer-managed-keys-sse-managed-disks-workflow.png)


I följande lista beskrivs diagrammet i detalj:

1. En Azure Key Vault administratör skapar nyckel valvs resurser.
1. Key Vault-administratören importerar antingen sina RSA-nycklar för att Key Vault eller generera nya RSA-nycklar i Key Vault.
1. Administratören skapar en instans av en resurs för disk krypterings uppsättning och anger ett Azure Key Vault-ID och en nyckel-URL. Disk krypterings uppsättning är en ny resurs som introducerats för att förenkla nyckel hanteringen för hanterade diskar. 
1. När en disk krypterings uppsättning skapas skapas en [systemtilldelad hanterad identitet](../../active-directory/managed-identities-azure-resources/overview.md) i Azure Active Directory (AD) och associeras med disk krypterings uppsättningen. 
1. Azure Key Vault-administratören ger sedan den hanterade identiteten behörighet att utföra åtgärder i nyckel valvet.
1. En VM-användare skapar diskar genom att associera dem med disk krypterings uppsättningen. Den virtuella dator användaren kan också aktivera kryptering på Server sidan med Kundhanterade nycklar för befintliga resurser genom att associera dem med disk krypterings uppsättningen. 
1. Managed disks använder den hanterade identiteten för att skicka begär anden till Azure Key Vault.
1. Vid läsning eller skrivning av data skickar Managed disks begär anden till Azure Key Vault att kryptera (omslutning) och dekryptera (unwrap) data krypterings nyckeln för att utföra kryptering och dekryptering av data. 

Information om hur du återkallar åtkomst till Kundhanterade nycklar finns i [Azure Key Vault PowerShell](https://docs.microsoft.com/powershell/module/azurerm.keyvault/) och [Azure Key Vault CLI](https://docs.microsoft.com/cli/azure/keyvault). Att återkalla åtkomsten på ett effektivt sätt blockerar åtkomsten till alla data i lagrings kontot, eftersom krypterings nyckeln inte är tillgänglig via Azure Storage.

### <a name="supported-regions"></a>Regioner som stöds

[!INCLUDE [virtual-machines-disks-encryption-regions](../../../includes/virtual-machines-disks-encryption-regions.md)]

### <a name="restrictions"></a>Begränsningar

För närvarande har Kundhanterade nycklar följande begränsningar:

- Om den här funktionen är aktive rad för disken kan du inte inaktivera den.
    Om du behöver kringgå detta måste du [Kopiera alla data](disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk) till en helt annan hanterad disk som inte använder Kundhanterade nycklar.
- Endast ["mjuka" och "hårda" RSA-nycklar](../../key-vault/keys/about-keys.md) med storleken 2080 stöds, inga andra nycklar eller storlekar.
- Diskar som har skapats från anpassade avbildningar som krypteras med kryptering på Server sidan och Kundhanterade nycklar måste krypteras med samma Kundhanterade nycklar och måste finnas i samma prenumeration.
- Ögonblicks bilder som har skapats från diskar som är krypterade med kryptering på Server sidan och Kundhanterade nycklar måste vara krypterade med samma Kundhanterade nycklar.
- Alla resurser som är relaterade till dina Kundhanterade nycklar (Azure Key Vaults, disk krypterings uppsättningar, virtuella datorer, diskar och ögonblicks bilder) måste finnas i samma prenumeration och region.
- Diskar, ögonblicks bilder och bilder som är krypterade med Kundhanterade nycklar kan inte flyttas till en annan prenumeration.
- Om du använder Azure Portal för att skapa disk krypterings uppsättningen kan du inte använda ögonblicks bilder för tillfället.
- Hanterade diskar som krypterats med kryptering på Server sidan med Kundhanterade nycklar kan inte också krypteras med Azure Disk Encryption och vice versa
- Information om hur du använder Kundhanterade nycklar med delade avbildnings gallerier finns i för [hands version: Använd Kundhanterade nycklar för att kryptera avbildningar](../image-version-encryption.md).

### <a name="powershell"></a>PowerShell

#### <a name="setting-up-your-azure-key-vault-and-diskencryptionset"></a>Konfigurera din Azure Key Vault och DiskEncryptionSet

1. Kontrol lera att du har installerat den senaste [versionen av Azure PowerShell](/powershell/azure/install-az-ps)och att du är inloggad på ett Azure-konto i med Connect-AzAccount

1. Skapa en instans av Azure Key Vault och krypterings nyckel.

    När du skapar Key Vault-instansen måste du aktivera mjuk borttagning och tömning av skydd. Mjuk borttagning garanterar att Key Vault innehåller en borttagen nyckel för en viss kvarhållningsperiod (90 dag standard). Med rensnings skyddet går det inte att ta bort en borttagen nyckel förrän kvarhållningsperioden upphörde. Dessa inställningar skyddar dig från att förlora data på grund av oavsiktlig borttagning. De här inställningarna är obligatoriska när du använder en Key Vault för att kryptera hanterade diskar.

    > [!IMPORTANT]
    > Kamel notation inte, om du gör det kan det uppstå problem när du tilldelar ytterligare diskar till resursen i Azure Portal.
    
    ```powershell
    $ResourceGroupName="yourResourceGroupName"
    $LocationName="westcentralus"
    $keyVaultName="yourKeyVaultName"
    $keyName="yourKeyName"
    $keyDestination="Software"
    $diskEncryptionSetName="yourDiskEncryptionSetName"

    $keyVault = New-AzKeyVault -Name $keyVaultName -ResourceGroupName $ResourceGroupName -Location $LocationName -EnableSoftDelete -EnablePurgeProtection

    $key = Add-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName -Destination $keyDestination  
    ```

1.    Skapa en instans av en DiskEncryptionSet. 
    
        ```powershell
        $desConfig=New-AzDiskEncryptionSetConfig -Location $LocationName -SourceVaultId $keyVault.ResourceId -KeyUrl $key.Key.Kid -IdentityType SystemAssigned
        
        $des=New-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName -InputObject $desConfig 
        ```

1.    Ge DiskEncryptionSet-resursen åtkomst till nyckel valvet.

        > [!NOTE]
        > Det kan ta några minuter för Azure att skapa identiteten för din DiskEncryptionSet i din Azure Active Directory. Om du får ett fel meddelande som "det går inte att hitta Active Directory-objektet" när du kör följande kommando, väntar du några minuter och försöker igen.
        
        ```powershell
        $identity = Get-AzADServicePrincipal -DisplayName myDiskEncryptionSet1  
         
        Set-AzKeyVaultAccessPolicy -VaultName $keyVaultName -ObjectId $des.Identity.PrincipalId -PermissionsToKeys wrapkey,unwrapkey,get
         
        New-AzRoleAssignment -ResourceName $keyVaultName -ResourceGroupName $ResourceGroupName -ResourceType "Microsoft.KeyVault/vaults" -ObjectId $des.Identity.PrincipalId -RoleDefinitionName "Reader" 
        ```

#### <a name="create-a-vm-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Skapa en virtuell dator med en Marketplace-avbildning, kryptera operativ system och data diskar med Kundhanterade nycklar

```powershell
$VMLocalAdminUser = "yourVMLocalAdminUserName"
$VMLocalAdminSecurePassword = ConvertTo-SecureString <password> -AsPlainText -Force
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$ComputerName = "yourComputerName"
$VMName = "yourVMName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourdiskEncryptionSetName"
    
$NetworkName = "yourNetworkName"
$NICName = "yourNICName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix
$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet
$NIC = New-AzNetworkInterface -Name $NICName -ResourceGroupName $ResourceGroupName -Location $LocationName -SubnetId $Vnet.Subnets[0].Id
    
$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
    
$VirtualMachine = New-AzVMConfig -VMName $VMName -VMSize $VMSize
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName $ComputerName -Credential $Credential -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2012-R2-Datacenter' -Version latest

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$VirtualMachine = Set-AzVMOSDisk -VM $VirtualMachine -Name $($VMName +"_OSDisk") -DiskEncryptionSetId $diskEncryptionSet.Id -CreateOption FromImage

$VirtualMachine = Add-AzVMDataDisk -VM $VirtualMachine -Name $($VMName +"DataDisk1") -DiskSizeInGB 128 -StorageAccountType Premium_LRS -CreateOption Empty -Lun 0 -DiskEncryptionSetId $diskEncryptionSet.Id 
    
New-AzVM -ResourceGroupName $ResourceGroupName -Location $LocationName -VM $VirtualMachine -Verbose
```

#### <a name="create-an-empty-disk-encrypted-using-server-side-encryption-with-customer-managed-keys-and-attach-it-to-a-vm"></a>Skapa en tom disk som är krypterad med kryptering på Server sidan med Kundhanterade nycklar och koppla den till en virtuell dator

```PowerShell
$vmName = "yourVMName"
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskSKU = "Premium_LRS"
$diskSizeinGiB = 30
$diskLUN = 1
$diskEncryptionSetName="yourDiskEncryptionSetName"


$vm = Get-AzVM -Name $vmName -ResourceGroupName $ResourceGroupName 

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

$vm = Add-AzVMDataDisk -VM $vm -Name $diskName -CreateOption Empty -DiskSizeInGB $diskSizeinGiB -StorageAccountType $diskSKU -Lun $diskLUN -DiskEncryptionSetId $diskEncryptionSet.Id 

Update-AzVM -ResourceGroupName $ResourceGroupName -VM $vm

```

#### <a name="encrypt-existing-managed-disks"></a>Kryptera befintliga hanterade diskar 

Dina befintliga diskar får inte anslutas till en virtuell dator som körs för att du ska kunna kryptera dem med följande skript:

```PowerShell
$rgName = "yourResourceGroupName"
$diskName = "yourDiskName"
$diskEncryptionSetName = "yourDiskEncryptionSetName"
 
$diskEncryptionSet = Get-AzDiskEncryptionSet -ResourceGroupName $rgName -Name $diskEncryptionSetName
 
New-AzDiskUpdateConfig -EncryptionType "EncryptionAtRestWithCustomerKey" -DiskEncryptionSetId $diskEncryptionSet.Id | Update-AzDisk -ResourceGroupName $rgName -DiskName $diskName
```

#### <a name="create-a-virtual-machine-scale-set-using-a-marketplace-image-encrypting-the-os-and-data-disks-with-customer-managed-keys"></a>Skapa en skalnings uppsättning för virtuella datorer med en Marketplace-avbildning, kryptera operativ system och data diskar med Kundhanterade nycklar

```PowerShell
$VMLocalAdminUser = "yourLocalAdminUser"
$VMLocalAdminSecurePassword = ConvertTo-SecureString Password@123 -AsPlainText -Force
$LocationName = "westcentralus"
$ResourceGroupName = "yourResourceGroupName"
$ComputerNamePrefix = "yourComputerNamePrefix"
$VMScaleSetName = "yourVMSSName"
$VMSize = "Standard_DS3_v2"
$diskEncryptionSetName="yourDiskEncryptionSetName"
    
$NetworkName = "yourVNETName"
$SubnetName = "yourSubnetName"
$SubnetAddressPrefix = "10.0.0.0/24"
$VnetAddressPrefix = "10.0.0.0/16"
    
$SingleSubnet = New-AzVirtualNetworkSubnetConfig -Name $SubnetName -AddressPrefix $SubnetAddressPrefix

$Vnet = New-AzVirtualNetwork -Name $NetworkName -ResourceGroupName $ResourceGroupName -Location $LocationName -AddressPrefix $VnetAddressPrefix -Subnet $SingleSubnet

$ipConfig = New-AzVmssIpConfig -Name "myIPConfig" -SubnetId $Vnet.Subnets[0].Id 

$VMSS = New-AzVmssConfig -Location $LocationName -SkuCapacity 2 -SkuName $VMSize -UpgradePolicyMode 'Automatic'

$VMSS = Add-AzVmssNetworkInterfaceConfiguration -Name "myVMSSNetworkConfig" -VirtualMachineScaleSet $VMSS -Primary $true -IpConfiguration $ipConfig

$diskEncryptionSet=Get-AzDiskEncryptionSet -ResourceGroupName $ResourceGroupName -Name $diskEncryptionSetName

# Enable encryption at rest with customer managed keys for OS disk by setting DiskEncryptionSetId property 

$VMSS = Set-AzVmssStorageProfile $VMSS -OsDiskCreateOption "FromImage" -DiskEncryptionSetId $diskEncryptionSet.Id -ImageReferenceOffer 'WindowsServer' -ImageReferenceSku '2012-R2-Datacenter' -ImageReferenceVersion latest -ImageReferencePublisher 'MicrosoftWindowsServer'

$VMSS = Set-AzVmssOsProfile $VMSS -ComputerNamePrefix $ComputerNamePrefix -AdminUsername $VMLocalAdminUser -AdminPassword $VMLocalAdminSecurePassword

# Add a data disk encrypted at rest with customer managed keys by setting DiskEncryptionSetId property 

$VMSS = Add-AzVmssDataDisk -VirtualMachineScaleSet $VMSS -CreateOption Empty -Lun 1 -DiskSizeGB 128 -StorageAccountType Premium_LRS -DiskEncryptionSetId $diskEncryptionSet.Id

$Credential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);

New-AzVmss -VirtualMachineScaleSet $VMSS -ResourceGroupName $ResourceGroupName -VMScaleSetName $VMScaleSetName
```

#### <a name="change-the-key-of-a-diskencryptionset-to-rotate-the-key-for-all-the-resources-referencing-the-diskencryptionset"></a>Ändra nyckeln för en DiskEncryptionSet för att rotera nyckeln för alla resurser som refererar till DiskEncryptionSet

```PowerShell
$ResourceGroupName="yourResourceGroupName"
$keyVaultName="yourKeyVaultName"
$keyName="yourKeyName"
$diskEncryptionSetName="yourDiskEncryptionSetName"

$keyVault = Get-AzKeyVault -VaultName $keyVaultName -ResourceGroupName $ResourceGroupName

$keyVaultKey = Get-AzKeyVaultKey -VaultName $keyVaultName -Name $keyName

Update-AzDiskEncryptionSet -Name $diskEncryptionSetName -ResourceGroupName $ResourceGroupName -SourceVaultId $keyVault.ResourceId -KeyUrl $keyVaultKey.Id
```

#### <a name="find-the-status-of-server-side-encryption-of-a-disk"></a>Hitta statusen för kryptering på Server sidan för en disk

```PowerShell
$ResourceGroupName="yourResourceGroupName"
$DiskName="yourDiskName"

$disk=Get-AzDisk -ResourceGroupName $ResourceGroupName -DiskName $DiskName
$disk.Encryption.Type

```

> [!IMPORTANT]
> Kundhanterade nycklar är beroende av hanterade identiteter för Azure-resurser, en funktion i Azure Active Directory (Azure AD). När du konfigurerar Kundhanterade nycklar, tilldelas en hanterad identitet automatiskt till dina resurser under försättsblad. Om du senare flyttar prenumerationen, resurs gruppen eller den hanterade disken från en Azure AD-katalog till en annan överförs inte den hanterade identitet som är kopplad till hanterade diskar till den nya klienten, så Kundhanterade nycklar kanske inte längre fungerar. Mer information finns i [överföra en prenumeration mellan Azure AD-kataloger](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

[!INCLUDE [virtual-machines-disks-encryption-portal](../../../includes/virtual-machines-disks-encryption-portal.md)]

> [!IMPORTANT]
> Kundhanterade nycklar är beroende av hanterade identiteter för Azure-resurser, en funktion i Azure Active Directory (Azure AD). När du konfigurerar Kundhanterade nycklar, tilldelas en hanterad identitet automatiskt till dina resurser under försättsblad. Om du senare flyttar prenumerationen, resurs gruppen eller den hanterade disken från en Azure AD-katalog till en annan överförs inte den hanterade identitet som är kopplad till hanterade diskar till den nya klienten, så Kundhanterade nycklar kanske inte längre fungerar. Mer information finns i [överföra en prenumeration mellan Azure AD-kataloger](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="server-side-encryption-versus-azure-disk-encryption"></a>Kryptering på Server sidan jämfört med Azure Disk Encryption

[Azure Disk Encryption](../../security/fundamentals/azure-disk-encryption-vms-vmss.md) utnyttjar [BitLocker](https://docs.microsoft.com/windows/security/information-protection/bitlocker/bitlocker-overview) -funktionen i Windows för att kryptera hanterade diskar med Kundhanterade nycklar i den virtuella gäst datorn.  Kryptering på Server sidan med Kundhanterade nycklar förbättrar på ADE genom att du kan använda alla OS-typer och avbildningar för dina virtuella datorer genom att kryptera data i lagrings tjänsten.

## <a name="next-steps"></a>Nästa steg

- [Utforska Azure Resource Manager mallar för att skapa krypterade diskar med Kundhanterade nycklar](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Vad är Azure Key Vault?](../../key-vault/general/overview.md)
- [Replikera datorer med Kundhanterade nycklar aktiverade diskar](../../site-recovery/azure-to-azure-how-to-enable-replication-cmk-disks.md)
- [Konfigurera katastrof återställning av virtuella VMware-datorer till Azure med PowerShell](../../site-recovery/vmware-azure-disaster-recovery-powershell.md#replicate-vmware-vms)
- [Konfigurera katastrof återställning till Azure för virtuella Hyper-V-datorer med PowerShell och Azure Resource Manager](../../site-recovery/hyper-v-azure-powershell-resource-manager.md#step-7-enable-vm-protection)
