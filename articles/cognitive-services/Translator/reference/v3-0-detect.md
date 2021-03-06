---
title: Translator Text API identifiera metod
titleSuffix: Azure Cognitive Services
description: Identifiera språket för en text del med identifierings metoden för Azure Cognitive Services-Translator Text API.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 02/01/2019
ms.author: swmachan
ms.openlocfilehash: 370f3b14c12fc05f181d6497b7069bbf1cf3c9cc
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "73837302"
---
# <a name="translator-text-api-30-detect"></a>Translator Text API 3,0: identifiera

Identifierar språket för en del av texten.

## <a name="request-url"></a>URL för begäran

Skicka en `POST` begäran till:

```HTTP
https://api.cognitive.microsofttranslator.com/detect?api-version=3.0
```

## <a name="request-parameters"></a>Parametrar för begäran

Parametrarna för begäran som skickades till frågesträngen är:

<table width="100%">
  <th width="20%">Frågeparameter</th>
  <th>Beskrivning</th>
  <tr>
    <td>api-version</td>
    <td>*Obligatorisk parameter*.<br/>Den version av API: t som klienten begär. Värdet måste vara `3.0`.</td>
  </tr>
</table> 

Begärandehuvuden innehåller:

<table width="100%">
  <th width="20%">Rubriker</th>
  <th>Beskrivning</th>
  <tr>
    <td>Authentication-huvud (er)</td>
    <td><em>Begär ande huvud för begäran</em>.<br/>Se <a href="https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication">tillgängliga alternativ för autentisering</a>.</td>
  </tr>
  <tr>
    <td>Content-Type</td>
    <td>*Begär ande huvud för begäran*.<br/>Anger nytto lastens innehålls typ. Möjliga värden är: `application/json`.</td>
  </tr>
  <tr>
    <td>Innehålls längd</td>
    <td>*Begär ande huvud för begäran*.<br/>Längden på begär ande texten.</td>
  </tr>
  <tr>
    <td>X-ClientTraceId</td>
    <td>*Valfritt*.<br/>Ett GUID som skapats av klienten för att unikt identifiera begäran. Observera att du kan utelämna detta sidhuvud om du inkluderar spårnings-ID: t i frågesträngen med hjälp av `ClientTraceId`en frågeparameter med namnet.</td>
  </tr>
</table> 

## <a name="request-body"></a>Begärandetext

Bröd texten i begäran är en JSON-matris. Varje mat ris element är ett JSON-objekt med en sträng `Text`egenskap med namnet. Språk identifiering tillämpas på `Text` egenskapens värde. En exempel begär ande text ser ut så här:

```json
[
    { "Text": "Ich würde wirklich gern Ihr Auto um den Block fahren ein paar Mal." }
]
```

Följande begränsningar gäller:

* Matrisen får innehålla högst 100 element.
* Text värden för ett mat ris element får inte överstiga 10 000 tecken inklusive blank steg.
* Hela texten som ingår i begäran får inte överstiga 50 000 tecken inklusive blank steg.

## <a name="response-body"></a>Själva svaret

Ett lyckat svar är en JSON-matris med ett resultat för varje sträng i den angivna matrisen. Ett resultat objekt innehåller följande egenskaper:

  * `language`: Kod för det identifierade språket.

  * `score`: Ett flyt värde som indikerar förtroendet i resultatet. Poängen är mellan noll och en och en låg poäng indikerar en låg exakthet.

  * `isTranslationSupported`: Ett booleskt värde som är sant om det identifierade språket är ett av de språk som stöds för text översättning.

  * `isTransliterationSupported`: Ett booleskt värde som är sant om det identifierade språket är ett av de språk som stöds för transkriberingsspråk.
  
  * `alternatives`: En matris med andra möjliga språk. Varje element i matrisen är ett annat objekt med samma egenskaper som anges ovan `language`: `score`, `isTranslationSupported` och `isTransliterationSupported`.

Ett exempel på JSON-svar är:

```json
[
  {
    "language": "de",
    "score": 0.92,
    "isTranslationSupported": true,
    "isTransliterationSupported": false,
    "alternatives": [
      {
        "language": "pt",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      },
      {
        "language": "sk",
        "score": 0.23,
        "isTranslationSupported": true,
        "isTransliterationSupported": false
      }
    ]
  }
]
```

## <a name="response-headers"></a>Svarshuvuden

<table width="100%">
  <th width="20%">Rubriker</th>
  <th>Beskrivning</th>
  <tr>
    <td>X-RequestId</td>
    <td>Värde som genereras av tjänsten för att identifiera begäran. Den används i fel söknings syfte.</td>
  </tr>
</table> 

## <a name="response-status-codes"></a>Svars status koder

Följande är de möjliga HTTP-statuskod som en begäran returnerar. 

<table width="100%">
  <th width="20%">Statuskod</th>
  <th>Beskrivning</th>
  <tr>
    <td>200</td>
    <td>Lyckades.</td>
  </tr>
  <tr>
    <td>400</td>
    <td>En av frågeparametrarna saknas eller är ogiltig. Korrigera parametrarna för begäran innan du försöker igen.</td>
  </tr>
  <tr>
    <td>401</td>
    <td>Det gick inte att autentisera begäran. Kontrol lera att autentiseringsuppgifterna har angetts och är giltiga.</td>
  </tr>
  <tr>
    <td>403</td>
    <td>Begäran är inte auktoriserad. Se fel meddelandet information. Detta indikerar ofta att alla kostnads fria översättningar som ingår i en utvärderings prenumeration har använts.</td>
  </tr>
  <tr>
    <td>429</td>
    <td>Servern avvisade begäran på grund av att klienten har överskridit gränsen för begäran.</td>
  </tr>
  <tr>
    <td>500</td>
    <td>Det uppstod ett oväntat fel. Om felet kvarstår rapporterar du det med: datum och tid för felet, begärande-ID från svars huvudet `X-RequestId`och klient-ID: n från begär `X-ClientTraceId`ande huvudet.</td>
  </tr>
  <tr>
    <td>503</td>
    <td>Servern är inte tillgänglig för tillfället. Gör om begäran. Om felet kvarstår rapporterar du det med: datum och tid för felet, begärande-ID från svars huvudet `X-RequestId`och klient-ID: n från begär `X-ClientTraceId`ande huvudet.</td>
  </tr>
</table> 

Om ett fel inträffar returnerar begäran även ett JSON-felsvar. Felkoden är ett 6-siffrigt tal som kombinerar den tresiffriga HTTP-statuskoden följt av ett 3-siffrigt nummer för att ytterligare kategorisera felet. Vanliga felkoder finns på [referens sidan för v3-Translator text API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors). 

## <a name="examples"></a>Exempel

I följande exempel visas hur du hämtar språk som stöds för text översättning.

```curl
curl -X POST "https://api.cognitive.microsofttranslator.com/detect?api-version=3.0" -H "Ocp-Apim-Subscription-Key: <client-secret>" -H "Content-Type: application/json" -d "[{'Text':'What language is this text written in?'}]"
```
