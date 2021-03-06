---
title: Azure Blob Storage-utlösare för Azure Functions
description: Lär dig hur du kör en Azure-funktion när Azure Blob Storage-data ändras.
author: craigshoemaker
ms.topic: reference
ms.date: 02/13/2020
ms.author: cshoe
ms.openlocfilehash: 61fbaf37577efdab0b147d437ae78fc4df0764cb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82084965"
---
# <a name="azure-blob-storage-trigger-for-azure-functions"></a>Azure Blob Storage-utlösare för Azure Functions

Blob Storage-utlösaren startar en funktion när en ny eller uppdaterad BLOB identifieras. BLOB-innehållet tillhandahålls som [indata till funktionen](./functions-bindings-storage-blob-input.md).

Azure Blob Storage-utlösaren kräver ett allmänt lagrings konto. Om du vill använda ett enbart BLOB-konto, eller om ditt program har särskilda behov, granskar du alternativen för att använda den här utlösaren.

Information om konfiguration och konfigurations information finns i [översikten](./functions-bindings-storage-blob.md).

## <a name="alternatives"></a>Alternativ

### <a name="event-grid-trigger"></a>Event Grid utlösare

[Event Grid-utlösaren](functions-bindings-event-grid.md) har också inbyggt stöd för [BLOB-händelser](../storage/blobs/storage-blob-event-overview.md). Använd Event Grid i stället för Blob Storage-utlösaren i följande scenarier:

