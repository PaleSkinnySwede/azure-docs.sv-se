---
title: Importera en logikapp som ett API med Azure Portal | Microsoft Docs
description: Den här självstudien visar hur du använder API Management (APIM) för att importera en logikapp som ett API.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/22/2020
ms.author: apimpm
ms.openlocfilehash: b6cb0fa3611da44726dabd879c37fec357fc3031
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82202782"
---
# <a name="import-a-logic-app-as-an-api"></a>Importera en logikapp som ett API

Den här artikeln visar hur du importerar en Logic app som ett API och testar det importerade API: et.

I den här artikeln kan du se hur du:

> [!div class="checklist"]
>
> -   Importera en logikapp som ett API
> -   Testa API:et i Azure Portal
> -   Testa API:et i Developer-portalen

## <a name="prerequisites"></a>Krav

-   Slutför följande snabbstart: [Skapa en Azure API Management-instans](get-started-create-service-instance.md)
-   Kontrollera att det finns en logikapp i din prenumeration som gör en HTTP-slutpunkt tillgänglig. Mer information finns i avsnittet om att [utlösa arbetsflöden med HTTP-slutpunkter](../logic-apps/logic-apps-http-endpoint.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importera och publicera ett serverdels-API

1. Gå till din API Management-tjänst i Azure Portal och välj **API: er** på menyn.
2. Välj **Logikapp** från listan **Lägg till ett nytt API**.

    ![Logikapp](./media/import-logic-app-as-api/logic-app-api.png)

3. Tryck på **Bläddra** för att se listan med Logic Apps med http-utlösare i din prenumeration. (Observera att Logic Apps utan HTTP-utlösare visas inte i listan.)
4. Välj appen. API Management hittar den Swagger som är associerad med den valda appen, hämtar den och importerar den.
5. Lägg till ett API URL-suffix. Suffixet är ett namn som identifierar det här specifika API:et i den här API Management-instansen. Det måste vara unikt i den här API Management-instansen.
6. Du kan publicera API:et genom att associera det med en produkt. I det här fallet används den _obegränsade_ produkten. Om du vill att API:et ska publiceras och vara tillgänglig för utvecklare, lägger du till det till en produkt. Du kan göra det vid API-skapandet eller ställa in det senare.

    Produkter är associationer med en eller flera API:er. Du kan inkludera flera API:er och erbjuda dem till utvecklare via utvecklarportalen. Utvecklare måste först prenumerera på en produkt för att få åtkomst till API:n. När de prenumererar få de en prenumerationsnyckel som går att använda till alla API:er i produkten. Om du har skapat API Management-instansen är du redan administratör, så du prenumererar på alla produkter som standard.

    Som standard medföljer två exempelprodukter varje API Management-instans:

    - **Starter**
    - **Obegränsat**

7. Ange andra API-inställningar. Du kan ange värden när du skapar eller konfigurerar dem senare genom att gå till fliken **Inställningar** . Inställningarna beskrivs i själv studie kursen [Importera och publicera din första API](import-and-publish.md#-import-and-publish-a-backend-api) .
8. Välj **Skapa**.

## <a name="test-the-api-in-the-azure-portal"></a>Testa API:et i Azure Portal

Du kan anropa åtgärder direkt från Azure Portal, vilket är ett enkelt sätt att visa och testa åtgärderna i ett API.

1. Välj det API som du skapade i föregående steg.
2. Tryck på fliken **Test**.
3. Välj någon åtgärd.

    Sidan visar fält för frågeparametrar och fält för sidhuvudena. Ett av huvudena är Ocp-Apim-prenumeration-Key, för prenumerationsnyckeln till den produkt som är associerad med det här API:et. Om du skapade API Management-instansen är du redan administratör, vilket innebär att nyckeln fylls i automatiskt.

4. Tryck på **Skicka**.

    Serverdelen svarar med **200 OK** och några data.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

>[!NOTE]
>Varje logikapp har åtgärden **manual-invoke**. Om du vill att ditt API utgörs av flera logikappar, måste du byta namn på funktionen för att undvika kollisioner.

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
>
> [Transformera och skydda ett publicerat API](transform-api.md)
