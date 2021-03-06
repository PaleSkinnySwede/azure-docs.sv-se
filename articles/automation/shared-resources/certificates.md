---
title: Hantera certifikat i Azure Automation
description: Azure Automation säkert lagrar certifikat så att Runbooks och DSC-konfigurationer kan komma åt dem för att autentisera mot Azure och resurser från tredje part. I den här artikeln förklaras information om certifikat och hur du arbetar med dem i både text-och grafisk redigering.
services: automation
ms.service: automation
ms.subservice: shared-capabilities
author: mgoedtel
ms.author: magoedte
ms.date: 04/02/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 2793679fb4588d00ea4e37340b19183398cb9d90
ms.sourcegitcommit: 602e6db62069d568a91981a1117244ffd757f1c2
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82864325"
---
# <a name="manage-certificates-in-azure-automation"></a>Hantera certifikat i Azure Automation

Azure Automation lagrar certifikat säkert för åtkomst av Runbooks och DSC-konfigurationer med hjälp av cmdleten [Get-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0) för Azure Resource Manager-resurser. Med säker certifikat lagring kan du skapa Runbooks och DSC-konfigurationer som använder certifikat för autentisering, eller lägga till dem i Azure eller från resurser från tredje part.

>[!NOTE]
>Säkra till gångar i Azure Automation inkluderar autentiseringsuppgifter, certifikat, anslutningar och krypterade variabler. Dessa till gångar krypteras och lagras i Automation med hjälp av en unik nyckel som genereras för varje Automation-konto. Automation lagrar nyckeln i den systemhanterade Key Vaults tjänsten. Innan du lagrar en säker till gång läser Automation in nyckeln från Key Vault och använder den för att kryptera till gången. 

>[!NOTE]
>Den här artikeln visar hur du använder modulen Azure PowerShell AZ. Du kan fortfarande använda AzureRM-modulen. Mer information om AZ-modulen och AzureRM-kompatibilitet finns i [Introduktion till den nya Azure PowerShell AZ-modulen](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0). Installations anvisningar för AZ-modulen på Hybrid Runbook Worker finns i [installera Azure PowerShell-modulen](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0). För ditt Automation-konto kan du uppdatera dina moduler till den senaste versionen med hjälp av [hur du uppdaterar Azure PowerShell moduler i Azure Automation](../automation-update-azure-modules.md).

## <a name="powershell-cmdlets-to-access-certificates"></a>PowerShell-cmdletar för att komma åt certifikat

