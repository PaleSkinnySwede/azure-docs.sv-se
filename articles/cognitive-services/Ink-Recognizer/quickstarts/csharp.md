---
title: 'Snabb start: identifiera digitalt bläck med hand SKRIFTS tolken REST API och C #'
titleSuffix: Azure Cognitive Services
description: 'Den här snabb starten visar hur du kan använda pennan tecknings-API: t för att börja känna igen digitala penndrag.'
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 12/17/2019
ms.author: aahi
ms.openlocfilehash: c24d055f1904453d2f512a278f00e23c6fea1d9b
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80371383"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-c"></a>Snabb start: identifiera digitalt bläck med hand SKRIFTS tolken REST API och C #

Använd den här snabb starten för att börja skicka digitala penndrag till färg igenkännings-API: et. Det här C#-programmet skickar en API-begäran som innehåller JSON-formaterade penndrag och hämtar svaret.

Även om det här programmet är skrivet i C#, är API:et en RESTful-webbtjänst som är kompatibel med de flesta programmeringsspråk.

Normalt anropar du API: et från en digital intecknings app. I den här snabb starten skickas Penn strecks data för följande handskrivna exempel från en JSON-fil.

![en bild av handskriven text](../media/handwriting-sample.jpg)

Du hittar käll koden för den här snabb starten på [GitHub](https://go.microsoft.com/fwlink/?linkid=2089502).

## <a name="prerequisites"></a>Krav

- Valfri version av [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/).
- [Newtonsoft.Json](https://www.newtonsoft.com/json)
    - Så här installerar du Newtonsoft. JSON som ett NuGet-paket i Visual Studio:
        1. Högerklicka på **lösnings hanteraren**
        2. Klicka på **Hantera NuGet-paket...**
        3. Sök efter `Newtonsoft.Json` och installera paketet
- Om du använder Linux/MacOS kan du köra det här programmet med [mono](https://www.mono-project.com/).

- Du hittar exempel Penn strecks data för den här snabb starten på [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Skapa en tryck färgs igenkännings resurs

[!INCLUDE [creating-an-ink-recognizer-resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Skapa ett nytt program

1. Skapa en ny konsol lösning i Visual Studio och Lägg till följande paket. 
    
    [!code-csharp[import statements](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=imports)]

2. Skapa variabler för din prenumerations nyckel och slut punkt och exempel-JSON-filen. Slut punkten kommer senare att kombineras `inkRecognitionUrl` med för att få åtkomst till API: et. 

    [!code-csharp[endpoint file and key variables](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=vars)]

## <a name="create-a-function-to-send-requests"></a>Skapa en funktion för att skicka begär Anden

1. Skapa en ny async-funktion `Request` som använder variablerna som skapats ovan.

2. Ange klientens säkerhets protokoll och huvud information med hjälp av `HttpClient` ett objekt. Se till att lägga till din prenumerations nyckel `Ocp-Apim-Subscription-Key` i rubriken. Skapa sedan ett `StringContent` objekt för begäran.
 
3. Skicka begäran med `PutAsync()`. Returnera svaret om begäran lyckas.  
    
    [!code-csharp[request example method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=request)]

## <a name="send-an-ink-recognition-request"></a>Skicka en begäran om hand SKRIFTS igenkänning

1. Skapa en ny funktion som `recognizeInk()`kallas. Skapa begäran och skicka den genom att anropa `Request()` funktionen med din slut punkt, prenumerations nyckel, URL: en för API: et och den digitala Penn linjens data.

2. Deserialisera JSON-objektet och skriv det till-konsolen. 
    
    [!code-csharp[request to recognize ink data](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=recognize)]

## <a name="load-your-digital-ink-data"></a>Läs in digitala pennan tecknings data

Skapa en funktion som `LoadJson()` kallas för att läsa in JSON-filen med bläck data. Använd en `StreamReader` och `JsonTextReader` för att skapa `JObject` en och returnera den.

[!code-csharp[load the JSON file](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=loadJson)]

## <a name="send-the-api-request"></a>Skicka API-begäran

1. I appens huvud metod läser du in dina JSON-data med funktionen som du skapade ovan. 

2. Anropa funktionen `recognizeInk()` som skapats ovan. Använd `System.Console.ReadKey()` för att låta konsol fönstret vara öppet när du har kört programmet.
    
    [!code-csharp[file main method](~/cognitive-services-rest-samples/dotnet/Vision/InkRecognition/quickstart/recognizeInk.cs?name=main)]


## <a name="run-the-application-and-view-the-response"></a>Kör programmet och Visa svaret

Kör appen. Ett lyckat svar returneras i JSON-format. Du kan också hitta JSON-svaret på [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/dotnet/Vision/InkRecognition/quickstart/example-response.json).


## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Referens för REST-API](https://go.microsoft.com/fwlink/?linkid=2089907)


Ta en titt på följande exempel program på GitHub för att se hur API: et för färg igenkänning fungerar i en digital inkungs app:
* [C# och Universal Windows-plattform (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# och Windows Presentation Foundation(WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [JavaScript-webbläsarappen](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Java- och Android-mobilapp](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Swift- och iOS-mobilapp](https://go.microsoft.com/fwlink/?linkid=2089805)
