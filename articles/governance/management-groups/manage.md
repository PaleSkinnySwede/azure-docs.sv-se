---
title: Arbeta med hanterings grupper – Azure-styrning
description: Lär dig att visa, underhålla, uppdatera och ta bort en hierarki för hanterings grupper.
ms.date: 04/15/2020
ms.topic: conceptual
ms.openlocfilehash: 423d1837c3d5710e24abb94f5411200319e8a8aa
ms.sourcegitcommit: d662eda7c8eec2a5e131935d16c80f1cf298cb6b
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/01/2020
ms.locfileid: "81381666"
---
# <a name="manage-your-resources-with-management-groups"></a>Hantera dina resurser med hanterings grupper

Om din organisation har många prenumerationer kan det behövas ett effektivt sätt att hantera åtkomst, principer och efterlevnad för prenumerationerna. Med Azures hanteringsgrupper får du en hanteringsnivå över prenumerationsnivån. Du kan ordna prenumerationerna i containrar som kallas hanteringsgrupper och tillämpa styrningsvillkor för hanteringsgrupperna. Alla prenumerationer i en hanteringsgrupp ärver automatiskt de villkor som tillämpas för hanteringsgruppen.

Hanteringsgrupper tillhandahåller hantering i företagsklass i stor skala oavsett vilken typ av prenumeration du har. Mer information om hanterings grupper finns i [ordna dina resurser med Azures hanterings grupper](./overview.md).

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

> [!IMPORTANT]
> Azure Resource Manager tokens och cache för hanterings grupper varar i 30 minuter innan de tvingas uppdatera. När du har utfört en åtgärd som att flytta en hanterings grupp eller prenumeration kan det ta upp till 30 minuter att visa. Om du vill se uppdateringarna tidigare måste du uppdatera din token genom att uppdatera webbläsaren, logga in och ut eller begära en ny token.  

## <a name="change-the-name-of-a-management-group"></a>Ändra namnet på en hanterings grupp

Du kan ändra namnet på hanterings gruppen med hjälp av portalen, PowerShell eller Azure CLI.

### <a name="change-the-name-in-the-portal"></a>Ändra namnet i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Välj den hanterings grupp som du vill byta namn på.

1. Välj **information**.

1. Välj alternativet för att **byta namn på grupp** överst på sidan.

   :::image type="content" source="./media/detail_action_small.png" alt-text="Byt namn på grupp alternativ på hanterings grupp Sidan" border="false":::

1. När menyn öppnas anger du det nya namn som du vill ska visas.

   :::image type="content" source="./media/rename_context.png" alt-text="Byt namn på grupp fönstret för att byta namn på hanterings grupp" border="false":::

1. Välj **Spara**.

### <a name="change-the-name-in-powershell"></a>Ändra namnet i PowerShell

Om du vill uppdatera visnings namnet använder du **Update-AzManagementGroup**. Om du till exempel vill ändra visnings namnet för en hanterings grupp från "contoso IT" till "contoso-grupp" kör du följande kommando:

```azurepowershell-interactive
Update-AzManagementGroup -GroupName 'ContosoIt' -DisplayName 'Contoso Group'
```

### <a name="change-the-name-in-azure-cli"></a>Ändra namnet i Azure CLI

För Azure CLI använder du kommandot Update.

```azurecli-interactive
az account management-group update --name 'Contoso' --display-name 'Contoso Group'
```

## <a name="delete-a-management-group"></a>Ta bort en hanterings grupp

Om du vill ta bort en hanterings grupp måste följande krav uppfyllas:

