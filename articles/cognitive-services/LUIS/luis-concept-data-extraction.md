---
title: Data extrahering – LUIS
description: Extrahera data från uttryck text med avsikter och entiteter. Lär dig vilken typ av data som kan extraheras från Language Understanding (LUIS).
author: diberry
ms.topic: conceptual
ms.date: 01/23/2020
ms.openlocfilehash: 3cea4a46564210ad8c37fdeda68e24337091d0bb
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82100302"
---
# <a name="extract-data-from-utterance-text-with-intents-and-entities"></a>Extrahera data från uttryck text med avsikter och entiteter
LUIS ger dig möjlighet att hämta information från en användares naturliga språk yttranden. Informationen extraheras på ett sätt som kan användas av ett program, program eller en chatt-robot för att vidta åtgärder. I följande avsnitt lär du dig vilka data som returneras från avsikter och entiteter med exempel på JSON.

De hårda data som ska extraheras är de data som lagras på datorn eftersom det inte är en exakt text matchning. Data extrahering av de datorbaserade [enheterna](luis-concept-entity-types.md) måste vara en del av [redigerings cykeln](luis-concept-app-iteration.md) tills du är säker på att du får de data du förväntar dig.

## <a name="data-location-and-key-usage"></a>Data plats och nyckel användning
LUIS tillhandahåller data från den publicerade [slut punkten](luis-glossary.md#endpoint). **Https-begäran** (post eller get) innehåller uttryck samt vissa valfria konfigurationer som till exempel mellanlagrings-eller produktions miljöer.

#### <a name="v2-prediction-endpoint-request"></a>[V2-begäran om slut punkts förutsägelse](#tab/V2)

`https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/<appID>?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&q=book 2 tickets to paris`

#### <a name="v3-prediction-endpoint-request"></a>[V3 förutsägelse slut punkts förfrågan](#tab/V3)

`https://westus.api.cognitive.microsoft.com/luis/v3.0-preview/apps/<appID>/slots/<slot-type>/predict?subscription-key=<subscription-key>&verbose=true&timezoneOffset=0&query=book 2 tickets to paris`

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

Är tillgänglig på sidan **Inställningar** i Luis-appen samt en del av URL: en (efter `/apps/`) när du redigerar den Luis-appen. `appID` `subscription-key` Är den slut punkts nyckel som används för att skicka frågor till din app. Även om du kan använda din kostnads fria redigerings-/start nyckel när du lär dig LUIS, är det viktigt att ändra slut punkts nyckeln till en nyckel som stöder din [förväntade Luis-användning](luis-limits.md#key-limits). `timezoneOffset` Enheten är minuter.

**Https-svaret** innehåller all information om Luis och entiteten som kan fastställas baserat på den aktuella publicerade modellen för antingen mellanlagrings-eller produktions slut punkten. Slut punkts-URL: en finns på [Luis](luis-reference-regions.md) -webbplatsen i avsnittet **Hantera** på sidan **nycklar och slut punkter** .

## <a name="data-from-intents"></a>Data från avsikter
Primära data är det översta namnet på bedömnings **avsikten**. Slut punktens svar är:

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

```JSON
{
  "query": "when do you open next?",
  "prediction": {
    "normalizedQuery": "when do you open next?",
    "topIntent": "GetStoreInfo",
    "intents": {
        "GetStoreInfo": {
            "score": 0.984749258
        }
    }
  },
  "entities": []
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

|Data objekt|Datatyp|Dataplats|Värde|
|--|--|--|--|
|Avsikt|Sträng|topScoringIntent. avsikt|"GetStoreInfo"|

Om din chattrobot-eller LUIS-anropande app fattar ett beslut baserat på fler än ett avsikts poäng, returnerar du alla Intents-resultat.


#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

Ange parametern QueryString, `verbose=true`. Slut punktens svar är:

```JSON
{
  "query": "when do you open next?",
  "topScoringIntent": {
    "intent": "GetStoreInfo",
    "score": 0.984749258
  },
  "intents": [
    {
      "intent": "GetStoreInfo",
      "score": 0.984749258
    },
    {
      "intent": "None",
      "score": 0.2040639
    }
  ],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

Ange parametern QueryString, `show-all-intents=true`. Slut punktens svar är:

```JSON
{
    "query": "when do you open next?",
    "prediction": {
        "normalizedQuery": "when do you open next?",
        "topIntent": "GetStoreInfo",
        "intents": {
            "GetStoreInfo": {
                "score": 0.984749258
            },
            "None": {
                 "score": 0.2040639
            }
        },
        "entities": {
        }
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

Avsikterna är sorterade från högsta till lägsta poäng.

|Data objekt|Datatyp|Dataplats|Värde|Poäng|
|--|--|--|--|:--|
|Avsikt|Sträng|avsikter [0]. avsikt|"GetStoreInfo"|0,984749258|
|Avsikt|Sträng|avsikter [1]. avsikt|Alternativet|0,0168218873|

Om du lägger till fördefinierade domäner, anger namn på avsikt domänen, t. ex. `Utilties` eller `Communication` och avsikten:

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
{
  "query": "Turn on the lights next monday at 9am",
  "topScoringIntent": {
    "intent": "Utilities.ShowNext",
    "score": 0.07842206
  },
  "intents": [
    {
      "intent": "Utilities.ShowNext",
      "score": 0.07842206
    },
    {
      "intent": "Communication.StartOver",
      "score": 0.0239675418
    },
    {
      "intent": "None",
      "score": 0.0168218873
    }],
  "entities": []
}
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

```JSON
{
    "query": "Turn on the lights next monday at 9am",
    "prediction": {
        "normalizedQuery": "Turn on the lights next monday at 9am",
        "topIntent": "Utilities.ShowNext",
        "intents": {
            "Utilities.ShowNext": {
                "score": 0.07842206
            },
            "Communication.StartOver": {
                "score": 0.0239675418
            },
            "None": {
                "score": 0.00085447653
            }
        },
        "entities": []
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

|Domain|Data objekt|Datatyp|Dataplats|Värde|
|--|--|--|--|--|
|Verktyg|Avsikt|Sträng|avsikter [0]. avsikt|"<b>Verktyg</b>. ShowNext"|
|Kommunikation|Avsikt|Sträng|avsikter [1]. avsikt|<b>Kommunikation</b>. StartOver"|
||Avsikt|Sträng|avsikter [2]. avsikt|Alternativet|


## <a name="data-from-entities"></a>Data från entiteter
De flesta chattrobotar och program behöver mer än namnet på avsikten. Detta ytterligare, valfria data kommer från entiteter som identifierats i uttryck. Varje typ av entitet returnerar annan information om matchningen.

Ett enstaka ord eller en fras i en uttryck kan matcha mer än en entitet. I så fall returneras varje matchande entitet med poängen.

Alla entiteter returneras i matrisen **entiteter** för svaret från slut punkten:

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
"entities": [
  {
    "entity": "bob jones",
    "type": "Name",
    "startIndex": 0,
    "endIndex": 8,
    "score": 0.473899543
  },
  {
    "entity": "3",
    "type": "builtin.number",
    "startIndex": 16,
    "endIndex": 16,
    "resolution": {
      "value": "3"
    }
  }
]
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

```JSON
"entities": {
    "name":["bob jones"],
    "number": [3]
}
```
Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

## <a name="tokenized-entity-returned"></a>En token enhet returnerades

Granska [token-stödet](luis-language-support.md#tokenization) i Luis.

## <a name="simple-entity-data"></a>Enkla enhets data

En [enkel entitet](reference-entity-simple.md) är ett värde som är känd för datorn. Det kan vara ett ord eller en fras.

## <a name="composite-entity-data"></a>Sammansatta enhets data

En [sammansatt entitet](reference-entity-composite.md) består av andra entiteter, till exempel färdiga entiteter, enkla, reguljära uttryck och list enheter. Separata entiteter utgör en hel entitet.

## <a name="list-entity-data"></a>Lista entitets data

[Lista entiteter](reference-entity-list.md) representerar en fast, avslutad uppsättning relaterade ord tillsammans med deras synonymer. LUIS identifierar inte ytterligare värden för List-entiteter. Använd **rekommendations** funktionen om du vill se förslag på nya ord baserade på den aktuella listan. Om det finns mer än en List-entitet med samma värde returneras varje entitet i slut punkts frågan.

## <a name="prebuilt-entity-data"></a>Färdiga enhets data
[Förbyggda](luis-concept-entity-types.md) entiteter identifieras baserat på en reguljär uttrycks matchning med hjälp av [text](https://github.com/Microsoft/Recognizers-Text) projekt med öppen källkod. Förbyggda entiteter returneras i entiteten entiteter och använder det typnamn som föregås av `builtin::`. Följande text är ett exempel på en uttryck med returnerade fördefinierade entiteter:

`Dec 5th send to +1 360-555-1212`

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
"entities": [
    {
      "entity": "dec 5th",
      "type": "builtin.datetimeV2.date",
      "startIndex": 0,
      "endIndex": 6,
      "resolution": {
        "values": [
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2017-12-05"
          },
          {
            "timex": "XXXX-12-05",
            "type": "date",
            "value": "2018-12-05"
          }
        ]
      }
    },
    {
      "entity": "1",
      "type": "builtin.number",
      "startIndex": 18,
      "endIndex": 18,
      "resolution": {
        "value": "1"
      }
    },
    {
      "entity": "360",
      "type": "builtin.number",
      "startIndex": 20,
      "endIndex": 22,
      "resolution": {
        "value": "360"
      }
    },
    {
      "entity": "555",
      "type": "builtin.number",
      "startIndex": 26,
      "endIndex": 28,
      "resolution": {
        "value": "555"
      }
    },
    {
      "entity": "1212",
      "type": "builtin.number",
      "startIndex": 32,
      "endIndex": 35,
      "resolution": {
        "value": "1212"
      }
    },
    {
      "entity": "5th",
      "type": "builtin.ordinal",
      "startIndex": 4,
      "endIndex": 6,
      "resolution": {
        "value": "5"
      }
    },
    {
      "entity": "1 360 - 555 - 1212",
      "type": "builtin.phonenumber",
      "startIndex": 18,
      "endIndex": 35,
      "resolution": {
        "value": "1 360 - 555 - 1212"
      }
    }
  ]
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

Utan parametern QueryString `verbose=true`:

```json
"entities": {
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "XXXX-12-05",
                    "value": "2018-12-05"
                },
                {
                    "timex": "XXXX-12-05",
                    "value": "2019-12-05"
                }
            ]
        }
    ],
    "ordinal": [
        {
            "offset": 5,
            "relativeTo": "start"
        }
    ],
    "ordinalV2": [
        {
            "offset": 5,
            "relativeTo": "start"
        }
    ],
    "number": [
        1360,
        555,
        1212
    ],
    "phonenumber": [
        "1 360-555-1212"
    ]
}
```

Med parametern QueryString `verbose=true`:

```json

"entities": {
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "XXXX-12-05",
                    "value": "2018-12-05"
                },
                {
                    "timex": "XXXX-12-05",
                    "value": "2019-12-05"
                }
            ]
        }
    ],
    "ordinal": [
        {
            "offset": 5,
            "relativeTo": "start"
        }
    ],
    "ordinalV2": [
        {
            "offset": 5,
            "relativeTo": "start"
        }
    ],
    "number": [
        1360,
        555,
        1212
    ],
    "phonenumber": [
        "1 360-555-1212"
    ],
    "$instance": {
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.date",
                "text": "Dec 5th",
                "startIndex": 0,
                "length": 7,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "ordinal": [
            {
                "type": "builtin.ordinal",
                "text": "5th",
                "startIndex": 4,
                "length": 3,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "ordinalV2": [
            {
                "type": "builtin.ordinalV2",
                "text": "5th",
                "startIndex": 4,
                "length": 3,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "number": [
            {
                "type": "builtin.number",
                "text": "1 360",
                "startIndex": 17,
                "length": 5,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            },
            {
                "type": "builtin.number",
                "text": "555",
                "startIndex": 23,
                "length": 3,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            },
            {
                "type": "builtin.number",
                "text": "1212",
                "startIndex": 27,
                "length": 4,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "phonenumber": [
            {
                "type": "builtin.phonenumber",
                "text": "1 360-555-1212",
                "startIndex": 17,
                "length": 14,
                "score": 1.0,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *
## <a name="regular-expression-entity-data"></a>Enhets data för reguljära uttryck

En [entitet för reguljära uttryck](reference-entity-regular-expression.md) extraherar en entitet baserat på ett mönster för reguljära uttryck som du anger.

## <a name="extracting-names"></a>Extraherar namn
Det är svårt att hämta namn från en uttryck eftersom ett namn kan vara praktiskt taget valfri kombination av bokstäver och ord. Beroende på vilken typ av namn du extraherar har du flera alternativ. Följande förslag är inte regler, men fler rikt linjer.

### <a name="add-prebuilt-personname-and-geographyv2-entities"></a>Lägg till färdiga PersonName-och GeographyV2-entiteter

[PersonName](luis-reference-prebuilt-person.md) -och [GeographyV2](luis-reference-prebuilt-geographyV2.md) -entiteter är tillgängliga i vissa [språk kulturer](luis-reference-prebuilt-entities.md).

### <a name="names-of-people"></a>Namn på personer

Personens namn kan ha lite format beroende på språk och kultur. Använd antingen en fördefinierad **[personName](luis-reference-prebuilt-person.md)** -entitet eller en **[enkel entitet](luis-concept-entity-types.md#simple-entity)** med [roller](luis-concept-roles.md) för för-och efter namn.

Om du använder den enkla entiteten ska du se till att ge exempel som använder det första och sista namnet i olika delar av uttryck, i yttranden med olika längd och yttranden i alla avsikter, inklusive ingen avsikt. [Granska](luis-how-to-review-endoint-utt.md) slut punkts yttranden regelbundet för att märka namn som inte har förutsägts korrekt.

### <a name="names-of-places"></a>Namn på platser

Plats namn anges och är kända som städer, regioner, stater, provinser och länder/regioner. Använd den fördefinierade entiteten **[geographyV2](luis-reference-prebuilt-geographyv2.md)** för att extrahera plats information.

### <a name="new-and-emerging-names"></a>Nya och framväxande namn

Vissa appar måste kunna hitta nya och nya namn, till exempel produkter eller företag. Dessa typer av namn är den svåraste typen av data extrahering. Börja med en **[enkel entitet](luis-concept-entity-types.md#simple-entity)** och Lägg till en [fras lista](luis-concept-feature.md). [Granska](luis-how-to-review-endoint-utt.md) slut punkts yttranden regelbundet för att märka namn som inte har förutsägts korrekt.

## <a name="pattern-roles-data"></a>Mönster Rolls data
Roller är Sammanhangs beroende skillnader i entiteter.


#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

Entitetsnamnet är `Location`, med två roller `Origin` och. `Destination`

```JSON
"entities": [
  {
    "entity": "bob jones",
    "type": "Employee",
    "startIndex": 5,
    "endIndex": 13,
    "score": 0.922820568,
    "role": ""
  },
  {
    "entity": "seattle",
    "type": "Location",
    "startIndex": 20,
    "endIndex": 26,
    "score": 0.948008537,
    "role": "Origin"
  },
  {
    "entity": "redmond",
    "type": "Location",
    "startIndex": 31,
    "endIndex": 37,
    "score": 0.7047979,
    "role": "Destination"
  }
]
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

I v3 är **roll namnet** det primära namnet på objektet.

Entitetsnamnet är `Location`, med två roller `Origin` och. `Destination`

Utan parametern QueryString `verbose=true`:

```json
"entities": {
    "Employee": [
        "bob jones"
    ],
    "Origin": [
        "seattle"
    ],
    "Destination": [
        "redmond"
    ]
}
```

Med parametern QueryString `verbose=true`:

```json
"entities": {
    "Employee": [
        "bob jones"
    ],
    "LocationOrigin": [
        "seattle"
    ],
    "LocationDestination": [
        "redmond"
    ],
    "$instance": {
        "Employee": [
            {
                "type": "Employee",
                "text": "bob jones",
                "startIndex": 5,
                "length": 9,
                "score": 0.982873261,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "Origin": [
            {
                "role": "Origin",
                "type": "Location",
                "text": "seattle",
                "startIndex": 20,
                "length": 7,
                "score": 0.9913306,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "Destination": [
            {
                "role": "Destination",
                "type": "Location",
                "text": "redmond",
                "startIndex": 31,
                "length": 7,
                "score": 0.898179531,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]

}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

## <a name="patternany-entity-data"></a>Mönster. alla entitets data

[Mönster.](reference-entity-pattern-any.md) det finns en plats hållare med variabel längd som bara används i ett mönsters mall uttryck för att markera var entiteten börjar och slutar.

## <a name="sentiment-analysis"></a>Sentimentanalys
Om sentiment-analysen har kon figurer ATS innehåller LUIS JSON-svaret sentiment analys. Läs mer om sentiment-analys i [textanalys](https://docs.microsoft.com/azure/cognitive-services/text-analytics/) -dokumentationen.

### <a name="sentiment-data"></a>Sentiment-data
Sentiment-data är ett resultat mellan 1 och 0 som anger positiv (närmare 1) eller negativ (närmare 0) sentiment av data.

När kulturen är `en-us`är svaret:

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

För alla andra kulturer är svaret:

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```


### <a name="key-phrase-extraction-entity-data"></a>Nyckel fras extrahering av enhets data
Extraherings enheten för nyckel fraser returnerar nyckel fraser i uttryck, som tillhandahålls av [textanalys](https://docs.microsoft.com/azure/cognitive-services/text-analytics/).


#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
{
  "query": "Is there a map of places with beautiful views on a favorite trail?",
  "topScoringIntent": {
    "intent": "GetJobInformation",
    "score": 0.764368951
  },
  "intents": [
    ...
  ],
  "entities": [
    {
      "entity": "beautiful views",
      "type": "builtin.keyPhrase",
      "startIndex": 30,
      "endIndex": 44
    },
    {
      "entity": "map of places",
      "type": "builtin.keyPhrase",
      "startIndex": 11,
      "endIndex": 23
    },
    {
      "entity": "favorite trail",
      "type": "builtin.keyPhrase",
      "startIndex": 51,
      "endIndex": 64
    }
  ]
}
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

Utan parametern QueryString `verbose=true`:

```json
"entities": {
    "keyPhrase": [
        "map of places",
        "beautiful views",
        "favorite trail"
    ]
}
```

Med parametern QueryString `verbose=true`:

```json
"entities": {
    "keyPhrase": [
        "map of places",
        "beautiful views",
        "favorite trail"
    ],
    "$instance": {
        "keyPhrase": [
            {
                "type": "builtin.keyPhrase",
                "text": "map of places",
                "startIndex": 11,
                "length": 13,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            },
            {
                "type": "builtin.keyPhrase",
                "text": "beautiful views",
                "startIndex": 30,
                "length": 15,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            },
            {
                "type": "builtin.keyPhrase",
                "text": "favorite trail",
                "startIndex": 51,
                "length": 14,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *


## <a name="data-matching-multiple-entities"></a>Data som matchar flera entiteter

LUIS returnerar alla entiteter som identifierats i uttryck. Därför kan din chattrobot behöva fatta beslut baserat på resultaten. En uttryck kan ha många entiteter i en uttryck:

`book me 2 adult business tickets to paris tomorrow on air france`

LUIS-slutpunkten kan upptäcka samma data i olika entiteter.

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
{
  "query": "book me 2 adult business tickets to paris tomorrow on air france",
  "topScoringIntent": {
    "intent": "BookFlight",
    "score": 1.0
  },
  "intents": [
    {
      "intent": "BookFlight",
      "score": 1.0
    },
    {
      "intent": "Concierge",
      "score": 0.04216196
    },
    {
      "intent": "None",
      "score": 0.03610297
    }
  ],
  "entities": [
    {
      "entity": "air france",
      "type": "Airline",
      "startIndex": 54,
      "endIndex": 63,
      "score": 0.8291798
    },
    {
      "entity": "adult",
      "type": "Category",
      "startIndex": 10,
      "endIndex": 14,
      "resolution": {
        "values": [
          "adult"
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Cities",
      "startIndex": 36,
      "endIndex": 40,
      "resolution": {
        "values": [
          "Paris"
        ]
      }
    },
    {
      "entity": "tomorrow",
      "type": "builtin.datetimeV2.date",
      "startIndex": 42,
      "endIndex": 49,
      "resolution": {
        "values": [
          {
            "timex": "2018-02-21",
            "type": "date",
            "value": "2018-02-21"
          }
        ]
      }
    },
    {
      "entity": "paris",
      "type": "Location::ToLocation",
      "startIndex": 36,
      "endIndex": 40,
      "score": 0.9730773
    },
    {
      "entity": "2",
      "type": "builtin.number",
      "startIndex": 8,
      "endIndex": 8,
      "resolution": {
        "value": "2"
      }
    },
    {
      "entity": "business",
      "type": "Seat",
      "startIndex": 16,
      "endIndex": 23,
      "resolution": {
        "values": [
          "business"
        ]
      }
    },
    {
      "entity": "2 adult business",
      "type": "TicketSeatOrder",
      "startIndex": 8,
      "endIndex": 23,
      "score": 0.8784727
    }
  ],
  "compositeEntities": [
    {
      "parentType": "TicketSeatOrder",
      "value": "2 adult business",
      "children": [
        {
          "type": "Category",
          "value": "adult"
        },
        {
          "type": "builtin.number",
          "value": "2"
        },
        {
          "type": "Seat",
          "value": "business"
        }
      ]
    }
  ]
}
```

#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

Utan `verbose=true` som en QueryString-parameter.

```json
"entities": {
    "TicketsOrder": [
        {
            "number": [
                2
            ],
            "PassengerCategory": [
                "adult"
            ],
            "TravelClass": [
                "business"
            ]
        }
    ],
    "Location::LocationTo": [
        "paris"
    ],
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "2019-09-28",
                    "value": "2019-09-28"
                }
            ]
        }
    ],
    "Airline": [
        "air france"
    ]
}
```

Med `verbose=true` som en QueryString-parameter.


```json
"entities": {
    "TicketsOrder": [
        {
            "number": [
                2
            ],
            "PassengerCategory": [
                "adult"
            ],
            "TravelClass": [
                "business"
            ],
            "$instance": {
                "number": [
                    {
                        "type": "builtin.number",
                        "text": "2",
                        "startIndex": 8,
                        "length": 1,
                        "modelTypeId": 2,
                        "modelType": "Prebuilt Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ],
                "PassengerCategory": [
                    {
                        "type": "PassengerCategory",
                        "text": "adult",
                        "startIndex": 10,
                        "length": 5,
                        "score": 0.9503733,
                        "modelTypeId": 3,
                        "modelType": "Hierarchical Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ],
                "TravelClass": [
                    {
                        "type": "TravelClass",
                        "text": "business",
                        "startIndex": 16,
                        "length": 8,
                        "score": 0.950095,
                        "modelTypeId": 3,
                        "modelType": "Hierarchical Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    ],
    "Location::LocationTo": [
        "paris"
    ],
    "datetimeV2": [
        {
            "type": "date",
            "values": [
                {
                    "timex": "2019-09-28",
                    "value": "2019-09-28"
                }
            ]
        }
    ],
    "Airline": [
        "air france"
    ],
    "$instance": {
        "TicketsOrder": [
            {
                "type": "TicketsOrder",
                "text": "2 adult business",
                "startIndex": 8,
                "length": 16,
                "score": 0.942183256,
                "modelTypeId": 4,
                "modelType": "Composite Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "Location::LocationTo": [
            {
                "type": "Location::LocationTo",
                "text": "paris",
                "startIndex": 36,
                "length": 5,
                "score": 0.9905354,
                "modelTypeId": 3,
                "modelType": "Hierarchical Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "datetimeV2": [
            {
                "type": "builtin.datetimeV2.date",
                "text": "tomorrow",
                "startIndex": 42,
                "length": 8,
                "modelTypeId": 2,
                "modelType": "Prebuilt Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ],
        "Airline": [
            {
                "type": "Airline",
                "text": "air france",
                "startIndex": 54,
                "length": 10,
                "score": 0.9455415,
                "modelTypeId": 1,
                "modelType": "Entity Extractor",
                "recognitionSources": [
                    "model"
                ]
            }
        ]
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

## <a name="data-matching-multiple-list-entities"></a>Data som matchar flera List enheter

Om ett ord eller en fras matchar mer än en List-entitet returnerar slut punkts frågan varje lista entitet.

För frågan `when is the best time to go to red rock?`och appen har ordet `red` i fler än en lista, känner Luis igen alla entiteter och returnerar en matris med entiteter som en del av JSON-slutpunktens svar:

#### <a name="v2-prediction-endpoint-response"></a>[Slut punkts svar för v2 förutsägelse](#tab/V2)

```JSON
{
  "query": "when is the best time to go to red rock?",
  "topScoringIntent": {
    "intent": "Calendar.Find",
    "score": 0.06701678
  },
  "entities": [
    {
      "entity": "red",
      "type": "Colors",
      "startIndex": 31,
      "endIndex": 33,
      "resolution": {
        "values": [
          "Red"
        ]
      }
    },
    {
      "entity": "red rock",
      "type": "Cities",
      "startIndex": 31,
      "endIndex": 38,
      "resolution": {
        "values": [
          "Destinations"
        ]
      }
    }
  ]
}
```



#### <a name="v3-prediction-endpoint-response"></a>[V3 slut punkts svar för förutsägelse](#tab/V3)

Utan `verbose=true` i frågesträngen:

```JSON
{
    "query": "when is the best time to go to red rock",
    "prediction": {
        "normalizedQuery": "when is the best time to go to red rock",
        "topIntent": "None",
        "intents": {
            "None": {
                "score": 0.823669851
            }
        },
        "entities": {
            "Colors": [
                [
                    "red"
                ]
            ],
            "Cities": [
                [
                    "Destinations"
                ]
            ]
        }
    }
}
```


Med `verbose=true` i frågesträngen:

```JSON
{
    "query": "when is the best time to go to red rock",
    "prediction": {
        "normalizedQuery": "when is the best time to go to red rock",
        "topIntent": "None",
        "intents": {
            "None": {
                "score": 0.823669851
            }
        },
        "entities": {
            "Colors": [
                [
                    "red"
                ]
            ],
            "Cities": [
                [
                    "Destinations"
                ]
            ],
            "$instance": {
                "Colors": [
                    {
                        "type": "Colors",
                        "text": "red",
                        "startIndex": 31,
                        "length": 3,
                        "modelTypeId": 5,
                        "modelType": "List Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ],
                "Cities": [
                    {
                        "type": "Cities",
                        "text": "red rock",
                        "startIndex": 31,
                        "length": 8,
                        "modelTypeId": 5,
                        "modelType": "List Entity Extractor",
                        "recognitionSources": [
                            "model"
                        ]
                    }
                ]
            }
        }
    }
}
```

Läs mer om [v3 förutsägelse slut punkten](luis-migration-api-v3.md).

* * *

## <a name="next-steps"></a>Nästa steg

Se [Lägg till entiteter](luis-how-to-add-entities.md) för att lära dig mer om hur du lägger till entiteter i Luis-appen.
