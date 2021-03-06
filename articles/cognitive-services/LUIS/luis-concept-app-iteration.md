---
title: Design av iterativa appar – LUIS
titleSuffix: Azure Cognitive Services
description: LUIS lär sig bäst i en iterativ cykel med modell ändringar, uttryck-exempel, publicering och insamling av data från slut punkts frågor.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/20/2019
ms.author: diberry
ms.openlocfilehash: c1c1b2df301634a435b610c395a1a58aa5573da3
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "74422600"
---
# <a name="iterative-app-design-for-luis"></a>Iterativ app-design för LUIS

En Language Understanding-app (LUIS) lär sig och fungerar effektivt med iteration. Här är en typisk upprepnings cykel:

* Skapa ny version
* Redigera LUIS app-schema. Du måste bland annat:
    * Avsikter med exempel yttranden
    * Entiteter
    * Funktioner
* Träna, testa och publicera
    * Testa vid förutsägelse slut punkten för aktiv inlärning
* Samla in data från slut punkts frågor

![Redigeringscykel](./media/luis-concept-app-iteration/iteration.png)

## <a name="building-a-luis-schema"></a>Skapa ett LUIS-schema

En Apps schema definierar vad användaren efterfrågar ( _avsikten eller avsikten_ ) och vilka delar av avsikten som tillhandahåller information (kallas _entiteter_ _) som_ används för att fastställa svaret. 

App-schemat måste vara särskilt för app-domänerna för att fastställa ord och fraser som är relevanta, samt för att fastställa vanlig ord ordning. 

Exempel yttranden representerar indata från användaren, till exempel identifierat tal eller text, som appen förväntar sig vid körning. 

Schemat kräver avsikter och _ska ha_ entiteter. 

### <a name="example-schema-of-intents"></a>Exempel på schema för avsikter

Det vanligaste schemat är ett avsikts schema som är ordnat med avsikter. Den här typen av schema använder LUIS för att fastställa användarens avsikt. 

Schema typen avsikt kan ha entiteter om den hjälper LUIS att fastställa användarens avsikt. Till exempel hjälper en frakten het (som en beskrivning till en avsikt) LUIS att fastställa en leverans. 

### <a name="example-schema-of-entities"></a>Exempel schema för entiteter

Ett entitets schema fokuserar på entiteter, vilket är de data som extraheras från User yttranden. Till exempel, om en användare skulle säga "Jag vill beställa tre pizzor". Det finns två entiteter som extraheras: _tre_ och _pizzor_. Dessa används för att hjälpa till att fullfölja avsikten, vilket var att göra en beställning. 

Avsikten med uttryck är mindre viktigt för klient programmet för ett enhets schema. 

En vanlig metod för att organisera ett enhets schema är att lägga till alla exempel yttranden i **ingen** avsikt. 

### <a name="example-of-a-mixed-schema"></a>Exempel på ett blandat schema

Det mest kraftfulla och mogna schemat är ett bevarande schema med en fullständig uppsättning entiteter och funktioner. Det här schemat kan inledas antingen som ett avsikts-eller enhets schema och växer för att inkludera koncepten för båda, eftersom klient programmet behöver dessa delar av information. 

## <a name="add-example-utterances-to-intents"></a>Lägg till exempel yttranden till avsikter

LUIS behöver några exempel yttranden i varje **avsikt**. Exemplet yttranden kräver tillräckligt många Variations alternativ och ord ordning för att kunna avgöra vilken avsikt som uttryck är avsedd för. 

> [!CAUTION]
> Lägg inte till exempel yttranden i bulk. Börja med 15 till 30 olika och varierande exempel. 

Varje uttryck måste ha alla **data som krävs för att extrahera** utformade och märkta med **entiteter**. 

|Nyckel element|Syfte|
|--|--|
|Avsikt|**Klassificera** användar yttranden i en enda avsikt eller åtgärd. Exempel: `BookFlight` och `GetWeather`.|
|Entitet|**Extrahera** data från uttryck som krävs för att slutföra avsikten. Exempel omfattar datum och tid för resor och plats.|

En LUIS-app kan utformas för att ignorera yttranden som inte är relevanta för en Apps domän genom att tilldela uttryck till **ingen** avsikt.

## <a name="test-and-train-your-app"></a>Testa och träna din app

När du har 15 till 30 olika exempel yttranden i varje avsikt, med de entiteter som krävs, måste du testa och [träna](luis-how-to-train.md) din Luis-app. 

## <a name="publish-to-a-prediction-endpoint"></a>Publicera till en förutsägelse slut punkt

LUIS-appen måste publiceras så att den är tillgänglig i list [förutsägelse slut punkts regionerna](luis-reference-regions.md).

## <a name="test-your-published-app"></a>Testa ditt publicerade program

Du kan testa din publicerade LUIS-app från HTTPS förutsägelse-slutpunkten. Om du testar från förutsägelse slut punkten kan LUIS välja valfri yttranden med låg exakthet för [granskning](luis-how-to-review-endpoint-utterances.md).  

## <a name="create-a-new-version-for-each-cycle"></a>Skapa en ny version för varje cykel

