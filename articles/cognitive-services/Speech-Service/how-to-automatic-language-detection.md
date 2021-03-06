---
title: Så här använder du automatisk språk identifiering för tal till text
titleSuffix: Azure Cognitive Services
description: 'Tal-SDK: n stöder automatisk språk identifiering för tal till text. När du använder den här funktionen jämförs ljudet med den angivna listan med språk och den mest sannolika matchningen fastställs. Det returnerade värdet kan sedan användas för att välja den språk modell som används för tal till text.'
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: trbye
zone_pivot_groups: programming-languages-set-two
ms.openlocfilehash: fefbe793fa4a6b90ba9bf8d468d42dcbd315759c
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "81402203"
---
# <a name="automatic-language-detection-for-speech-to-text"></a>Automatisk språk identifiering för tal till text

Automatisk språk identifiering används för att avgöra den mest sannolika matchningen för ljud som skickas till tal-SDK: n jämfört med en lista med angivna språk. Värdet som returneras av automatisk språk identifiering används sedan för att välja språk modell för tal till text, vilket ger en mer exakt avskrift. Information om vilka språk som är tillgängliga finns i [språk stöd](language-support.md).

I den här artikeln får du lära dig hur du `AutoDetectSourceLanguageConfig` använder för att `SpeechRecognizer` skapa ett objekt och hämta det identifierade språket.

> [!IMPORTANT]
> Den här funktionen är endast tillgänglig för tal-SDK för C#, C++, Java och python.

## <a name="automatic-language-detection-with-the-speech-sdk"></a>Automatisk språk identifiering med talet SDK

Automatisk språk identifiering har för närvarande en gräns på Server sidan på två språk per identifiering. Behåll den här begränsningen i åtanke när `AudoDetectSourceLanguageConfig` du har byggt ditt objekt. I exemplen nedan skapar du en `AutoDetectSourceLanguageConfig`och använder den sedan för att skapa en. `SpeechRecognizer`

> [!TIP]
> Du kan också ange en anpassad modell som ska användas när du utför tal till text. Mer information finns i [använda en anpassad modell för automatisk språk identifiering](#use-a-custom-model-for-automatic-language-detection).

Följande kodfragment visar hur du använder automatisk språk identifiering i dina appar:

::: zone pivot="programming-language-csharp"

```csharp
var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromLanguages(
        new string[] { "en-US", "de-DE" });

using (var recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig))
{
    var speechRecognitionResult = await recognizer.RecognizeOnceAsync();
    var autoDetectSourceLanguageResult =
        AutoDetectSourceLanguageResult.FromResult(speechRecognitionResult);
    var detectedLanguage = autoDetectSourceLanguageResult.Language;
}
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromLanguages({ "en-US", "de-DE" });

auto recognizer = SpeechRecognizer::FromConfig(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig);

speechRecognitionResult = recognizer->RecognizeOnceAsync().get();
auto autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult::FromResult(speechRecognitionResult);
auto detectedLanguage = autoDetectSourceLanguageResult->Language;
```

::: zone-end

::: zone pivot="programming-language-java"

```java
AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromLanguages(Arrays.asList("en-US", "de-DE"));

SpeechRecognizer recognizer = new SpeechRecognizer(
    speechConfig,
    autoDetectSourceLanguageConfig,
    audioConfig);

Future<SpeechRecognitionResult> future = recognizer.recognizeOnceAsync();
SpeechRecognitionResult result = future.get(30, TimeUnit.SECONDS);
AutoDetectSourceLanguageResult autoDetectSourceLanguageResult =
    AutoDetectSourceLanguageResult.fromResult(result);
String detectedLanguage = autoDetectSourceLanguageResult.getLanguage();

recognizer.close();
speechConfig.close();
autoDetectSourceLanguageConfig.close();
audioConfig.close();
result.close();
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
auto_detect_source_language_config = \
        speechsdk.languageconfig.AutoDetectSourceLanguageConfig(languages=["en-US", "de-DE"])
speech_recognizer = speechsdk.SpeechRecognizer(
        speech_config=speech_config, 
        auto_detect_source_language_config=auto_detect_source_language_config, 
        audio_config=audio_config)
result = speech_recognizer.recognize_once()
auto_detect_source_language_result = speechsdk.AutoDetectSourceLanguageResult(result)
detected_language = auto_detect_source_language_result.language
```

::: zone-end

## <a name="use-a-custom-model-for-automatic-language-detection"></a>Använd en anpassad modell för automatisk språk identifiering

Förutom språk identifiering med tal tjänst modeller kan du ange en anpassad modell för bättre igenkänning. Om ingen anpassad modell har angetts använder tjänsten standard språk modellen.

Kodfragmenten nedan visar hur du anger en anpassad modell i ditt anrop till röst tjänsten. Om det identifierade språket `en-US`är, används standard modellen. Om det identifierade språket `fr-FR`är, används slut punkten för den anpassade modellen:

::: zone pivot="programming-language-csharp"

```csharp
var sourceLanguageConfigs = new SourceLanguageConfig[]
{
    SourceLanguageConfig.FromLanguage("en-US"),
    SourceLanguageConfig.FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR")
};
var autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-cpp"

```cpp
std::vector<std::shared_ptr<SourceLanguageConfig>> sourceLanguageConfigs;
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("en-US"));
sourceLanguageConfigs.push_back(
    SourceLanguageConfig::FromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

auto autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig::FromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-java"

```java
List sourceLanguageConfigs = new ArrayList<SourceLanguageConfig>();
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("en-US"));
sourceLanguageConfigs.add(
    SourceLanguageConfig.fromLanguage("fr-FR", "The Endpoint Id for custom model of fr-FR"));

AutoDetectSourceLanguageConfig autoDetectSourceLanguageConfig =
    AutoDetectSourceLanguageConfig.fromSourceLanguageConfigs(
        sourceLanguageConfigs);
```

::: zone-end

::: zone pivot="programming-language-python"

```Python
 en_language_config = speechsdk.languageconfig.SourceLanguageConfig("en-US")
 fr_language_config = speechsdk.languageconfig.SourceLanguageConfig("fr-FR", "The Endpoint Id for custom model of fr-FR")
 auto_detect_source_language_config = speechsdk.languageconfig.AutoDetectSourceLanguageConfig(
        sourceLanguageConfigs=[en_language_config, fr_language_config])
```

::: zone-end

## <a name="next-steps"></a>Nästa steg

- [Dokumentation om tal SDK-referens](speech-sdk.md)
