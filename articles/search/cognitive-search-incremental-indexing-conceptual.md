---
title: Stegvis anrikning (för hands version)
titleSuffix: Azure Cognitive Search
description: Cachelagra mellanliggande innehåll och stegvisa ändringar från AI-pipeline i Azure Storage för att bevara investeringar i befintliga bearbetade dokument. Den här funktionen är för närvarande i allmänt tillgänglig förhandsversion.
manager: nitinme
author: Vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/09/2020
ms.openlocfilehash: 09003c26ead9108d07ae339fcf64235c246474a4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77024151"
---
# <a name="introduction-to-incremental-enrichment-and-caching-in-azure-cognitive-search"></a>Introduktion till stegvis anrikning och cachelagring i Azure Kognitiv sökning

> [!IMPORTANT] 
> Stegvis anrikning är för närvarande en offentlig för hands version. Den här förhandsversionen tillhandahålls utan serviceavtal och rekommenderas inte för produktionsarbetsbelastningar. Mer information finns i [Kompletterande villkor för användning av Microsoft Azure-förhandsversioner](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Den [REST API version 2019-05-06 – för hands version](search-api-preview.md) innehåller den här funktionen. Det finns för närvarande ingen portal eller .NET SDK-support.

Avancerad berikning ger till gång till cachelagring och statefulness till en anriknings pipeline, som bevarar din investering i befintliga utdata och bara ändrar de dokument som påverkas av särskilda ändringar. Detta bevarar inte bara din monetära investering i bearbetningen (särskilt OCR-bearbetning och bild bearbetning), men det gör också ett mer effektivt system. När strukturer och innehåll cachelagras kan en indexerare avgöra vilka kunskaper som har ändrats och bara köra de som har ändrats, samt eventuella underordnade beroende kunskaper. 

## <a name="indexer-cache"></a>Indexerare-cache

Stegvis anrikning lägger till en cache i pipelinen. Indexeraren cachelagrar resultaten från dokument sprickor plus utdata för varje kunskap för varje dokument. När en färdigheter uppdateras, körs bara de ändrade eller underordnade färdigheterna igen. De uppdaterade resultaten skrivs till cachen och dokumentet uppdateras i Sök indexet eller i kunskaps lagret.

Fysiskt lagras cacheminnet i en BLOB-behållare i ditt Azure Storage-konto. Cachen använder också Table Storage för en intern post av bearbetnings uppdateringar. Alla index i en Sök tjänst kan dela samma lagrings konto för indexeraren cache. Varje indexerare tilldelas en unik och oföränderlig cache-identifierare till den behållare som den använder.

## <a name="cache-configuration"></a>Cache-konfiguration

Du måste ange `cache` egenskapen på indexeraren för att starta får från en stegvis berikning. I följande exempel illustreras en indexerare med cachelagring aktiverat. Vissa delar av den här konfigurationen beskrivs i följande avsnitt. Mer information finns i [Konfigurera stegvis berikning](search-howto-incremental-index.md).

```json
{
    "name": "myIndexerName",
    "targetIndexName": "myIndex",
    "dataSourceName": "myDatasource",
    "skillsetName": "mySkillset",
    "cache" : {
        "storageConnectionString" : "Your storage account connection string",
        "enableReprocessing": true
    },
    "fieldMappings" : [],
    "outputFieldMappings": [],
    "parameters": []
}
```

Om du anger den här egenskapen för en befintlig indexerare måste du återställa och köra indexeraren igen, vilket leder till att alla dokument i data källan bearbetas igen. Det här steget är nödvändigt för att eliminera dokument som har berikats av tidigare versioner av färdigheter. 

## <a name="cache-management"></a>Hantering av cache

Livs längden för cachen hanteras av indexeraren. Om `cache` egenskapen på indexeraren har angetts till null eller om anslutnings strängen ändras, tas den befintliga cachen bort när nästa indexerare körs. Cachens livs cykel är också kopplad till indexerare livs cykel. Om en indexerare tas bort, tas även den tillhör ande cachen bort.

Även om den stegvisa berikningen är utformad för att identifiera och svara på ändringar utan någon åtgärd från din sida, finns det parametrar som du kan använda för att åsidosätta standard beteenden:

+ Prioritera nya dokument
+ Kringgå färdigheter-kontroller
+ Kringgå kontroller av data Källa
+ Framtvinga färdigheter-utvärdering

### <a name="prioritize-new-documents"></a>Prioritera nya dokument

Ange `enableReprocessing` egenskapen för att styra bearbetningen av inkommande dokument som redan visas i cacheminnet. När `true` (standard), kommer dokument som redan finns i cacheminnet att ombearbetas när du kör om indexeraren, förutsatt att din kunskaps uppdatering påverkar dokumentet. 

När `false`de befintliga dokumenten inte bearbetas igen, prioriteras nya, inkommande innehåll i befintligt innehåll. Du bör bara `enableReprocessing` ställas `false` in på temporär basis. För att säkerställa konsekvens över sökkorpus `enableReprocessing` bör du vara `true` det mesta av tiden, vilket säkerställer att alla dokument, både nya och befintliga, är giltiga enligt den aktuella färdigheter-definitionen.

### <a name="bypass-skillset-evaluation"></a>Kringgå färdigheter-utvärdering

Det går inte att ändra en färdigheter och ombearbeta den färdigheter. Vissa ändringar i en färdigheter bör dock inte resultera i ombearbetning (till exempel att distribuera en anpassad färdighet till en ny plats eller med en ny åtkomst nyckel). De flesta sannolika är att de är kring utrustnings ändringar som inte har någon verklig inverkan på själva färdigheter. 

Om du vet att en ändring i färdigheter verkligen är ytlig, bör du åsidosätta färdigheter-utvärderingen genom att `disableCacheReprocessingChangeDetection` ange parametern `true`till:

1. Anropa Update färdigheter och ändra färdigheter-definitionen.
1. Lägg till `disableCacheReprocessingChangeDetection=true` parametern i begäran.
1. Skicka ändringen.

Genom att ange den här parametern ser du till att endast uppdateringar av färdigheter-definitionen genomförs och ändringen utvärderas inte för effekter på den befintliga sökkorpus.

I följande exempel visas en uppdatering färdigheter-begäran med parametern:

```http
PUT https://customerdemos.search.windows.net/skillsets/callcenter-text-skillset?api-version=2019-05-06-Preview&disableCacheReprocessingChangeDetection=true
```

### <a name="bypass-data-source-validation-checks"></a>Kringgå verifierings kontroller för data Källa

De flesta ändringar av en data käll definition kommer att ogiltig förklara cachen. Men för scenarier där du vet att en ändring inte ska göra detta ska du inte göra en ändring i cachen, till exempel ändra en anslutnings sträng eller rotera nyckeln på lagrings kontot – Lägg till`ignoreResetRequirement` parametern i uppdateringen av data källan. Om du anger den `true` här parametern så att incheckning kan gå igenom, utan att utlösa ett återställnings villkor som skulle resultera i att alla objekt byggs om och fylls från från början.

```http
PUT https://customerdemos.search.windows.net/datasources/callcenter-ds?api-version=2019-05-06-Preview&ignoreResetRequirement=true
```

### <a name="force-skillset-evaluation"></a>Framtvinga färdigheter-utvärdering

Syftet med cachen är att undvika onödig bearbetning, men vi antar att du gör en ändring i en färdighet som indexeraren inte identifierar (till exempel ändra något i extern kod, till exempel en anpassad färdighet).

I det här fallet kan du använda [återställnings kunskaper](https://docs.microsoft.com/rest/api/searchservice/2019-05-06-preview/reset-skills) för att tvinga ombearbetning av en viss färdighet, inklusive eventuella efterföljande kunskaper som har ett beroende på den aktuella färdighetens utdata. Detta API accepterar en POST-begäran med en lista med kunskaper som ska ogiltig förklaras och markeras för ombearbetning. När du har återställt färdigheter kan du köra indexeraren för att anropa pipelinen.

## <a name="change-detection"></a>Ändrings identifiering

När du aktiverar ett cacheminne utvärderar indexeraren ändringar i din pipeline-sammansättning för att avgöra vilket innehåll som kan återanvändas och vilka som behöver bearbetas igen. I det här avsnittet räknas ändringar som inte validerar cachen direkt, följt av ändringar som utlöser stegvis bearbetning. 

### <a name="changes-that-invalidate-the-cache"></a>Ändringar som invaliderar cachen

En ogiltig ändring är en där hela cachen inte längre är giltig. Ett exempel på en ogiltig ändring är en plats där data källan uppdateras. Här är en fullständig lista över ändringar som skulle göra din cache ogiltig:

* Ändra till typ av data Källa
* Ändra till data källans behållare
* Autentiseringsuppgifter för datakälla
* Princip för ändrings identifiering av data Källa
* Ta bort identifierings princip för data källor
* Fält mappningar för indexerare
* Indexerings parametrar
    * Tolknings läge
    * Uteslutna fil namns tillägg
    * Indexerade fil namns tillägg
    * Indexera lagrings metadata endast för dokument med storleks storlek
    * Avgränsade text rubriker
    * Avgränsad text avgränsare
    * Dokument rot
    * Avbildnings åtgärd (ändringar i hur bilder extraheras)

### <a name="changes-that-trigger-incremental-processing"></a>Ändringar som utlöser stegvis bearbetning

Stegvis bearbetning utvärderar din färdigheter-definition och avgör vilka kunskaper som ska köras igen och selektivt uppdaterar de berörda delarna i dokument trädet. Här är en fullständig lista över ändringar som resulterar i stegvisa anrikning:

* Kompetensen i färdigheter har en annan typ. OData-typen för kunskapen uppdateras
* Kunskapsbaserade parametrar har uppdaterats, till exempel URL, standardinställningar eller andra parametrar
* Ändringar i kunskaps utmatningar, kunskapen returnerar ytterligare eller andra utdata
* Kunskaps uppdateringar som uppstår är olika föregångare, kunskaps kedjan har ändrats, dvs. kompetens inmatningar
* Eventuella indata från en överordnad kompetens, om en färdighet som tillhandahåller indata till den här kompetensen uppdateras
* Uppdateringar till platsen för kunskaps lager projektion, resultat i omprojektering av dokument
* Ändringar i kunskaps lagrets projektioner, resultat i omprojektering av dokument
* Mappningar av utdatakolumner som har ändrats på en indexerare resulterar i omprojektering av dokument till indexet

## <a name="api-reference"></a>API-referens

REST API- `2019-05-06-Preview` versionen ger stegvis berikning genom ytterligare egenskaper för indexerare, färdighetsuppsättningar och data källor. Utöver referens dokumentationen finns mer information om hur du anropar API: erna i [Konfigurera cachelagring för stegvis berikning](search-howto-incremental-index.md) .

+ [Skapa indexerare (API-version = 2019-05 -06 – för hands version)](https://docs.microsoft.com/rest/api/searchservice/2019-05-06-preview/create-indexer) 

+ [Uppdatera indexerare (API-version = 2019-05 -06 – för hands version)](https://docs.microsoft.com/rest/api/searchservice/2019-05-06-preview/update-indexer) 

+ [Uppdatera färdigheter (API-version = 2019-05 -06-Preview)](https://docs.microsoft.com/rest/api/searchservice/2019-05-06-preview/update-skillset) (ny URI-parameter på begäran)

+ [Återställ kompetens (api-version=2019-05-06-förhandsversion)](https://docs.microsoft.com/rest/api/searchservice/2019-05-06-preview/reset-skills)

+ Databas indexerare (Azure SQL, Cosmos DB). Vissa indexerare hämtar data via frågor. För frågor som hämtar data stöder [uppdaterings data källan](https://docs.microsoft.com/rest/api/searchservice/update-data-source) en ny parameter på en begäran- **ignoreResetRequirement**, som ska ställas `true` in på när din uppdaterings åtgärd inte ska göra en ogiltig verifiering av cachen. 

  Använd **ignoreResetRequirement** sparsamt eftersom det kan leda till oavsiktlig inkonsekvens i dina data som inte kommer att identifieras enkelt.

## <a name="next-steps"></a>Nästa steg

Stegvis anrikning är en kraftfull funktion som utökar ändrings spårningen till färdighetsuppsättningar och AI-anrikning. AIncremental-anrikning möjliggör åter användning av befintligt bearbetat innehåll när du itererar över färdigheter-designen.

I nästa steg ska du aktivera cachelagring på en befintlig indexerare eller lägga till ett cacheminne när du definierar en ny indexerare.

> [!div class="nextstepaction"]
> [Konfigurera cachelagring för stegvis anrikning](search-howto-incremental-index.md)
