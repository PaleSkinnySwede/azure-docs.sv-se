---
title: 'Snabb start: Textanalys klient bibliotek för ruby | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: I den här snabb starten identifierar du språk genom att använda ruby-Textanalys klient biblioteket från Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 02/26/2020
ms.author: aahi
ms.openlocfilehash: 0d4d32a413dd22c55f1b2f01dce3a3df81f5f729
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77919676"
---
# <a name="quickstart-use-the-text-analytics-client-library-for-ruby"></a>Snabb start: Använd Textanalys klient bibliotek för ruby

Kom igång med Textanalys klient biblioteket. Följ de här stegen för att installera paketet och prova exempel koden för grundläggande uppgifter.

Använd Textanalys klient bibliotek för att utföra:

* Sentimentanalys
* Språkidentifiering
* Enhetsidentifiering
* Extrahering av nyckelfraser

> [!NOTE]
> Den här snabb starten gäller endast Textanalys version 2,1. För närvarande är ett v3-klient bibliotek för ruby inte tillgängligt.

[Referens dokumentation](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/textanalytics?view=azure-python) | [bibliotek käll kods](https://github.com/Azure/azure-sdk-for-ruby/tree/master/data/azure_cognitiveservices_textanalytics) | [paket (rubygems)](https://rubygems.org/gems/azure_cognitiveservices_textanalytics) | [exempel](https://github.com/Azure-Samples/cognitive-services-quickstart-code)

<a name="HOLTop"></a>

## <a name="prerequisites"></a>Krav

* En Azure-prenumeration – [skapa en kostnads fritt](https://azure.microsoft.com/free/)
* Aktuell version av [ruby](https://www.ruby-lang.org/)
* När du har en Azure <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics"  title="-prenumeration skapar du en textanalys"  target="_blank">resurs skapa en textanalys <span class="docon docon-navigate-external x-hidden-focus"></span> </a> resurs i Azure Portal för att hämta din nyckel och slut punkt. 
    * Du behöver nyckeln och slut punkten från den resurs som du skapar för att ansluta ditt program till API för textanalys. Du kommer att göra detta senare i snabb starten.
    * Du kan använda den kostnads fria pris nivån för att testa tjänsten och senare uppgradera till en betald nivå för produktion.

## <a name="setting-up"></a>Konfigurera

### <a name="create-a-new-ruby-application"></a>Skapa ett nytt ruby-program

I ett konsol fönster (till exempel cmd, PowerShell eller bash) skapar du en ny katalog för din app och navigerar till den. Skapa sedan en fil med `GemFile`namnet och en ruby-fil för din kod.

```console
mkdir myapp && cd myapp
```

`GemFile`I lägger du till följande rader för att lägga till klient biblioteket som ett beroende.

```ruby
source 'https://rubygems.org'
gem 'azure_cognitiveservices_textanalytics', '~>0.17.3'
```

Importera följande paket i ruby-filen.

[!code-ruby[Import statements](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=includeStatement)]

Skapa variabler för resursens Azure-slutpunkt och nyckel. 

[!INCLUDE [text-analytics-find-resource-information](../includes/find-azure-resource-info.md)]

```ruby
const subscription_key = '<paste-your-text-analytics-key-here>'
const endpoint = `<paste-your-text-analytics-endpoint-here>`
```

## <a name="object-model"></a>Objekt modell 

Textanalys klienten autentiserar till Azure med hjälp av din nyckel. Klienten tillhandahåller flera metoder för att analysera text, som en enskild sträng eller en batch. 

Text skickas till API: et som en lista över `documents`, som är `dictionary` objekt som innehåller en kombination `id`av `text`,, `language` och attribut beroende på vilken metod som används. `text` Attributet lagrar texten som ska analyseras i ursprunget `language`och `id` kan vara vilket värde som helst. 

Objektet Response är en lista som innehåller analys informationen för varje dokument. 

## <a name="code-examples"></a>Kodexempel

De här kodfragmenten visar hur du gör följande med Textanalys klient biblioteket för python:

* [Autentisera klienten](#authenticate-the-client)
* [Attitydanalys](#sentiment-analysis)
* [Språk identifiering](#language-detection)
* [Enhets igenkänning](#entity-recognition)
* [Extrahering av nyckel fraser](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>Autentisera klienten

Skapa en klass med `TextAnalyticsClient`namnet. 

```ruby
class TextAnalyticsClient
  @textAnalyticsClient
  #...
end
```

I den här klassen skapar du en funktion `initialize` som kallas för att autentisera klienten med hjälp av din nyckel och slut punkt. 

[!code-ruby[initialize function for authentication](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=initialize)]

Utanför klassen använder du klientens `new()` funktion för att instansiera den.

[!code-ruby[client creation](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=clientCreation)] 

<a name="SentimentAnalysis"></a>

## <a name="sentiment-analysis"></a>Sentimentanalys

I objektet klient skapar du en funktion som kallas `AnalyzeSentiment()` för en lista med indatamängds dokument som ska skapas senare. Anropa klientens `sentiment()` funktion och få resultatet. Iterera sedan igenom resultaten och skriv ut varje dokuments ID och sentiment poäng. En poäng närmare 0 anger ett negativt sentiment, medan ett resultat närmare 1 anger en positiv sentiment.

[!code-ruby[client method for sentiment analysis](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=analyzeSentiment)] 

Utanför klient funktionen skapar du `SentimentAnalysisExample()` `TextAnalyticsClient` en ny funktion som tar objektet som skapats tidigare. Skapa en lista med `MultiLanguageInput` objekt som innehåller dokumenten som du vill analysera. Varje-objekt kommer att `id`innehålla `Language` ett- `text` och-attribut. `text` Attributet lagrar texten som ska analyseras, `language` är språket i dokumentet och `id` kan vara vilket värde som helst. Anropa sedan klientens `AnalyzeSentiment()` funktion.

[!code-ruby[sentiment analysis document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=sentimentCall)] 

Anropa funktionen `SentimentAnalysisExample()`.

```ruby
SentimentAnalysisExample(textAnalyticsClient)
```

### <a name="output"></a>Resultat

```console
===== SENTIMENT ANALYSIS =====
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

<a name="LanguageDetection"></a>

## <a name="language-detection"></a>Språkidentifiering

I objektet klient skapar du en funktion som kallas `DetectLanguage()` för en lista med indatamängds dokument som ska skapas senare. Anropa klientens `detect_language()` funktion och få resultatet. Iterera sedan igenom resultaten och skriv ut varje dokuments ID och identifierat språk.

[!code-ruby[client method for language detection](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguage)] 

Utanför klient funktionen skapar du `DetectLanguageExample()` `TextAnalyticsClient` en ny funktion som tar objektet som skapats tidigare. Skapa en lista med `LanguageInput` objekt som innehåller dokumenten som du vill analysera. Varje-objekt kommer att `id`innehålla ett- `text` och-attribut. `text` Attributet lagrar texten som ska analyseras och `id` kan vara vilket värde som helst. Anropa sedan klientens `DetectLanguage()` funktion.

[!code-ruby[language detection document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguageCall)] 

Anropa funktionen `DetectLanguageExample()`.

```ruby
DetectLanguageExample(textAnalyticsClient)
```

### <a name="output"></a>Resultat

```console
===== LANGUAGE EXTRACTION ======
Document ID: 1 , Language: English
Document ID: 2 , Language: Spanish
Document ID: 3 , Language: Chinese_Simplified
```

<a name="EntityRecognition"></a>

## <a name="entity-recognition"></a>Enhetsidentifiering

I objektet klient skapar du en funktion som kallas `RecognizeEntities()` för en lista med indatamängds dokument som ska skapas senare. Anropa klientens `entities()` funktion och få resultatet. Iterera sedan igenom resultaten och skriv ut varje dokuments ID och de identifierade enheterna.

[!code-ruby[client method for entity recognition](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntities)]

Utanför klient funktionen skapar du `RecognizeEntitiesExample()` `TextAnalyticsClient` en ny funktion som tar objektet som skapats tidigare. Skapa en lista med `MultiLanguageInput` objekt som innehåller dokumenten som du vill analysera. Varje-objekt kommer att `id`innehålla ett `language` `text` attribut, a och. `text` Attributet lagrar texten som ska analyseras, `language` är språk för texten och `id` kan vara vilket värde som helst. Anropa sedan klientens `RecognizeEntities()` funktion.

[!code-ruby[entity recognition documents and method call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntitiesCall)] 

Anropa funktionen `RecognizeEntitiesExample()`.

```ruby
RecognizeEntitiesExample(textAnalyticsClient)
```

### <a name="output"></a>Resultat

```console
===== ENTITY RECOGNITION =====
Document ID: 1
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 0, Length: 9,   Score: 1.0

        Name: Bill Gates,       Type: Person,   Sub-Type: N/A
        Offset: 25, Length: 10, Score: 0.999847412109375

        Name: Paul Allen,       Type: Person,   Sub-Type: N/A
        Offset: 40, Length: 10, Score: 0.9988409876823425

        Name: April 4,  Type: Other,    Sub-Type: N/A
        Offset: 54, Length: 7,  Score: 0.8

        Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
        Offset: 54, Length: 13, Score: 0.8

        Name: BASIC,    Type: Other,    Sub-Type: N/A
        Offset: 89, Length: 5,  Score: 0.8

        Name: Altair 8800,      Type: Other,    Sub-Type: N/A
        Offset: 116, Length: 11,        Score: 0.8

Document ID: 2
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 21, Length: 9,  Score: 0.999755859375

        Name: Redmond (Washington),     Type: Location, Sub-Type: N/A
        Offset: 60, Length: 7,  Score: 0.9911284446716309

        Name: 21 kilómetros,    Type: Quantity, Sub-Type: Dimension
        Offset: 71, Length: 13, Score: 0.8

        Name: Seattle,  Type: Location, Sub-Type: N/A
        Offset: 88, Length: 7,  Score: 0.9998779296875
```

<a name="KeyPhraseExtraction"></a>

## <a name="key-phrase-extraction"></a>Extrahering av nyckelfraser

I objektet klient skapar du en funktion som kallas `ExtractKeyPhrases()` för en lista med indatamängds dokument som ska skapas senare. Anropa klientens `key_phrases()` funktion och få resultatet. Iterera sedan igenom resultaten och skriv ut varje dokuments ID och de extraherade nyckel fraserna.

[!code-ruby[key phrase extraction client method](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=extractKeyPhrases)] 

Utanför klient funktionen skapar du `KeyPhraseExtractionExample()` `TextAnalyticsClient` en ny funktion som tar objektet som skapats tidigare. Skapa en lista med `MultiLanguageInput` objekt som innehåller dokumenten som du vill analysera. Varje-objekt kommer att `id`innehålla ett `language` `text` attribut, a och. `text` Attributet lagrar texten som ska analyseras, `language` är språk för texten och `id` kan vara vilket värde som helst. Anropa sedan klientens `ExtractKeyPhrases()` funktion.

[!code-ruby[key phrase document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=keyPhrasesCall)]


Anropa funktionen `KeyPhraseExtractionExample()`.

```ruby
KeyPhraseExtractionExample(textAnalyticsClient)
```

### <a name="output"></a>Resultat

```console
Document ID: 1
         Key phrases:
                幸せ
Document ID: 2
         Key phrases:
                Stuttgart
                Hotel
                Fahrt
                Fu
Document ID: 3
         Key phrases:
                cat
                rock
Document ID: 4
         Key phrases:
                fútbol
```
