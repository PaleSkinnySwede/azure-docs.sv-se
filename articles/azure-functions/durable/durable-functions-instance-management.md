---
title: Hantera instanser i Durable Functions – Azure
description: Lär dig hur du hanterar instanser i Durable Functions-tillägget för Azure Functions.
author: cgillum
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: azfuncdf
ms.openlocfilehash: 1837d342c4476633ee33a8579abe7389ac9bbddf
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80476826"
---
# <a name="manage-instances-in-durable-functions-in-azure"></a>Hantera instanser i Durable Functions i Azure

Om du använder [Durable Functions](durable-functions-overview.md) -tillägget för Azure Functions eller om du vill börja göra det kontrollerar du att du får den bästa användningen av det. Du kan optimera dina Durable Functions Orchestration-instanser genom att lära dig mer om hur du hanterar dem. Den här artikeln går till information om varje instans hanterings åtgärd.

Du kan starta och avsluta instanser, till exempel och du kan fråga instanser, inklusive möjligheten att fråga alla instanser och fråga efter instanser med filter. Dessutom kan du skicka händelser till instanser, vänta på att dirigeringen ska slutföras och hämta URL: er för HTTP Management webhook. I den här artikeln beskrivs även andra hanterings åtgärder, inklusive omspolning av instanser, rensning av instans historik och borttagning av aktivitets nav.

