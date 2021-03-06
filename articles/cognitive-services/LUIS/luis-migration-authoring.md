---
title: Migrera till Azure-resurs för redigering
titleSuffix: Azure Cognitive Services
description: Migrera till en resurs nyckel för Azure-redigering.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 02/28/2020
ms.author: diberry
ms.openlocfilehash: 679073715588a4a81e69e3e7ba2d18341b1bab4b
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82096630"
---
# <a name="migrate-to-an-azure-resource-authoring-key"></a>Migrera till en Azure-resurs redigerings nyckel

Language Understanding (LUIS) som redigerar autentiseringen har ändrats från ett e-postkonto till en Azure-resurs. När det inte krävs för närvarande kommer växling till en Azure-resurs att verkställas i framtiden.

## <a name="why-migrate"></a>Varför ska man migrera?

Med hjälp av en Azure-resurs för redigering kan du, som ägare av resursen, kontrol lera åtkomsten till redigering. Du kan skapa och namnge att skriva resurser för att hantera olika grupper av författare.

Till exempel är du ägare till 2 LUIS-appar och du har olika medlemmar som är medarbetare på varje app. Du kan skapa två olika redigerings resurser och tilldela varje app till varje separat resurs. Tilldela sedan varje medlem som deltagare till rätt redigerings resurs beroende på vilken app de samarbetar om. Azure Authoring-resursen styr auktoriseringen.

> [!Note]
> Innan migreringen kallas medförfattare _för medarbetare på Luis_ app-nivå. Efter migreringen används Azure-rollen för _deltagare_ för samma funktioner, men på Azures resurs nivå.

## <a name="what-is-migrating"></a>Vad är migrering?

Migreringen omfattar:

* Alla användare av LUIS, ägare och deltagare.
* **Alla** appar.
* En **enkelriktad migrering.**

Ägaren kan inte välja en delmängd av appar som ska migreras och processen inte kan ångras.

Migreringen är inte:

