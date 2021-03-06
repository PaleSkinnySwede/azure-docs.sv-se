---
title: Migrera från Translator Speech API till tal tjänsten
titleSuffix: Azure Cognitive Services
description: Lär dig hur du migrerar dina program från Translator Speech API till tal tjänsten.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 01/21/2020
ms.author: aahi
ms.openlocfilehash: 75a456c4a297b0465c34b8e0af2e87056ad565b3
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "77560906"
---
# <a name="migrate-from-the-translator-speech-api-to-the-speech-service"></a>Migrera från Translator Speech API till tal tjänsten

Använd den här artikeln för att migrera dina program från Microsoft Translator Speech API till [tal tjänsten](index.yml). I den här guiden beskrivs skillnaderna mellan Translator Speech API-och tal tjänsten och en översikt över strategier för att migrera dina program.

> [!NOTE]
> Din Translator Speech API prenumerations nyckel accepteras inte av tal tjänsten. Du måste skapa en ny röst tjänst prenumeration.

## <a name="comparison-of-features"></a>Jämförelse av funktioner

| Funktion                                           | Translator Speech API                                  | Tjänst för taligenkänning | Information                                                                                                                                                                                                                                                                            |
|---------------------------------------------------|-----------------------------------------------------------------|------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Översättning till text                               | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Översättning till tal                             | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Global slut punkt                                   | :heavy_check_mark:                                              | : heavy_minus_sign:                 | Tal tjänsten erbjuder inte en global slut punkt. En global slut punkt kan automatiskt dirigera trafik till närmaste regionala slut punkt, vilket minskar svars tiden i ditt program.                                                    |
| Regionala slut punkter                                | : heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Tids gräns för anslutning                             | 90 minuter                                               | Obegränsat med SDK. 10 minuter med en WebSockets-anslutning.                                                                                                                                                                                                                                                                                   |
| Auth Key i rubrik                                | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Flera språk som har översatts i en enskild begäran | : heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| SDK: er är tillgängliga                                    | : heavy_minus_sign:                                              | :heavy_check_mark:                 | Se [dokumentationen för tal tjänst](index.yml) för tillgängliga SDK: er.                                                                                                                                                    |
| WebSockets-anslutningar                            | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Språk-API                                     | :heavy_check_mark:                                              | : heavy_minus_sign:                 | Tal tjänsten har stöd för samma mängd språk som beskrivs i artikeln [Translator-API språk referens](../translator-speech/languages-reference.md) . |
| Filter och markör för svordomar                       | : heavy_minus_sign:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| . WAV/PCM som inmatade                                 | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Andra filtyper som indata                         | : heavy_minus_sign:                                              | : heavy_minus_sign:                 |                                                                                                                                                                                                                                                                                    |
| Partiella resultat                                   | :heavy_check_mark:                                              | :heavy_check_mark:                 |                                                                                                                                                                                                                                                                                    |
| Timing-information                                       | :heavy_check_mark:                                              | : heavy_minus_sign:                 |                                                                                                                                                                 |
| Korrelations-ID                                    | :heavy_check_mark:                                              | : heavy_minus_sign:                 |                                                                                                                                                                                                                                                                                    |
| Anpassade tal modeller                              | : heavy_minus_sign:                                              | :heavy_check_mark:                 | Tal tjänsten erbjuder anpassade tal modeller som gör att du kan anpassa tal igenkänning till organisationens unika vokabulär.                                                                                                                                           |
| Anpassade översättnings modeller                         | : heavy_minus_sign:                                              | :heavy_check_mark:                 | Om du prenumererar på Microsoft text översättnings-API: et kan du använda [anpassad översättare](https://www.microsoft.com/translator/business/customization/) för att använda dina egna data för mer exakta översättningar.                                                 |

## <a name="migration-strategies"></a>Migreringsstrategier

Om du eller din organisation har program som är i utvecklings-eller produktions miljön som använder Translator Speech API bör du uppdatera dem så att de använder tal tjänsten. I dokumentationen för [tal tjänsten](index.yml) finns information om tillgängliga SDK: er, kod exempel och självstudier. Tänk på följande när du migrerar:

* Tal tjänsten erbjuder inte en global slut punkt. Ta reda på om programmet fungerar effektivt när det använder en enda regional slut punkt för all trafik. Om inte, Använd geolokalisering för att fastställa den effektiva slut punkten.

* Om programmet använder långsiktiga anslutningar och inte kan använda de tillgängliga SDK: erna kan du använda en WebSockets-anslutning. Hantera tids gränsen på 10 minuter genom att ansluta vid rätt tidpunkter.

* Om programmet använder Translator Text API och Translator Speech API för att aktivera anpassade översättnings modeller, kan du lägga till kategori-ID: n direkt med hjälp av tal tjänsten.

* Till skillnad från Translator Speech API kan tal tjänsten slutföra översättningar till flera språk i en enskild begäran.

## <a name="next-steps"></a>Nästa steg

* [Prova röst tjänsten kostnads fritt](get-started.md)
* [Snabb start: identifiera tal i en UWP-app med hjälp av talet SDK](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=uwp)

## <a name="see-also"></a>Se även

* [Vad är tal tjänsten](overview.md)
* [Dokumentation om Speech service och Speech SDK](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-devices-sdk-qsg)