- **Enbart BLOB-lagrings konton**: [BLOB-endast lagrings konton](../storage/common/storage-account-overview.md#types-of-storage-accounts) stöds för BLOB-indata och utgående bindningar, men inte för BLOB-utlösare.

- **Hög skala**: hög skala kan lösas fritt som behållare som har fler än 100 000 blobbar i dem eller lagrings konton som har fler än 100 BLOB-uppdateringar per sekund.

- **Minimera latens**: om din Function-app är i förbruknings planen kan det vara upp till 10 minuter att bearbeta nya blobbar om en Function-app har varit inaktiv. För att undvika den här fördröjningen kan du växla till en App Service plan med Always on Enabled. Du kan också använda en [Event Grid-utlösare](functions-bindings-event-grid.md) med ditt Blob Storage-konto. Ett exempel finns i [Event Grid själv studie kursen](../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2Fazure%2Fazure-functions%2Ftoc.json).

Se [bild storleken med event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md) själv studie kurs om ett event Grid exempel.

### <a name="queue-storage-trigger"></a>Kölagringsutlösare

En annan metod för att bearbeta blobbar är att skriva Kömeddelanden som motsvarar blobbar som skapas eller ändras och sedan använda en [kö Storage-utlösare](./functions-bindings-storage-queue.md) för att påbörja bearbetningen.

## <a name="example"></a>Exempel

# <a name="c"></a>[C #](#tab/csharp)

I följande exempel visas en [C#-funktion](functions-dotnet-class-library.md) som skriver en logg när en BLOB läggs till eller uppdateras i `samples-workitems` behållaren.

```csharp
[FunctionName("BlobTriggerCSharp")]        
public static void Run([BlobTrigger("samples-workitems/{name}")] Stream myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

Strängen `{name}` i BLOB trigger-sökvägen `samples-workitems/{name}` skapar ett [bindnings uttryck](./functions-bindings-expressions-patterns.md) som du kan använda i funktions koden för att komma åt fil namnet för den Utlös ande blobben. Mer information finns i [BLOB Name-mönster](#blob-name-patterns) längre fram i den här artikeln.

Mer information om `BlobTrigger` attributet finns i [attribut och anteckningar](#attributes-and-annotations).

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

I följande exempel visas en BLOB-utlösnings bindning i en *Function. JSON* -fil och kod som använder bindningen. Funktionen skriver en logg när en BLOB läggs till eller uppdateras i `samples-workitems` [behållaren](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Här är bindnings data i *Function. JSON* -filen:

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Strängen `{name}` i BLOB trigger-sökvägen `samples-workitems/{name}` skapar ett [bindnings uttryck](./functions-bindings-expressions-patterns.md) som du kan använda i funktions koden för att komma åt fil namnet för den Utlös ande blobben. Mer information finns i [BLOB Name-mönster](#blob-name-patterns) längre fram i den här artikeln.

Mer information om egenskaper för *Function. JSON* -filen finns i [konfigurations](#configuration) avsnittet förklarar dessa egenskaper.

Här är C#-skript kod som binder till en `Stream`:

```cs
public static void Run(Stream myBlob, string name, ILogger log)
{
   log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
}
```

Här är C#-skript kod som binder till en `CloudBlockBlob`:

```cs
#r "Microsoft.WindowsAzure.Storage"

using Microsoft.WindowsAzure.Storage.Blob;

public static void Run(CloudBlockBlob myBlob, string name, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name}\nURI:{myBlob.StorageUri}");
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

I följande exempel visas en BLOB-utlösnings bindning i en *Function. JSON* -fil och [JavaScript-kod](functions-reference-node.md) som använder bindningen. Funktionen skriver en logg när en BLOB läggs till eller uppdateras i `samples-workitems` behållaren.

Här är *Function. JSON* -filen:

```json
{
    "disabled": false,
    "bindings": [
        {
            "name": "myBlob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Strängen `{name}` i BLOB trigger-sökvägen `samples-workitems/{name}` skapar ett [bindnings uttryck](./functions-bindings-expressions-patterns.md) som du kan använda i funktions koden för att komma åt fil namnet för den Utlös ande blobben. Mer information finns i [BLOB Name-mönster](#blob-name-patterns) längre fram i den här artikeln.

Mer information om egenskaper för *Function. JSON* -filen finns i [konfigurations](#configuration) avsnittet förklarar dessa egenskaper.

Här är JavaScript-koden:

```javascript
module.exports = function(context) {
    context.log('Node.js Blob trigger function processed', context.bindings.myBlob);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

I följande exempel visas en BLOB-utlösnings bindning i en *Function. JSON* -fil och [python-kod](functions-reference-python.md) som använder bindningen. Funktionen skriver en logg när en BLOB läggs till eller uppdateras i `samples-workitems` [behållaren](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources).

Här är *Function. JSON* -filen:

```json
{
    "scriptFile": "__init__.py",
    "disabled": false,
    "bindings": [
        {
            "name": "myblob",
            "type": "blobTrigger",
            "direction": "in",
            "path": "samples-workitems/{name}",
            "connection":"MyStorageAccountAppSetting"
        }
    ]
}
```

Strängen `{name}` i BLOB trigger-sökvägen `samples-workitems/{name}` skapar ett [bindnings uttryck](./functions-bindings-expressions-patterns.md) som du kan använda i funktions koden för att komma åt fil namnet för den Utlös ande blobben. Mer information finns i [BLOB Name-mönster](#blob-name-patterns) längre fram i den här artikeln.

Mer information om egenskaper för *Function. JSON* -filen finns i [konfigurations](#configuration) avsnittet förklarar dessa egenskaper.

Här är python-koden:

```python
import logging
import azure.functions as func


def main(myblob: func.InputStream):
    logging.info('Python Blob trigger function processed %s', myblob.name)
```

# <a name="java"></a>[Java](#tab/java)

Den här funktionen skriver en logg när en BLOB läggs till eller uppdateras i `myblob` behållaren.

```java
@FunctionName("blobprocessor")
public void run(
  @BlobTrigger(name = "file",
               dataType = "binary",
               path = "myblob/{name}",
               connection = "MyStorageAccountAppSetting") byte[] content,
  @BindingName("name") String filename,
  final ExecutionContext context
) {
  context.getLogger().info("Name: " + filename + " Size: " + content.length + " bytes");
}
```

---

## <a name="attributes-and-annotations"></a>Attribut och anteckningar

# <a name="c"></a>[C #](#tab/csharp)

Använd följande attribut i [C#-klass bibliotek](functions-dotnet-class-library.md)för att konfigurera en BLOB-utlösare:

* [BlobTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs.Extensions.Storage/Blobs/BlobTriggerAttribute.cs)

  Attributets konstruktor tar en Sök vägs sträng som visar behållaren för att titta på och eventuellt ett [BLOB Name-mönster](#blob-name-patterns). Här är ett exempel:

  ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
  ```

  Du kan ställa in `Connection` egenskapen för att ange det lagrings konto som ska användas, som du ser i följande exempel:

   ```csharp
  [FunctionName("ResizeImage")]
  public static void Run(
      [BlobTrigger("sample-images/{name}", Connection = "StorageConnectionAppSetting")] Stream image,
      [Blob("sample-images-md/{name}", FileAccess.Write)] Stream imageSmall)
  {
      ....
  }
   ```

  Ett fullständigt exempel finns i [utlösare exempel](#example).

* [StorageAccountAttribute](https://github.com/Azure/azure-webjobs-sdk/blob/master/src/Microsoft.Azure.WebJobs/StorageAccountAttribute.cs)

  Ger ett annat sätt att ange det lagrings konto som ska användas. Konstruktorn tar namnet på en app-inställning som innehåller en lagrings anslutnings sträng. Attributet kan användas på parameter-, metod-eller klass nivå. I följande exempel visas klass nivå och metod nivå:

  ```csharp
  [StorageAccount("ClassLevelStorageAppSetting")]
  public static class AzureFunctions
  {
      [FunctionName("BlobTrigger")]
      [StorageAccount("FunctionLevelStorageAppSetting")]
      public static void Run( //...
  {
      ....
  }
  ```

Lagrings kontot som ska användas fastställs i följande ordning:

* `BlobTrigger` Attributets `Connection` egenskap.
* `StorageAccount` Attributet som används för samma parameter som `BlobTrigger` attributet.
* `StorageAccount` Attributet som används för funktionen.
* `StorageAccount` Attributet som används för klassen.
* Standard lagrings kontot för app-inställningen ("AzureWebJobsStorage").

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

Attribut stöds inte av C#-skript.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Attribut stöds inte av Java Script.

# <a name="python"></a>[Python](#tab/python)

Attribut stöds inte av python.

# <a name="java"></a>[Java](#tab/java)

`@BlobTrigger` Attributet används för att ge dig åtkomst till den blob som utlöste funktionen. Mer information finns i [utlösnings exemplet](#example) .

---

## <a name="configuration"></a>Konfiguration

I följande tabell förklaras de egenskaper för bindnings konfiguration som du anger i filen *Function. JSON* och `BlobTrigger` attributet.

|function. JSON-egenskap | Attributets egenskap |Beskrivning|
|---------|---------|----------------------|
|**bastyp** | saknas | Måste anges till `blobTrigger`. Den här egenskapen anges automatiskt när du skapar utlösaren i Azure Portal.|
|**riktning** | saknas | Måste anges till `in`. Den här egenskapen anges automatiskt när du skapar utlösaren i Azure Portal. Undantag anges i [användnings](#usage) avsnittet. |
|**Namn** | saknas | Namnet på variabeln som representerar blobben i funktions koden. |
|**sökväg** | **BlobPath** |Den [behållare](../storage/blobs/storage-blobs-introduction.md#blob-storage-resources) som ska övervakas.  Kan vara ett [BLOB Name-mönster](#blob-name-patterns). |
|**anslutningen** | **Anslutning** | Namnet på en app-inställning som innehåller den lagrings anslutnings sträng som ska användas för den här bindningen. Om appens inställnings namn börjar med "AzureWebJobs" kan du bara ange resten av namnet här. Om du till exempel anger `connection` "unstorage" söker funktions körningen efter en app-inställning med namnet "AzureWebJobsMyStorage". Om du lämnar `connection` tomt använder Functions-körningen standard anslutnings strängen för lagring i den angivna `AzureWebJobsStorage`app-inställningen.<br><br>Anslutnings strängen måste vara för ett allmänt lagrings konto, inte ett [Blob Storage-konto](../storage/common/storage-account-overview.md#types-of-storage-accounts).|

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Användning

# <a name="c"></a>[C #](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-trigger.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Få åtkomst till BLOB `context.bindings.<NAME>` - `<NAME>` data med WHERE matchar värdet som definierats i *Function. JSON*.

# <a name="python"></a>[Python](#tab/python)

Få åtkomst till BLOB-data via parametern som anges som [InputStream](https://docs.microsoft.com/python/api/azure-functions/azure.functions.inputstream?view=azure-python). Mer information finns i [utlösnings exemplet](#example) .

# <a name="java"></a>[Java](#tab/java)

`@BlobTrigger` Attributet används för att ge dig åtkomst till den blob som utlöste funktionen. Mer information finns i [utlösnings exemplet](#example) .

---

## <a name="blob-name-patterns"></a>BLOB Name-mönster

Du kan ange ett BLOB Name-mönster i `path` egenskapen i *Function. JSON* eller i `BlobTrigger` attributhierarkin. Namn mönstret kan vara ett [filter eller ett bindnings uttryck](./functions-bindings-expressions-patterns.md). I följande avsnitt finns exempel.

### <a name="get-file-name-and-extension"></a>Hämta fil namn och tillägg

I följande exempel visas hur du binder till BLOB-filens namn och tillägg separat:

```json
"path": "input/{blobname}.{blobextension}",
```

Om blobben heter *original-Blob1. txt*, är `blobname` värdena för variablerna `blobextension` och i funktions koden *ursprungliga – Blob1* och *txt*.

### <a name="filter-on-blob-name"></a>Filtrera på BLOB-namn

Följande exempel utlöser bara blobar i `input` behållaren som börjar med strängen "original-":

```json
"path": "input/original-{name}",
```

Om BLOB-namnet är *original-Blob1. txt*, är `name` `Blob1`värdet för variabeln i funktions koden.

### <a name="filter-on-file-type"></a>Filtrera efter filtyp

Följande exempel utlöser endast på *png* -filer:

```json
"path": "samples/{name}.png",
```

### <a name="filter-on-curly-braces-in-file-names"></a>Filtrera på klammerparenteser i fil namn

Om du vill söka efter klammerparenteser i fil namn, escapea klammerparenteser med två klammerparenteser. I följande exempel filtreras blobbar som har klammerparenteser i namnet:

```json
"path": "images/{{20140101}}-{name}",
```

Om blobben heter * {20140101}-Soundfile. mp3*, är `name` variabelvärdet i funktions koden *Soundfile. mp3*.

## <a name="metadata"></a>Metadata

# <a name="c"></a>[C #](#tab/csharp)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# <a name="c-script"></a>[C#-skript](#tab/csharp-script)

[!INCLUDE [functions-bindings-blob-storage-trigger](../../includes/functions-bindings-blob-storage-metadata.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```javascript
module.exports = function (context, myBlob) {
    context.log("Full blob path:", context.bindingData.blobTrigger);
    context.done();
};
```

# <a name="python"></a>[Python](#tab/python)

Metadata är inte tillgängliga i python.

# <a name="java"></a>[Java](#tab/java)

Metadata är inte tillgängliga i Java.

---

## <a name="blob-receipts"></a>BLOB-kvitton

Azure Functions runtime ser till att ingen BLOB-utlösnings funktion anropas mer än en gång för samma nya eller uppdaterade blob. För att avgöra om en specifik blob-version har bearbetats upprätthåller *BLOB-kvitton*.

Azure Functions lagrar BLOB-kvitton i en behållare med namnet *Azure-WebJobs-hosts* i Azure Storage-kontot för din Function-app (definieras `AzureWebJobsStorage`av appens inställning). Ett BLOB-kvitto har följande information:

* Funktionen triggerd ("*&lt;Function App Name>*. Funktionen. funktions namn>", till exempel:" MyFunctionApp. functions. CopyBlob ") * &lt; *
* Behållarens namn
* Blob-typ ("BlockBlob" eller "PageBlob")
* BLOB-namnet
* ETag (en BLOB versions identifierare, till exempel: "0x8D1DC6E70A277EF")

Om du vill framtvinga en ombearbetning av en BLOB tar du bort BLOB-kvittot för blobben från behållaren *Azure-WebJobs-hosts* manuellt. Det kan hända att ombearbetningen inte sker omedelbart, men det är garanterat att det sker vid en senare tidpunkt. För att ombearbeta omedelbart kan *scaninfo* -blobben i *Azure-WebJobs-hosts/blobscaninfo* uppdateras. Alla blobbar med en senast ändrad tidstämpel efter att `LatestScan` egenskapen genomsöks igen.

## <a name="poison-blobs"></a>Poison-blobbar

När en BLOB-utlösare Miss lyckas för en specifik BLOB, Azure Functions försök som fungerar totalt 5 gånger som standard.

Om alla fem försöken inte fungerar lägger Azure Functions till ett meddelande i en lagrings kö med namnet *WebJobs-en-Poison*. Det maximala antalet återförsök kan konfigureras. Samma MaxDequeueCount-inställning används för hantering av skadlig blob och meddelande hantering för hantering av skadlig kö. Queue-meddelandet för Poison-blobbar är ett JSON-objekt som innehåller följande egenskaper:

* FunctionId (i format * &lt;funktionens program namn>*. Funktionen. funktions namn>) * &lt; *
* BlobType ("BlockBlob" eller "PageBlob")
* ContainerName
* BlobName
* ETag (en BLOB versions identifierare, till exempel: "0x8D1DC6E70A277EF")

## <a name="concurrency-and-memory-usage"></a>Samtidighet och minnes användning

BLOB-utlösaren använder en kö internt, så det maximala antalet samtidiga funktions anrop styrs av [köernas konfiguration i Host. JSON](functions-host-json.md#queues). Standardinställnings gränsen samtidighet till 24 anrop. Den här gränsen gäller separat för varje funktion som använder en BLOB-utlösare.

[Förbruknings planen](functions-scale.md#how-the-consumption-and-premium-plans-work) begränsar en Function-app på en virtuell dator (VM) till 1,5 GB minne. Minne används av varje intern körning av funktions instansen och av Functions-körningen. Om en BLOB-utlöst funktion läser in hela blobben i minnet är den maximala mängd minne som används av den funktionen bara för blobbar 24 * maximal BLOB-storlek. Till exempel skulle en Function-app med tre BLOB-utlöst funktioner och standardinställningarna ha ett maximalt antal per VM-concurrency på 3 * 24 = 72 funktions anrop.

Java Script-och Java-funktioner läser in hela blobben i minnet och C#-funktionerna gör det om `string`du `Byte[]`binder till, eller Poco.

## <a name="polling"></a>Avsöknings

Avsökning fungerar som en hybrid mellan att inspektera loggar och köra regelbundna behållar genomsökningar. Blobbar genomsöks i grupper om 10 000 i taget med en fortsättnings-token som används mellan intervall.

> [!WARNING]
> Dessutom [skapas lagrings loggar på grund av "bästa prestanda"](/rest/api/storageservices/About-Storage-Analytics-Logging) . Det finns ingen garanti för att alla händelser har registrerats. Under vissa omständigheter kan loggarna missas.
> 
> Om du behöver en snabbare eller mer tillförlitlig BLOB-bearbetning bör du överväga att skapa ett [Queue-meddelande](../storage/queues/storage-dotnet-how-to-use-queues.md) när du skapar blobben. Använd sedan en [Queue-utlösare](functions-bindings-storage-queue.md) i stället för en BLOB-utlösare för att bearbeta blobben. Ett annat alternativ är att använda Event Grid; i självstudien får du [automatiskt ändra storlek på överförda bilder med hjälp av event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md).
>

## <a name="next-steps"></a>Nästa steg

- [Läsa Blob Storage-data när en funktion körs](./functions-bindings-storage-blob-input.md)
- [Skriv Blob Storage-data från en funktion](./functions-bindings-storage-blob-output.md)
