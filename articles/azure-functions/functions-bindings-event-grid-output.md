---
title: Azure Event Grid utgående bindning för Azure Functions
description: Lär dig att skicka en Event Grid-händelse i Azure Functions.
author: craigshoemaker
ms.topic: reference
ms.date: 02/14/2020
ms.author: cshoe
ms.custom: fasttrack-edit
ms.openlocfilehash: 0237bcbf98578d9f83f3c9652661c786df54e73a
ms.sourcegitcommit: 1895459d1c8a592f03326fcb037007b86e2fd22f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/01/2020
ms.locfileid: "82627695"
---
# <a name="azure-event-grid-output-binding-for-azure-functions"></a>Azure Event Grid utgående bindning för Azure Functions

Använd Event Grid utgående bindning för att skriva händelser till ett anpassat ämne. Du måste ha en giltig [åtkomst nyckel för det anpassade ämnet](../event-grid/security-authentication.md#authenticate-publishing-clients-using-sas-or-key).

Information om konfiguration och konfigurations information finns i [översikten](./functions-bindings-event-grid.md).

> [!NOTE]
> Event Grid utgående bindning har inte stöd för signaturer för delad åtkomst (SAS-token). Du måste använda ämnets åtkomst nyckel.

> [!IMPORTANT]
> Den Event Grid utgående bindningen är endast tillgänglig för funktionerna 2. x och högre.

## <a name="example"></a>Exempel

# <a name="c"></a>[C #](#tab/csharp)

I följande exempel visas en [C#-funktion](functions-dotnet-class-library.md) som skriver ett meddelande till ett event Grid anpassat ämne med hjälp av metodens retur värde som utdata:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static EventGridEvent Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    return new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

I följande exempel visas hur du använder `IAsyncCollector` gränssnittet för att skicka en batch med meddelanden.

```csharp
[FunctionName("EventGridAsyncOutput")]
public static async Task Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo myTimer,
    [EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]IAsyncCollector<EventGridEvent> outputEvents,
    ILogger log)
{
    for (var i = 0; i < 3; i++)
    {
        var myEvent = new EventGridEvent("message-id-" + i, "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
        await outputEvents.AddAsync(myEvent);
    }
}
```

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

I följande exempel visas Event Grid utgående bindnings data i *Function. JSON* -filen.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Här är C#-skript kod som skapar en händelse:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, out EventGridEvent outputEvent, ILogger log)
{
    outputEvent = new EventGridEvent("message-id", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0");
}
```

Här är C#-skript kod som skapar flera händelser:

```cs
#r "Microsoft.Azure.EventGrid"
using System;
using Microsoft.Azure.EventGrid.Models;
using Microsoft.Extensions.Logging;

public static void Run(TimerInfo myTimer, ICollector<EventGridEvent> outputEvent, ILogger log)
{
    outputEvent.Add(new EventGridEvent("message-id-1", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
    outputEvent.Add(new EventGridEvent("message-id-2", "subject-name", "event-data", "event-type", DateTime.UtcNow, "1.0"));
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

I följande exempel visas Event Grid utgående bindnings data i *Function. JSON* -filen.

```json
{
    "type": "eventGrid",
    "name": "outputEvent",
    "topicEndpointUri": "MyEventGridTopicUriSetting",
    "topicKeySetting": "MyEventGridTopicKeySetting",
    "direction": "out"
}
```

Här är JavaScript-kod som skapar en enskild händelse:

```javascript
module.exports = async function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = {
        id: 'message-id',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    };
    context.done();
};
```

Här är JavaScript-kod som skapar flera händelser:

```javascript
module.exports = function(context) {
    var timeStamp = new Date().toISOString();

    context.bindings.outputEvent = [];

    context.bindings.outputEvent.push({
        id: 'message-id-1',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.bindings.outputEvent.push({
        id: 'message-id-2',
        subject: 'subject-name',
        dataVersion: '1.0',
        eventType: 'event-type',
        data: "event-data",
        eventTime: timeStamp
    });
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Den Event Grid utgående bindningen är inte tillgänglig för python.

# <a name="java"></a>[Java](#tab/java)

Den Event Grid utgående bindningen är inte tillgänglig för Java.

---

## <a name="attributes-and-annotations"></a>Attribut och anteckningar

# <a name="c"></a>[C #](#tab/csharp)

För [klass bibliotek i C#](functions-dotnet-class-library.md)använder du attributet [EventGridAttribute](https://github.com/Azure/azure-functions-eventgrid-extension/blob/dev/src/EventGridExtension/OutputBinding/EventGridAttribute.cs) .

Attributets konstruktor tar namnet på en app-inställning som innehåller namnet på det anpassade ämnet och namnet på en app-inställning som innehåller ämnes nyckeln. Mer information om de här inställningarna finns i [output-Configuration](#configuration). Här är ett `EventGrid` exempel på attribut:

```csharp
[FunctionName("EventGridOutput")]
[return: EventGrid(TopicEndpointUri = "MyEventGridTopicUriSetting", TopicKeySetting = "MyEventGridTopicKeySetting")]
public static string Run([TimerTrigger("0 */5 * * * *")] TimerInfo myTimer, ILogger log)
{
    ...
}
```

Ett fullständigt exempel finns i [exemplet](#example).

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

Attribut stöds inte av C#-skript.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Attribut stöds inte av Java Script.

# <a name="python"></a>[Python](#tab/python)

Den Event Grid utgående bindningen är inte tillgänglig för python.

# <a name="java"></a>[Java](#tab/java)

Den Event Grid utgående bindningen är inte tillgänglig för Java.

---

## <a name="configuration"></a>Konfiguration

I följande tabell förklaras de egenskaper för bindnings konfiguration som du anger i filen *Function. JSON* och `EventGrid` attributet.

|function. JSON-egenskap | Attributets egenskap |Beskrivning|
|---------|---------|----------------------|
|**bastyp** | saknas | Måste vara inställd på "eventGrid". |
|**position** | saknas | Måste anges till "out". Den här parametern anges automatiskt när du skapar bindningen i Azure Portal. |
|**Namn** | saknas | Variabel namnet som används i funktions koden som representerar händelsen. |
|**topicEndpointUri** |**TopicEndpointUri** | Namnet på en app-inställning som innehåller URI: n för det anpassade ämnet, till `MyTopicEndpointUri`exempel. |
|**topicKeySetting** |**TopicKeySetting** | Namnet på en app-inställning som innehåller en åtkomst nyckel för det anpassade ämnet. |

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

> [!IMPORTANT]
> Se till att du anger värdet för `TopicEndpointUri` konfigurations egenskapen till namnet på en app-inställning som innehåller URI: n för det anpassade ämnet. Ange inte URI för det anpassade ämnet direkt i den här egenskapen.

## <a name="usage"></a>Användning

# <a name="c"></a>[C #](#tab/csharp)

Skicka meddelanden med hjälp av en metod parameter, `out EventGridEvent paramName`till exempel. Om du vill skriva flera meddelanden kan du `ICollector<EventGridEvent>` använda `IAsyncCollector<EventGridEvent>` eller i stället `out EventGridEvent`för.

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

Skicka meddelanden med hjälp av en metod parameter, `out EventGridEvent paramName`till exempel. I C#- `paramName` skript är värdet som anges i `name` egenskapen för *Function. JSON*. Om du vill skriva flera meddelanden kan du `ICollector<EventGridEvent>` använda `IAsyncCollector<EventGridEvent>` eller i stället `out EventGridEvent`för.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Få åtkomst till utdata-händelsen `context.bindings.<name>` med `<name>` hjälp av WHERE är värdet som `name` anges i egenskapen för *Function. JSON*.

# <a name="python"></a>[Python](#tab/python)

Den Event Grid utgående bindningen är inte tillgänglig för python.

# <a name="java"></a>[Java](#tab/java)

Den Event Grid utgående bindningen är inte tillgänglig för Java.

---

## <a name="next-steps"></a>Nästa steg

* [Skicka en Event Grid-händelse](./functions-bindings-event-grid-trigger.md)
