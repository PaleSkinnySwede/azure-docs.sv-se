---
title: Konfigurera Kundhanterade nycklar för ditt Azure Cosmos DB-konto
description: Lär dig hur du konfigurerar Kundhanterade nycklar för ditt Azure Cosmos DB-konto med Azure Key Vault
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 03/19/2020
ms.author: thweiss
ROBOTS: noindex, nofollow
ms.openlocfilehash: 8f58887a056c8ca0cd175a44127556562338de38
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81450040"
---
# <a name="configure-customer-managed-keys-for-your-azure-cosmos-account-with-azure-key-vault"></a>Konfigurera Kundhanterade nycklar för ditt Azure Cosmos-konto med Azure Key Vault

> [!NOTE]
> För tillfället måste du begära åtkomst för att använda den här funktionen. Om du vill göra det kontaktar [azurecosmosdbcmk@service.microsoft.com](mailto:azurecosmosdbcmk@service.microsoft.com)du.

Data som lagras i ditt Azure Cosmos-konto är automatiskt och sömlöst krypterade med nycklar som hanteras av Microsoft (**tjänst hanterade nycklar**). Du kan också välja att lägga till ett andra lager av kryptering med nycklar som du hanterar (**Kundhanterade nycklar**).

![Lager för kryptering runt kund information](./media/how-to-setup-cmk/cmk-intro.png)

Du måste lagra Kundhanterade nycklar i [Azure Key Vault](../key-vault/general/overview.md) och tillhandahålla en nyckel för varje Azure Cosmos-konto som är aktiverat med Kundhanterade nycklar. Den här nyckeln används för att kryptera alla data som lagras i kontot.

> [!NOTE]
> För närvarande är Kundhanterade nycklar endast tillgängliga för nya Azure Cosmos-konton. Du bör konfigurera dem när kontot skapas.

## <a name="register-the-azure-cosmos-db-resource-provider-for-your-azure-subscription"></a><a id="register-resource-provider"></a>Registrera Azure Cosmos DB Resource Provider för din Azure-prenumeration

