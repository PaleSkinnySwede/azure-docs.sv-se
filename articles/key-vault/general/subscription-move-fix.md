---
title: Ändra klient-ID för Key Vault efter att prenumerationen har flyttats Azure Key Vault | Microsoft Docs
description: Lär dig hur du växlar klient-ID för ett nyckelvalv när en prenumeration flyttas till en annan klient
services: key-vault
author: amitbapat
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/12/2019
ms.author: ambapat
ms.openlocfilehash: 94bcba80e5768d57e3dc97bed1a74a8369ac60b9
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "81422895"
---
# <a name="change-a-key-vault-tenant-id-after-a-subscription-move"></a>Ändra nyckelvalvsklient-ID efter en prenumerationsflytt

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]


När du skapar ett nytt nyckelvalv i en prenumeration knyts det automatiskt till det standardmässiga Azure Active Directory-klient-ID:t för den prenumerationen. Alla åtkomstprincipposter knyts också till detta klient-ID. 

Om du flyttar din Azure-prenumeration från klient A till klient B är dina befintliga nyckel valv otillgängliga för huvud kontona (användare och program) i klient B. För att åtgärda det här problemet måste du:

* Ändra det klient-ID som är associerat med alla befintliga nyckel valv i prenumerationen till klient B.
* Ta bort alla åtkomstprincipposter.
* Lägg till nya åtkomst princip poster som är associerade med klient B.

Om du till exempel har nyckel valvet "valv" i en prenumeration som har flyttats från klient A till klient B, kan du använda Azure PowerShell för att ändra klient-ID och ta bort gamla åtkomst principer.

```azurepowershell
Select-AzSubscription -SubscriptionId <your-subscriptionId>                # Select your Azure Subscription
$vaultResourceId = (Get-AzKeyVault -VaultName myvault).ResourceId          # Get your key vault's Resource ID 
$vault = Get-AzResource –ResourceId $vaultResourceId -ExpandProperties     # Get the properties for your key vault
$vault.Properties.TenantId = (Get-AzContext).Tenant.TenantId               # Change the Tenant that your key vault resides in
$vault.Properties.AccessPolicies = @()                                     # Access policies can be updated with real
                                                                           # applications/users/rights so that it does not need to be                             # done after this whole activity. Here we are not setting 
                                                                           # any access policies. 
Set-AzResource -ResourceId $vaultResourceId -Properties $vault.Properties  # Modifies the key vault's properties.
````

Du kan också använda Azure CLI.

```azurecli
az account set -s <your-subscriptionId>                                    # Select your Azure Subscription
tenantId=$(az account show --query tenantId)                               # Get your tenantId
az keyvault update -n myvault --remove Properties.accessPolicies           # Remove the access policies
az keyvault update -n myvault --set Properties.tenantId=$tenantId          # Update the key vault tenantId
```

Nu när ditt valv är associerat med rätt klient-ID och gamla åtkomst princip poster tas bort, anger du nya åtkomst princip poster med Azure PowerShell cmdleten [set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/Set-azKeyVaultAccessPolicy) eller Azure CLI [-AZ](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) för nyckel valv.

Om du använder en hanterad identitet för Azure-resurser måste du också uppdatera den till den nya Azure AD-klienten. Mer information om hanterade identiteter finns i [tillhandahålla Key Vault autentisering med en hanterad identitet](managed-identity.md).


Om du använder MSI måste du också uppdatera MSI-identiteten eftersom den gamla identiteten inte längre kommer att finnas i rätt AAD-klient.

## <a name="next-steps"></a>Nästa steg

Om du har några frågor om Azure Key Vault kan du besöka [Azure Key Vault-forumet](https://social.msdn.microsoft.com/forums/azure/home?forum=AzureKeyVault).
