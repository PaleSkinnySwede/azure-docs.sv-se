---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/05/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 7a5717f7bcb891f78054e3093c89f75622494aff
ms.sourcegitcommit: 11572a869ef8dbec8e7c721bc7744e2859b79962
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/05/2020
ms.locfileid: "82837811"
---
|Name |Beskrivning |Påverkan (ar) |Version |GitHub |
|---|---|---|---|---|
|[Azure Backup ska vara aktiverat för Virtual Machines](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F013e242c-8828-4970-87b3-ab247555486d) |Den här principen hjälper dig att granska om Azure Backup-tjänsten är aktive rad för alla virtuella datorer. Azure Backup är en kostnads effektiv lösning för säkerhets kopiering med enkel klickning som fören klar data återställningen och är enklare att aktivera än andra moln tjänster för säkerhets kopiering. |AuditIfNotExists, inaktiverat |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachines_EnableAzureBackup_Audit.json) |
|[Konfigurera säkerhets kopiering på virtuella datorer för en plats till ett befintligt centralt valv på samma plats](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F09ce66bc-1220-4153-8104-e3f51c936913) |Den här principen konfigurerar Azure Backup skydd på virtuella datorer på en specifik plats till ett befintligt centralt valv på samma plats. Den gäller endast för de virtuella datorer som inte redan har kon figurer ATS för säkerhets kopiering. Vi rekommenderar att den här principen är tilldelad till högst 200 virtuella datorer. Om principen är tilldelad för fler än 200 virtuella datorer kan det leda till att säkerhets kopieringen utlöses några timmar utöver det definierade schemat. Den här principen kommer att förbättras för att stödja fler VM-avbildningar. |deployIfNotExists, auditIfNotExists, inaktiverat |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Backup/VirtualMachineBackup_Backup_DeployIfNotExists.json) |
|[Distribuera diagnostikinställningar för Recovery Services valv till Log Analytics arbets yta för resursbaserade kategorier.](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc717fb0c-d118-4c43-ab3d-ece30ac81fb3) |Distribuera diagnostikinställningar för Recovery Services valv för att strömma till Log Analytics arbets ytan för resursbaserade kategorier. Om någon av resurs kategorierna inte är aktiverade skapas en ny diagnostisk inställning. |deployIfNotExists |1.0.0 – för hands version |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Monitoring/EnableRecoveryServiceVaultDiagnosticSetting_Backup_DeployIfNotExist.json) |
