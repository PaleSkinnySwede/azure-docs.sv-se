---
author: trevorbye
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 01/27/2020
ms.author: trbye
ms.openlocfilehash: a41c9dc888e85baf021712e3302da7ae6106db47
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81422008"
---
För att slutföra snabb starten för avsikts igenkänning måste du skapa ett LUIS-konto och ett projekt med hjälp av LUIS Preview Portal. Den här snabb starten kräver bara en LUIS-prenumeration. En röst tjänst prenumeration krävs *inte* .

Det första du behöver göra är att skapa ett LUIS-konto och en app med hjälp av LUIS Preview Portal. LUIS-appen som du skapar använder en fördefinierad domän för start automatisering, som tillhandahåller avsikter, entiteter och exempel yttranden. När du är klar har du en LUIS-slutpunkt som körs i molnet som du kan ringa med hjälp av tal-SDK. 

Följ de här anvisningarna för att skapa en LUIS-app:

* <a href="https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app" target="_blank">Snabb start: skapa en fördefinierad domän-app<span class="docon docon-navigate-external x-hidden-focus"></span></a>

När du är klar behöver du fyra saker:

* Publicera igen med **tal Prima** aktiverat
* Din LUIS- **primära nyckel**
* Din LUIS- **plats**
* Ditt LUIS **app-ID**

Här kan du hitta den här informationen i [Luis Preview Portal](https://preview.luis.ai/):

1. Välj din app från LUIS Preview-portalen och välj sedan knappen **publicera** .

2. Välj **produktions** platsen, om du använder `en-US` växla alternativet för **tal Prima** till **på** plats. Välj sedan knappen **publicera** .

    > [!IMPORTANT]
    > **Tal Prima** rekommenderas eftersom det kommer att förbättra precisionen för tal igenkänning.

    > [!div class="mx-imgBorder"]
    > ![Publicera LUIS till slut punkten](../../../media/luis/publish-app-popup.png)

3. Välj **Hantera**på Luis Preview-portalen och välj sedan **Azure-resurser**. På den här sidan hittar du din LUIS-nyckel och plats (kallas ibland _region_).

   > [!div class="mx-imgBorder"]
   > ![LUIS nyckel och plats](../../../media/luis/luis-key-region.png)

4. När du har fått nyckeln och platsen behöver du app-ID: t. Välj **program inställningar** – ditt app-ID är tillgängligt på den här sidan.

   > [!div class="mx-imgBorder"]
   > ![LUIS app-ID](../../../media/luis/luis-app-id.png)
