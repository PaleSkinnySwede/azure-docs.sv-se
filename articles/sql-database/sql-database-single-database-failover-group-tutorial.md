---
title: 'Självstudie: lägga till en enskild databas i en failover-grupp'
description: Lägg till en Azure SQL Database enkel databas i en failover-grupp med hjälp av Azure Portal, PowerShell eller Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: sstein, carlrab
ms.date: 06/19/2019
ms.openlocfilehash: c5ce6a1c2f231d372a2a8113eb9043a236090388
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80061697"
---
# <a name="tutorial-add-an-azure-sql-database-single-database-to-a-failover-group"></a>Självstudie: lägga till en Azure SQL Database enkel databas i en failover-grupp

En [failover-grupp](sql-database-auto-failover-group.md) är ett deklarativ abstraktions lager som gör att du kan gruppera flera-geo-replikerade databaser. Lär dig hur du konfigurerar en failover-grupp för en Azure SQL Database enkel databas och redundanstest med antingen Azure Portal, PowerShell eller Azure CLI.  I den här självstudien får du lära dig hur man:

> [!div class="checklist"]
> - Skapa en Azure SQL Database enskild databas.
> - Skapa en failover-grupp för en enskild databas mellan två logiska SQL-servrar.
> - Redundanstest.

## <a name="prerequisites"></a>Krav

# <a name="portal"></a>[Portalen](#tab/azure-portal)
För att kunna följa den här självstudien måste du ha: 

- En Azure-prenumeration. [Skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) om du inte redan har ett.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Kontrol lera att du har följande objekt för att slutföra självstudien:

- En Azure-prenumeration. [Skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) om du inte redan har ett.
- [Azure PowerShell](/powershell/azureps-cmdlets-docs)


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Kontrol lera att du har följande objekt för att slutföra självstudien:

- En Azure-prenumeration. [Skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) om du inte redan har ett.
- Den senaste versionen av [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest). 

---

## <a name="1---create-a-single-database"></a>1 – Skapa en enskild databas 

[!INCLUDE [sql-database-create-single-database](includes/sql-database-create-single-database.md)]

## <a name="2---create-the-failover-group"></a>2 – Skapa redundans gruppen 
I det här steget ska du skapa en [redundans grupp](sql-database-auto-failover-group.md) mellan en befintlig Azure SQL-Server och en ny Azure SQL-Server i en annan region. Lägg sedan till exempel databasen i gruppen för växling vid fel. 

# <a name="portal"></a>[Portalen](#tab/azure-portal)
Skapa din grupp för redundans och Lägg till din enda databas i den med hjälp av Azure Portal. 