Cmdletarna i följande tabell skapar och hanterar Automation-certifikat med PowerShell. De levereras som en del av [AZ-modulerna](modules.md#az-modules).

|Cmdlet |Beskrivning|
| --- | ---|
|[Get-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0)|Hämtar information om ett certifikat som ska användas i en Runbook-eller DSC-konfiguration. Du kan bara hämta själva certifikatet med hjälp av den interna `Get-AutomationCertificate` cmdleten.|
|[New-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/New-AzAutomationCertificate?view=azps-3.7.0)|Skapar ett nytt certifikat i Automation.|
|[Remove-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/Remove-AzAutomationCertificate?view=azps-3.7.0)|Tar bort ett certifikat från Automation.|
|[Set-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/Set-AzAutomationCertificate?view=azps-3.7.0)|Anger egenskaperna för ett befintligt certifikat, inklusive överföring av certifikat filen och inställning av lösen ordet för en **PFX** -fil.|

Cmdleten [Add-AzureCertificate](/powershell/module/servicemanagement/azure/add-azurecertificate) kan också användas för att ladda upp ett tjänst certifikat för den angivna moln tjänsten.

## <a name="internal-cmdlets-to-access-certificates"></a>Interna cmdlets för att komma åt certifikat

Den interna cmdleten i följande tabell används för att komma åt certifikat i dina runbooks. Denna cmdlet ingår i den globala modulen `Orchestrator.AssetManagement.Cmdlets`. Mer information finns i [interna cmdletar](modules.md#internal-cmdlets).

| Intern cmdlet | Beskrivning |
|:---|:---|
|`Get-AutomationCertificate`|Hämtar ett certifikat som ska användas i en Runbook-eller DSC-konfiguration. Returnerar ett [system. Security. Cryptography. X509Certificates. X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2) -objekt.|

> [!NOTE] 
> Du bör undvika att använda variabler i `Name` -parametern `Get-AutomationCertificate` i en Runbook-eller DSC-konfiguration. Sådana variabler kan komplicera identifieringen av beroenden mellan Runbooks och DSC-konfigurationer och automation-variabler i design läge.

## <a name="python-2-functions-to-access-certificates"></a>Python 2-funktioner för att få åtkomst till certifikat

Använd funktionen i följande tabell för att komma åt certifikat i en python 2-Runbook.

| Funktion | Beskrivning |
|:---|:---|
| `automationassets.get_automation_certificate` | Hämtar information om en certifikat till gång. |

> [!NOTE]
> Du måste importera `automationassets` modulen i början av din python-Runbook för att få åtkomst till till gångs funktionerna.

## <a name="create-a-new-certificate"></a>Skapa ett nytt certifikat

När du skapar ett nytt certifikat laddar du upp en CER-eller PFX-fil som ska automatiseras. Om du markerar att certifikatet kan exporteras kan du överföra det från automations certifikat arkiv. Om den inte kan exporteras kan du bara använda den för signering i Runbook-eller DSC-konfigurationen. Automation kräver att certifikatet har leverantören **Microsoft Enhanced RSA och AES Cryptographic Provider**.

### <a name="create-a-new-certificate-with-the-azure-portal"></a>Skapa ett nytt certifikat med Azure Portal

1. Från ditt Automation-konto väljer du **till gångar** > **certifikat** > **Lägg till ett certifikat**.
1. I fältet **namn** anger du ett namn för certifikatet.
1. Om du vill bläddra efter en **CER** -eller **PFX** -fil går du till **överför en certifikat fil**och väljer **Välj en fil**. Om du väljer en **. pfx** -fil anger du ett lösen ord och anger om det kan exporteras.
1. Välj **skapa** för att spara den nya certifikat till gången.

### <a name="create-a-new-certificate-with-powershell"></a>Skapa ett nytt certifikat med PowerShell

Följande exempel visar hur du skapar ett nytt Automation-certifikat och markerar det som kan exporteras. I det här exemplet importeras en befintlig **. pfx** -fil.

```powershell-interactive
$certificateName = 'MyCertificate'
$PfxCertPath = '.\MyCert.pfx'
$CertificatePassword = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force
$ResourceGroup = "ResourceGroup01"

New-AzAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certificateName -Path $PfxCertPath –Password $CertificatePassword -Exportable -ResourceGroupName $ResourceGroup
```

### <a name="create-a-new-certificate-with-a-resource-manager-template"></a>Skapa ett nytt certifikat med en Resource Manager-mall

Följande exempel visar hur du distribuerar ett certifikat till ditt Automation-konto med hjälp av en Resource Manager-mall via PowerShell:

```powershell-interactive
$AutomationAccountName = "<automation account name>"
$PfxCertPath = '<PFX cert path'
$CertificatePassword = '<password>'
$certificateName = '<certificate name>'
$AutomationAccountName = '<automation account name>'
$flags = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::PersistKeySet `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::MachineKeySet
# Load the certificate into memory
$PfxCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($PfxCertPath, $CertificatePassword, $flags)
# Export the certificate and convert into base 64 string
$Base64Value = [System.Convert]::ToBase64String($PfxCert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12))
$Thumbprint = $PfxCert.Thumbprint


$json = @"
{
    '`$schema': 'https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#',
    'contentVersion': '1.0.0.0',
    'resources': [
        {
            'name': '$AutomationAccountName/$certificateName',
            'type': 'Microsoft.Automation/automationAccounts/certificates',
            'apiVersion': '2015-10-31',
            'properties': {
                'base64Value': '$Base64Value',
                'thumbprint': '$Thumbprint',
                'isExportable': true
            }
        }
    ]
}
"@

$json | out-file .\template.json
New-AzResourceGroupDeployment -Name NewCert -ResourceGroupName TestAzureAuto -TemplateFile .\template.json
```

## <a name="get-a-certificate"></a>Hämta ett certifikat

Använd den interna `Get-AutomationCertificate` cmdleten för att hämta ett certifikat. Du kan inte använda cmdleten [Get-AzAutomationCertificate](https://docs.microsoft.com/powershell/module/Az.Automation/Get-AzAutomationCertificate?view=azps-3.7.0) eftersom den returnerar information om certifikat till gången, men inte själva certifikatet.

### <a name="textual-runbook-example"></a>Exempel på text Runbook

I följande exempel visas hur du lägger till ett certifikat i en moln tjänst i en Runbook. I det här exemplet hämtas lösen ordet från en krypterad Automation-variabel.

```powershell-interactive
$serviceName = 'MyCloudService'
$cert = Get-AutomationCertificate -Name 'MyCertificate'
$certPwd = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyCertPassword'
Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert
```

### <a name="graphical-runbook-example"></a>Exempel på grafisk Runbook

Lägg till en aktivitet för den `Get-AutomationCertificate` interna cmdleten i en grafisk Runbook genom att högerklicka på certifikatet i fönstret Bibliotek och välja **Lägg till på arbets ytan**.

![Skärm bild som visar hur du lägger till ett certifikat på arbets ytan](../media/certificates/automation-certificate-add-to-canvas.png)

Följande bild visar ett exempel på hur du använder ett certifikat i en grafisk Runbook. 

![Skärm bild av ett exempel på grafisk redigering](../media/certificates/graphical-runbook-add-certificate.png)

### <a name="python-2-example"></a>Exempel på python 2

I följande exempel visas hur du får åtkomst till certifikat i python 2-Runbooks.

```python
# get a reference to the Azure Automation certificate
cert = automationassets.get_automation_certificate("AzureRunAsCertificate")

# returns the binary cert content  
print cert
```

## <a name="next-steps"></a>Nästa steg

* Mer information om de cmdletar som används för att komma åt certifikat finns [i hantera moduler i Azure Automation](modules.md).
* Allmän information om Runbooks finns [i Runbook-körning i Azure Automation](../automation-runbook-execution.md).
* Mer information om DSC-konfigurationer finns i [Översikt över tillstånds konfiguration](../automation-dsc-overview.md). 