Varje version är en ögonblicks bild i LUIS-appen. Innan du gör ändringar i appen skapar du en ny version. Det är enklare att gå tillbaka till en äldre version än att försöka ta bort intentor och yttranden till ett tidigare tillstånd.

Versions-ID: t består av tecken, siffror eller "." och får inte vara längre än 10 tecken.

Den första versionen (0,1) är den aktiva standard versionen. 

### <a name="begin-by-cloning-an-existing-version"></a>Börja med att klona en befintlig version

Klona en befintlig version som ska användas som utgångs punkt för varje ny version. När du har klonat en version blir den nya versionen den **aktiva** versionen. 

### <a name="publishing-slots"></a>Publicerings platser

Du kan publicera på antingen scenen och/eller produktions fack. Varje plats kan ha en annan version eller samma version. Detta är användbart för att verifiera ändringar innan publicering till produktion, som är tillgängligt för robotar eller andra LUIS som anropar appar. 

Utbildade versioner är inte automatiskt tillgängliga i LUIS-appens [slut punkt](luis-glossary.md#endpoint). Du måste [publicera](luis-how-to-publish-app.md) eller publicera om en version för att den ska vara tillgänglig i Luis-appens slut punkt. Du kan publicera till **mellanlagring** och **produktion**, vilket ger dig två versioner av appen tillgängliga vid slut punkten. Om fler versioner av appen måste vara tillgängliga vid en slut punkt bör du exportera versionen och importera den till en ny app. Den nya appen har ett annat app-ID.

### <a name="import-and-export-a-version"></a>Importera och exportera en version

En version kan importeras på App-nivå. Den versionen blir den aktiva versionen och använder versions-ID: `versionId` t i egenskapen för app-filen. Du kan också importera till en befintlig app på versions nivå. Den nya versionen blir den aktiva versionen. 

En version kan också exporteras på program-eller versions nivå. Den enda skillnaden är att den version som exporter ATS på program nivå är den för närvarande aktiva versionen på versions nivå. du kan välja vilken version du vill exportera på sidan **[Inställningar](luis-how-to-manage-versions.md)** . 

Den exporterade filen innehåller **inte** :

* Information som har belärts av datorn, eftersom appen omtränas efter att den har importer ATS
* Deltagar information

För att kunna säkerhetskopiera LUIS-appen exporterar du en version från [Luis-portalen](https://www.luis.ai/applications).

## <a name="manage-contributor-changes-with-versions-and-contributors"></a>Hantera deltagar ändringar med versioner och deltagare

LUIS använder begreppet deltagare i en app genom att tillhandahålla Azure-behörigheter på resurs nivå. Kombinera det här konceptet med versions hantering för att ge riktat samarbete. 

Använd följande tekniker för att hantera deltagar ändringar i din app.

### <a name="manage-multiple-versions-inside-the-same-app"></a>Hantera flera versioner i samma app

Börja med att [klona](luis-how-to-manage-versions.md#clone-a-version) från en bas version för varje författare. 

Varje författare gör ändringar i sin egen version av appen. När författaren är nöjd med modellen exporterar du de nya versionerna till JSON-filer.  

Exporterade appar,. JSON-eller. lu-filer kan jämföras med ändringar. Kombinera filerna för att skapa en enda fil i den nya versionen. Ändra `versionId` egenskapen för att ange den nya sammanfogade versionen. Importera den versionen till den ursprungliga appen. 

Med den här metoden kan du ha en aktiv version, en fas version och en publicerad version. Du kan jämföra resultatet av den aktiva versionen med en publicerad version (fas eller produktion) i fönstret för [interaktiv testning](luis-interactive-test.md).

### <a name="manage-multiple-versions-as-apps"></a>Hantera flera versioner som appar

[Exportera](luis-how-to-manage-versions.md#export-version) bas versionen. Varje författare importerar versionen. Den person som importerar appen är ägaren till versionen. När du är färdig med att ändra appen exporterar du versionen. 

Exporterade appar är JSON-formaterade filer som kan jämföras med bas exporten för ändringar. Kombinera filerna för att skapa en enda JSON-fil för den nya versionen. Ändra egenskapen **versionId** i JSON-filen för att ange den nya sammanfogade versionen. Importera den versionen till den ursprungliga appen.

Läs mer om hur du redigerar bidrag från [medarbetare](luis-how-to-collaborate.md).

## <a name="review-endpoint-utterances-to-begin-the-new-iterative-cycle"></a>Granska slut punkts yttranden för att påbörja den nya iterativa cykeln

När du är färdig med en upprepnings cykel kan du upprepa processen. Börja med att [Granska förutsägelse slut punkt yttranden](luis-how-to-review-endpoint-utterances.md) Luis markerad med låg exakthet. Kontrol lera dessa yttranden för både korrekt förväntad avsikt och korrekt och fullständig extraherad enhet. När du har granskat och godkänt ändringarna bör gransknings listan vara tom.  

## <a name="next-steps"></a>Nästa steg

Lär dig begrepp om [samarbete](luis-concept-keys.md).
