---
title: 'Snabb start: identifiera avvikelser som en batch med hjälp av avvikelse detektor REST API och python'
titleSuffix: Azure Cognitive Services
description: 'Använd API: t för avvikelse detektor för att identifiera avvikelser i din data serie antingen som en batch eller strömmande data i den här snabb starten.'
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: quickstart
ms.date: 03/24/2020
ms.author: aahi
ms.openlocfilehash: 684aba561dc50b64dd7cc564cff8e55229ce1429
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80239032"
---
# <a name="quickstart-detect-anomalies-in-your-time-series-data-using-the-anomaly-detector-rest-api-and-python"></a>Snabb start: identifiera avvikelser i dina tids serie data med hjälp av avvikelse detektor REST API och python

Använd den här snabb starten för att börja använda de två identifierings lägena för avvikelse detektor API: erna för att identifiera avvikelser i dina tids serie data. Detta python-program skickar två API-begäranden som innehåller JSON-formaterade Time Series-data och hämtar svaren.

| API-begäran                                        | Programutdata                                                                                                                         |
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| Identifiera avvikelser som en batch                        | JSON-svaret som innehåller avvikelse status (och andra data) för varje data punkt i tids serie data och positionerna för identifierade avvikelser. |
| Identifiera avvikelse statusen för den senaste data punkten | JSON-svaret som innehåller avvikelse status (och andra data) för den senaste data punkten i tids serie data.                                                                                                                                         |

 Även om det här programmet är skrivet i Python, är API:et en RESTful-webbtjänst som är kompatibel med de flesta programmeringsspråk. Du hittar käll koden för den här snabb starten på [GitHub](https://github.com/Azure-Samples/AnomalyDetector/blob/master/quickstarts/python-detect-anomalies.py).

## <a name="prerequisites"></a>Krav

- [Python 2. x eller 3. x](https://www.python.org/downloads/)
- En avvikelse nyckel och slut punkt
- [Bibliotek för begär Anden](https://pypi.org/project/requests/) för python

- En JSON-fil som innehåller tids serie data punkter. Exempel data för den här snabb starten finns på [GitHub](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/request-data.json).

### <a name="create-an-anomaly-detector-resource"></a>Skapa en resurs för avvikelse detektor

[!INCLUDE [anomaly-detector-resource-creation](../../../../includes/cognitive-services-anomaly-detector-resource-cli.md)]


## <a name="create-a-new-application"></a>Skapa ett nytt program

1. Skapa en ny python-fil och Lägg till följande importer.

    [!code-python[import statements](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=imports)]

2. Skapa variabler för din prenumerations nyckel och din slut punkt. Nedan visas de URI: er som du kan använda för avvikelse identifiering. Dessa kommer att läggas till i tjänst slut punkten senare för att skapa API-begärandena URL: er.

    |Identifieringsmetod  |URI  |
    |---------|---------|
    |Batch-identifiering    | `/anomalydetector/v1.0/timeseries/entire/detect`        |
    |Identifiering på den senaste data punkten     | `/anomalydetector/v1.0/timeseries/last/detect`        |

    [!code-python[initial endpoint and key variables](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=vars)]

3. Läs i JSON-datafilen genom att öppna den och använda `json.load()`.

    [!code-python[Open JSON file and read in the data](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=fileLoad)]

## <a name="create-a-function-to-send-requests"></a>Skapa en funktion för att skicka begär Anden

1. Skapa en ny funktion `send_request()` som använder variablerna som skapats ovan. Utför sedan följande steg.

2. Skapa en ord lista för begärandehuvuden. Ange `Content-Type` till `application/json`och Lägg till din prenumerations nyckel i `Ocp-Apim-Subscription-Key` rubriken.

3. Skicka begäran med hjälp `requests.post()`av. Kombinera URL: en för slut punkt och avvikelse identifiering för URL: en för fullständig begäran och inkludera dina sidhuvud och data för JSON-begäran. Och returnerar sedan svaret.

    [!code-python[request method](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=request)]

## <a name="detect-anomalies-as-a-batch"></a>Identifiera avvikelser som en batch

1. Skapa en metod som `detect_batch()` kallas för att identifiera avvikelser i alla data som en batch. Anropa `send_request()` metoden som skapades ovan med din slut punkt, URL, prenumerations nyckel och JSON-data.

2. Anropa `json.dumps()` resultatet för att formatera det och skriva ut det till-konsolen.

3. Om fältet svar innehåller `code` skriver du ut felkoden och fel meddelandet.

4. Annars hittar du positionerna för avvikelser i data uppsättningen. Svarets `isAnomaly` fält innehåller ett booleskt värde som relaterar till om en viss data punkt är en avvikelse. Upprepa i listan och skriv ut indexet för alla `True` värden. Dessa värden motsvarar indexet för avvikande data punkter, om sådana hittades.

    [!code-python[detection as a batch](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=detectBatch)]

## <a name="detect-the-anomaly-status-of-the-latest-data-point"></a>Identifiera avvikelse statusen för den senaste data punkten

1. Skapa en metod som `detect_latest()` kallas för att avgöra om den senaste data punkten i tids serien är en avvikelse. Anropa `send_request()` metoden ovan med din slut punkt, URL, prenumerations nyckel och JSON-data. 

2. Anropa `json.dumps()` resultatet för att formatera det och skriva ut det till-konsolen.

    [!code-python[Latest point detection](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=detectLatest)]

## <a name="send-the-request"></a>Skicka begäran

Anropa de metoder för avvikelse identifiering som skapats ovan.

[!code-python[Method calls](~/samples-anomaly-detector/quickstarts/python-detect-anomalies.py?name=methodCalls)]

### <a name="example-response"></a>Exempelsvar

Ett lyckat svar returneras i JSON-format. Klicka på länkarna nedan om du vill visa JSON-svaret på GitHub:
* [Exempel svar för batch-identifiering](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/batch-response.json)
* [Exempel svar på senaste Poäng identifiering](https://github.com/Azure-Samples/anomalydetector/blob/master/example-data/latest-point-response.json)

[!INCLUDE [anomaly-detector-next-steps](../includes/quickstart-cleanup-next-steps.md)]
