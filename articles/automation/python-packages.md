---
title: Hantera python 2-paket i Azure Automation
description: Den här artikeln beskriver hur du hanterar python 2-paket i Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 02/25/2019
ms.topic: conceptual
ms.openlocfilehash: 701a5aab7a0061f8b5abfaac1b699034db2671b9
ms.sourcegitcommit: eaec2e7482fc05f0cac8597665bfceb94f7e390f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82508997"
---
# <a name="manage-python-2-packages-in-azure-automation"></a>Hantera python 2-paket i Azure Automation

Med Azure Automation kan du köra python 2-Runbooks på Azure och på Linux hybrid Runbook Worker. För att under lätta för Runbooks kan du använda python-paket för att importera de moduler som du behöver. Den här artikeln beskriver hur du hanterar och använder python-paket i Azure Automation.

## <a name="import-packages"></a>Importera paket

I ditt Automation-konto väljer du **python 2-paket** under **delade resurser**. Klicka på **+ Lägg till ett python 2-paket**.

![Lägg till python-paket](media/python-packages/add-python-package.png)

På sidan Lägg till python 2-paket väljer du ett lokalt paket att ladda upp. Paketet kan vara en **. WHL** -eller **. tjär. gz** -fil. När paketet är markerat klickar du på **OK** för att ladda upp det.

![Lägg till python-paket](media/python-packages/upload-package.png)

När ett paket har importer ATS visas det på sidan python 2-paket i ditt Automation-konto. Om du behöver ta bort ett paket markerar du paketet och klickar på **ta bort**.

![Paket lista](media/python-packages/package-list.png)

## <a name="import-packages-with-dependencies"></a>Importera paket med beroenden

Azure Automation löser inte beroenden för python-paket under import processen. Det finns två sätt att importera ett paket med alla dess beroenden. Endast ett av följande steg måste användas för att importera paketen till ditt Automation-konto.

### <a name="manually-download"></a>Hämta manuellt

På en Windows 64-dator med [python 2.7](https://www.python.org/downloads/release/latest/python2) och [pip](https://pip.pypa.io/en/stable/) installerat kör du följande kommando för att ladda ned ett paket och alla dess beroenden:

```cmd
C:\Python27\Scripts\pip2.7.exe download -d <output dir> <package name>
```

När paketen har hämtats kan du importera dem till ditt Automation-konto.

### <a name="runbook"></a>Runbook

 Om du vill hämta en Runbook [importerar du python 2-paket från pypi till Azure Automation konto](https://gallery.technet.microsoft.com/scriptcenter/Import-Python-2-packages-57f7d509) från galleriet till ditt Automation-konto. Se till att körnings inställningarna är inställda på **Azure** och starta runbooken med parametrarna. Runbook kräver ett Kör som-konto för att Automation-kontot ska fungera. För varje parameter måste du starta den med växeln som visas i följande lista och bild:

* -s \<subscriptionId\>
* – g \<resourceGroup\>
* – en \<automationAccount\>
* -m \<modulePackage\>

![Paket lista](media/python-packages/import-python-runbook.png)

I Runbook-flödet kan du ange vilket paket som ska laddas ned. Om du `Azure` till exempel använder parametern hämtas alla Azure-moduler och alla beroenden (cirka 105).

När runbooken har slutförts kan du kontrol lera **python 2-paketen** under **delade resurser** i ditt Automation-konto för att kontrol lera att paketet har importer ATS korrekt.

## <a name="use-a-package-in-a-runbook"></a>Använda ett paket i en Runbook

När ett paket har importer ATS kan du använda det i en Runbook. I följande exempel används [paketet Azure Automation-verktyget](https://github.com/azureautomation/azure_automation_utility). Det här paketet gör det enklare att använda python med Azure Automation. Om du vill använda paketet följer du anvisningarna i GitHub-lagringsplatsen och lägger till det i Runbook-flödet. Du kan till exempel använda `from azure_automation_utility import get_automation_runas_credential` för att importera funktionen för att hämta kör som-kontot.

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

## <a name="develop-and-test-runbooks-offline"></a>Utveckla och testa Runbooks offline

För att utveckla och testa dina python 2-Runbooks offline kan du använda modulen [Azure Automation python-emulerade resurser](https://github.com/azureautomation/python_emulated_assets) på GitHub. Med den här modulen kan du referera till dina delade resurser, till exempel autentiseringsuppgifter, variabler, anslutningar och certifikat.

## <a name="next-steps"></a>Nästa steg

För att komma igång med python 2-Runbooks, se [min första python 2-Runbook](automation-first-runbook-textual-python2.md).
