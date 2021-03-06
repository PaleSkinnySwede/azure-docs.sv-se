---
title: Azure Automation Linux Hybrid Runbook Worker
description: Den här artikeln innehåller information om hur du installerar en Azure Automation Hybrid Runbook Worker så att du kan köra Runbooks på Linux-baserade datorer i ditt lokala data Center eller i moln miljön.
services: automation
ms.subservice: process-automation
ms.date: 03/02/2020
ms.topic: conceptual
ms.openlocfilehash: 82f6d9e56e5d5745077ef512cb3392c16b95961f
ms.sourcegitcommit: f57297af0ea729ab76081c98da2243d6b1f6fa63
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82872181"
---
# <a name="deploy-a-linux-hybrid-runbook-worker"></a>Distribuera ett Linux-Hybrid Runbook Worker

Du kan använda Hybrid Runbook Worker funktionen i Azure Automation för att köra Runbooks direkt på datorn som är värd för rollen och mot resurser i miljön för att hantera de lokala resurserna. Linux-Hybrid Runbook Worker Kör Runbooks som en särskild användare som kan utökas för att köra kommandon som behöver förhöjda. Runbooks lagras och hanteras i Azure Automation och levereras sedan till en eller flera angivna datorer.

Den här artikeln beskriver hur du installerar Hybrid Runbook Worker på en Linux-dator, hur du tar bort en arbets grupp och hur du tar bort en Hybrid Runbook Worker grupp.

## <a name="supported-linux-operating-systems"></a>Linux-operativsystem som stöds

Funktionen Hybrid Runbook Worker stöder följande distributioner:

* Amazon Linux 2012,09 till 2015,09 (x86/x64)
* CentOS Linux 5, 6, och 7 (x86/x64)
* Oracle Linux 5, 6, och 7 (x86/x64)
* Red Hat Enterprise Linux Server 5, 6, och 7 (x86/x64)
* Debian GNU/Linux 6, 7 och 8 (x86/x64)
* Ubuntu 12,04 LTS, 14,04 LTS, 16,04 LTS och 18,04 (x86/x64)
* SUSE Linux Enterprise Server 11 och 12 (x86/x64)

## <a name="supported-runbook-types"></a>Runbook-typer som stöds

Linux hybrid Runbook Worker stöder inte den fullständiga uppsättningen av Run Book typer i Azure Automation.

Följande Runbook-typer fungerar på en Linux-Hybrid Worker:

* Python 2
* PowerShell

  > [!NOTE]
  > PowerShell-Runbooks kräver att PowerShell Core installeras på Linux-datorn. Se [Installera PowerShell Core på Linux](/powershell/scripting/install/installing-powershell-core-on-linux) för att lära dig hur du installerar det.

Följande Runbook-typer fungerar inte på en Linux-Hybrid Worker:

* PowerShell-arbetsflöde
* Grafisk
* Grafiskt PowerShell-arbetsflöde

## <a name="deployment-requirements"></a>Distributionskrav

Minimi kraven för en Linux-Hybrid Runbook Worker är:

* Två kärnor
* 4 GB RAM
* Port 443 (utgående)

### <a name="package-requirements"></a>Paket krav

| **Nödvändigt paket** | **Beskrivning** | **Lägsta version**|
|--------------------- | --------------------- | -------------------|
|Glibc |GNU C-bibliotek| 2.5-12 |
|Openssl| OpenSSL-bibliotek | 1,0 (TLS 1,1 och TLS 1,2 stöds)|
|Curl | webb klient för sväng | 7.15.5|
|Python – ctypes | Python 2. x krävs |
|PAM | Pluggable Authentication Modules|
| **Valfritt paket** | **Beskrivning** | **Lägsta version**|
| PowerShell Core | För att köra PowerShell-Runbooks måste PowerShell installeras, se [Installera PowerShell Core på Linux](/powershell/scripting/install/installing-powershell-core-on-linux) för att lära dig hur du installerar det.  | 6.0.0 |

## <a name="install-a-linux-hybrid-runbook-worker"></a>Installera en Linux-Hybrid Runbook Worker

Om du vill installera och konfigurera en Hybrid Runbook Worker på Linux-datorn följer du en enkel manuell process. Det kräver att du aktiverar Automation Hybrid Worker-lösningen i Azure Log Analytics-arbetsytan och sedan kör en uppsättning kommandon för att registrera datorn som en anställd och lägga till den i en grupp.

