---
title: ta med fil
description: ta med fil
services: cognitive-services
author: diberry
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 02/14/2020
ms.topic: include
ms.custom: include file
ms.author: diberry
ms.openlocfilehash: 53e6382cf8d046b2c9818b906890bc64642fd2ed
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77372136"
---
Använd det Language Understanding (LUIS) som redigerar klient biblioteket för .NET till:

* Skapa en app
* Lägg till avsikter, entiteter och exempel yttranden
* Lägga till funktioner som en fras lista
* Träna och publicera appen

[Referens dokumentation](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/languageunderstanding?view=azure-dotnet) | [bibliotek käll kods](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.LUIS.Authoring) | [redigerings paket (NuGet) C#-](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/) | [exempel](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/documentation-samples/quickstarts/LUIS/LUIS.cs)

## <a name="prerequisites"></a>Krav

* Language Understanding (LUIS) Portal konto – [skapa ett kostnads fritt](https://www.luis.ai)
* Den aktuella versionen av [.net Core](https://dotnet.microsoft.com/download/dotnet-core).


## <a name="setting-up"></a>Konfigurera

### <a name="get-your-language-understanding-luis-starter-key"></a>Hämta din Language Understandings start nyckel (LUIS)

Hämta din [Start nyckel](../luis-how-to-azure-subscription.md#starter-key) genom att skapa en Luis Authoring-resurs. Behåll din nyckel och regionen för nyckeln för nästa steg.

### <a name="create-an-environment-variable"></a>Skapa en miljö variabel

Med din nyckel och region för nyckeln skapar du två miljövariabler för autentisering:

* `COGNITIVESERVICE_AUTHORING_KEY`– Resurs nyckeln för autentisering av dina begär Anden.
* `COGNITIVESERVICE_REGION`– Den region som är kopplad till din nyckel. Till exempel `westus`.

Följ anvisningarna för ditt operativ system.

#### <a name="windows"></a>[Windows](#tab/windows)

```console
setx COGNITIVESERVICE_AUTHORING_KEY <replace-with-your-authoring-key>
setx COGNITIVESERVICE_REGION <replace-with-your-authoring-region>
```

Starta om konsol fönstret när du har lagt till miljövariabeln.

#### <a name="linux"></a>[Linux](#tab/linux)

```bash
export COGNITIVESERVICE_AUTHORING_KEY=<replace-with-your-authoring-key>
export COGNITIVESERVICE_REGION=<replace-with-your-authoring-region>
```

När du har lagt till miljövariabeln så kör `source ~/.bashrc` från konsolfönstret så att ändringarna träder i kraft.

#### <a name="macos"></a>[macOS](#tab/unix)

Redigera din `.bash_profile`och Lägg till miljövariabeln:

```bash
export COGNITIVESERVICE_AUTHORING_KEY=<replace-with-your-authoring-key>
export COGNITIVESERVICE_REGION=<replace-with-your-authoring-region>
```

När du har lagt till miljövariabeln så kör `source .bash_profile` från konsolfönstret så att ändringarna träder i kraft.
***

### <a name="create-a-new-c-application"></a>Skapa ett nytt C#-program

Skapa ett nytt .NET Core-program i önskat redigerings program eller IDE.

1. I ett konsol fönster (till exempel cmd, PowerShell eller bash) använder du kommandot dotNet `new` för att skapa en ny konsol app med namnet. `language-understanding-quickstart` Det här kommandot skapar ett enkelt "Hello World" C#-projekt med en enda käll `Program.cs`fil:.

    ```dotnetcli
    dotnet new console -n language-understanding-quickstart
    ```

1. Ändra katalogen till mappen nyligen skapade appar.

1. Du kan bygga programmet med:

    ```dotnetcli
    dotnet build
    ```

    Build-utdata får inte innehålla varningar eller fel.

    ```console
    ...
    Build succeeded.
     0 Warning(s)
     0 Error(s)
    ...
    ```


### <a name="install-the-sdk"></a>Installera SDK:n

I program katalogen installerar du klient biblioteket Language Understanding (LUIS) för .NET med följande kommando:

```dotnetcli
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring --version 3.0.0
```

Om du använder Visual Studio IDE är klient biblioteket tillgängligt som ett nedladdnings Bart NuGet-paket.


## <a name="object-model"></a>Objekt modell

Redigerings klienten för Language Understanding (LUIS) är ett [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet) -objekt som autentiserar till Azure, som innehåller din redigerings nyckel.

När klienten har skapats använder du den här klienten för att få åtkomst till funktioner, inklusive:

* Appar- [skapa](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet), [ta bort](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.deleteasync?view=azure-dotnet), [publicera](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet)
* Exempel yttranden – [Lägg till via batch](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.batchasync?view=azure-dotnet), [ta bort efter ID](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.deleteasync?view=azure-dotnet)
* Funktioner – hantera [fras listor](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.featuresextensions.addphraselistasync?view=azure-dotnet)
* Modell-hantera [avsikter](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet) och entiteter
* Mönster – hantera [mönster](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.patternextensions?view=azure-dotnet)
* Träna – [träna](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.trainversionasync?view=azure-dotnet) appen och avsökningen efter [kurs status](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet)
* [Versioner](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.versionsextensions?view=azure-dotnet) – hantera med klona, exportera och ta bort


## <a name="code-examples"></a>Kodexempel

De här kodfragmenten visar hur du gör följande med redigerings klient biblioteket för Language Understanding (LUIS) för .NET:

* [Skapa en app](#create-a-luis-app)
* [Lägg till entiteter](#create-entities-for-the-app)
* [Lägg till avsikter](#create-intent-for-the-app)
* [Lägga till exempelyttranden](#add-example-utterance-to-intent)
* [Träna appen](#train-the-app)
* [Publicera appen](#publish-a-language-understanding-app)

## <a name="add-the-dependencies"></a>Lägg till beroenden

Från projekt katalogen öppnar du *program.cs* -filen i önskat redigerings program eller IDE. Ersätt den befintliga `using` koden med följande `using` direktiv:

[!code-csharp[Using statements](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=Dependencies)]

## <a name="authenticate-the-client"></a>Autentisera klienten

1. Skapa en variabel för att hantera redigerings nyckeln som hämtas från en miljö variabel med `COGNITIVESERVICES_AUTHORING_KEY`namnet. Om du har skapat miljövariabeln när programmet har startats måste redigeraren, IDE eller gränssnittet som kör den stängas och läsas in igen för att få åtkomst till variabeln. Metoderna kommer att skapas senare.

1. Skapa variabler för att hålla din redigerings region och slut punkt. Redigerings nyckelns region är beroende av var du redigerar. De [tre redigerings regionerna](../luis-reference-regions.md) är:

    * Ske`australiaeast`
    * Östeuropa`westeurope`
    * USA och andra regioner – `westus` (standard)

    [!code-csharp[Authorization to resource key](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=Variables)]

1. Skapa ett [ApiKeyServiceClientCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.apikeyserviceclientcredentials?view=azure-dotnet) -objekt med din nyckel och Använd den med slut punkten för att skapa ett [LUISAuthoringClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.luisauthoringclient?view=azure-dotnet) -objekt.

    [!code-csharp[Create LUIS client object](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringCreateClient)]

## <a name="create-a-luis-app"></a>Skapa en LUIS-app

1. Skapa en LUIS-app som innehåller NLP-modellen (Natural Language Processing), entiteter och exempel yttranden.

1. Skapa en [ApplicationCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.applicationcreateobject?view=azure-dotnet). Namnet och språk kulturen är obligatoriska egenskaper.

1. Anropa metoden [Apps. AddAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.addasync?view=azure-dotnet) . Svaret är app-ID: t.

    [!code-csharp[Create a LUIS app](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringCreateApplication)]

## <a name="create-intent-for-the-app"></a>Skapa avsikt för appen
Det primära objektet i en LUIS-Apps modell är avsikten. Avsikten med en gruppering av användarens _uttryck._ En användare kan ställa en fråga eller göra en instruktion som söker efter ett särskilt _avsett_ svar från en robot (eller något annat klient program). Exempel på avsikt är att boka en flygning och fråga om väder i en destinations ort och fråga om kontakt information för kund tjänst.

Skapa en [ModelCreateObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modelcreateobject?view=azure-dotnet) med namnet på den unika avsikten och skicka sedan app-ID, VERSIONS-ID och ModelCreateObject till metoden [Model. AddIntentAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addintentasync?view=azure-dotnet) . Svaret är det ID för avsikten.

[!code-csharp[Create intent](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringAddIntents)]

## <a name="create-entities-for-the-app"></a>Skapa entiteter för appen

Även om entiteter inte krävs finns de i de flesta appar. Entiteten extraherar information från användar uttryck, som krävs för att fullfil användarens avsikt. Det finns flera typer av [färdiga](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions.addprebuiltasync?view=azure-dotnet) och anpassade entiteter, var och en med sina egna DTO-modeller (data Transformation Object).  Vanliga fördefinierade entiteter som ska läggas till i din app är [Number](../luis-reference-prebuilt-number.md), [datetimeV2](../luis-reference-prebuilt-datetimev2.md), [geographyV2](../luis-reference-prebuilt-geographyv2.md), [ordinal](../luis-reference-prebuilt-ordinal.md).

Den här **AddEntities** -metoden `Location` skapade en enkel entitet med två roller `Class` , en enkel entitet `Flight` , en sammansatt entitet och lägger till flera fördefinierade entiteter.

Det är viktigt att veta att entiteter inte har marker ATS med ett avsikts syfte. De kan och används vanligt vis för många syften. Endast yttranden är markerade för ett visst, enskilt syfte.

Skapande metoder för entiteter ingår i [modell](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.modelextensions?view=azure-dotnet) klassen. Varje entitetstyp har en egen DTO-modell (data Transformation Object) som vanligt vis innehåller ordet `model` i namn området [modeller](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models?view=azure-dotnet) .

[!code-csharp[Create entities](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringAddEntities)]

## <a name="add-example-utterance-to-intent"></a>Lägg till exempel uttryck till avsikt

Appen behöver exempel på yttranden för att kunna fastställa en uttryck för avsikt och extrahering av entiteter. Exemplen måste vara riktade mot en viss, enskild avsikt och ska markera alla anpassade entiteter. Fördefinierade entiteter behöver inte markeras.

Lägg till exempel yttranden genom att skapa en lista över [ExampleLabelObject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.examplelabelobject?view=azure-dotnet) -objekt, ett objekt för varje exempel uttryck. Varje exempel bör markera alla entiteter med en ord lista med namn/värde-par för enhets namn och enhets värde. Enhet svärdet bör vara exakt så som det visas i texten i exemplet uttryck.

Anropa [exempel. BatchAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.examplesextensions.batchasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_ExamplesExtensions_BatchAsync_Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_IExamples_System_Guid_System_String_System_Collections_Generic_IList_Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_Models_ExampleLabelObject__System_Threading_CancellationToken_) med app-ID, VERSIONS-ID och listan med exempel. Anropet svarar med en lista över resultat. Du måste kontrol lera varje exempels resultat för att se till att det har lagts till i modellen.

[!code-csharp[Add example utterances to a specific intent](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringBatchAddUtterancesForIntent)]

Metoderna **CreateUtterance** och **CreateLabel** är verktygs metoder som hjälper dig att skapa objekt.

## <a name="train-the-app"></a>Träna appen

När modellen har skapats måste LUIS-appen tränas för den här versionen av modellen. En utbildad modell kan användas i en [behållare](../luis-container-howto.md)eller [publiceras](../luis-how-to-publish-app.md) på mellanlagrings-eller produkt platserna.

Metoden [Train. TrainVersionAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions?view=azure-dotnet) kräver app-ID och VERSIONS-ID.

En mycket liten modell, till exempel den här snabb starten visar, kommer att träna mycket snabbt. Utbildning appen måste innehålla ett avsöknings anrop till [GetStatusAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.trainextensions.getstatusasync?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_TrainExtensions_GetStatusAsync_Microsoft_Azure_CognitiveServices_Language_LUIS_Authoring_ITrain_System_Guid_System_String_System_Threading_CancellationToken_) -metoden för att fastställa när en utbildning har genomförts för program på produktions nivå. Svaret är en lista över [ModelTrainingInfo](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.models.modeltraininginfo?view=azure-dotnet) -objekt med separat status för varje objekt. Alla objekt måste lyckas för att träningen ska anses vara slutförd.

[!code-csharp[Train the app's version](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringTrainVersion)]

## <a name="publish-a-language-understanding-app"></a>Publicera en Language Understanding app

Publicera LUIS-appen med [PublishAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.publishasync?view=azure-dotnet) -metoden. Detta publicerar den aktuella utbildade versionen till den angivna platsen vid slut punkten. Klient programmet använder den här slut punkten för att skicka användarens yttranden för förutsägelse av avsikt och extrahering av enheter.

[!code-csharp[Create entities](~/cognitive-services-dotnet-sdk-samples/documentation-samples/quickstarts/LUIS/LUIS.cs?name=AuthoringPublishVersionAndSlot)]

## <a name="run-the-application"></a>Köra appen

Kör programmet med `dotnet run` kommandot från program katalogen.

```dotnetcli
dotnet run
```

## <a name="clean-up-resources"></a>Rensa resurser

Om du vill rensa upp kan du ta bort LUIS-appen. Borttagning av appen görs med metoden [Apps. DeleteAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.luis.authoring.appsextensions.deleteasync?view=azure-dotnet) . Du kan också ta bort appen från [Luis-portalen](https://www.luis.ai).