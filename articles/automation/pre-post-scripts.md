---
title: Hantera för skript och efter skript i din Uppdateringshantering distribution i Azure
description: Den här artikeln beskriver hur du konfigurerar och hanterar för-skript och post-skript för uppdaterings distributioner.
services: automation
ms.subservice: update-management
ms.date: 05/17/2019
ms.topic: conceptual
ms.openlocfilehash: f55ebb3270fdd97a1fdbbf5a56f9703c08933f9f
ms.sourcegitcommit: c535228f0b77eb7592697556b23c4e436ec29f96
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82855340"
---
# <a name="manage-pre-scripts-and-post-scripts"></a>Hantera för skript och efter skript

För skript och post-skript är Runbooks som ska köras i ditt Azure Automation konto före (före aktivitet) och efter (efter aktivitet) en uppdaterings distribution. För skript och efter skript körs i Azure-kontexten, inte lokalt. För skript körs i början av uppdaterings distributionen. Efter att skripten körs i slutet av distributionen och efter omstarter som har kon figurer ATS.

>[!NOTE]
>Den här artikeln har uppdaterats till att använda den nya Azure PowerShell Az-modulen. Du kan fortfarande använda modulen AzureRM som kommer att fortsätta att ta emot felkorrigeringar fram till december 2020 eller längre. Mer information om den nya Az-modulen och AzureRM-kompatibilitet finns i [Introduktion till den nya Azure PowerShell Az-modulen](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0). Installations anvisningar för AZ-modulen på Hybrid Runbook Worker finns i [installera Azure PowerShell-modulen](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0). För ditt Automation-konto kan du uppdatera dina moduler till den senaste versionen med hjälp av [hur du uppdaterar Azure PowerShell moduler i Azure Automation](automation-update-azure-modules.md).

## <a name="pre-script-and-post-script-requirements"></a>Krav för skript och efter skript

