---
title: Vad är nytt i API för textanalys
titleSuffix: Text Analytics - Azure Cognitive Services
description: Den här artikeln innehåller information om nya versioner och funktioner för Azure Cognitive Services Textanalys.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: aahi
ms.openlocfilehash: 162e60ac8d33dc5d1951a58b0a9643b668608d7b
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77188806"
---
# <a name="whats-new-in-the-text-analytics-api"></a>Vad är nytt i API:et för textanalys?

API för textanalys uppdateras regelbundet. Den här artikeln innehåller information om nya versioner och funktioner för att hålla dig uppdaterad med den senaste utvecklingen.

## <a name="february-2020"></a>Februari 2020

### <a name="sdk-support-for-text-analytics-api-v3-public-preview"></a>SDK-stöd för API för textanalys v3 offentlig för hands version

Som en del av den [enhetliga Azure SDK-versionen](https://techcommunity.microsoft.com/t5/azure-sdk/january-2020-unified-azure-sdk-release/ba-p/1097290)är API för TEXTANALYS v3 SDK nu tillgängligt som en offentlig för hands version för följande programmeringsspråk:
   * [C #](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/text-analytics-sdk?tabs=version-3&pivots=programming-language-csharp)
   * [Python](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/text-analytics-sdk?tabs=version-3&pivots=programming-language-python)
   * [Java Script (Node. js)](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/text-analytics-sdk?tabs=version-3&pivots=programming-language-javascript)
   * [Java](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/text-analytics-sdk?tabs=version-3&pivots=programming-language-java)

> [!div class="nextstepaction"]
> [Läs mer om API för textanalys v3 SDK](https://docs.microsoft.com/azure/cognitive-services/text-analytics/quickstarts/text-analytics-sdk?tabs=version-3)

### <a name="named-entity-recognition-v3-public-preview"></a>Namngiven enhets igenkänning v3 offentlig för hands version

Ytterligare entitetstyper är nu tillgängliga i den namngivna enhets igenkännings tjänsten (NER) v3 offentlig för hands version eftersom vi expanderar identifieringen av allmänna och personliga informations enheter i text. Den här uppdateringen introducerar [modell versionen](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-versions-and-features) `2020-02-01`, som innehåller:

* Igenkänning av följande allmänna entitetstyper (endast engelska):
    * PersonType
    * Produkt
    * Händelse
    * GPE (politisk entitet) som en undertyp under plats
    * Kvalifikation

* Igenkänning av följande entitets typer för personlig information (endast engelska):
    * Person
    * Organisation
    * Ålder som underordnad kvantitet
    * Datum som en undertyp under DateTime
    * E-post 
    * Telefonnummer (endast USA)
    * URL
    * IP-adress

> [!div class="nextstepaction"]
> [Läs mer om namngiven enhets igenkänning v3](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-versions-and-features)

### <a name="october-2019"></a>Oktober 2019

#### <a name="named-entity-recognition-ner"></a>Igenkänning av namngivna enheter (NER)

* En [ny slut punkt](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0-Preview-1/operations/EntitiesRecognitionPii) för att identifiera entitets typer för personlig information (endast engelska)

* Separata slut punkter för [entitets igenkänning](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0-Preview-1/operations/EntitiesRecognitionGeneral) och [entitet länkning](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0-Preview-1/operations/EntitiesLinking).

* [Modell version](how-tos/text-analytics-how-to-entity-linking.md#named-entity-recognition-versions-and-features) `2019-10-01`, som innehåller:
    * Utökad identifiering och kategorisering av entiteter som finns i text. 
    * Igenkänning av följande nya enhets typer:
        * Telefonnummer
        * IP-adress

Enhets länkning stöder engelska och spanska. Språk stöd för NER varierar beroende på enhets typen.

#### <a name="sentiment-analysis-v3-public-preview"></a>Attitydanalys v3 offentlig för hands version

* En [ny slut punkt](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-v3-0-Preview-1/operations/Sentiment) för att analysera sentiment.
* [Modell version](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features) `2019-10-01`, som innehåller:

    * Betydande förbättringar av precisionen och detaljerna i API: ns text kategorisering och poängsättning.
    * Automatisk etikettering för olika sentiment i text.
    * Sentiment analys och utdata på en dokument-och menings nivå. 

Det stöder engelska (`en`), japanska (`ja`), kinesiska (förenklad`zh-Hans`), kinesiska (traditionell`zh-Hant`), franska (`fr`), italienska (`it`), spanska (`es`), nederländska (`nl`), portugisiska`pt``de`() och tyska (), och är tillgängliga i följande regioner: `Australia East`, `Central Canada`, `Central US` `East Asia` `East US` `East US 2` `North Europe` `Southeast Asia` `South Central US` `UK South` `West Europe`,,,,,,,, och. `West US 2` 

> [!div class="nextstepaction"]
> [Läs mer om Attitydanalys v3](how-tos/text-analytics-how-to-sentiment-analysis.md#sentiment-analysis-versions-and-features)

## <a name="next-steps"></a>Nästa steg

* [Vad är API för textanalys?](overview.md)  
* [Exempel på användarscenarier](text-analytics-user-scenarios.md)
* [Sentimentanalys](how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Språk identifiering](how-tos/text-analytics-how-to-language-detection.md)
* [Enhets igenkänning](how-tos/text-analytics-how-to-entity-linking.md)
* [Extrahering av nyckel fraser](how-tos/text-analytics-how-to-keyword-extraction.md)