I Durable Functions har du alternativ för hur du vill implementera var och en av dessa hanterings åtgärder. Den här artikeln innehåller exempel som använder [Azure Functions Core tools](../functions-run-local.md) för både .net (C#) och Java Script.

## <a name="start-instances"></a>Start instanser

Det är viktigt att kunna starta en instans av Orchestration. Detta görs vanligt vis när du använder en Durable Functions-bindning i en annan funktions utlösare.

Metoden `StartNewAsync` (.net) eller `startNew` (Java Script) i [Dirigerings klientens bindning](durable-functions-bindings.md#orchestration-client) startar en ny instans. Internt kommer den här metoden att köa ett meddelande i kontroll kön, som sedan utlöser starten av en funktion med det angivna namnet som använder [bindningen för Orchestration-utlösaren](durable-functions-bindings.md#orchestration-trigger).

Den här asynkrona åtgärden slutförs när Orchestration-processen har schemalagts.

Parametrarna för att starta en ny Dirigerings instans är följande:

* **Namn**: namnet på Orchestrator-funktionen som ska schemaläggas.
* **Indata**: alla JSON-serialiserbara data som ska skickas som indata till Orchestrator-funktionen.
* **InstanceID**: (valfritt) det unika ID: t för instansen. Om du inte anger den här parametern använder metoden ett slumpmässigt ID.

> [!TIP]
> Använd en slumpmässig identifierare för instans-ID: t. Slumpmässiga instans-ID: n hjälper till att säkerställa en likvärdig belastnings distribution när du skalar Orchestrator-funktioner över flera virtuella datorer. Den tid det tar att använda icke-slumpmässiga instans-ID: n är när ID: t måste komma från en extern källa eller när du implementerar [singleton Orchestrator](durable-functions-singletons.md) -mönstret.

Följande kod är en exempel funktion som startar en ny Dirigerings instans:

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("HelloWorldQueueTrigger")]
public static async Task Run(
    [QueueTrigger("start-queue")] string input,
    [DurableClient] IDurableOrchestrationClient starter,
    ILogger log)
{
    string instanceId = await starter.StartNewAsync("HelloWorld", input);
    log.LogInformation($"Started orchestration with ID = '{instanceId}'.");
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

<a name="javascript-function-json"></a>Om inget annat anges använder exemplen på den här sidan HTTP-utlösaren med följande funktion. JSON.

**function. JSON**

```json
{
  "bindings": [
    {
      "name": "req",
      "type": "httpTrigger",
      "direction": "in",
      "methods": ["post"]
    },
    {
      "name": "$return",
      "type": "http",
      "direction": "out"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

> [!NOTE]
> Det här exemplet riktar sig Durable Functions version 2. x. I version 1. x använder `orchestrationClient` du i stället `durableClient`för.

**index. js**

```javascript
const df = require("durable-functions");

module.exports = async function(context, input) {
    const client = df.getClient(context);

    const instanceId = await client.startNew("HelloWorld", undefined, input);
    context.log(`Started orchestration with ID = ${instanceId}.`);
};
```

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Du kan också starta en instans direkt genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable start-new` . Det tar följande parametrar:

* (obligatoriskt): namnet på den funktion som ska startas. ** `function-name` **
* (valfritt): indata till funktionen, antingen i rad eller via en JSON-fil. ** `input` ** För filer lägger du till ett prefix till sökvägen till filen med `@`, till exempel. `@path/to/file.json`
* (valfritt): ID för Orchestration-instansen. ** `id` ** Om du inte anger den här parametern använder kommandot ett slumpmässigt GUID.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är AzureWebJobsStorage.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är DurableFunctionsHub. Du kan också ange detta i [Host. JSON](durable-functions-bindings.md#host-json) med hjälp av DurableTask: HubName.

> [!NOTE]
> Kommandona för Core tools förutsätter att du kör dem från rot katalogen i en Function-app. Om du uttryckligen anger parametrarna `connection-string-setting` och `task-hub-name` kan du köra kommandona från vilken katalog som helst. Även om du kan köra dessa kommandon utan att köra en Function-värd som kör, kan du upptäcka att du inte kan observera vissa effekter om inte värden körs. `start-new` Kommandot kommer till exempel att köa ett Start meddelande i hubben för mål, men dirigeringen körs inte om det inte finns någon funktion för att köra en Function-värd process som kan bearbeta meddelandet.

Följande kommando startar funktionen HelloWorld och skickar innehållet i filen `counter-data.json` till den:

```bash
func durable start-new --function-name HelloWorld --input @counter-data.json --task-hub-name TestTaskHub
```

## <a name="query-instances"></a>Fråga efter instanser

Som en del av din ansträngning för att hantera dina dirigeringar behöver du förmodligen samla in information om statusen för en Dirigerings instans (till exempel om den har slutförts eller inte).

Metoden `GetStatusAsync` (.net) eller `getStatus` (Java Script) i [Dirigerings klientens bindning](durable-functions-bindings.md#orchestration-client) frågar efter status för en Dirigerings instans.

Det tar `instanceId` (krävs), `showHistory` (valfritt), `showHistoryOutput` (valfritt) och `showInput` (valfritt) som parametrar.

* **`showHistory`**: Om värdet `true`är, innehåller svaret körnings historiken.
* **`showHistoryOutput`**: Om det är `true`inställt på, innehåller körnings historiken aktivitets utdata.
* **`showInput`**: Om det är `false`inställt på, innehåller svaret inte indatamängden för funktionen. Standardvärdet är `true`.

Metoden returnerar ett objekt med följande egenskaper:

* **Namn**: namnet på Orchestrator-funktionen.
* **InstanceID**: instans-ID: t för dirigeringen (ska vara detsamma som `instanceId` indatamängden).
* **CreatedTime**: tiden då Orchestrator-funktionen började köras.
* **LastUpdatedTime**: tiden då dirigeringen senast pågick.
* **Inmatade**: indatamängden för funktionen som ett JSON-värde. Det här fältet är inte ifyllt om `showInput` är falskt.
* **CustomStatus**: anpassad Dirigerings status i JSON-format.
* **Output**: resultatet av funktionen som ett JSON-värde (om funktionen har slutförts). Om Orchestrator-funktionen misslyckades innehåller den här egenskapen fel information. Om Orchestrator-funktionen avbröts innehåller den här egenskapen orsaken till uppsägningen (om det finns några).
* **RuntimeStatus**: ett av följande värden:
  * **Väntar**: instansen har schemalagts men har ännu inte börjat köras.
  * **Körs**: instansen har börjat köras.
  * **Slutfört**: instansen har slutförts normalt.
  * **ContinuedAsNew**: instansen har startats om med en ny historik. Det här läget är ett tillfälligt tillstånd.
  * **Misslyckades**: instansen misslyckades med ett fel.
  * **Avslutad**: instansen avbröts plötsligt.
* **Historik**: körnings historiken för dirigeringen. Det här fältet fylls bara i `showHistory` om är inställt på `true`.

Den här metoden `null` returnerar (.net) `undefined` eller (Java Script) om instansen inte finns.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("GetStatus")]
public static async Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("check-status-queue")] string instanceId)
{
    DurableOrchestrationStatus status = await client.GetStatusAsync(instanceId);
    // do something based on the current status.
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const status = await client.getStatus(instanceId);
    // do something based on the current status.
}
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Det är också möjligt att hämta statusen för en Dirigerings instans direkt genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable get-runtime-status` . Det tar följande parametrar:

* (obligatoriskt): ID för Orchestration-instansen. ** `id` **
* (valfritt): om det är `true`inställt på, innehåller svaret indatatypen för funktionen. ** `show-input` ** Standardvärdet är `false`.
* (valfritt): om det är `true`inställt på, innehåller svaret resultatet av funktionen. ** `show-output` ** Standardvärdet är `false`.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i [Host. JSON](durable-functions-bindings.md#host-json), genom att använda DurableTask: HubName.

Följande kommando hämtar status (inklusive indata och utdata) för en instans med ett Dirigerings instans-ID för 0ab8c55a66644d68a3a8b220b12d209c. Det förutsätter att du kör `func` kommandot från rot katalogen i Function-appen:

```bash
func durable get-runtime-status --id 0ab8c55a66644d68a3a8b220b12d209c --show-input true --show-output true
```

Du kan använda `durable get-history` kommandot för att hämta historiken för en Dirigerings instans. Det tar följande parametrar:

* (obligatoriskt): ID för Orchestration-instansen. ** `id` **
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i Host. JSON, genom att använda durableTask: HubName.

```bash
func durable get-history --id 0ab8c55a66644d68a3a8b220b12d209c
```

## <a name="query-all-instances"></a>Fråga alla instanser

I stället för att skicka frågor till en instans i din organisation i taget, kan det vara mer effektivt att fråga alla dem samtidigt.

Du kan använda metoden `GetStatusAsync` (.net) eller `getStatusAll` (Java Script) för att fråga efter status för alla Dirigerings instanser. I .NET kan du skicka ett `CancellationToken` objekt om du vill avbryta det. Metoden returnerar objekt med samma egenskaper som `GetStatusAsync` metoden med parametrar.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("GetAllStatus")]
public static async Task Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")]HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    IList<DurableOrchestrationStatus> instances = await client.GetStatusAsync(); // You can pass CancellationToken as a parameter.
    foreach (var instance in instances)
    {
        log.LogInformation(JsonConvert.SerializeObject(instance));
    };
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, req) {
    const client = df.getClient(context);

    const instances = await client.getStatusAll();
    instances.forEach((instance) => {
        context.log(JSON.stringify(instance));
    });
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Det är också möjligt att fråga instanser direkt, genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable get-instances` . Det tar följande parametrar:

* (valfritt): det här kommandot stöder sid indelning. ** `top` ** Den här parametern motsvarar antalet instanser som hämtas per begäran. Standardvärdet är 10.
* (valfritt): en token för att ange vilken sida eller vilket avsnitt av instanser som ska hämtas. ** `continuation-token` ** Varje `get-instances` körning returnerar en token till nästa uppsättning instanser.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i [Host. JSON](durable-functions-bindings.md#host-json), genom att använda DurableTask: HubName.

```bash
func durable get-instances
```

## <a name="query-instances-with-filters"></a>Fråga efter instanser med filter

Vad händer om du verkligen behöver ha all information som en standard instans fråga kan ge? Vad händer till exempel om du bara söker efter skapande tid för Orchestration eller om Dirigerings körnings status? Du kan begränsa frågan genom att använda filter.

Använd metoden `GetStatusAsync` (.net) eller `getStatusBy` (Java Script) för att hämta en lista över Dirigerings instanser som matchar en uppsättning fördefinierade filter.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("QueryStatus")]
public static async Task Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")]HttpRequestMessage req,
    [DurableClient] IDurableOrchestrationClient client,
    ILogger log)
{
    var runtimeStatus = new List<OrchestrationRuntimeStatus> {
        OrchestrationRuntimeStatus.Completed,
        OrchestrationRuntimeStatus.Running
    };
    IList<DurableOrchestrationStatus> instances = await starter.GetStatusAsync(
        new DateTime(2018, 3, 10, 10, 1, 0),
        new DateTime(2018, 3, 10, 10, 23, 59),
        runtimeStatus
    ); // You can pass CancellationToken as a parameter.
    foreach (var instance in instances)
    {
        log.LogInformation(JsonConvert.SerializeObject(instance));
    };
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, req) {
    const client = df.getClient(context);

    const runtimeStatus = [
        df.OrchestrationRuntimeStatus.Completed,
        df.OrchestrationRuntimeStatus.Running,
    ];
    const instances = await client.getStatusBy(
        new Date(2018, 3, 10, 10, 1, 0),
        new Date(2018, 3, 10, 10, 23, 59),
        runtimeStatus
    );
    instances.forEach((instance) => {
        context.log(JSON.stringify(instance));
    });
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

I Azure Functions Core Tools kan du också använda `durable get-instances` kommandot med filter. Förutom `top`de ovannämnda `continuation-token` `connection-string-setting`parametrarna,, och `task-hub-name` kan du använda tre filter parametrar (`created-after`, `created-before`, och `runtime-status`).

* (valfritt): Hämta instanserna som skapats efter detta datum/tid (UTC). ** `created-after` ** ISO 8601 formaterade datetimes har godkänts.
* (valfritt): Hämta instanserna som skapades före detta datum/tid (UTC). ** `created-before` ** ISO 8601 formaterade datetimes har godkänts.
* (valfritt): Hämta instanserna med en viss status (till exempel körs eller slutfört). ** `runtime-status` ** Kan ge flera (blankstegsavgränsad) status.
* (valfritt): antal instanser som hämtats per begäran. ** `top` ** Standardvärdet är 10.
* (valfritt): en token för att ange vilken sida eller vilket avsnitt av instanser som ska hämtas. ** `continuation-token` ** Varje `get-instances` körning returnerar en token till nästa uppsättning instanser.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i [Host. JSON](durable-functions-bindings.md#host-json), genom att använda DurableTask: HubName.

Om du inte anger några filter (`created-after`, `created-before`eller `runtime-status`) hämtar kommandot bara `top` instanser, utan hänsyn till körnings status eller skapande tid.

```bash
func durable get-instances --created-after 2018-03-10T13:57:31Z --created-before  2018-03-10T23:59Z --top 15
```

## <a name="terminate-instances"></a>Avsluta instanser

Om du har en Dirigerings instans som tar för lång tid att köra, eller om du bara behöver stoppa den innan den har slutförts av någon anledning, kan du välja att avsluta den.

Du kan använda `TerminateAsync` (.net) eller `terminate` (JavaScript)-metoden i [Dirigerings klient bindningen](durable-functions-bindings.md#orchestration-client) för att avsluta instanser. De två parametrarna är en `instanceId` och en `reason` sträng som skrivs till loggar och instans status.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("TerminateInstance")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("terminate-queue")] string instanceId)
{
    string reason = "It was time to be done.";
    return client.TerminateAsync(instanceId, reason);
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const reason = "It was time to be done.";
    return client.terminate(instanceId, reason);
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

En avslutad instans övergår eventuellt till `Terminated` statusen. Den här över gången sker dock inte omedelbart. I stället placeras den avbrytande åtgärden i aktivitets navet tillsammans med andra åtgärder för den instansen. Du kan använda API: erna för [instansen för instansen](#query-instances) för att veta `Terminated` när en avslutad instans faktiskt har nått status.

> [!NOTE]
> Instans avslutningen sprids inte för närvarande. Aktivitets funktioner och under dirigering körs till slut för ande, oavsett om du har avslutat Orchestration-instansen som anropade dem.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Du kan också avsluta en Dirigerings instans direkt genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable terminate` . Det tar följande parametrar:

* (obligatoriskt): ID för Orchestration-instansen som ska avbrytas. ** `id` **
* (valfritt): orsak till avslutning. ** `reason` **
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i [Host. JSON](durable-functions-bindings.md#host-json), genom att använda DurableTask: HubName.

Följande kommando avslutar en Dirigerings instans med ID: t 0ab8c55a66644d68a3a8b220b12d209c:

```bash
func durable terminate --id 0ab8c55a66644d68a3a8b220b12d209c --reason "It was time to be done."
```

## <a name="send-events-to-instances"></a>Skicka händelser till instanser

I vissa fall är det viktigt att dina Orchestrator-funktioner kan vänta och lyssna efter externa händelser. Detta omfattar [övervaknings funktioner](durable-functions-overview.md#monitoring) och funktioner som väntar på [mänsklig interaktion](durable-functions-overview.md#human).

Skicka händelse meddelanden till instanser som körs med hjälp `RaiseEventAsync` av metoden (.net) eller `raiseEvent` (Java Script) i [Dirigerings klient bindningen](durable-functions-bindings.md#orchestration-client). Instanser som kan hantera dessa händelser är de som väntar på ett anrop till `WaitForExternalEvent` (.net) eller ger ett `waitForExternalEvent` (JavaScript)-anrop.

Parametrarna till `RaiseEventAsync` (.net) och `raiseEvent` (Java Script) är följande:

* **InstanceID**: instansens unika ID.
* **EventName**: namnet på händelsen som ska skickas.
* **EventData**: en JSON-serialiserbar nytto last som ska skickas till instansen.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("RaiseEvent")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("event-queue")] string instanceId)
{
    int[] eventData = new int[] { 1, 2, 3 };
    return client.RaiseEventAsync(instanceId, "MyEvent", eventData);
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const eventData = [ 1, 2, 3 ];
    return client.raiseEvent(instanceId, "MyEvent", eventData);
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

> [!NOTE]
> Om det inte finns någon Dirigerings instans med angivet instans-ID, ignoreras händelse meddelandet. Om en instans finns men inte är i vänte läge för händelsen, lagras händelsen i instansen tills den är redo att tas emot och bearbetas.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Du kan också utlösa en händelse till en Orchestration-instans direkt genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable raise-event` . Det tar följande parametrar:

* (obligatoriskt): ID för Orchestration-instansen. ** `id` **
* **`event-name`**: Namnet på händelsen som ska upphöjas.
* (valfritt): data som ska skickas till Orchestration-instansen. ** `event-data` ** Detta kan vara sökvägen till en JSON-fil eller så kan du ange data direkt på kommando raden.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Standardvärdet är `DurableFunctionsHub`. Den kan också anges i [Host. JSON](durable-functions-bindings.md#host-json), genom att använda DurableTask: HubName.

```bash
func durable raise-event --id 0ab8c55a66644d68a3a8b220b12d209c --event-name MyEvent --event-data @eventdata.json
```

```bash
func durable raise-event --id 1234567 --event-name MyOtherEvent --event-data 3
```

## <a name="wait-for-orchestration-completion"></a>Vänta på slut för ande av dirigering

I långvariga dirigeringar kanske du vill vänta och få resultatet av ett Dirigerings resultat. I dessa fall är det också användbart att kunna definiera en tids gräns för dirigering. Om tids gränsen överskrids ska dirigeringens tillstånd returneras i stället för resultatet.

Metoden `WaitForCompletionOrCreateCheckStatusResponseAsync` (.net) eller `waitForCompletionOrCreateCheckStatusResponse` (Java Script) kan användas för att hämta faktiska utdata från en Dirigerings instans synkront. Som standard använder dessa metoder ett standardvärde på 10 sekunder för `timeout`och 1 sekund för. `retryInterval`  

Här är ett exempel på en HTTP-utlösnings funktion som visar hur du använder det här API: et:

# <a name="c"></a>[C #](#tab/csharp)

[!code-csharp[Main](~/samples-durable-functions/samples/precompiled/HttpSyncStart.cs)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

[!code-javascript[Main](~/samples-durable-functions/samples/javascript/HttpSyncStart/index.js)]

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

Anropa funktionen med följande rad. Använd 2 sekunder för timeout och 0,5 sekunder för återförsöksintervall:

```bash
    http POST http://localhost:7071/orchestrators/E1_HelloSequence/wait?timeout=2&retryInterval=0.5
```

Det finns två fall beroende på hur lång tid det tar att hämta svaret från Orchestration-instansen:

* Dirigerings instanserna slutförs inom den angivna tids gränsen (i det här fallet 2 sekunder) och svaret är den faktiska Dirigerings instansens utdata som levereras synkront:

    ```http
        HTTP/1.1 200 OK
        Content-Type: application/json; charset=utf-8
        Date: Thu, 14 Dec 2018 06:14:29 GMT
        Transfer-Encoding: chunked

        [
            "Hello Tokyo!",
            "Hello Seattle!",
            "Hello London!"
        ]
    ```

* Dirigerings instanserna kan inte slutföras inom den angivna tids gränsen och svaret är standard som beskrivs i [http API URL-identifiering](durable-functions-http-api.md):

    ```http
        HTTP/1.1 202 Accepted
        Content-Type: application/json; charset=utf-8
        Date: Thu, 14 Dec 2018 06:13:51 GMT
        Location: http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}
        Retry-After: 10
        Transfer-Encoding: chunked

        {
            "id": "d3b72dddefce4e758d92f4d411567177",
            "sendEventPostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177/raiseEvent/{eventName}?taskHub={taskHub}&connection={connection}&code={systemKey}",
            "statusQueryGetUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177?taskHub={taskHub}&connection={connection}&code={systemKey}",
            "terminatePostUri": "http://localhost:7071/runtime/webhooks/durabletask/instances/d3b72dddefce4e758d92f4d411567177/terminate?reason={text}&taskHub={taskHub}&connection={connection}&code={systemKey}"
        }
    ```

> [!NOTE]
> Formatet för webhook-URL: erna kan variera beroende på vilken version av Azure Functions-värden som du kör. Föregående exempel är för Azure Functions 2,0-värden.

## <a name="retrieve-http-management-webhook-urls"></a>Hämta URL: er för HTTP Management webhook

Du kan använda ett externt system för att övervaka eller utlösa händelser till en dirigering. Externa system kan kommunicera med Durable Functions via webhook-URL: er som ingår i standardsvaret som beskrivs i [http API URL-identifiering](durable-functions-http-features.md#http-api-url-discovery). Webhook-URL: er kan också nås via programmering med hjälp av [Dirigerings klientens bindning](durable-functions-bindings.md#orchestration-client). Metoderna `CreateHttpManagementPayload` (.net) eller `createHttpManagementPayload` (Java Script) kan användas för att hämta ett serialiserbar objekt som innehåller dessa webhook-URL: er.

Metoderna `CreateHttpManagementPayload` (.net) och `createHttpManagementPayload` (Java Script) har en parameter:

* **InstanceID**: instansens unika ID.

Metoderna returnerar ett objekt med följande sträng egenskaper:

* **ID**: instans-ID för dirigeringen (ska vara detsamma som `InstanceId` indatatypen).
* **StatusQueryGetUri**: status-URL: en för Orchestration-instansen.
* **SendEventPostUri**: URL: en "öka händelse" för Orchestration-instansen.
* **TerminatePostUri**: "Terminate"-URL: en för Orchestration-instansen.
* **PurgeHistoryDeleteUri**: URL: en för "Rensa historik" för Orchestration-instansen.

Funktioner kan skicka instanser av dessa objekt till externa system för att övervaka eller utlösa händelser i motsvarande Orchestration, som du ser i följande exempel:

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("SendInstanceInfo")]
public static void SendInstanceInfo(
    [ActivityTrigger] IDurableActivityContext ctx,
    [DurableClient] IDurableOrchestrationClient client,
    [DocumentDB(
        databaseName: "MonitorDB",
        collectionName: "HttpManagementPayloads",
        ConnectionStringSetting = "CosmosDBConnection")]out dynamic document)
{
    HttpManagementPayload payload = client.CreateHttpManagementPayload(ctx.InstanceId);

    // send the payload to Cosmos DB
    document = new { Payload = payload, id = ctx.InstanceId };
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du använda `DurableActivityContext` `IDurableActivityContext` `OrchestrationClient` attributet i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

modules.exports = async function(context, ctx) {
    const client = df.getClient(context);

    const payload = client.createHttpManagementPayload(ctx.instanceId);

    // send the payload to Cosmos DB
    context.bindings.document = JSON.stringify({
        id: ctx.instanceId,
        payload,
    });
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

## <a name="rewind-instances-preview"></a>Spola tillbaka instanser (förhands granskning)

Om du har ett Dirigerings fel av oväntad anledning kan du *spola tillbaka* instansen till ett tidigare fungerande tillstånd med hjälp av ett API som skapats för detta ändamål.

> [!NOTE]
> Detta API är inte avsett att vara en ersättning för korrekt fel hantering och principer för återförsök. I stället är det endast avsett att användas i fall där Orchestration-instanser inte kan utföra oväntade orsaker. Mer information om fel hantering och principer för återförsök finns i artikeln [fel hantering](durable-functions-error-handling.md) .

Använd `RewindAsync` (.net) eller `rewind` (Java Script) metoden för [Dirigerings klient bindningen](durable-functions-bindings.md#orchestration-client) för att dirigera om dirigeringen till *körnings* tillstånd. Den här metoden kör även de körnings försök för aktiviteter eller under dirigering som orsakade ett Orchestration-haveri.

Anta till exempel att du har ett arbets flöde som innehåller en serie med [mänsklig godkännande](durable-functions-overview.md#human). Anta att det finns en serie aktivitets funktioner som meddelar någon att deras godkännande krävs och väntar på svar i real tid. När alla godkännande aktiviteter har fått svar eller nått tids gränsen, antar vi att en annan aktivitet Miss lyckas på grund av en fel konfiguration i programmet, t. ex. en ogiltig databas anslutnings sträng. Resultatet är ett djupgående problem i arbets flödet. Med API `RewindAsync` : et (.net `rewind` ) eller (Java Script) kan en program administratör åtgärda konfigurations felet och spola tillbaka den misslyckade dirigeringen till statusen omedelbart före felet. Ingen av de mänskliga interaktions stegen måste godkännas igen och dirigeringen kan nu slutföras.

> [!NOTE]
> Funktionen *spola tillbaka* stöder inte omspolning av Orchestration-instanser som använder varaktiga timers.

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("RewindInstance")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("rewind-queue")] string instanceId)
{
    string reason = "Orchestrator failed and needs to be revived.";
    return client.RewindAsync(instanceId, reason);
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);

    const reason = "Orchestrator failed and needs to be revived.";
    return client.rewind(instanceId, reason);
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Du kan också spola en Orchestration-instans direkt genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable rewind` . Det tar följande parametrar:

* (obligatoriskt): ID för Orchestration-instansen. ** `id` **
* (valfritt): skäl för att spola tillbaka Orchestration-instansen. ** `reason` **
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Som standard används aktivitets nav namnet i [Host. JSON](durable-functions-bindings.md#host-json) -filen.

```bash
func durable rewind --id 0ab8c55a66644d68a3a8b220b12d209c --reason "Orchestrator failed and needs to be revived."
```

## <a name="purge-instance-history"></a>Rensa instans historik

Om du vill ta bort alla data som är associerade med ett dirigering kan du rensa instans historiken. Du kanske till exempel vill ta bort alla Azure Table-rader och stora meddelande-blobar som är associerade med en slutförd instans. Det gör du genom att använda `PurgeInstanceHistoryAsync` metoden (.net) `purgeInstanceHistory` eller (Java Script) i [Dirigerings klient bindningen](durable-functions-bindings.md#orchestration-client).

Den här metoden har två överlagringar. Den första överlagringen rensar historiken med ID: t för Orchestration-instansen:

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("PurgeInstanceHistory")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [QueueTrigger("purge-queue")] string instanceId)
{
    return client.PurgeInstanceHistoryAsync(instanceId);
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
const df = require("durable-functions");

module.exports = async function(context, instanceId) {
    const client = df.getClient(context);
    return client.purgeInstanceHistory(instanceId);
};
```

Se [Start instanser](#javascript-function-json) för function. JSON-konfigurationen.

---

I nästa exempel visas en timer-utlöst funktion som rensar historiken för alla Dirigerings instanser som har slutförts efter angivet tidsintervall. I det här fallet tar den bort data för alla instanser som slutförts 30 eller fler dagar sedan. Det är schemalagt att köras en gång per dag, kl. 12:

# <a name="c"></a>[C #](#tab/csharp)

```csharp
[FunctionName("PurgeInstanceHistory")]
public static Task Run(
    [DurableClient] IDurableOrchestrationClient client,
    [TimerTrigger("0 0 12 * * *")]TimerInfo myTimer)
{
    return client.PurgeInstanceHistoryAsync(
        DateTime.MinValue,
        DateTime.UtcNow.AddDays(-30),  
        new List<OrchestrationStatus>
        {
            OrchestrationStatus.Completed
        });
}
```

> [!NOTE]
> Föregående C#-kod är för Durable Functions 2. x. För Durable Functions 1. x måste du `OrchestrationClient` använda attribut i stället för `DurableClient` attributet och du måste använda `DurableOrchestrationClient` parameter typen i stället för. `IDurableOrchestrationClient` Mer information om skillnaderna mellan versioner finns i artikeln [Durable Functions versioner](durable-functions-versions.md) .

# <a name="javascript"></a>[JavaScript](#tab/javascript)

`purgeInstanceHistoryBy` Metoden kan användas för att villkorligt rensa instans historik för flera instanser.

**function. JSON**

```json
{
  "bindings": [
    {
      "schedule": "0 0 12 * * *",
      "name": "myTimer",
      "type": "timerTrigger",
      "direction": "in"
    },
    {
      "name": "starter",
      "type": "durableClient",
      "direction": "in"
    }
  ],
  "disabled": false
}
```

> [!NOTE]
> Det här exemplet riktar sig Durable Functions version 2. x. I version 1. x använder `orchestrationClient` du i stället `durableClient`för.

**index. js**

```javascript
const df = require("durable-functions");

module.exports = async function (context, myTimer) {
    const client = df.getClient(context);
    const createdTimeFrom = new Date(0);
    const createdTimeTo = new Date().setDate(today.getDate() - 30);
    const runtimeStatuses = [ df.OrchestrationRuntimeStatus.Completed ];
    return client.purgeInstanceHistoryBy(createdTimeFrom, createdTimeTo, runtimeStatuses);
};
```

---

> [!NOTE]
> För att rensnings historiken ska fungera måste körnings status för mål instansen vara **slutförd**, **avslutad**eller **misslyckad**.

### <a name="azure-functions-core-tools"></a>Azure Functions Core Tools

Du kan rensa en Dirigerings instanss historik genom att använda kommandot [Azure Functions Core tools](../functions-run-local.md) `durable purge-history` . I likhet med det andra C#-exemplet i föregående avsnitt rensas historiken för alla Dirigerings instanser som skapats under ett angivet tidsintervall. Du kan filtrera rensade instanser ytterligare efter körnings status. Kommandot har flera parametrar:

* (valfritt): Rensa historiken för instanser som skapats efter detta datum/tid (UTC). ** `created-after` ** ISO 8601 formaterade datetimes har godkänts.
* (valfritt): Rensa historiken för instanser som skapats före detta datum/tid (UTC). ** `created-before` ** ISO 8601 formaterade datetimes har godkänts.
* (valfritt): Rensa historiken för instanser med en viss status (till exempel körs eller slutfört). ** `runtime-status` ** Kan ge flera (blankstegsavgränsad) status.
* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Som standard används aktivitets nav namnet i [Host. JSON](durable-functions-bindings.md#host-json) -filen.

Följande kommando tar bort historiken för alla misslyckade instanser som skapats före den 14 november 2018 vid 7:35 PM (UTC).

```bash
func durable purge-history --created-before 2018-11-14T19:35:00.0000000Z --runtime-status failed
```

## <a name="delete-a-task-hub"></a>Ta bort en aktivitets hubb

Med hjälp av kommandot [Azure Functions Core tools](../functions-run-local.md) `durable delete-task-hub` kan du ta bort alla lagrings artefakter som är associerade med en viss aktivitets hubb, inklusive Azure Storage-tabeller, köer och blobar. Kommandot har två parametrar:

* (valfritt): namnet på den program inställning som innehåller den lagrings anslutnings sträng som ska användas. ** `connection-string-setting` ** Standardvärdet är `AzureWebJobsStorage`.
* (valfritt): namnet på den Durable Functions aktivitets hubb som ska användas. ** `task-hub-name` ** Som standard används aktivitets nav namnet i [Host. JSON](durable-functions-bindings.md#host-json) -filen.

Följande kommando tar bort alla Azure Storage-data som är `UserTest` associerade med aktivitets navet.

```bash
func durable delete-task-hub --task-hub-name UserTest
```

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Lär dig hur du hanterar versions hantering](durable-functions-versioning.md)

> [!div class="nextstepaction"]
> [Inbyggd HTTP API-referens för instans hantering](durable-functions-http-api.md)