Notera Log Analytics arbets ytan som ditt Automation-konto är länkat till innan du fortsätter. Observera också den primära nyckeln för ditt Automation-konto. Du kan hitta båda från Azure Portal genom att välja ditt Automation-konto, välja **arbets yta** för arbetsyte-ID och välja **nycklar** för den primära nyckeln. Information om portar och adresser som du behöver för Hybrid Runbook Worker finns i [Konfigurera nätverket](automation-hybrid-runbook-worker.md#network-planning).

>[!NOTE]
> [Nxautomation-kontot](automation-runbook-execution.md#log-analytics-agent-for-linux) med motsvarande sudo-behörigheter måste finnas under installationen av Linux-hybrid Worker. Om du försöker installera Worker och kontot inte finns eller inte har rätt behörighet, Miss lyckas installationen.

1. Aktivera Automation Hybrid Worker lösning i Azure med någon av följande metoder:

   * Lägg till Automation Hybrid Worker-lösningen i din prenumeration genom att följa anvisningarna i [lägga till Azure Monitor logg lösningar på din arbets yta](../log-analytics/log-analytics-add-solutions.md).
   * Kör följande cmdlet:

        ```azurepowershell-interactive
         Set-AzureRmOperationalInsightsIntelligencePack -ResourceGroupName  <ResourceGroupName> -WorkspaceName <WorkspaceName> -IntelligencePackName  "AzureAutomation" -Enabled $true
        ```

1. Installera Log Analytics agent för Linux genom att köra följande kommando. Ersätt \<WorkspaceID\> och \<WorkspaceKey\> med lämpliga värden från din arbets yta.

   [!INCLUDE [log-analytics-agent-note](../../includes/log-analytics-agent-note.md)]

   ```bash
   wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <WorkspaceID> -s <WorkspaceKey>
   ```

1. Kör följande kommando och ändra värdena för parametrarna *-w*, *-k*, *-g*och *-e*. För parametern *-g* ersätter du värdet med namnet på den hybrid Runbook Worker grupp som den nya Linux-hybrid Runbook Worker ska ansluta till. Om namnet inte finns i ditt Automation-konto görs en ny Hybrid Runbook Worker grupp med det namnet.

   ```bash
   sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <LogAnalyticsworkspaceId> -k <AutomationSharedKey> -g <hybridgroupname> -e <automationendpoint>
   ```

1. När kommandot har slutförts visar sidan Hybrid Worker grupper i Azure Portal den nya gruppen och antalet medlemmar. Om det här är en befintlig grupp ökar antalet medlemmar. Du kan välja gruppen i listan på sidan Hybrid Worker grupper och sedan välja panelen **hybrid personal** . På sidan hybrid Worker visas varje medlem i gruppen i listan.

> [!NOTE]
> Om du använder tillägget Azure Monitor virtuell dator för Linux för en virtuell Azure-dator rekommenderar vi att `autoUpgradeMinorVersion` du anger till false som versioner för automatisk uppgradering kan orsaka problem med hybrid Runbook Worker. Information om hur du uppgraderar tillägget manuellt finns i [Azure CLI-distribution](../virtual-machines/extensions/oms-linux.md#azure-cli-deployment).

## <a name="turn-off-signature-validation"></a>Inaktivera verifiering av signatur

Som standard kräver Linux hybrid Runbook Worker verifiering av signatur. Om du kör en osignerad Runbook mot en arbets grupp visas ett `Signature validation failed` fel meddelande. Kör följande kommando för att inaktivera verifiering av signatur. Ersätt den andra parametern med ditt Log Analytics arbetsyte-ID.

 ```bash
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <LogAnalyticsworkspaceId>
 ```

## <a name="remove-the-hybrid-runbook-worker-from-an-on-premises-linux-computer"></a><a name="remove-linux-hybrid-runbook-worker"></a>Ta bort Hybrid Runbook Worker från en lokal Linux-dator

Du kan använda kommandot `ls /var/opt/microsoft/omsagent` på Hybrid Runbook Worker för att hämta arbetsyte-ID: t. En mapp skapas med namnet med arbetsyte-ID: t.

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> Den här koden tar inte bort Log Analytics agenten för Linux från datorn. Den tar bara bort funktionaliteten och konfigurationen av Hybrid Runbook Workers rollen.

## <a name="remove-a-hybrid-worker-group"></a>Ta bort en Hybrid Worker-grupp

Om du vill ta bort en Hybrid Runbook Worker grupp Linux-datorer använder du samma steg som för en Windows hybrid Worker-grupp. Se [ta bort en hybrid Worker grupp](automation-windows-hrw-install.md#remove-a-hybrid-worker-group).

## <a name="next-steps"></a>Nästa steg

* Information om hur du konfigurerar dina runbooks för att automatisera processer i ditt lokala data Center eller någon annan moln miljö finns i [köra Runbooks på en hybrid Runbook Worker](automation-hrw-run-runbooks.md).
* Information om hur du felsöker dina hybrid Runbook Worker finns i [Felsöka Linux hybrid Runbook Worker](troubleshoot/hybrid-runbook-worker.md#linux)