För att en Runbook ska kunna användas som ett för skript eller efter skript måste du importera den till ditt Automation-konto och [publicera runbooken](manage-runbooks.md#publish-a-runbook).

## <a name="pre-script-and-post-script-parameters"></a>Parametrar för skript och efter skript

När du konfigurerar för skript och efter skript kan du skicka in parametrar precis som schemaläggning av en Runbook. Parametrarna definieras vid tidpunkten för skapandet av uppdaterings distributionen. För skript och post-scripts stöder följande typer:

* hängande
* stor
* int
* som
* decimal
* samma
* Dubbelklicka
* DateTime
* nollängd

Runbook-parametrarna för skript och efter skript stöder inte booleska, objekt eller mat ris typer. Dessa värden gör att Runbooks inte fungerar. 

Om du behöver en annan objekt typ kan du omvandla den till en annan typ med din egen logik i runbooken.

Förutom dina standard parametrar för Runbook anges `SoftwareUpdateConfigurationRunContext` parametern (typ JSON-sträng). Om du definierar parametern i skriptet för för skript eller efter skript skickas den automatiskt av uppdaterings distributionen. Parametern innehåller information om uppdaterings distributionen, som är en del av informationen som returneras av [SoftwareUpdateconfigurations-API: et](/rest/api/automation/softwareupdateconfigurations/getbyname#updateconfiguration). I avsnitten nedan definieras de associerade egenskaperna.

### <a name="softwareupdateconfigurationruncontext-properties"></a>Egenskaper för SoftwareUpdateConfigurationRunContext

|Egenskap  |Beskrivning  |
|---------|---------|
|SoftwareUpdateConfigurationName     | Namnet på program uppdaterings konfigurationen.        |
|SoftwareUpdateConfigurationRunId     | Unikt ID för körningen.        |
|SoftwareUpdateConfigurationSettings     | En samling egenskaper relaterade till program uppdaterings konfigurationen.         |
|SoftwareUpdateConfigurationSettings. Opera ting system     | De operativ system som är avsedda för uppdaterings distributionen.         |
|SoftwareUpdateConfigurationSettings. duration     | Maximal varaktighet för uppdaterings distributionen som körs enligt `PT[n]H[n]M[n]S` vad som är tillåtet per iso8601; kallas även underhålls period.          |
|SoftwareUpdateConfigurationSettings. Windows     | En samling egenskaper som är relaterade till Windows-datorer.         |
|SoftwareUpdateConfigurationSettings. Windows. excludedKbNumbers     | En lista över KB som är undantagna från uppdaterings distributionen.        |
|SoftwareUpdateConfigurationSettings. Windows. includedUpdateClassifications     | Uppdaterings klassificeringar som valts för uppdaterings distributionen.        |
|SoftwareUpdateConfigurationSettings. Windows. rebootSetting     | Starta om inställningarna för uppdaterings distributionen.        |
|azureVirtualMachines     | En lista över resourceIds för virtuella Azure-datorer i uppdaterings distributionen.        |
|nonAzureComputerNames|En lista över FQDN för icke-Azure-datorer i uppdaterings distributionen.|

Följande exempel är en JSON-sträng som skickas till parametern **SoftwareUpdateConfigurationRunContext** :

```json
"SoftwareUpdateConfigurationRunContext":{
      "SoftwareUpdateConfigurationName":"sampleConfiguration",
      "SoftwareUpdateConfigurationRunId":"00000000-0000-0000-0000-000000000000",
      "SoftwareUpdateConfigurationSettings":{
         "operatingSystem":"Windows",
         "duration":"PT2H0M",
         "windows":{
            "excludedKbNumbers":[
               "168934",
               "168973"
            ],
            "includedUpdateClassifications":"Critical",
            "rebootSetting":"IfRequired"
         },
         "azureVirtualMachines":[
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-01",
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-02",
            "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresources/providers/Microsoft.Compute/virtualMachines/vm-03"
         ],
         "nonAzureComputerNames":[
            "box1.contoso.com",
            "box2.contoso.com"
         ]
      }
   }
```

Du hittar ett fullständigt exempel med alla egenskaper på: [Hämta program uppdaterings konfiguration efter namn](/rest/api/automation/softwareupdateconfigurations/getbyname#examples).

> [!NOTE]
> `SoftwareUpdateConfigurationRunContext` Objektet kan innehålla dubbla poster för datorer. Detta kan orsaka att för skript och efter skript körs flera gånger på samma dator. Undvik problemet genom att använda `Sort-Object -Unique` för att välja enbart unika VM-namn.

## <a name="using-a-pre-script-or-post-script-in-a-deployment"></a>Använda ett för skript eller efter skript i en distribution

Om du vill använda ett för skript eller efter skript i en uppdaterings distribution börjar du med att skapa en uppdaterings distribution. Välj **för skript + efter skript**. Den här åtgärden öppnar sidan **Välj före skript + efter skript** .

![Välj skript](./media/pre-post-scripts/select-scripts.png)

Välj det skript som du vill använda. I det här exemplet använder vi **UpdateManagement-TurnOnVms-** runbooken. När du väljer Runbook öppnas sidan **Konfigurera skript** . Välj **för skript**och välj sedan **OK**.

Upprepa processen för **UpdateManagement-TurnOffVms-** skriptet. Men när du väljer **skript typen**väljer du **efter skript**.

Avsnittet **valda objekt** visar nu båda skripten som valts. En är ett för skript och det andra är ett efter skript:

![Valda objekt](./media/pre-post-scripts/selected-items.png)

Slutför konfigurationen av uppdaterings distributionen.

När distributionen av uppdateringen är klar kan du gå till **uppdaterings distributioner** för att visa resultaten. Som du kan se anges statusen för för-skriptet och efter skriptet:

![Uppdatera resultat](./media/pre-post-scripts/update-results.png)

Genom att välja körningen av uppdaterings distributionen visas ytterligare information om för-skript och post-skript. En länk till skript källan vid körningen tillhandahålls.

![Körnings resultat för distribution](./media/pre-post-scripts/deployment-run.png)

es i skriptet.

## <a name="stopping-a-deployment"></a>Stoppa en distribution

Om du vill stoppa en distribution som baseras på ett för skript, måste du [utlösa](automation-runbook-execution.md#throw) ett undantag. Om du inte gör det kommer distribution och post script fortfarande att köras. Följande kodfragment visar hur du genererar ett undantag.

```powershell
#In this case, we want to terminate the patch job if any run fails.
#This logic might not hold for all cases - you might want to allow success as long as at least 1 run succeeds
foreach($summary in $finalStatus)
{
    if ($summary.Type -eq "Error")
    {
        #We must throw in order to fail the patch deployment.
        throw $summary.Summary
    }
}
```



## <a name="interacting-with-machines"></a>Interagera med datorer

För skript och post-uppgifter körs som Runbooks i ditt Automation-konto och inte direkt på datorerna i distributionen. Aktiviteter och aktiviteter körs också i Azure-kontexten och har inte åtkomst till datorer som inte är Azure-datorer. I följande avsnitt visas hur du kan interagera med datorerna direkt, oavsett om de är virtuella Azure-datorer eller datorer som inte är Azure-datorer.

### <a name="interact-with-azure-machines"></a>Interagera med Azure-datorer

Aktiviteter och aktiviteter körs som Runbooks och körs inte internt på dina virtuella Azure-datorer i distributionen. Om du vill interagera med dina virtuella Azure-datorer måste du ha följande objekt:

* Ett Kör som-konto
* En Runbook som du vill köra

Om du vill interagera med Azure-datorer ska du använda cmdleten [Invoke-AzVMRunCommand](https://docs.microsoft.com/powershell/module/az.compute/invoke-azvmruncommand?view=azps-3.7.0) för att interagera med dina virtuella Azure-datorer. Ett exempel på hur du gör detta finns i Runbook-exemplet [uppdateringshantering – kör skript med kommandot kör](https://gallery.technet.microsoft.com/Update-Management-Run-40f470dc).

### <a name="interact-with-non-azure-machines"></a>Interagera med datorer som inte är Azure-datorer

Aktiviteter och aktiviteter som körs i Azure-kontexten och som inte har åtkomst till datorer som inte är Azure-datorer. Om du vill interagera med datorer som inte är Azure-datorer måste du ha följande objekt:

* Ett Kör som-konto
* Hybrid Runbook Worker installerat på datorn
* En Runbook som du vill köra lokalt
* En överordnad Runbook

För att interagera med datorer som inte är Azure-datorer körs en överordnad Runbook i Azure-kontexten. Denna Runbook anropar en underordnad Runbook med cmdleten [Start-AzAutomationRunbook](https://docs.microsoft.com/powershell/module/Az.Automation/Start-AzAutomationRunbook?view=azps-3.7.0) . Du måste ange `RunOn` parametern och ange namnet på Hybrid Runbook Worker som skriptet ska köras på. Se Runbook-exemplet [uppdateringshantering – kör skriptet lokalt](https://gallery.technet.microsoft.com/Update-Management-Run-6949cc44).

## <a name="aborting-patch-deployment"></a>Avbryter korrigerings distribution

Om ditt för skript returnerar ett fel kanske du vill avbryta distributionen. Om du vill göra det måste du [utlösa](/powershell/module/microsoft.powershell.core/about/about_throw) ett fel i skriptet för vilken logik som skulle utgöra ett fel.

```powershell
if (<My custom error logic>)
{
    #Throw an error to fail the patch deployment.
    throw "There was an error, abort deployment"
}
```

## <a name="samples"></a>Exempel

Exempel för för skript och post-skript finns i [Script Center-galleriet](https://gallery.technet.microsoft.com/scriptcenter/site/search?f%5B0%5D.Type=RootCategory&f%5B0%5D.Value=WindowsAzure&f%5B0%5D.Text=Windows%20Azure&f%5B1%5D.Type=SubCategory&f%5B1%5D.Value=WindowsAzure_automation&f%5B1%5D.Text=Automation&f%5B2%5D.Type=SearchText&f%5B2%5D.Value=update%20management&f%5B3%5D.Type=Tag&f%5B3%5D.Value=Patching&f%5B3%5D.Text=Patching&f%5B4%5D.Type=ProgrammingLanguage&f%5B4%5D.Value=PowerShell&f%5B4%5D.Text=PowerShell) och i [PowerShell-galleriet](https://www.powershellgallery.com/packages?q=Tags%3A%22UpdateManagement%22+Tags%3A%22Automation%22), eller så kan du importera dem via Azure Portal. Om du vill göra det går du till ditt Automation-konto, under **process automatisering**, väljer **Runbooks-Galleri**. Använd **uppdateringshantering** för filtret.

![Galleri lista](./media/pre-post-scripts/runbook-gallery.png)

Du kan också söka efter dem med deras skript namn, som du ser i följande lista:

* Uppdateringshantering – Aktivera virtuella datorer
* Uppdateringshantering – Stäng av virtuella datorer
* Uppdateringshantering-kör skript lokalt
* Uppdateringshantering-mall för pre/post-skript
* Uppdateringshantering-kör skript med kommandot kör

> [!IMPORTANT]
> När du har importerat Runbooks måste du publicera dem innan de kan användas. Det gör du genom att leta upp Runbook i ditt Automation-konto, välja **Redigera**och sedan **publicera**.

Exemplen är alla baserade på den grundläggande mall som definieras i följande exempel. Den här mallen kan användas för att skapa en egen Runbook som ska användas med för-skript och efter skript. Den nödvändiga logiken för autentisering med Azure och hantering av `SoftwareUpdateConfigurationRunContext` -parametern ingår.

```powershell
<#
.SYNOPSIS
 Barebones script for Update Management Pre/Post

.DESCRIPTION
  This script is intended to be run as a part of Update Management pre/post-scripts.
  It requires a RunAs account.

.PARAMETER SoftwareUpdateConfigurationRunContext
  This is a system variable which is automatically passed in by Update Management during a deployment.
#>

param(
    [string]$SoftwareUpdateConfigurationRunContext
)
#region BoilerplateAuthentication
#This requires a RunAs account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Add-AzAccount `
    -ServicePrincipal `
    -TenantId $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzureContext = Select-AzSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID
#endregion BoilerplateAuthentication

#If you wish to use the run context, it must be converted from JSON
$context = ConvertFrom-Json  $SoftwareUpdateConfigurationRunContext
#Access the properties of the SoftwareUpdateConfigurationRunContext
$vmIds = $context.SoftwareUpdateConfigurationSettings.AzureVirtualMachines | Sort-Object -Unique
$runId = $context.SoftwareUpdateConfigurationRunId

Write-Output $context

#Example: How to create and write to a variable using the pre-script:
<#
#Create variable named after this run so it can be retrieved
New-AzAutomationVariable -ResourceGroupName $ResourceGroup –AutomationAccountName $AutomationAccount –Name $runId -Value "" –Encrypted $false
#Set value of variable
Set-AutomationVariable –Name $runId -Value $vmIds
#>

#Example: How to retrieve information from a variable set during the pre-script
<#
$variable = Get-AutomationVariable -Name $runId
#>
```

> [!NOTE]
> För icke-grafiska PowerShell-Runbooks `Add-AzAccount` och `Add-AzureRMAccount` är alias för [Connect-AzAccount](https://docs.microsoft.com/powershell/module/az.accounts/connect-azaccount?view=azps-3.5.0). Du kan använda dessa cmdletar, eller så kan du [Uppdatera dina moduler](automation-update-azure-modules.md) i ditt Automation-konto till de senaste versionerna. Du kan behöva uppdatera dina moduler även om du precis har skapat ett nytt Automation-konto.

## <a name="next-steps"></a>Nästa steg

Gå vidare till följande självstudie och lär dig hur du hanterar uppdateringar för virtuella Windows-datorer:

> [!div class="nextstepaction"]
> [Hantera uppdateringar och korrigeringar för dina virtuella Windows-datorer i Azure](automation-tutorial-update-management.md)