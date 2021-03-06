---
title: ta med fil
description: ta med fil
services: functions
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/04/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 439b312050f657566026a36c145e7b6dd5cc9bad
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2020
ms.locfileid: "83116462"
---
1. I menyn i Azure-portalen eller på sidan **Start** väljer du **Skapa en resurs**.

1. På den **nya** sidan väljer du **Compute**  >  **Funktionsapp**.

1. På sidan **grundläggande** inställningar använder du funktionen appinställningar som anges i följande tabell.

    | Inställningen      | Föreslaget värde  | Beskrivning |
    | ------------ | ---------------- | ----------- |
    | **Prenumeration** | Din prenumeration | Prenumerationen som den nya funktionsappen skapas under. |
    | **[Resurs grupp](../articles/azure-resource-manager/management/overview.md)** |  *myResourceGroup* | Namnet på den nya resursgrupp där du vill skapa funktionsappen. |
    | **Funktionsapp namn** | Globalt unikt namn | Namn som identifierar din nya funktionsapp. Giltiga tecken är `a-z` (Skift läges okänsligt), `0-9` och `-` .  |
    |**Publicera**| Kod | Alternativ för att publicera kodfiler eller en Docker-container. |
    | **Körningsstack** | Önskat språk | Välj en körning som stöder det funktionsprogrammeringsspråk som du föredrar. Välj **.net Core** för C# och F # functions. |
    |**Version**| Versionsnummer | Välj den version av den installerade körnings miljön.  |
    |**Nationella**| Önskad region | Välj en [plats](https://azure.microsoft.com/regions/) nära dig eller nära andra tjänster som kommer att användas i dina funktioner. |

    ![Grundläggande inställningar](./media/functions-create-function-app-portal/function-app-create-basics.png)

1. Välj **Nästa: värd**. Ange följande inställningar på sidan **värd** .

    | Inställningen      | Föreslaget värde  | Beskrivning |
    | ------------ | ---------------- | ----------- |
    | **[Lagringskonto](../articles/storage/common/storage-account-create.md)** |  Globalt unikt namn |  Skapa ett lagringskonto som används av din funktionsapp. Lagrings konto namn måste innehålla mellan 3 och 24 tecken och får bara innehålla siffror och gemena bokstäver. Du kan också använda ett befintligt konto som måste uppfylla kraven för [lagrings kontot](../articles/azure-functions/functions-scale.md#storage-account-requirements). |
    |**Operativsystem**| Önskat operativ system | Ett operativ system är i förväg valt för dig baserat på ditt val av körnings stack, men du kan ändra inställningen om det behövs. |
    | **[Planera](../articles/azure-functions/functions-scale.md)** | **Förbrukning (utan server)** | Värdplan som definierar hur resurser allokeras till din funktionsapp. I standard **förbruknings** planen läggs resurser till dynamiskt enligt vad som krävs av dina funktioner. I den här värdbaserade [servern](https://azure.microsoft.com/overview/serverless-computing/) betalar du bara för den tid som dina funktioner körs. När du använder en App Service-plan måste du hantera [funktionsappens skalning](../articles/azure-functions/functions-scale.md).  |

    ![Värd](./media/functions-create-function-app-portal/function-app-create-hosting.png)

1. Välj **Nästa: övervakning**. Ange följande inställningar på sidan **övervakning** .

    | Inställningen      | Föreslaget värde  | Beskrivning |
    | ------------ | ---------------- | ----------- |
    | **[Application Insights](../articles/azure-functions/functions-monitoring.md)** | Standardvärde | Skapar en Application Insights resurs av samma *app-namn* i den närmaste region som stöds. Genom att utöka den här inställningen eller välja **Skapa ny**kan du ändra Application Insights namn eller välja en annan region i ett [Azure-geografiskt](https://azure.microsoft.com/global-infrastructure/geographies/) område där du vill lagra dina data. |

    ![Övervakning](./media/functions-create-function-app-portal/function-app-create-monitoring.png)

1. Välj **Granska + skapa** för att granska konfigurations valen för appen.

1. På sidan **Granska + skapa** granskar du inställningarna och väljer sedan **skapa** för att etablera och distribuera Function-appen.

1. Välj **aviserings** ikonen i det övre högra hörnet i portalen och titta efter ett meddelande om att **distributionen har slutförts** .

1. Välj **Gå till resurs** att visa den nya funktionsappen. Du kan också välja **Fäst vid instrument panelen**. Genom att fästa blir det enklare att återgå till den här funktions program resursen från instrument panelen.

    ![Distributionsmeddelande](./media/functions-create-function-app-portal/function-app-create-notification2.png)