1. Logga in på [Azure Portal](https://portal.azure.com/), gå till din Azure-prenumeration och välj **resurs leverantörer** på fliken **Inställningar** :

   !["Resource providers"-post från den vänstra menyn](./media/how-to-setup-cmk/portal-rp.png)

1. Sök efter **Microsoft. DocumentDB** -resurs leverantören. Kontrol lera att resurs leverantören redan har marker ATS som registrerad. Annars väljer du resurs leverantören och väljer **Registrera**:

   ![Registrera Microsoft. DocumentDB Resource Provider](./media/how-to-setup-cmk/portal-rp-register.png)

## <a name="configure-your-azure-key-vault-instance"></a>Konfigurera Azure Key Vault-instansen

Om du använder Kundhanterade nycklar med Azure Cosmos DB måste du ange två egenskaper för den Azure Key Vault-instans som du planerar att använda som värd för dina krypterings nycklar. Dessa egenskaper inkluderar **mjuk borttagning** och **Rensa inte**. Dessa egenskaper är inte aktiverade som standard. Du kan aktivera dem med hjälp av antingen PowerShell eller Azure CLI.

Information om hur du aktiverar dessa egenskaper på en befintlig Azure Key Vault instans finns i avsnitten "Aktivera mjuk borttagning" och "Aktivera rensnings skydd" i någon av följande artiklar:

- [Använda mjuk borttagning med PowerShell](../key-vault/general/soft-delete-powershell.md)
- [Använda mjuk borttagning med Azure CLI](../key-vault/general/soft-delete-cli.md)

## <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Lägg till en åtkomst princip till Azure Key Vault-instansen

1. Från Azure Portal går du till den Azure Key Vault instans som du planerar att använda som värd för dina krypterings nycklar. Välj **åtkomst principer** på den vänstra menyn:

   !["Åtkomst principer" på den vänstra menyn](./media/how-to-setup-cmk/portal-akv-ap.png)

1. Välj **+ Lägg till åtkomst princip**.

1. Under List rutan **nyckel behörigheter** väljer du **Hämta**, **packa upp nyckel**och packa upp **nyckel** behörigheter:

   ![Välja rätt behörigheter](./media/how-to-setup-cmk/portal-akv-add-ap-perm2.png)

1. Under **Välj huvud konto**väljer du **ingen vald**. Sök sedan efter **Azure Cosmos DB** huvud konto och välj det (för att göra det lättare att hitta, kan du också söka efter huvud- `a232010e-820c-4083-83bb-3ace5fc29d0b` ID: för alla Azure-regioner utom Azure Government regioner där ägar `57506a73-e302-42a9-b869-6f12d9ec29e9`-ID är). Slutligen väljer du **Välj** längst ned. Om **Azure Cosmos DB** -huvudobjektet inte finns i listan kan du behöva registrera om **Microsoft. DocumentDB** Resource Provider enligt beskrivningen i avsnittet [Registrera resurs leverantören](#register-resource-provider) i den här artikeln.

   ![Välj Azure Cosmos DB huvud konto](./media/how-to-setup-cmk/portal-akv-add-ap.png)

1. Välj **Lägg** till för att lägga till den nya åtkomst principen.

## <a name="generate-a-key-in-azure-key-vault"></a>Generera en nyckel i Azure Key Vault

1. Från Azure Portal, gå till den Azure Key Vault instans som du planerar att använda som värd för dina krypterings nycklar. Välj sedan **nycklar** i den vänstra menyn:

   !["Nycklar"-post från den vänstra menyn](./media/how-to-setup-cmk/portal-akv-keys.png)

1. Välj **generera/importera**, ange ett namn för den nya nyckeln och välj en nyckel storlek för RSA. Minst 3072 rekommenderas för bästa säkerhet. Välj sedan **skapa**:

   ![Skapa en ny nyckel](./media/how-to-setup-cmk/portal-akv-gen.png)

1. När nyckeln har skapats väljer du den nya nyckeln och sedan dess aktuella version.

1. Kopiera nyckelns **nyckel identifierare**, förutom delen efter det sista snedstrecket:

   ![Nyckelns nyckel identifierare kopieras](./media/how-to-setup-cmk/portal-akv-keyid.png)

## <a name="create-a-new-azure-cosmos-account"></a>Skapa ett nytt Azure Cosmos-konto

### <a name="using-the-azure-portal"></a>Använda Azure Portal

När du skapar ett nytt Azure Cosmos DB konto från Azure Portal väljer du **kundhanterad nyckel** i **krypterings** steget. I fältet **nyckel-URI** klistrar du in URI/nyckel-ID för den Azure Key Vault nyckel som du kopierade från föregående steg:

![Ange CMK-parametrar i Azure Portal](./media/how-to-setup-cmk/portal-cosmos-enc.png)

### <a name="using-azure-powershell"></a>Använda Azure PowerShell

När du skapar ett nytt Azure Cosmos DB-konto med PowerShell:

- Skicka URI: n för den Azure Key Vault nyckel som kopierats tidigare under egenskapen **keyVaultKeyUri** i **PropertyObject**.

- Använd **2019-12-12** som API-version.

> [!IMPORTANT]
> Du måste ange `Location` parametern explicit för att kontot ska kunna skapas med Kundhanterade nycklar.

```powershell
$resourceGroupName = "myResourceGroup"
$accountLocation = "West US 2"
$accountName = "mycosmosaccount"

$failoverLocations = @(
    @{ "locationName"="West US 2"; "failoverPriority"=0 }
)

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$failoverLocations;
    "keyVaultKeyUri" = "https://<my-vault>.vault.azure.net/keys/<my-key>";
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2019-12-12" -ResourceGroupName $resourceGroupName `
    -Location $accountLocation -Name $accountName -PropertyObject $CosmosDBProperties
```

### <a name="using-an-azure-resource-manager-template"></a>Använda en Azure Resource Manager mall

När du skapar ett nytt Azure Cosmos-konto via en Azure Resource Manager-mall:

- Skicka URI: n för den Azure Key Vault nyckel som du kopierade tidigare under egenskapen **keyVaultKeyUri** i objektet **Properties** .

- Använd **2019-12-12** som API-version.

> [!IMPORTANT]
> Du måste ange `Location` parametern explicit för att kontot ska kunna skapas med Kundhanterade nycklar.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "keyVaultKeyUri": {
            "type": "string"
        }
    },
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[parameters('accountName')]",
            "apiVersion": "2019-12-12",
            "kind": "GlobalDocumentDB",
            "location": "[parameters('location')]",
            "properties": {
                "locations": [ 
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    }
                ],
                "databaseAccountOfferType": "Standard",
                "keyVaultKeyUri": "[parameters('keyVaultKeyUri')]"
            }
        }
    ]
}

```

Distribuera mallen med följande PowerShell-skript:

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

### <a name="using-the-azure-cli"></a>Använda Azure CLI

När du skapar ett nytt Azure Cosmos-konto via Azure CLI skickar du URI: n för den Azure Key Vault nyckel som du kopierade tidigare under parametern **--Key-URI** .

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
keyVaultKeyUri = 'https://<my-vault>.vault.azure.net/keys/<my-key>'

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --key-uri $keyVaultKeyUri
```

## <a name="frequently-asked-questions"></a>Vanliga frågor och svar

### <a name="is-there-any-additional-charge-for-using-customer-managed-keys"></a>Finns det ytterligare avgifter för att använda Kundhanterade nycklar?

Ja. För att ta hänsyn till den ytterligare beräknings belastning som krävs för att hantera data kryptering och dekryptering med Kundhanterade nycklar, förbrukar sig alla åtgärder som utförs mot Azure Cosmos-kontot en ökning på 25 procent i [enheter för programbegäran](./request-units.md).

### <a name="what-data-gets-encrypted-with-the-customer-managed-keys"></a>Vilka data krypteras med kundens Kundhanterade nycklar?

Alla data som lagras i ditt Azure Cosmos-konto krypteras med kundens hanterade nycklar, förutom följande metadata:

- Namnen på dina Azure Cosmos DB [-konton, databaser och behållare](./account-overview.md#elements-in-an-azure-cosmos-account)

- Namnen på dina [lagrade procedurer](./stored-procedures-triggers-udfs.md)

- Egenskaps Sök vägarna som deklareras i dina [indexerings principer](./index-policy.md)

- Värdena för dina behållar [partitionerings nycklar](./partitioning-overview.md)

### <a name="are-customer-managed-keys-supported-for-existing-azure-cosmos-accounts"></a>Stöds kund hanterade nycklar för befintliga Azure Cosmos-konton?

Den här funktionen är för närvarande endast tillgänglig för nya konton.

### <a name="is-there-a-plan-to-support-finer-granularity-than-account-level-keys"></a>Finns det någon plan för att ge bättre granularitet än nycklar på konto nivå?

Inte för närvarande, men nycklar för container nivå beaktas.

### <a name="how-do-customer-managed-keys-affect-a-backup"></a>Hur påverkar kund hanterade nycklar en säkerhets kopia?

Azure Cosmos DB använder [regelbundna och automatiska säkerhets kopieringar](./online-backup-and-restore.md) av de data som lagras i ditt konto. Den här åtgärden säkerhetskopierar krypterade data. Om du vill använda den återställda säkerhets kopian krävs den krypterings nyckel som du använde vid tidpunkten för säkerhets kopieringen. Det innebär att ingen åter kallelse görs och den version av nyckeln som användes vid tidpunkten för säkerhets kopieringen fortfarande kommer att aktive ras.

### <a name="how-do-i-revoke-an-encryption-key"></a>Hur gör jag för att återkalla en krypterings nyckel?

Nyckel återkallning görs genom att den senaste versionen av nyckeln inaktive ras:

![Inaktivera en nyckels version](./media/how-to-setup-cmk/portal-akv-rev2.png)

Om du vill återkalla alla nycklar från en Azure Key Vault-instans kan du också ta bort åtkomst principen som beviljats till Azure Cosmos DB-huvudobjektet:

![Tar bort åtkomst principen för Azure Cosmos DB-huvudobjektet](./media/how-to-setup-cmk/portal-akv-rev.png)

### <a name="what-operations-are-available-after-a-customer-managed-key-is-revoked"></a>Vilka åtgärder är tillgängliga när en kundhanterad nyckel har återkallats?

Den enda åtgärd som är möjlig när krypterings nyckeln har återkallats är borttagning av kontot.

## <a name="next-steps"></a>Nästa steg

- Läs mer om [data kryptering i Azure Cosmos DB](./database-encryption-at-rest.md).
- Få en översikt över [säker åtkomst till data i Cosmos DB](secure-access-to-data.md).
