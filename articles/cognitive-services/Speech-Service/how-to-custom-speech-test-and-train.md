---
title: Förbereda data för Custom Speech tal-tjänsten
titleSuffix: Azure Cognitive Services
description: När du testar precisionen för Microsoft tal igenkänning eller tränar dina anpassade modeller, behöver du ljud-och text data. På den här sidan tar vi upp de typer av data, hur de används och hur de hanteras.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 78857709447f99895c36f23d8760f44f8468ba7c
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "81402131"
---
# <a name="prepare-data-for-custom-speech"></a>Förbereda data för Custom Speech

När du testar precisionen för Microsoft tal igenkänning eller tränar dina anpassade modeller, behöver du ljud-och text data. På den här sidan tar vi upp de typer av data, hur de används och hur de hanteras.

## <a name="data-types"></a>Datatyper

I den här tabellen listas godkända data typer, när varje datatyp ska användas och den rekommenderade kvantiteten. Det krävs inte alla data typer för att skapa en modell. Data kraven varierar beroende på om du skapar ett test eller tränar en modell.

| Datatyp | Används för testning | Rekommenderad kvantitet | Används för utbildning | Rekommenderad kvantitet |
|-----------|-----------------|----------|-------------------|----------|
| [Ljud](#audio-data-for-testing) | Ja<br>Används för visuell granskning | 5 + ljudfiler | Nej | Ej tillämpligt |
| [Ljud + medmärkta avskrifter](#audio--human-labeled-transcript-data-for-testingtraining) | Ja<br>Används för att utvärdera noggrannhet | 0,5 – 5 timmars ljud | Ja | 1 – 1000 timmars ljud |
| [Relaterad text](#related-text-data-for-training) | Nej | Ej tillämpligt | Ja | 1-200 MB relaterad text |

Filerna ska grupperas efter typ i en data uppsättning och laddas upp som en zip-fil. Varje data uppsättning får bara innehålla en enda datatyp.

> [!TIP]
> Överväg att använda exempel data för att snabbt komma igång. <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/sampledata/customspeech" target="_target">Exempel Custom Speech data <span class="docon docon-navigate-external x-hidden-focus"></span> </a> finns i den här GitHub-lagringsplatsen

## <a name="upload-data"></a>Ladda upp data

Om du vill överföra dina data går du till <a href="https://speech.microsoft.com/customspeech" target="_blank">Custom Speech <span class="docon docon-navigate-external x-hidden-focus"> </span>Portal </a>. Från portalen klickar du på **överför data** för att starta guiden och skapa din första data uppsättning. Du uppmanas att välja en tal data typ för din data uppsättning innan du tillåter att du överför dina data.

![Välj ljud från tal portalen](./media/custom-speech/custom-speech-select-audio.png)

Varje data uppsättning som du överför måste uppfylla kraven för den datatyp som du väljer. Dina data måste vara korrekt formaterade innan de överförs. Korrekt formaterade data säkerställer att de bearbetas korrekt av tjänsten Custom Speech. Kraven visas i följande avsnitt.

När din data uppsättning har laddats upp har du några alternativ:

* Du kan navigera till fliken **test** och visuellt inspektera enbart ljud eller ljud + mänskligt avskrifts data.
* Du kan navigera till fliken **utbildning** och använda ljud-och färg SKRIFTS data eller relaterade text data för att träna en anpassad modell.

## <a name="audio-data-for-testing"></a>Ljud data för testning

Ljuddata är optimala för att testa noggrannheten hos Microsofts bas linje tal-till-text-modell eller en anpassad modell. Kom ihåg att ljuddata används för att kontrol lera precisionen för tal med avseende på en speciell modells prestanda. Om du vill kvantifiera precisionen för en modell använder du [ljud + mänskligt avskrifts data](#audio--human-labeled-transcript-data-for-testingtraining).

Använd den här tabellen för att se till att ljudfilerna är korrekt formaterade för användning med Custom Speech:

| Egenskap                 | Värde                 |
|--------------------------|-----------------------|
| Fil format              | RIFF (WAV)            |
| Samplings frekvens              | 8 000 Hz eller 16 000 Hz |
| Kanaler                 | 1 (mono)              |
| Maximal längd per ljud | 2 timmar               |
| Exempel format            | PCM, 16-bitars           |
| Arkiv format           | .zip                  |
| Maximal Arkiv storlek     | 2 GB                  |

[!INCLUDE [supported-audio-formats](includes/supported-audio-formats.md)]

> [!TIP]
> När du överför utbildning och testar data får. zip-filens storlek inte överskrida 2 GB. Om du behöver mer data för utbildning delar du in det i flera. zip-filer och laddar upp dem separat. Senare kan du välja att träna från *flera* data uppsättningar. Du kan dock bara testa från en *enda* data uppsättning.

Använd <a href="http://sox.sourceforge.net" target="_blank" rel="noopener">SoX <span class="docon docon-navigate-external x-hidden-focus"></span> </a> för att verifiera ljud egenskaperna eller konvertera det befintliga ljudet till rätt format. Nedan visas några exempel på hur var och en av dessa aktiviteter kan göras via SoX-kommando raden:

| Aktivitet | Beskrivning | SoX-kommando |
|----------|-------------|-------------|
| Kontrol lera ljud formatet | Använd det här kommandot för att kontrol lera<br>ljud fil formatet. | `sox --i <filename>` |
| Konvertera ljud format | Använd det här kommandot för att konvertera<br>ljud filen till en kanal, 16-bitars 16 KHz. | `sox <input> -b 16 -e signed-integer -c 1 -r 16k -t wav <output>.wav` |

## <a name="audio--human-labeled-transcript-data-for-testingtraining"></a>Ljud + mänskligt avskrifts data för testning/utbildning

För att mäta noggrannheten hos Microsofts tal-till-text-precision vid bearbetning av ljudfiler, måste du tillhandahålla medmärkta avskrifter (ord för ord) för jämförelse. Även om det ofta finns tids krävande avskrifter, är det nödvändigt att utvärdera precisionen och träna modellen för dina användnings fall. Kom ihåg att förbättringarna i igenkänningen bara är lika lämpliga som de data som tillhandahålls. Därför är det viktigt att endast avskrifter av hög kvalitet överförs.

| Egenskap                 | Värde                               |
|--------------------------|-------------------------------------|
| Fil format              | RIFF (WAV)                          |
| Samplings frekvens              | 8 000 Hz eller 16 000 Hz               |
| Kanaler                 | 1 (mono)                            |
| Maximal längd per ljud | 2 timmar (testning)/60 s (utbildning) |
| Exempel format            | PCM, 16-bitars                         |
| Arkiv format           | .zip                                |
| Maximal zip-storlek         | 2 GB                                |

[!INCLUDE [supported-audio-formats](includes/supported-audio-formats.md)]

> [!NOTE]
> När du överför utbildning och testar data får. zip-filens storlek inte överskrida 2 GB. Du kan bara testa från en *enda* data uppsättning, se till att hålla den inom lämplig fil storlek. Dessutom kan varje tränings fil inte överstiga 60 sekunder, annars kommer den att vara fel.

För att lösa problem som Word-borttagning eller ersättning krävs en stor mängd data för att förbättra igenkänningen. I allmänhet rekommenderar vi att du ger ord för ord-avskrifter i ungefär 10 till 1 000 timmar av ljud. Transkriptioner för alla WAV-filer bör ingå i en enda fil med oformaterad text. Varje rad i transkriptionsfilen ska innehålla namnet på en av ljudfilerna följt av motsvarande transkription. Filnamnet och transkriptionen ska separeras med ett tabbtecken (\t).

  Ett exempel:
```
  speech01.wav  speech recognition is awesome
  speech02.wav  the quick brown fox jumped all over the place
  speech03.wav  the lazy dog was not amused
```

> [!IMPORTANT]
> Transkriptionen ska kodas som UTF-8-byteordningsmärke (BOM).

Transkriptionerna textnormaliseras så att de kan bearbetas av systemet. Det finns dock några viktiga normaliseringar som måste utföras innan data överförs till tal Studio. För det språk som ska användas när du förbereder dina avskrifter, se [så här skapar du en](how-to-custom-speech-human-labeled-transcriptions.md) medhjälpad avskrift

När du har samlat in dina ljudfiler och motsvarande avskrifter, paketera dem som en enda. zip-fil innan du överför till <a href="https://speech.microsoft.com/customspeech" target="_blank">Custom Speech Portal <span class="docon docon-navigate-external x-hidden-focus"> </span> </a>. Nedan visas ett exempel på en data uppsättning med tre ljudfiler och en fil med mänsklig märkning:

> [!div class="mx-imgBorder"]
> ![Välj ljud från tal portalen](./media/custom-speech/custom-speech-audio-transcript-pairs.png)

## <a name="related-text-data-for-training"></a>Relaterade text data för utbildning

Produkt namn eller funktioner som är unika bör innehålla relaterade text data för utbildning. Relaterad text hjälper till att säkerställa korrekt igenkänning. Två typer av relaterade text data kan tillhandahållas för att förbättra igenkänningen:

| Datatyp | Hur dessa data förbättrar igenkänningen |
|-----------|------------------------------------|
| Meningar (yttranden) | Förbättra precisionen när du känner igen produkt namn eller branschspecifika vokabulär inom ramen för en mening. |
| Uttal | Förbättra uttal av ovanliga termer, akronymer eller andra ord med odefinierade uttal. |

Meningar kan anges som en enda textfil eller flera textfiler. För att förbättra precisionen använder du text data som är närmare den förväntade talade yttranden. Uttal ska anges som en enskild textfil. Allt kan paketeras som en enda zip-fil och överföras till <a href="https://speech.microsoft.com/customspeech" target="_blank">Custom Speech <span class="docon docon-navigate-external x-hidden-focus"> </span>Portal </a>.

### <a name="guidelines-to-create-a-sentences-file"></a>Rikt linjer för att skapa en menings fil

Om du vill skapa en anpassad modell med hjälp av meningar måste du ange en lista över exempel-yttranden. Yttranden behöver _inte_ vara slutförd eller grammatiskt korrekt, men de måste avspegla den talade ingången som du förväntar dig i produktionen. Om du vill att vissa villkor har ökat vikt, lägger du till flera meningar som innehåller dessa specifika villkor.

Som allmän vägledning är modell anpassningen effektiv när övnings texten är så nära som möjligt för den verkliga text som förväntas i produktionen. Kundspecifika jargong och fraser som du riktar in dig på bör tas med i övnings texten. När det är möjligt kan du försöka att ha en mening eller ett nyckelord som styrs på en separat rad. För nyckelord och fraser som är viktiga för dig (till exempel produkt namn) kan du kopiera dem några gånger. Men kom ihåg att inte kopiera för mycket – det kan påverka den övergripande igenkännings hastigheten.

Använd den här tabellen för att se till att din relaterade datafil för yttranden är korrekt formaterad:

| Egenskap | Värde |
|----------|-------|
| Textkodning | UTF-8 BOM |
| antal yttrande per rad | 1 |
| Maximal filstorlek | 200 MB |

Dessutom bör du ta hänsyn till följande begränsningar:

* Undvik upprepade tecken mer än fyra gånger. Till exempel: "AAAA" eller "uuuu".
* Använd inte specialtecken eller UTF-8-tecken ovan `U+00A1`.
* URI: er kommer att avvisas.

### <a name="guidelines-to-create-a-pronunciation-file"></a>Rikt linjer för att skapa en uttal-fil

Om det finns ovanliga termer utan standard uttal som användarna kommer att stöta på eller använder, kan du ange en anpassad uttal-fil för att förbättra igenkänningen.

> [!IMPORTANT]
> Vi rekommenderar inte att du använder anpassade uttal-filer för att ändra uttal av vanliga ord.

Detta inkluderar exempel på en talade uttryck och ett anpassat uttal för var och en:

| Identifierat/visat formulär | Tal format |
|--------------|--------------------------|
| 3CPO | tre c p o |
| CNTK | c n t k |
| STANDARDEN | i tredubbel e |

Det talade formuläret är den fonetiska sekvensen som har stavats ut. Det kan bestå av bokstäver, ord, stavelser eller en kombination av alla tre.

Ett anpassat uttal är tillgängligt på engelska`en-US`() och tyska`de-DE`(). I den här tabellen visas tecken som stöds efter språk:

| Språk | Nationell inställning | Tabbtecken |
|----------|--------|------------|
| Svenska | `en-US` | `a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z` |
| Tyska | `de-DE` | `ä, ö, ü, a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z` |

Använd följande tabell för att kontrol lera att den relaterade data filen för uttal är korrekt formaterad. Uttal-filerna är små och får bara bestå av några kilobyte i storlek.

| Egenskap | Värde |
|----------|-------|
| Textkodning | UTF-8-struktur (ANSI stöds också för engelska) |
| antal uttal per rad | 1 |
| Maximal filstorlek | 1 MB (1 KB ledig nivå) |

## <a name="next-steps"></a>Nästa steg

* [Inspektera dina data](how-to-custom-speech-inspect-data.md)
* [Utvärdera dina data](how-to-custom-speech-evaluate-data.md)
* [Träna din modell](how-to-custom-speech-train-model.md)
* [Distribuera din modell](how-to-custom-speech-deploy-model.md)
