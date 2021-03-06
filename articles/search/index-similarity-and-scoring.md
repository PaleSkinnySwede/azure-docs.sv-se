---
title: Översikt över likhet och bedömning
titleSuffix: Azure Cognitive Search
description: Förklarar koncepten för likhet och bedömning, och vad en utvecklare kan göra för att anpassa Poäng resultatet.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 4b02039c86f43e6bebed58dfff475816f09a3da1
ms.sourcegitcommit: b396c674aa8f66597fa2dd6d6ed200dd7f409915
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2020
ms.locfileid: "82890148"
---
# <a name="similarity-and-scoring-in-azure-cognitive-search"></a>Likhet och bedömning i Azure Kognitiv sökning

Poängsättning avser beräkningen av en Sök Poäng för varje objekt som returneras i Sök Resultat för fullständiga texts öknings frågor. Poängen är en indikator för ett objekts relevans i kontexten för den aktuella sökåtgärden. Ju högre poäng, desto mer relevant är objektet. I Sök resultaten rangordnas objekt rang från hög till låg, baserat på Sök resultaten som beräknas för varje objekt. 

Som standard returneras de översta 50 i svaret, men du kan använda parametern **$Top** för att returnera ett mindre eller större antal objekt (upp till 1000 i ett enda svar) och **$Skip** för att hämta nästa uppsättning resultat.

Sök poängen beräknas baserat på statistiska egenskaper för data och frågan. Azure Kognitiv sökning hittar dokument som matchar på Sök villkoren (vissa eller alla, beroende på [searchMode](https://docs.microsoft.com/rest/api/searchservice/search-documents#searchmodeany--all-optional)) och prioriterar dokument som innehåller många instanser av Sök termen. Sök poängen går till ännu högre om termen är sällsynt över data indexet, men vanligt i dokumentet. Grunden för den här metoden för att beräkna relevans kallas *TF-IDF eller* term frekvens – invertering av dokument frekvens.

Sök Resultat värden kan upprepas i en resultat uppsättning. Om flera träffar har samma Sök poäng, är ordningen för samma poäng objekt inte definierad och är inte stabil. Kör frågan igen så kan du se objekt förändrings positionen, särskilt om du använder den kostnads fria tjänsten eller en fakturerbar tjänst med flera repliker. Om två objekt har samma poäng finns det ingen garanti för att det visas först.

Om du vill dela upp anknyten mellan upprepade Poäng kan du lägga till en **$OrderBy** -sats i första ordningen efter poäng och sedan Sortera efter ett annat sorterbart fält `$orderby=search.score() desc,Rating desc`(till exempel). Mer information finns i [$OrderBy](https://docs.microsoft.com/azure/search/search-query-odata-orderby).

> [!NOTE]
> A `@search.score = 1.00` indikerar en oväntad resultat uppsättning. Poängen är enhetlig för alla resultat. Oväntade resultat inträffar när fråge formuläret är suddigt Sök-, jokertecken-eller regex-frågor eller ett **$filter** -uttryck. 

## <a name="scoring-profiles"></a>Poängprofiler

Du kan anpassa hur olika fält rangordnas genom att definiera en anpassad *bedömnings profil*. Med bedömnings profiler får du bättre kontroll över rankningen av objekt i Sök resultaten. Du kanske till exempel vill öka objekt baserat på deras intäkts potential, befordra nya objekt eller kanske förbättra objekt som har varit i lager för länge. 

En bedömnings profil är en del av index definitionen, som består av viktade fält, funktioner och parametrar. Mer information om att definiera en finns i [bedömnings profiler](index-add-scoring-profiles.md).

## <a name="scoring-statistics"></a>Bedömnings statistik

För skalbarhet distribuerar Azure Kognitiv sökning varje index vågrätt genom en horisontell partitionering process, vilket innebär att delar av ett index är fysiskt åtskilda.

Som standard beräknas poängen för ett dokument baserat på statistiska egenskaper för data *i en Shard*. Den här metoden är vanligt vis inte ett problem för en stor sökkorpus och ger bättre prestanda än att behöva beräkna poängen baserat på information i alla Shards. Detta innebär att med hjälp av den här prestanda optimeringen kan det orsaka två mycket lika stora dokument (eller till och med identiska dokument) för att få en annan relevans om de blir i olika Shards.

Om du föredrar att beräkna poängen baserat på de statistiska egenskaperna för alla Shards kan du göra det genom att lägga till *scoringStatistics = global* som en [frågeparameter](https://docs.microsoft.com/rest/api/searchservice/search-documents) (eller lägga till *"scoringStatistics": "global"* som en text parameter för [förfrågan](https://docs.microsoft.com/rest/api/searchservice/search-documents)).

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs?scoringStatistics=global
  Content-Type: application/json
  api-key: [admin key]  
```

> [!NOTE]
> En Admin-API-nyckel krävs för `scoringStatistics` parametern.

## <a name="similarity-ranking-algorithms"></a>Algoritmer för rangordning av likhet

Azure Kognitiv sökning har stöd för två olika algoritmer för rangordning av likheter: en *klassisk likhets* algoritm och den officiella implementeringen av *Okapi BM25* -algoritmen (för närvarande i för hands version). Algoritmen för klassisk likhet är standardalgoritmen, men från och med den 15 juli kommer alla nya tjänster som skapats efter det datumet att använda den nya BM25-algoritmen. Det är den enda algoritm som är tillgänglig för nya tjänster.

Nu kan du ange vilken algoritm för rangordning av likheter som du vill använda. Mer information finns i [rangordning av algoritmer](index-ranking-similarity.md).

Följande video segment snabb Spolar framåt till en förklaring av de rangordnings algoritmer som används i Azure Kognitiv sökning. Du kan se hela videon om du vill ha mer bakgrund.

> [!VIDEO https://www.youtube.com/embed/Y_X6USgvB1g?version=3&start=322&end=643]

## <a name="see-also"></a>Se även

 [Bedömnings profiler](index-add-scoring-profiles.md) [REST API referens](https://docs.microsoft.com/rest/api/searchservice/)   
 [Sök i dokument-API](https://docs.microsoft.com/rest/api/searchservice/search-documents)   
 [Azure Kognitiv sökning .NET SDK](https://docs.microsoft.com/dotnet/api/overview/azure/search?view=azure-dotnet)  
