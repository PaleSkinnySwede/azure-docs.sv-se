---
title: Gränser – LUIS
description: Den här artikeln innehåller de kända gränserna för Azure Cognitive Services Language Understanding (LUIS). LUIS har flera gräns områden. Modell gräns kontroller för avsikter, entiteter och funktioner i LUIS. Kvot gränser baserat på nyckel typ. Tangent bords kombinationen styr LUIS-webbplatsen.
ms.topic: reference
ms.date: 04/02/2020
ms.openlocfilehash: 0a734091ad2c9812f079d77c97c22872717aa7c9
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82103596"
---
# <a name="limits-for-your-luis-model-and-keys"></a>Begränsningar för din LUIS-modell och nycklar
LUIS har flera gräns områden. Den första är [modell gränsen](#model-limits), som styr avsikter, entiteter och funktioner i Luis. Det andra avsnittet är [kvot gränser](#key-limits) baserat på nyckel typ. Ett tredje begränsat utrymme är [tangentkombinationen](#keyboard-controls) för att kontrol lera Luis-webbplatsen. Ett fjärde område är [Mappning av världs regionen](luis-reference-regions.md) mellan Luis Authoring-webbplatsen och API: erna för Luis- [slutpunkt](luis-glossary.md#endpoint) .

<a name="model-boundaries"></a>

## <a name="model-limits"></a>Modell gränser

Om din app överskrider LUIS-modellens gränser bör du överväga att använda en [Luis sändnings](luis-concept-enterprise.md#dispatch-tool-and-model) app eller en [Luis-behållare](luis-container-howto.md).

|Område|Gräns|
|--|:--|
| [App-namn][luis-get-started-create-app] | * Standard-största antal bokstäver |
| Program| 500 program per Azure Authoring-resurs |
| [Batch-testning][batch-testing]| 10 data uppsättningar, 1000 yttranden per data uppsättning|
| Explicit lista | 50 per program|
| Externa entiteter | inga gränser |
| [Avsikter][intents]|500 per program: 499 anpassade avsikter och det krävs _ingen_ avsikt.<br>[Skickat](https://aka.ms/dispatch-tool) program har motsvarande 500-sändnings källor.|
| [Listentiteter](./luis-concept-entity-types.md) | Överordnad: 50, underordnad: 20 000 objekt. Kanoniskt namn är * standard tecknen max. Synonym värden har ingen längd begränsning. |
| [Kundlärt entiteter + roller](./luis-concept-entity-types.md):<br> sammansättning<br>gång<br>enhets roll|En gräns på antingen 100 överordnade entiteter eller 330 entiteter, beroende på vilken gräns användaren träffar först. En roll räknas som en entitet för den här begränsningen. Ett exempel är ett sammansatt objekt med en enkel entitet, som har två roller: 1 sammansatt + 1 enkla + 2 roller = 4 av 330-enheterna.<br>Del komponenter kan kapslas upp till 5 nivåer.|
|Modell som en funktion| Det maximala antalet modeller som kan användas som en beskrivning (funktion) till en speciell modell som är 10 modeller. Det maximala antalet fras listor som används som en beskrivning (funktion) för en speciell modell som är 10 fras listor.|
| [Förhands granskning-dynamiska List enheter](https://aka.ms/luis-api-v3-doc#dynamic-lists-passed-in-at-prediction-time)|2 listor med ~ 1 kB per förfrågan för förutsägelse slut punkt|
| [Mönster](luis-concept-patterns.md)|500 mönster per program.<br>Den maximala längden för mönstret är 400 tecken.<br>3 mönster. alla entiteter per mönster<br>Högst 2 kapslade valfria texter i mönstret|
| [Mönster. alla](./luis-concept-entity-types.md)|100 per program, 3 mönster. alla entiteter per mönster |
| [Fras lista][phrase-list]|500 fras listor. 10 global fras listor på grund av modellen som en funktions gräns. Listan över icke-utbytbara fraser har högst 5 000 fraser. Listan över utbytbara fraser har högst 50 000 fraser. Maximalt antal fraser per program 500 000-fraser.|
| [Fördefinierade entiteter](./luis-prebuilt-entities.md) | ingen gräns|
| [Lägga till entiteter för reguljära uttryck](./luis-concept-entity-types.md)|20 entiteter<br>500-Character max. per reguljärt uttryck enhets mönster|
| [Roller](luis-concept-roles.md)|300 roller per program. 10 roller per entitet|
| [Yttrande][utterances] | 500 tecken|
| [Yttranden][utterances] | 15 000 per program-det finns ingen gräns för antalet yttranden per avsikt|
| [5](luis-concept-version.md)| 100-versioner per program |
| [Versions namn][luis-how-to-manage-versions] | 10 tecken som är begränsade till alfanumeriska tecken och punkter (.) |

* Standard tecken Max är 50 tecken.

<a name="intent-and-entity-naming"></a>

## <a name="name-uniqueness"></a>Namn unikhet

Använd följande unika namngivnings regler.

Följande måste vara unikt inom en LUIS-app:

* versions namn
* avsikt
* entitetsrelation
* roles

Följande måste vara unikt inom omfånget som används:

* fras lista

## <a name="object-naming"></a>Objekt namn

Använd inte följande tecken i följande namn.

|Objekt|Uteslut tecken|
|--|--|
|Namn på avsikt, entitet och roll|`:`<br>`$` <br> `&`|
|Versions namn|`\`<br> `/`<br> `:`<br> `?`<br> `&`<br> `=`<br> `*`<br> `+`<br> `(`<br> `)`<br> `%`<br> `@`<br> `$`<br> `~`<br> `!`<br> `#`|

## <a name="resource-usage-and-limits"></a>Resursanvändning och begränsningar

Språk förståelsen har separata resurser, en typ för redigering och en typ för att skicka frågor till förutsägelse slut punkten. Mer information om skillnaderna mellan nyckel typer finns i [Redigera och köra slut punkts nycklar för förutsägelse i Luis](luis-concept-keys.md).

<a name="key-limits"></a>

### <a name="authoring-resource-limits"></a>Redigera resurs gränser

Använd _typen_, `LUIS.Authoring`när du filtrerar resurser i Azure Portal. LUIS begränsar 500 program per Azure Authoring-resurs.

|Skapar resurs|Redigera TPS|
|--|--|
|Starter|1 miljon/månad, 5/sekund|
|F0 – kostnads fri nivå |1 miljon/månad, 5/sekund|

* TPS = transaktioner per sekund

[Läs mer om priser.][pricing]

### <a name="query-prediction-resource-limits"></a>Resurs gränser för fråga förutsägelse

Använd _typen_, `LUIS`när du filtrerar resurser i Azure Portal. Slut punkts resursen för LUIS-som används i körnings miljön är bara giltig för slut punkts frågor.

|Fråga förutsägelse resurs|Fråga TPS|
|--|--|
|F0 – kostnads fri nivå |10 000 per månad, 5/sekund|
|S0 – standard nivå|50/sekund|

### <a name="sentiment-analysis"></a>Sentimentanalys

[Sentiment analys integrering](luis-how-to-publish-app.md#enable-sentiment-analysis), som innehåller information om sentiment, tillhandahålls utan någon annan Azure-resurs.

### <a name="speech-integration"></a>Tal integrering

[Tal integrering](../speech-service/how-to-recognize-intents-from-speech-csharp.md) ger 1000 slut punkts förfrågningar per enhets kostnad.

[Läs mer om priser.][pricing]

## <a name="keyboard-controls"></a>Tangent bords kontroller

|Tangent bords inmatare | Beskrivning |
|--|--|
|Kontroll + E|växlar mellan token och entiteter i yttranden-listan|

## <a name="website-sign-in-time-period"></a>Inloggnings tids period för webbplats

Din inloggnings åtkomst är i **60 minuter**. Efter den här tids perioden får du det här felet. Du måste logga in igen.

[luis-get-started-create-app]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app
[batch-testing]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-test#batch-testing
[intents]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-intent
[phrase-list]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-feature
[utterances]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-utterance
[luis-how-to-manage-versions]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-how-to-manage-versions
[pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
<!-- TBD: fix this link -->
[speech-to-intent-pricing]: https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/
