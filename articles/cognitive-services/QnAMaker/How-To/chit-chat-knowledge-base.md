---
title: Lägga till en CHI2TEST-chatt till en QnA Maker kunskaps bas
titleSuffix: Azure Cognitive Services
description: Om du lägger till en personlig CHI2TEST i din robot blir det mer samtal och engagerande när du skapar en KB. Med QnA Maker kan du enkelt lägga till en i förväg ifylld uppsättning av den främsta CHI2TEST i din KB.
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: diberry
ms.custom: seodec18
ms.openlocfilehash: a9a14056e6be62fc1c1b5e542c1a3acceb738eac
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "79220714"
---
# <a name="add-chit-chat-to-a-knowledge-base"></a>Lägga till en Chi2test-chatt i en kunskaps bas

Att lägga till en CHI2TEST i din robot gör det mer samtal och engagerande. Med funktionen CHI2TEST i QnA Maker kan du enkelt lägga till en i förväg ifylld uppsättning av den främsta CHI2TEST i din kunskaps bas (KB). Detta kan vara en utgångs punkt för din robots personlighet, och det sparar tid och kostnad för att skriva dem från grunden.  

Den här data uppsättningen har cirka 100 scenarier med CHI2TEST i en röst av flera personer, som professionella, användarvänliga och witty. Välj den person som bäst liknar din robots röst. Med en användar fråga försöker QnA Maker matcha den med den närmast kända QnA i CHI2TEST.  

Några exempel på olika personliga uppgifter finns nedan. Du kan se alla personliga [data uppsättningar](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets) tillsammans med information om dina personliga uppgifter.

För användar frågan för `When is your birthday?`, har varje personlighet ett formaterat svar:

<!-- added quotes so acrolinx doesn't score these sentences -->
|Personlighet|Exempel|
|--|--|
|Professionell|Åldern gäller egentligen inte för mig.|
|Eget|Jag har verkligen ingen ålder.|
|Witty|Jag är ålders fri.|
|Caring|Jag har ingen ålder.|
|Entusiastisk|Jag är en bot, så jag har ingen ålder.|
||


## <a name="language-support"></a>Stöd för språk

Data uppsättningar för CHI2TEST-chatt stöds på följande språk:

|Språk|
|--|
|Kinesiska|
|Svenska|
|Franska|
|Tyskland|
|Italienska|
|Japanska|
|Koreansk|
|Portugisiska|
|Spanska|


## <a name="add-chit-chat-during-kb-creation"></a>Lägga till en CHI2TEST-chatt när KB skapas
När du har lagt till käll-URL: er och filer i kunskaps basen finns det ett alternativ för att lägga till en CHI2TEST. Välj den personlighet som du vill använda som din databas för CHI2TEST. Om du inte vill lägga till en CHI2TEST, eller om du redan har stöd för CHI2TEST i dina data källor, väljer du **ingen**. 

## <a name="add-chit-chat-to-an-existing-kb"></a>Lägga till en Chi2test i en befintlig KB
Välj din KB och gå till sidan **Inställningar** . Det finns en länk till alla data uppsättningar för CHI2TEST i lämpligt **. tsv** -format. Ladda ned den personlighet som du vill ha och ladda sedan upp den som en fil källa. Se till att du inte redigerar formatet eller metadata när du laddar ned och laddar upp filen. 
  
![Lägg till en CHI2TEST till befintlig KB](../media/qnamaker-how-to-chit-chat/add-chit-chat-dataset.png)

## <a name="edit-your-chit-chat-questions-and-answers"></a>Redigera frågor och svar om CHI2TEST
När du redigerar din KB visas en ny källa för CHI2TEST, baserat på den personlighet du har valt. Du kan nu lägga till ändrade frågor eller redigera svaren, precis som med andra källor. 

![Redigera kring för CHI2TEST](../media/qnamaker-how-to-chit-chat/edit-chit-chat.png)

Om du vill visa metadata väljer du **visnings alternativ** i verktygsfältet och väljer sedan **Visa metadata**.

## <a name="add-additional-chit-chat-questions-and-answers"></a>Lägg till fler frågor och svar om CHI2TEST
Du kan lägga till en ny CHI2TEST QnA som inte finns i den fördefinierade uppsättningen. Se till att du inte duplicerar ett QnA-par som redan ingår i uppsättningen CHI2TEST. När du lägger till nya QnA i CHI2TEST läggs den till i din **redigerings** källa. För att se till att rangordningen förstår att detta är CHI2TEST, Lägg till nyckel/värde-paret "redaktionell: ChitChat", som visas i följande bild:
   
![! [Lägg till CHI2TEST kring] (.. /media/qnamaker-how-to-chit-chat/add-new-chit-chat.png)](../media/qnamaker-how-to-chit-chat/add-new-chit-chat.png#lightbox)

## <a name="delete-chit-chat-from-an-existing-kb"></a>Ta bort CHI2TEST från en befintlig KB
Välj din KB och gå till sidan **Inställningar** . Din aktuella CHI2TEST-Chat-källa visas som en fil med det valda personliga namnet. Du kan ta bort den som en käll fil.

![Ta bort CHI2TEST från KB](../media/qnamaker-how-to-chit-chat/delete-chit-chat.png)

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Importera en kunskaps bas](../Tutorials/migrate-knowledge-base.md)

## <a name="see-also"></a>Se även 

[Översikt över QnA Maker](../Overview/overview.md)
