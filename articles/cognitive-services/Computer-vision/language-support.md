---
title: Språk stöd – Visuellt innehåll
titleSuffix: Azure Cognitive Services
description: Den här artikeln innehåller en lista över naturliga språk som stöds av Visuellt innehåll funktioner; OCR, Identifiera text och läsa.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 04/17/2019
ms.author: pafarley
ms.openlocfilehash: a834c68119340d796f87971912a07fc0524a6d21
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "79220142"
---
# <a name="language-support-for-computer-vision"></a>Språk stöd för Visuellt innehåll

Vissa funktioner i Visuellt innehåll stöd för flera språk. de funktioner som inte nämns här stöder bara engelska.

## <a name="text-recognition"></a>Textigenkänning

Visuellt innehåll kan identifiera text på många språk. Mer specifikt stöder [OCR](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc) API en mängd olika språk, medan API: et för [läs](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/2afb498089f74080d7ef85eb) -och [identifiera text](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/587f2c6a154055056008f200) stöder endast engelska. Se [identifiera utskrift och handskriven text](concept-recognizing-text.md) för mer information om den här funktionen och fördelarna med varje API.

OCR identifierar automatiskt språket för det inmatade materialet, så det finns inget behov av att ange en språkkod i API-anropet. Språk koder returneras dock alltid som värdet för `"language"` NODEN i JSON-svaret.

|Språk| Språkkod | OCR-API |
|:-----|:----:|:-----:|
|Arabiska | `ar`|✔ |
|Kinesiska (förenklad) | `zh-Hans`|✔ |
|Kinesiska (traditionell) | `zh-Hant`|✔ |
|Tjeckiska | `cs` |✔ |
|Danska | `da` |✔ |
|Nederländska | `nl` |✔ |
|Svenska | `en` |✔ |
|Finska | `fi` |✔ |
|Franska | `fr` |✔ |
|Tyska | `de` |✔ |
|Grekiska | `el` |✔ |
|Ungerska | `hu` |✔ |
|Italienska | `it` |✔ |
|Japanska | `ja` |✔ |
|Koreansk | `ko` |✔ |
|Norska | `nb` |✔ |
|Polska | `pl` |✔ |
|Portugisiska | `pt` |✔ |
|Rumänska | `ro` |✔ |
|Ryska | `ru` |✔ |
|Serbiska (kyrillisk) | `sr-Cyrl` |✔ |
|Serbiska (latinsk) (Serbien) | `sr-Latn` |✔ |
|Slovakiska | `sk` |✔ |
|Spanska | `es` |✔ |
|Svenska | `sw` |✔ |
|Turkiska | `tr` |✔ |

## <a name="image-analysis"></a>Bild analys

Vissa åtgärder för API: et [analys-image](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa) kan returnera resultat på andra språk, som anges `language` med Frågeparametern. Andra åtgärder returnerar resultat på engelska oavsett vilket språk som anges, och andra genererar ett undantag för språk som inte stöds. Åtgärder anges med parametrarna `visualFeatures` och, `details` i [översikten](home.md) finns en lista över alla åtgärder som du kan utföra med bild analys.

|Språk | Språkkod | Kategorier | Taggar | Beskrivning | Vuxen | Varumärken | Färg | Ansikten | ImageType | Objekt | Kändisar | Landmärken |
|:---|:---:|:----:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|Kinesiska | `zh`    | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Svenska | `en`   | ✔ | ✔| ✔|✔|✔|✔|✔|✔|✔|✔|✔|
|Japanska | `ja`   | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Portugisiska | `pt` | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|
|Spanska | `es`    | ✔ | ✔| ✔|-|-|-|-|-|❌|✔|✔|

## <a name="next-steps"></a>Nästa steg

Kom igång med Visuellt innehåll funktionerna som nämns i den här hand boken.

* [Analysera en lokal avbildning (REST)](./quickstarts/csharp-analyze.md)
* [Extrahera utskriven text (REST)](./quickstarts/csharp-print-text.md)