* En process som samlar in medarbetare och automatiskt flyttar eller lägger till i Azure Authoring-resursen. Du, som appens ägare, måste slutföra det här steget. Det här steget kräver behörighet till lämplig resurs.
* En process för att skapa och tilldela en förutsägelse körnings resurs. Om du behöver en förutsägelse körnings resurs, är det [en separat process](luis-how-to-azure-subscription.md#create-resources-in-the-azure-portal) och är oförändrad.

## <a name="how-are-the-apps-migrating"></a>Hur migrerar appar?

[Luis-portalen](https://www.luis.ai) innehåller migreringsprocessen.

Du uppmanas att migrera om:

* Du har appar i systemet för e-postautentisering för redigering.
* Och du är appens ägare.

Du kan försena migreringsprocessen genom att avbryta fönstret. Du uppmanas regelbundet att migrera tills du migrerar eller så skickas tids gränsen för migreringen. Du kan starta migreringsprocessen från det övre navigerings fältets lås ikon.

## <a name="migration-for-the-app-owner"></a>Migrering av appens ägare

### <a name="before-you-migrate"></a>Innan du migrerar

* **Krävs**måste du ha en Azure- [prenumeration](https://azure.microsoft.com/free/). En del av prenumerations processen kräver fakturerings information. Du kan dock använda den kostnads fria (`F0`) pris nivån när du använder Luis.
* **Du kan också**säkerhetskopiera apparna från Luis-portalens lista med appar genom att exportera varje app eller använda export- [API: et](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40).
* **Du kan också**Spara varje apps collaborator's-lista. Den här e-postlistan tillhandahålls som en del av migreringsprocessen.


**Redigering av din Luis-app är kostnads fri**, vilket `F0` anges av nivån. Läs [mer om pris nivåer](luis-limits.md#key-limits).

Om du inte har någon Azure-prenumeration kan du [Registrera dig](https://azure.microsoft.com/free/).

### <a name="migration-steps"></a>Migreringsanvisningar

Följ [dessa steg för migrering](luis-migration-authoring-steps.md).

### <a name="after-you-migrate"></a>När du har migrerat

Efter migreringsprocessen är alla dina LUIS-appar nu kopplade till en enda LUIS Authoring-resurs.

Du kan skapa fler redigerings resurser och tilldela dem från sidan **hantera > Azure-resurser** på Luis- _portalen_.

Du kan lägga till deltagare till redigerings resursen från _Azure Portal_på sidan **Access Control (IAM)** för resursen. Mer information finns i [lägga till deltagar åtkomst](luis-migration-authoring-steps.md#after-the-migration-process-add-contributors-to-your-authoring-resource).

|Portalen|Syfte|
|--|--|
|[Azure](https://azure.microsoft.com/free/)|* Skapa förutsägelse-och redigera resurser.<br>* Tilldela deltagare.|
|[LUIS](https://www.luis.ai)|* Migrera till nya redigerings resurser.<br>* Tilldela eller ta bort tilldelningar av förutsägelse och redigering av resurser till appar från sidan **hantera > Azure-resurser** .|

## <a name="migration-for-the-app-contributor"></a>Migrering av appens deltagare

Varje användare av LUIS måste migreras, inklusive medarbetare/deltagare. En medarbetare måste migrera för att få åtkomst till appen.

> [!Note]
> Om ägaren av LUIS-appen migrerat och lagt till medarbetaren som deltagare på Azure-resursen, kommer medarbetaren fortfarande inte att ha åtkomst till appen om de också migrerar.

### <a name="before-the-app-is-migrated"></a>Innan appen migreras

Du kan välja att exportera en app som du är en medarbetare på och sedan importera appen tillbaka till LUIS. Import processen skapar en ny app med ett nytt app-ID som du är ägare till.

### <a name="after-the-app-is-migrated"></a>När appen har migrerats

Appens ägare måste [lägga till ditt e-postmeddelande till Azure Authoring-resursen som en medarbetare](luis-how-to-collaborate.md#add-contributor-to-azure-authoring-resource).

Efter migreringen är alla appar som du äger tillgängliga på sidan **Mina appar** i Luis-portalen.

## <a name="troubleshooting-the-migration-process-for-luis-authoring"></a>Felsöka migreringsprocessen för LUIS-redigering

* LUIS redigerings nycklar visas bara i LUIS-portalen när migreringen är klar. Om du skapar redigerings nycklarna, till exempel med LUIS CLI, måste användaren fortfarande slutföra migreringsprocessen i LUIS-portalen.
* Om en migrerad användare lägger till en icke-migrerad användare som deltagare i sin Azure-resurs får den icke-migrerade användaren ingen åtkomst till apparna om de inte migrerar.
* Om en icke-migrerad användare inte är ägare till några appar, men är en medarbetare till andra appar som ägs av andra och ägarna har genomgått migreringsprocessen, måste användaren migrera för att få åtkomst till apparna.
* Om en icke-migrerad användare har lagt till en annan migrerad användare som en medarbetare i sin app, uppstår ett fel eftersom du inte kan lägga till en migrerad användare som medarbetare i en app. Den icke-migrerade användaren måste sedan gå igenom migreringsprocessen och skapa en Azure-resurs och lägga till den migrerade användaren som deltagare i resursen.

Du får ett fel meddelande under migreringsprocessen om:
* Din prenumeration tillåter inte att du skapar Cognitive Services resurser
* Migreringen påverkar negativt alla program körningar. När du migrerar tas alla medarbetare bort från dina appar och du tas bort som en medarbetare från andra appar. Den här processen innebär att de nycklar som du har tilldelat också tas bort. Migreringen blockeras om du har tilldelat nycklar i andra appar. Ta bort den nyckel du tilldelade säkert innan du migrerar. Om du vet att den nyckel som du har tilldelat inte används i körnings miljön måste du ta bort den för att kunna fortsätta med migreringen.

Få åtkomst till appens Azure Resource List med följande URL-format:

`https://www.luis.ai/applications/REPLACE-WITH-YOUR-APP-ID/versions/REPLACE-WITH-YOUR-VERSION-ID/manage/resources`

## <a name="next-steps"></a>Nästa steg

* [Så här migrerar du din app till en redigerings resurs](luis-migration-authoring-steps.md)
