---
title: 'Snabb start: skapa en Azure-brandvägg med Tillgänglighetszoner-Resource Manager-mall'
description: Distribuera Azure-brandväggen med en mall. Det virtuella nätverket har ett virtuellt nätverk med tre undernät. Två virtuella Windows Server-datorer distribueras. en hopp ruta och en server.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: quickstart
ms.date: 04/30/2020
ms.author: victorh
ms.openlocfilehash: 17ab693033b61c25ba2f5b5bd588ef52caf8c046
ms.sourcegitcommit: acc558d79d665c8d6a5f9e1689211da623ded90a
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82597713"
---
# <a name="quickstart-deploy-azure-firewall-with-availability-zones---resource-manager-template"></a>Snabb start: Distribuera Azure-brandväggen med Tillgänglighetszoner Resource Manager-mall

I den här snabb starten använder du en Resource Manager-mall för att distribuera en Azure-brandvägg i tre Tillgänglighetszoner. 

Mallen skapar en test nätverks miljö med en brand vägg. Nätverket har ett virtuellt nätverk (VNet) med tre undernät: *AzureFirewallSubnet*, *ServersSubnet*och *JumpboxSubnet*. *ServersSubnet* -och *JumpboxSubnet* -undernätet har en enda virtuell Windows Server-dator med två kärnor.

Brand väggen finns i *AzureFirewallSubnet* -undernätet och har en samling med program regler med en enda regel som tillåter åtkomst `www.microsoft.com`till.

En användardefinierad väg pekar nätverks trafik från *ServersSubnet* -undernätet via brand väggen, där brand Väggs reglerna tillämpas.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Mer information om Azure-brandväggen finns i [distribuera och konfigurera Azure-brandväggen med hjälp av Azure Portal](tutorial-firewall-deploy-portal.md).

## <a name="prerequisites"></a>Krav

- Ett Azure-konto med en aktiv prenumeration. [Skapa ett konto kostnads fritt](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-an-azure-firewall-with-availability-zones"></a>Skapa en Azure-brandvägg med Tillgänglighetszoner

Den här mallen skapar en Azure-brandvägg med Tillgänglighetszoner, tillsammans med nödvändiga resurser för att stödja Azure-brandväggen.

### <a name="review-the-template"></a>Granska mallen

Mallen som används i den här snabb starten är från [Azure snabb starts-mallar](https://github.com/Azure/azure-quickstart-templates/blob/master/101-azurefirewall-with-zones-sandbox/azuredeploy.json).

:::code language="json" source="~/quickstart-templates/101-azurefirewall-with-zones-sandbox/azuredeploy.json" range="001-444" highlight="369-442":::

Flera Azure-resurser definieras i mallen:

- [**Microsoft. Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft. Network/networkSecurityGroups**](/azure/templates/microsoft.network/networksecuritygroups)
- [**Microsoft. Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft. Compute/virtualMachines**](/azure/templates/microsoft.compute/virtualmachines)
- [**Microsoft. Network/networkInterfaces**](/azure/templates/microsoft.network/networkinterfaces)
- [**Microsoft. Storage/storageAccounts**](/azure/templates/microsoft.storage/storageAccounts)
- [**Microsoft. Network/azureFirewalls**](/azure/templates/microsoft.network/azureFirewalls)
- [**Microsoft. Network/routeTables**](/azure/templates/microsoft.network/routeTables)

### <a name="deploy-the-template"></a>Distribuera mallen

Distribuera Resource Manager-mallen till Azure:

1. Välj **distribuera till Azure** för att logga in på Azure och öppna mallen. Mallen skapar en Azure-brandvägg, nätverks infrastrukturen och två virtuella datorer.

   [![Distribuera till Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurefirewall-with-zones-sandbox%2Fazuredeploy.json)

2. I portalen, på sidan **skapa en sandbox-installation av Azure-brandvägg med zoner** , skriver eller väljer du följande värden:
   - **Resurs grupp**: Välj **Skapa ny**, Skriv ett namn för resurs gruppen och välj **OK**. 
   - **Virtual Network namn**: Ange ett namn för det nya virtuella nätverket. 
   - **Admin-användar namn**: Ange ett användar namn för administratörs användar kontot.
   - **Administratörs lösen ord**: Ange ett administratörs lösen ord. 

3. Läs igenom villkoren och välj sedan **Jag accepterar villkoren ovan** och välj sedan **köp**. Distributionen kan ta 10 minuter eller längre att slutföra.

## <a name="review-deployed-resources"></a>Granska distribuerade resurser

Utforska de resurser som har skapats med brand väggen.

Information om JSON-syntaxen och egenskaperna för en brand vägg i en mall finns i [Microsoft. Network/azureFirewalls](/azure/templates/microsoft.network/azurefirewalls).

## <a name="clean-up-resources"></a>Rensa resurser

När du inte längre behöver dem kan du ta bort resurs gruppen, brand väggen och alla relaterade resurser genom att `Remove-AzResourceGroup` köra PowerShell-kommandot. Om du vill ta bort en resurs grupp med namnet *MyResourceGroup*kör du: 

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyResourceGroup
```
Ta inte bort resurs gruppen och brand väggen om du planerar att fortsätta till själv studie kursen om brand Väggs övervakning. 

## <a name="next-steps"></a>Nästa steg

Sedan kan du övervaka Azure Firewall-loggarna:

[Självstudie: Övervaka Azure Firewall-loggar](./tutorial-diagnostics.md)
