---
title: Gränser och gränser – QnA Maker
description: QnA Maker har meta-gränser för delar av kunskaps basen och tjänsten. Det är viktigt att du håller kunskaps basen inom dessa gränser för att testa och publicera.
ms.topic: reference
ms.date: 02/14/2020
ms.openlocfilehash: 6375a6c6efc0c7016d9947e04e9479385aa80af5
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "78273343"
---
# <a name="qna-maker-knowledge-base-limits-and-boundaries"></a>QnA Maker grundläggande gränser och gränser för kunskap

QnA Maker gränser som anges nedan är en kombination av [gränserna för Azure kognitiv sökning-pris nivåer](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity) och [QNA Maker pris nivå gränser](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/). Du måste känna till båda uppsättningarna med gränser för att förstå hur många kunskaps baser du kan skapa per resurs och hur stor varje kunskaps bas kan växa.

## <a name="knowledge-bases"></a>Kunskaps banker

Det maximala antalet kunskaps baser baseras på [gränserna för Azure kognitiv sökning nivån](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity).

|**Azure Kognitiv sökning-nivå** | **Kostnadsfri** | **Basic** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|Högsta tillåtna antal publicerade kunskaps banker|2|14|49|199|199|2 999|

 Om din nivå till exempel har 15 tillåtna index, kan du publicera 14 kunskaps banker (1 index per publicerad kunskaps bas). Det femtonde indexet `testkb`används för alla kunskaps banker för redigering och testning.

## <a name="extraction-limits"></a>Extraherings gränser

### <a name="file-naming-constraints"></a>Begränsningar för fil namn givning

Fil namn får inte innehålla följande tecken:

|Använd inte Character|
|--|
|Enkelt citat tecken`'`|
|Dubbelt citat tecken`"`|

### <a name="maximum-file-size"></a>Maximal filstorlek

|Format|Maximal fil storlek (MB)|
|--|--|
|`.docx`|10|
|`.pdf`|25|
|`.tsv`|10|
|`.txt`|10|
|`.xlsx`|3|

### <a name="maximum-number-of-files"></a>Maximalt antal filer

Det maximala antalet filer som kan extraheras och maximal fil storlek baseras på **[QNA Maker pris nivå gränser](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/)**.

### <a name="maximum-number-of-deep-links-from-url"></a>Maximalt antal djup länkar från URL

Det maximala antalet djup länkar som kan crawlas för extrahering av kring från en URL-sida är **20**.

## <a name="metadata-limits"></a>Metadata-gränser

Metadata visas som en textbaserad nyckel: värde par, till exempel `product:windows 10`. Den lagras och jämförs med gemener.

### <a name="by-azure-cognitive-search-pricing-tier"></a>Av Azure Kognitiv sökning pris nivå

Det maximala antalet metadata-fält per kunskaps bas baseras på **[Azure kognitiv sökning nivå gränser](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity)**.

|**Azure Kognitiv sökning-nivå** | **Kostnadsfri** | **Basic** |**S1** | **S2**| **S3** |**S3 HD**|
|---|---|---|---|---|---|----|
|Fält för maximalt antal metadata per QnA Maker tjänst (i alla KB)|1,000|100 *|1,000|1,000|1,000|1,000|

### <a name="by-name-and-value"></a>Efter namn och värde

Längden på och godkända tecken för namn och värde för metadata visas i följande tabell.

|Objekt|Tillåtna tecken|Matchning av regex-mönster|Max tecken|
|--|--|--|--|
|Namn (nyckel)|Innebär<br>alfanumeriskt (bokstäver och siffror)<br>`_`under streck<br> Får inte innehålla blank steg.|`^[a-zA-Z0-9_]+$`|100|
|Värde|Tillåter allt utom<br>`:`kolon<br>`|`(lodrät pipe)<br>Endast ett värde tillåts.|`^[^:|]+$`|500|
|||||

## <a name="knowledge-base-content-limits"></a>Innehålls gränser för kunskaps bas
Övergripande begränsningar för innehållet i kunskaps basen:
* Svars textens längd: 25 000
* Text längd: 1 000
* Längd på nyckel/värde-text för metadata: 100
* Tecken som stöds för namn på metadata: alfabet, siffror och`_`
* Tecken som stöds för metadata-värde: `:` alla utom och`|`
* Fil namnets längd: 200
* Fil format som stöds: ". tsv", ". pdf", ". txt", ". docx", ". xlsx".
* Maximalt antal alternativa frågor: 300
* Maximalt antal frågor/svar-par: beror på den valda **[Azure kognitiv sökning-nivån](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity#document-limits)** . Ett fråge-och svars par mappar till ett dokument i Azure Kognitiv sökning index.
* URL/HTML-sida: 1 000 000 tecken

## <a name="create-knowledge-base-call-limits"></a>Skapa svars gränser för kunskaps bas:
Dessa motsvarar gränserna för varje åtgärd för att skapa kunskaps bas. Det gör du genom att klicka på *skapa KB* eller anropa CreateKnowledgeBase-API: et.
* Maximalt antal alternativa frågor per svar: 300
* Maximalt antal URL: er: 10
* Maximalt antal filer: 10

## <a name="update-knowledge-base-call-limits"></a>Uppdatera anrops gränser för kunskaps bas
Dessa representerar gränserna för varje uppdaterings åtgärd. Det gör du genom att klicka på *Spara och träna* eller anropa UpdateKnowledgeBase-API: et.
* Längd för varje käll namn: 300
* Maximalt antal alternativa frågor som lagts till eller tagits bort: 300
* Maximalt antal metadata-fält har lagts till eller tagits bort: 10
* Maximalt antal URL: er som kan uppdateras: 5

## <a name="next-steps"></a>Nästa steg

Lär dig när och hur du ändrar [pris nivåer för tjänster](How-To/set-up-qnamaker-service-azure.md#upgrade-qna-maker-sku).