1. Det finns inga underordnade hanterings grupper eller prenumerationer under hanterings gruppen.

   - Om du vill flytta en prenumeration eller hanterings grupp till en annan hanterings grupp läser du [flytta hanterings grupper och prenumerationer i hierarkin](#moving-management-groups-and-subscriptions).

1. Du behöver Skriv behörighet för hanterings gruppen ("ägare", "deltagare" eller "hanterings grupp deltagare"). Om du vill se vilka behörigheter du har väljer du hanterings gruppen och väljer sedan **IAM**. Mer information om RBAC-roller finns i  
   [Hantera åtkomst och behörigheter med RBAC](../../role-based-access-control/overview.md).

### <a name="delete-in-the-portal"></a>Ta bort i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Välj den hanterings grupp som du vill ta bort.

1. Välj **information**.

1. Välj **ta bort**

   :::image type="content" source="./media/delete.png" alt-text="Alternativet ta bort grupp" border="false":::

   > [!TIP]
   > Om ikonen är inaktive rad och Hovra med mus väljaren över ikonen visas orsaken.

1. Det finns ett fönster som öppnas som bekräftar att du vill ta bort hanterings gruppen.

   :::image type="content" source="./media/delete_confirm.png" alt-text="Ta bort grupp bekräftelse fönster" border="false":::

1. Välj **Ja**.

### <a name="delete-in-powershell"></a>Ta bort i PowerShell

Använd kommandot **Remove-AzManagementGroup** i PowerShell för att ta bort hanterings grupper.

```azurepowershell-interactive
Remove-AzManagementGroup -GroupName 'Contoso'
```

### <a name="delete-in-azure-cli"></a>Borttagning i Azure CLI

Med Azure CLI använder du kommandot AZ Account Management-Group Delete.

```azurecli-interactive
az account management-group delete --name 'Contoso'
```

## <a name="view-management-groups"></a>Visa hanterings grupper

Du kan visa en hanterings grupp som har en direkt eller ärvd RBAC-roll på.  

### <a name="view-in-the-portal"></a>Visa i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Sidan för hanterings gruppens hierarki kommer att läsas in. På den här sidan kan du utforska alla hanterings grupper och prenumerationer som du har åtkomst till. Om du väljer grupp namnet tar du en nivå i hierarkin. Navigeringen fungerar på samma sätt som i Utforskaren.

1. Om du vill se information om hanterings gruppen väljer du länken **(information)** bredvid rubriken för hanterings gruppen. Om den här länken inte är tillgänglig har du inte behörighet att visa den hanterings gruppen.

   :::image type="content" source="./media/main.png" alt-text="Huvudformulär" border="false":::

### <a name="view-in-powershell"></a>Visa i PowerShell

Du använder kommandot Get-AzManagementGroup för att hämta alla grupper. Se [AZ. Resources](/powershell/module/az.resources/Get-AzManagementGroup) -moduler för den fullständiga listan över hanterings grupper Hämta PowerShell-kommandon.  

```azurepowershell-interactive
Get-AzManagementGroup
```

För en enskild hanterings grupps information använder du parametern-GroupName

```azurepowershell-interactive
Get-AzManagementGroup -GroupName 'Contoso'
```

Om du vill returnera en bestämd hanterings grupp och alla nivåer i hierarkin använder du parametrarna **-Expand** och **-rekursivt** .  

```azurepowershell-interactive
PS C:\> $response = Get-AzManagementGroup -GroupName TestGroupParent -Expand -Recurse
PS C:\> $response

Id                : /providers/Microsoft.Management/managementGroups/TestGroupParent
Type              : /providers/Microsoft.Management/managementGroups
Name              : TestGroupParent
TenantId          : 00000000-0000-0000-0000-000000000000
DisplayName       : TestGroupParent
UpdatedTime       : 2/1/2018 11:15:46 AM
UpdatedBy         : 00000000-0000-0000-0000-000000000000
ParentId          : /providers/Microsoft.Management/managementGroups/00000000-0000-0000-0000-000000000000
ParentName        : 00000000-0000-0000-0000-000000000000
ParentDisplayName : 00000000-0000-0000-0000-000000000000
Children          : {TestGroup1DisplayName, TestGroup2DisplayName}

PS C:\> $response.Children[0]

Type        : /managementGroup
Id          : /providers/Microsoft.Management/managementGroups/TestGroup1
Name        : TestGroup1
DisplayName : TestGroup1DisplayName
Children    : {TestRecurseChild}

PS C:\> $response.Children[0].Children[0]

Type        : /managementGroup
Id          : /providers/Microsoft.Management/managementGroups/TestRecurseChild
Name        : TestRecurseChild
DisplayName : TestRecurseChild
Children    :
```

### <a name="view-in-azure-cli"></a>Visa i Azure CLI

Du kan använda kommandot lista för att hämta alla grupper.  

```azurecli-interactive
az account management-group list
```

Använd kommandot show för en enskild hanterings grupps information

```azurecli-interactive
az account management-group show --name 'Contoso'
```

Om du vill returnera en bestämd hanterings grupp och alla nivåer i hierarkin använder du parametrarna **-Expand** och **-rekursivt** .

```azurecli-interactive
az account management-group show --name 'Contoso' -e -r
```

## <a name="moving-management-groups-and-subscriptions"></a>Flytta hanterings grupper och prenumerationer   

En orsak till att skapa en hanterings grupp är att bunta ihop prenumerationer. Endast hanterings grupper och prenumerationer kan göras underordnade till en annan hanterings grupp. En prenumeration som flyttas till en hanterings grupp ärver alla användares åtkomst och principer från den överordnade hanterings gruppen

När du flyttar en hanterings grupp eller en prenumeration till en underordnad till en annan hanterings grupp måste tre regler utvärderas som sant.

Om du utför flytt åtgärden behöver du: 

- Hanterings gruppens Skriv-och roll tilldelning Skriv behörigheter för den underordnade prenumerationen eller hanterings gruppen.
  - Exempel på inbyggd roll exempel **ägare**
- Skriv åtkomst till hanterings grupp för den överordnade mål hanterings gruppen.
  - Exempel på inbyggda roller: **ägare**, **deltagare**, **hanterings grupp deltagare**
- Skriv åtkomst till hanterings grupp för den befintliga överordnade hanterings gruppen.
  - Exempel på inbyggda roller: **ägare**, **deltagare**, **hanterings grupp deltagare**

**Undantag**: om målet eller den befintliga överordnade hanterings gruppen är rot hanterings gruppen gäller inte behörighets kraven. Eftersom rot hanterings gruppen är standard landnings platsen för alla nya hanterings grupper och prenumerationer behöver du inte behörigheter för den för att flytta ett objekt.

Om ägar rollen för prenumerationen ärvs från den aktuella hanterings gruppen är dina flyttnings mål begränsade. Du kan bara flytta prenumerationen till en annan hanterings grupp där du har ägar rollen. Du kan inte flytta den till en hanterings grupp där du är deltagare eftersom du förlorar prenumerationens ägarskap. Om du är direkt tilldelad till ägar rollen för prenumerationen (ärvs inte från hanterings gruppen) kan du flytta den till en hanterings grupp där du är deltagare.

Om du vill se vilka behörigheter du har i Azure Portal väljer du hanterings gruppen och väljer sedan **IAM**. Mer information om RBAC-roller finns i [Hantera åtkomst och behörigheter med RBAC](../../role-based-access-control/overview.md).

## <a name="move-subscriptions"></a>Flytta prenumerationer 

### <a name="add-an-existing-subscription-to-a-management-group-in-the-portal"></a>Lägga till en befintlig prenumeration i en hanterings grupp i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Välj den hanterings grupp som du planerar att vara överordnad.

1. Välj **Lägg till prenumeration**längst upp på sidan.

1. Välj prenumerationen i listan med rätt ID.

   :::image type="content" source="./media/add_context_sub.png" alt-text="Tillgängliga prenumerationer som ska läggas till i en hanterings grupp" border="false":::

1. Välj "Spara".

### <a name="remove-a-subscription-from-a-management-group-in-the-portal"></a>Ta bort en prenumeration från en hanterings grupp i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Välj den hanterings grupp som du planerar som är den aktuella överordnade.  

1. Välj ellipsen i slutet av raden för prenumerationen i listan som du vill flytta.

   :::image type="content" source="./media/move_small.png" alt-text="Flytta alternativ i en hanterings grupp" border="false":::

1. Välj **Flytta**.

1. På menyn som öppnas väljer du den **överordnade hanterings gruppen**.

   :::image type="content" source="./media/move_small_context.png" alt-text="Flytta fönster för att ändra överordnad grupp" border="false":::

1. Välj **Spara**.

### <a name="move-subscriptions-in-powershell"></a>Flytta prenumerationer i PowerShell

Om du vill flytta en prenumeration i PowerShell använder du kommandot New-AzManagementGroupSubscription.  

```azurepowershell-interactive
New-AzManagementGroupSubscription -GroupName 'Contoso' -SubscriptionId '12345678-1234-1234-1234-123456789012'
```

Om du vill ta bort länken mellan och-prenumerationen och hanterings gruppen använder du kommandot Remove-AzManagementGroupSubscription.

```azurepowershell-interactive
Remove-AzManagementGroupSubscription -GroupName 'Contoso' -SubscriptionId '12345678-1234-1234-1234-123456789012'
```

### <a name="move-subscriptions-in-azure-cli"></a>Flytta prenumerationer i Azure CLI

Om du vill flytta en prenumeration i CLI använder du kommandot Add.

```azurecli-interactive
az account management-group subscription add --name 'Contoso' --subscription '12345678-1234-1234-1234-123456789012'
```

Om du vill ta bort prenumerationen från hanterings gruppen använder du kommandot för att ta bort prenumeration.  

```azurecli-interactive
az account management-group subscription remove --name 'Contoso' --subscription '12345678-1234-1234-1234-123456789012'
```

## <a name="move-management-groups"></a>Flytta hanterings grupper 

### <a name="move-management-groups-in-the-portal"></a>Flytta hanterings grupper i portalen

1. Logga in på [Azure Portal](https://portal.azure.com).

1. Välj **alla tjänst** > **hanterings grupper**.

1. Välj den hanterings grupp som du planerar att vara överordnad.

1. Längst upp på sidan väljer du **Lägg till hanterings grupp**.

1. I menyn som öppnas väljer du om du vill ha en ny eller använda en befintlig hanterings grupp.

   - Om du väljer ny skapas en ny hanterings grupp.
   - Om du väljer en befintlig visas en listruta över alla hanterings grupper som du kan flytta till den här hanterings gruppen.  

   :::image type="content" source="./media/add_context_MG.png" alt-text="Flytta en hanterings grupp till en ny eller befintlig grupp" border="false":::

1. Välj **Spara**.

### <a name="move-management-groups-in-powershell"></a>Flytta hanterings grupper i PowerShell

Använd kommandot Update-AzManagementGroup i PowerShell för att flytta en hanterings grupp under en annan grupp.

```azurepowershell-interactive
$parentGroup = Get-AzManagementGroup -GroupName ContosoIT
Update-AzManagementGroup -GroupName 'Contoso' -ParentId $parentGroup.id
```  

### <a name="move-management-groups-in-azure-cli"></a>Flytta hanterings grupper i Azure CLI

Använd kommandot Update för att flytta en hanterings grupp med Azure CLI.

```azurecli-interactive
az account management-group update --name 'Contoso' --parent ContosoIT
```

## <a name="audit-management-groups-using-activity-logs"></a>Granska hanteringsgrupper med hjälp av aktivitetsloggar

Hanteringsgrupper kan användas i [Azure-aktivitetsloggar](../../azure-monitor/platform/platform-logs-overview.md). Du kan fråga alla händelser som sker i en hanterings grupp på samma centrala plats som andra Azure-resurser. Du kan till exempel se alla ändringar för rolltilldelningar eller principtilldelningar som gjorts i en viss hanteringsgrupp.

:::image type="content" source="./media/al-mg.png" alt-text="Aktivitets loggar med hanterings grupper" border="false":::

När du vill fråga hanteringsgrupper utanför Microsoft Azure-portalen är målområdet för hanteringsgrupper: **"/providers/Microsoft.Management/managementGroups/{yourMgID}"**.

## <a name="referencing-management-groups-from-other-resource-providers"></a>Referera till hanterings grupper från andra resurs leverantörer

När du refererar till hanterings grupper från andra resurs leverantörs åtgärder använder du följande sökväg som definitions område. Den här sökvägen används när du använder PowerShell-, Azure CLI-och REST-API: er.  

`/providers/Microsoft.Management/managementGroups/{yourMgID}`

Ett exempel på hur du använder den här sökvägen är när du tilldelar en ny roll tilldelning till en hanterings grupp i PowerShell:

```azurepowershell-interactive
New-AzRoleAssignment -Scope "/providers/Microsoft.Management/managementGroups/Contoso"
```

Samma sökväg för omfånget används när en princip definition hämtas i en hanterings grupp.

```http
GET https://management.azure.com/providers/Microsoft.Management/managementgroups/MyManagementGroup/providers/Microsoft.Authorization/policyDefinitions/ResourceNaming?api-version=2018-05-01
```

## <a name="next-steps"></a>Nästa steg

Läs mer om hanteringslösningar här:

- [Skapa hanteringsgrupper för att organisera Azure-resurser](./create.md)
- [Så här ändrar, raderar och hanterar du dina hanteringsgrupper](./manage.md)
- [Granska hanteringsgrupper i Azure PowerShell-resursmodulen](/powershell/module/az.resources#resources)
- [Granska hanteringsgrupper i REST API](/rest/api/resources/managementgroups)
- [Granska hanteringsgrupper i Azure CLI](/cli/azure/account/management-group)