1. Välj **Azure SQL** i den vänstra menyn i [Azure Portal](https://portal.azure.com). Om **Azure SQL** inte finns i listan väljer du **alla tjänster**och skriver sedan Azure SQL i sökrutan. Valfritt Välj stjärnan bredvid **Azure SQL** för att Favorita den och lägga till den som ett objekt i navigeringen till vänster. 
1. Välj den enkla databasen som `mySampleDatabase`skapades i avsnitt 1, till exempel. 
1. Redundansväxlingen kan konfigureras på server nivå. Välj namnet på servern under **Server namn** för att öppna inställningarna för servern.

   ![Öppna Server för enskild databas](media/sql-database-single-database-failover-group-tutorial/open-sql-db-server.png)

1. Välj **grupper för växling vid fel** i fönstret **Inställningar** och välj sedan **Lägg till grupp** för att skapa en ny grupp för redundans. 

    ![Lägg till ny redundans grupp](media/sql-database-single-database-failover-group-tutorial/sqldb-add-new-failover-group.png)

1. På sidan **redundans** anger eller väljer du följande värden och väljer sedan **skapa**:
    - **Namn på redundans grupp**: Ange ett unikt namn på redundans grupp, till `failovergrouptutorial`exempel. 
    - **Sekundär server**: Välj alternativet för att *Konfigurera nödvändiga inställningar* och välj sedan att **skapa en ny server**. Alternativt kan du välja en redan befintlig server som den sekundära servern. När du har angett följande värden väljer du **Välj**. 
        - **Server namn**: Ange ett unikt namn på den sekundära servern, till exempel `mysqlsecondary`. 
        - **Inloggning för Server administratör**: typ`azureuser`
        - **Lösen ord**: Ange ett komplext lösen ord som uppfyller lösen ords kraven.
        - **Plats**: Välj en plats i list rutan, till exempel `East US`. Den här platsen kan inte vara samma plats som den primära servern.

    > [!NOTE]
    > Inställningarna för Server inloggning och brand vägg måste matcha den primära servern. 
    
      ![Skapa en sekundär server för redundans gruppen](media/sql-database-single-database-failover-group-tutorial/create-secondary-failover-server.png)

   - **Databaser i gruppen**: när du har valt en sekundär server blir det här alternativet olåst. Välj den för att **välja databaser som ska läggas till** och välj sedan den databas som du skapade i avsnitt 1. Om du lägger till databasen i gruppen för växling vid fel startas automatiskt processen för geo-replikering. 
        
    ![Lägg till SQL DB i redundans gruppen](media/sql-database-single-database-failover-group-tutorial/add-sqldb-to-failover-group.png)
        

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Skapa din grupp för redundans och Lägg till din databas i den med hjälp av PowerShell. 

   > [!NOTE]
   > Inställningarna för Server inloggning och brand vägg måste matcha den primära servern. 

   ```powershell-interactive
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "West US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $databaseName = "mySampleDatabase"
   $drLocation = "East US"
   $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # The ip address range that you want to allow to access your server 
   # (leaving at 0.0.0.0 will prevent outside-of-azure connections to your DB)
   $startIp = "0.0.0.0"
   $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "DR Server name is" $drServerName 
   Write-host "Failover group name is" $failoverGroupName
   
   # Create a secondary server in the failover region
   Write-host "Creating a secondary logical server in the failover region..."
   $drServer = New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $drServer

   # Create a server firewall rule that allows access from the specified IP range
   Write-host "Configuring firewall for secondary logical server..."
   $serverFirewallRule = New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   $serverFirewallRule   
   
   # Create a failover group between the servers
   $failovergroup = Write-host "Creating a failover group between the primary and secondary server..."
   New-AzSqlDatabaseFailoverGroup `
      –ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -PartnerServerName $drServerName  `
      –FailoverGroupName $failoverGroupName `
      –FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   $failovergroup
   
   # Add the database to the failover group
   Write-host "Adding the database to the failover group..." 
   Get-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName | `
   Add-AzSqlDatabaseToFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Successfully added the database to the failover group..." 
   ```

I den här delen av självstudien används följande PowerShell-cmdletar:

| Kommando | Obs! |
|---|---|
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Skapar en SQL Database-server som är värd för enkla databaser och elastiska pooler. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Skapar en brand Väggs regel för en logisk server. | 
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Skapar en ny Azure SQL Database enskild databas. | 
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Skapar en ny grupp för redundans. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Hämtar en eller flera SQL-databaser. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Lägger till en eller flera Azure SQL-databaser i en failover-grupp. |

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Skapa din grupp för redundans och Lägg till din enda databas i den med hjälp av AZ CLI. 

   > [!NOTE]
   > Inställningarna för Server inloggning och brand vägg måste matcha den primära servern. 

   ```azurecli-interactive
   #!/bin/bash
   # set variables
   $failoverLocation = "West US"
   $failoverServer = "failoverServer-$randomIdentifier"
   $failoverGroup = "failoverGroup-$randomIdentifier"

   echo "Creating a secondary logical server in the DR region..."
   az sql server create --name $failoverServer --resource-group $resourceGroup --location $failoverLocation --admin-user $login --admin-password $password
   
   echo "Creating a failover group between the two servers..."
   az sql failover-group create --name $failoverGroup --partner-server $failoverServer --resource-group $resourceGroup --server $server --add-db $database --failover-policy Automatic
   ```

Den här delen av självstudien använder följande AZ CLI-cmdletar:

| Kommando | Obs! |
|---|---|
| [az sql server create](/cli/azure/sql/server#az-sql-server-create) | Skapar en SQL Database-server som är värd för enkla databaser och elastiska pooler. |
| [AZ SQL Server-brandvägg-regel skapa](/cli/azure/sql/server/firewall-rule) | Skapar en servers brand Väggs regler. | 
| [AZ SQL-redundans – grupp skapa](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-create) | Skapar en grupp för redundans. | 

---

## <a name="3---test-failover"></a>3-testa redundans 
I det här steget kommer du inte att kunna redundansväxla gruppen till den sekundära servern och sedan växla tillbaka med Azure Portal. 

# <a name="portal"></a>[Portalen](#tab/azure-portal)
Testa redundans med Azure Portal. 

1. Välj **Azure SQL** i den vänstra menyn i [Azure Portal](https://portal.azure.com). Om **Azure SQL** inte finns i listan väljer du **alla tjänster**och skriver sedan Azure SQL i sökrutan. Valfritt Välj stjärnan bredvid **Azure SQL** för att Favorita den och lägga till den som ett objekt i navigeringen till vänster. 
1. Välj den enkla databasen som `mySampleDatbase`skapades i avsnitt 2, till exempel. 
1. Välj namnet på servern under **Server namn** för att öppna inställningarna för servern.

   ![Öppna Server för enskild databas](media/sql-database-single-database-failover-group-tutorial/open-sql-db-server.png)

1. Välj **grupper för växling vid fel** i fönstret **Inställningar** och välj sedan den grupp för växling vid fel som du skapade i avsnitt 2. 
  
   ![Välj gruppen redundans från portalen](media/sql-database-single-database-failover-group-tutorial/select-failover-group.png)

1. Granska vilken server som är primär och vilken server som är sekundär. 
1. Välj **redundans** från åtgärds fönstret för att Redundansväxla din redundanskonfiguration som innehåller din exempel enkla databas. 
1. Välj **Ja** i varningen som meddelar dig att TDS-sessioner kommer att kopplas bort. 

   ![Redundansväxla din failover-grupp som innehåller din SQL-databas](media/sql-database-single-database-failover-group-tutorial/failover-sql-db.png)

1. Granska den server som nu är primär och vilken server som är sekundär. Om redundans har slutförts ska de två servrarna ha växlade roller. 
1. Välj **redundans** igen för att återställa servrarna till deras ursprungliga roller. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Testa redundans med PowerShell. 


Kontrol lera den sekundära replikens roll: 

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"
   
   # Check role of secondary replica
   Write-host "Confirming the secondary replica is secondary...." 
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole
   ```

Redundansväxla till den sekundära servern: 

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"
   
   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..." 
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group successfully to" $drServerName 
   ```

Återställ redundansväxlingen tillbaka till den primära servern:

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $serverName = "mysqlserver-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"
   
   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...." 
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group successfully back to" $serverName
   ```

I den här delen av självstudien används följande PowerShell-cmdletar:

| Kommando | Obs! |
|---|---|
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Hämtar eller listar Azure SQL Database redundans grupper. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Kör en redundansväxling av en Azure SQL Database redundans grupp. |



# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Testa redundans med AZ CLI. 

Verifiera vilken server som är sekundär:

   
   ```azurecli-interactive
   echo "Verifying which server is in the secondary role..."
   az sql failover-group list --server $server --resource-group $resourceGroup
   ```

Redundansväxla till den sekundära servern: 

   ```azurecli-interactive
   echo "Failing over group to the secondary server..."
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $failoverServer
   echo "Successfully failed failover group over to" $failoverServer
   ```

Återställ redundansväxlingen tillbaka till den primära servern:

   ```azurecli-interactive
   echo "Failing over group back to the primary server..."
   az sql failover-group set-primary --name $failoverGroup --resource-group $resourceGroup --server $server
   echo "Successfully failed failover group back to" $server
   ```

Den här delen av självstudien använder följande AZ CLI-cmdletar:

| Kommando | Obs! |
|---|---|
| [AZ SQL-redundans – grupp lista](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-list) | Visar en lista över failover-grupper på en server. |
| [AZ SQL-redundans – grupp uppsättning – primär](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-set-primary) | Ange den primära gruppen för redundans genom att redundansväxla alla databaser från den aktuella primära servern. | 

---

## <a name="clean-up-resources"></a>Rensa resurser 
Rensa resurser genom att ta bort resurs gruppen. 

# <a name="portal"></a>[Portalen](#tab/azure-portal)
Ta bort resurs gruppen med hjälp av Azure Portal. 

1. Navigera till din resurs grupp i [Azure Portal](https://portal.azure.com).
1. Välj **ta bort resurs grupp** för att ta bort alla resurser i gruppen, samt själva resurs gruppen. 
1. Skriv namnet på resurs gruppen `myResourceGroup`, i text rutan och välj sedan **ta bort** för att ta bort resurs gruppen.  

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Ta bort resurs gruppen med hjälp av PowerShell. 

   ```powershell-interactive
   # Set variables
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"

   # Remove the resource group
   Write-host "Removing resource group..."
   Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
   Write-host "Resource group removed =" $resourceGroupName
   ```

I den här delen av självstudien används följande PowerShell-cmdletar:

| Kommando | Obs! |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Tar bort en resurs grupp | 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Ta bort resurs gruppen med AZ CLI. 


   ```azurecli-interactive
   echo "Cleaning up resources by removing the resource group..."
   az group delete --name $resourceGroup
   echo "Successfully removed resource group" $resourceGroup
   ```

Den här delen av självstudien använder följande AZ CLI-cmdletar:

| Kommando | Obs! |
|---|---|
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | Tar bort en resursgrupp, inklusive alla kapslade resurser. |

---


> [!IMPORTANT]
> Om du vill behålla resurs gruppen men ta bort den sekundära databasen tar du bort den från gruppen växling vid fel innan du tar bort den. Om du tar bort en sekundär databas innan den tas bort från redundans gruppen kan det orsaka oförutsägbart beteende. 


## <a name="full-scripts"></a>Fullständiga skript

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!code-powershell-interactive[main](../../powershell_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-ps.ps1 "Add single database to a failover group")]

Det här skriptet använder följande kommandon. Varje kommando i tabellen länkar till kommandospecifik dokumentation.

| Kommando | Obs! |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Skapar en resursgrupp där alla resurser lagras. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Skapar en SQL Database-server som är värd för enkla databaser och elastiska pooler. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Skapar en brand Väggs regel för en logisk server. | 
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Skapar en ny Azure SQL Database enskild databas. | 
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Skapar en ny grupp för redundans. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Hämtar en eller flera SQL-databaser. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Lägger till en eller flera Azure SQL-databaser i en failover-grupp. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Hämtar eller listar Azure SQL Database redundans grupper. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Kör en redundansväxling av en Azure SQL Database redundans grupp. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Tar bort en resurs grupp | 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!code-azurecli-interactive[main](../../cli_scripts/sql-database/failover-groups/add-single-db-to-failover-group-az-cli.sh "Add single database to a failover group")]

Det här skriptet använder följande kommandon. Varje kommando i tabellen länkar till kommandospecifik dokumentation.

| Kommando | Obs! |
|---|---|
| [AZ-konto uppsättning](/cli/azure/account?view=azure-cli-latest#az-account-set) | Anger att en prenumeration är den aktuella aktiva prenumerationen. | 
| [az group create](/cli/azure/group#az-group-create) | Skapar en resursgrupp där alla resurser lagras. |
| [az sql server create](/cli/azure/sql/server#az-sql-server-create) | Skapar en SQL Database-server som är värd för enkla databaser och elastiska pooler. |
| [AZ SQL Server-brandvägg-regel skapa](/cli/azure/sql/server/firewall-rule) | Skapar en servers brand Väggs regler. | 
| [az sql db create](/cli/azure/sql/db?view=azure-cli-latest) | Skapar en databas. | 
| [AZ SQL-redundans – grupp skapa](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-create) | Skapar en grupp för redundans. | 
| [AZ SQL-redundans – grupp lista](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-list) | Visar en lista över failover-grupper på en server. |
| [AZ SQL-redundans – grupp uppsättning – primär](/cli/azure/sql/failover-group?view=azure-cli-latest#az-sql-failover-group-set-primary) | Ange den primära gruppen för redundans genom att redundansväxla alla databaser från den aktuella primära servern. | 
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#az-vm-extension-set) | Tar bort en resursgrupp, inklusive alla kapslade resurser. |

# <a name="portal"></a>[Portalen](#tab/azure-portal)
Det finns inga tillgängliga skript för Azure Portal. 
 
---

Du kan hitta andra Azure SQL Database skript här: [Azure PowerShell](sql-database-powershell-samples.md) och [Azure CLI](sql-database-cli-samples.md). 

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du lagt till en Azure SQL Database enkel databas i en failover-grupp och testat redundansväxlingen. Du har lärt dig att: 

> [!div class="checklist"]
> - Skapa en Azure SQL Database enskild databas. 
> - Skapa en [failover-grupp](sql-database-auto-failover-group.md) för en enskild databas mellan två logiska SQL-servrar.
> - Redundanstest.

Gå vidare till nästa självstudie om hur du lägger till en elastisk pool i en failover-grupp. 

> [!div class="nextstepaction"]
> [Självstudie: lägga till en Azure SQL Database elastisk pool i en failover-grupp](sql-database-elastic-pool-failover-group-tutorial.md)
