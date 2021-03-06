---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 05/05/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 5ddb8e8987e1bd42f28f98b6a609bb460ce4fc7e
ms.sourcegitcommit: 11572a869ef8dbec8e7c721bc7744e2859b79962
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/05/2020
ms.locfileid: "82838288"
---
|Name |Beskrivning |Påverkan (ar) |Version |GitHub |
|---|---|---|---|---|
|[Tillåtna platser](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe56962a6-4747-49cd-b67b-bf8b01975c4c) |Med den här principen kan du begränsa vilka platser som organisationen kan ange när den distribuerar resurser. Den används för att genomdriva kraven på geo-efterlevnad. Resursgrupper, Microsoft.AzureActiveDirectory/b2cDirectories och resurser som använder regionen ”global” undantas. |autentiseringsregel |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/AllowedLocations_Deny.json) |
|[Tillåtna platser för resurs grupper](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fe765b5de-1225-4ba3-bd56-1ac6695af988) |Med den här principen kan du begränsa vilka platser som organisationen kan skapa resurs grupper i. Den används för att genomdriva kraven på geo-efterlevnad. |autentiseringsregel |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/ResourceGroupAllowedLocations_Deny.json) |
|[Tillåtna resurstyper](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa08ec900-254a-4555-9bf5-e42af04b5c5c) |Med den här principen kan du ange vilka resurs typer som din organisation kan distribuera. Endast resurs typer som stöder ' taggar ' och ' Location ' kommer att påverkas av den här principen. Om du vill begränsa alla resurser duplicerar du principen och ändrar "mode" till "alla". |autentiseringsregel |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/AllowedResourceTypes_Deny.json) |
|[Granska resurs plats matchar resurs gruppens plats](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F0a914e76-4921-4c19-b460-a2d36003525a) |Granska att resurs platsen matchar resurs gruppens plats |händelse |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/ResourcesInResourceGroupLocation_Audit.json) |
|[Granska användningen av anpassade RBAC-regler](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa451c1ef-c6ca-483d-87ed-f49761e3ffb5) |Granska inbyggda roller som "ägare, Contribute", läsare "i stället för anpassade RBAC-roller som är fel känsliga. Användning av anpassade roller behandlas som ett undantag och kräver en rigorös gransknings-och hot modellering |Granskning, inaktive rad |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/Subscription_AuditCustomRBACRoles_Audit.json) |
|[Roller för anpassade prenumerationer får inte finnas](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F10ee2ea2-fb4d-45b8-a7e9-a2e770044cd9) |Den här principen säkerställer att det inte finns några anpassade prenumerations ägare roller. |Granskning, inaktive rad |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/CustomSubscription_OwnerRole_Audit.json) |
|[Ej tillåtna resurs typer](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F6c112d4e-5bc7-47ae-a041-ea2d9dccd749) |Med den här principen kan du ange de resurs typer som din organisation inte kan distribuera. |Neka |1.0.0 |[Länk](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/General/InvalidResourceTypes_Deny.json) |
