---
title: Hantera indexeringsprinciper i Azure Cosmos DB
description: 'Lär dig hur du hanterar indexerings principer, tar med eller undantar en egenskap från indexering, hur du definierar indexering med hjälp av olika Azure Cosmos DB SDK: er'
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/28/2020
ms.author: tisande
ms.openlocfilehash: b913ba58252f4cb84d010aea39d371316582bd6d
ms.sourcegitcommit: f57297af0ea729ab76081c98da2243d6b1f6fa63
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82869925"
---
# <a name="manage-indexing-policies-in-azure-cosmos-db"></a>Hantera indexeringsprinciper i Azure Cosmos DB

I Azure Cosmos DB indexeras data efter [indexerings principer](index-policy.md) som definierats för varje behållare. Standardprincipen för indexering för nyligen skapade containrar framtvingar intervallindex för strängar eller nummer. Du kan åsidosätta den här principen med din egen anpassade indexeringsprincip.

## <a name="indexing-policy-examples"></a>Exempel på indexeringsprincip

Här följer några exempel på indexerings principer som visas i [deras JSON-format](index-policy.md#include-exclude-paths), vilket är hur de exponeras på Azure Portal. Samma parametrar kan ställas in via Azure CLI eller SDK.

### <a name="opt-out-policy-to-selectively-exclude-some-property-paths"></a>Opt-out-princip för att selektivt utesluta vissa egenskaps Sök vägar

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

Den här indexerings principen motsvarar den som anges nedan och som manuellt ```kind```ställer ```dataType```in, ```precision``` och till standardvärdena. De här egenskaperna är inte längre nödvändiga för att uttryckligen anges och du kan utesluta dem från din indexerings princip helt (se exemplet ovan).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number",
                        "precision": -1
                    },
                    {
                        "kind": "Range",
                        "dataType": "String",
                        "precision": -1
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/path/to/single/excluded/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/excluded/properties/*"
            }
        ]
    }
```

### <a name="opt-in-policy-to-selectively-include-some-property-paths"></a>Opt-in-princip för att selektivt inkludera vissa egenskaps Sök vägar

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?"
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*"
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

Den här indexerings principen motsvarar den som anges nedan och som manuellt ```kind```ställer ```dataType```in, ```precision``` och till standardvärdena. De här egenskaperna är inte längre nödvändiga för att uttryckligen anges och du kan utesluta dem från din indexerings princip helt (se exemplet ovan).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [
            {
                "path": "/path/to/included/property/?",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            },
            {
                "path": "/path/to/root/of/multiple/included/properties/*",
                "indexes": [
                    {
                        "kind": "Range",
                        "dataType": "Number"
                    },
                    {
                        "kind": "Range",
                        "dataType": "String"
                    }
                ]
            }
        ],
        "excludedPaths": [
            {
                "path": "/*"
            }
        ]
    }
```

> [!NOTE]
> Vi rekommenderar vanligt vis att du använder en **opt-out-** indexerings princip för att låta Azure Cosmos DB indexera alla nya egenskaper som kan läggas till i din modell proaktivt.

### <a name="using-a-spatial-index-on-a-specific-property-path-only"></a>Använda ett rums index för en speciell egenskaps Sök väg

```json
{
    "indexingMode": "consistent",
    "automatic": true,
    "includedPaths": [
        {
            "path": "/*"
        }
    ],
    "excludedPaths": [
        {
            "path": "/_etag/?"
        }
    ],
    "spatialIndexes": [
        {
            "path": "/path/to/geojson/property/?",
            "types": [
                "Point",
                "Polygon",
                "MultiPolygon",
                "LineString"
            ]
        }
    ]
}
```

## <a name="composite-indexing-policy-examples"></a>Exempel på sammansatta indexerings principer

Förutom att inkludera eller exkludera sökvägar för enskilda egenskaper kan du också ange ett sammansatt index. Om du vill utföra en fråga som har en `ORDER BY` sats för flera egenskaper, krävs ett [sammansatt index](index-policy.md#composite-indexes) för dessa egenskaper. Dessutom får sammansatta index en prestanda förmån för frågor som har ett filter och har en ORDER BY-sats för olika egenskaper.

> [!NOTE]
> Sammansatta sökvägar `/?` har ett implicit eftersom endast det skalära värdet på den sökvägen indexeras. `/*` Jokertecknet stöds inte i sammansatta sökvägar. Du bör inte `/?` ange `/*` eller i en sammansatt sökväg.

### <a name="composite-index-defined-for-name-asc-age-desc"></a>Sammansatt index definierat för (namn ASC, ålders DESC):

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

Det sammansatta indexet för namn och ålder krävs för fråga #1 och fråga #2:

Fråga #1:

```sql
    SELECT *
    FROM c
    ORDER BY c.name ASC, c.age DESC
```

Fråga #2:

```sql
    SELECT *
    FROM c
    ORDER BY c.name DESC, c.age ASC
```

Det sammansatta indexet kommer att dra nytta av frågan #3 och fråga #4 och optimera filtren:

Fråga #3:

```sql
SELECT *
FROM c
WHERE c.name = "Tim"
ORDER BY c.name DESC, c.age ASC
```

Fråga #4:

```sql
SELECT *
FROM c
WHERE c.name = "Tim" AND c.age > 18
```

### <a name="composite-index-defined-for-name-asc-age-asc-and-name-asc-age-desc"></a>Sammansatt index definierat för (namn ASC, ålder ASC) och (namn ASC, ålders DESC):

Du kan definiera flera olika sammansatta index i samma indexerings princip.

```json
    {  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"ascending"
                }
            ],
            [  
                {  
                    "path":"/name",
                    "order":"ascending"
                },
                {  
                    "path":"/age",
                    "order":"descending"
                }
            ]
        ]
    }
```

### <a name="composite-index-defined-for-name-asc-age-asc"></a>Sammansatt index definierat för (namn ASC, ålder ASC):

Det är valfritt att ange ordningen. Om detta inte anges, är ordningen stigande.

```json
{  
        "automatic":true,
        "indexingMode":"Consistent",
        "includedPaths":[  
            {  
                "path":"/*"
            }
        ],
        "excludedPaths":[],
        "compositeIndexes":[  
            [  
                {  
                    "path":"/name",
                },
                {  
                    "path":"/age",
                }
            ]
        ]
}
```

### <a name="excluding-all-property-paths-but-keeping-indexing-active"></a>Exkludera alla egenskaps Sök vägar men behålla indexering aktiv

Den här principen kan användas i situationer där [TTL-funktionen (Time-to-Live)](time-to-live.md) är aktiv men inget sekundärt index krävs (för att använda Azure Cosmos DB som ett rent nyckel värdes lager).

```json
    {
        "indexingMode": "consistent",
        "includedPaths": [],
        "excludedPaths": [{
            "path": "/*"
        }]
    }
```

### <a name="no-indexing"></a>Ingen indexering

Med den här principen stängs indexeringen av. Om `indexingMode` är inställt på `none`kan du inte ange ett TTL-värde för behållaren.

```json
    {
        "indexingMode": "none"
    }
```

## <a name="updating-indexing-policy"></a>Uppdaterar indexerings princip

I Azure Cosmos DB kan indexerings principen uppdateras med någon av metoderna nedan:

- från Azure Portal
- använda Azure CLI
- använda PowerShell
- använda en av SDK: erna

En [indexerings princip uppdatering](index-policy.md#modifying-the-indexing-policy) utlöser en index omvandling. Förloppet för den här omvandlingen kan också spåras från SDK: er.

> [!NOTE]
> När du uppdaterar indexerings principen avbryts skrivningen till Azure Cosmos DB. Vid Omindexering kan frågor returnera partiella resultat när indexet uppdateras.

## <a name="use-the-azure-portal"></a>Använda Azure-portalen

Azure Cosmos-behållare lagrar sin indexerings princip som ett JSON-dokument som Azure Portal gör att du kan redigera den direkt.

1. Logga in på [Azure-portalen](https://portal.azure.com/).

1. Skapa ett nytt Azure Cosmos DB-konto eller välj ett befintligt konto.

1. Öppna fönstret **datautforskaren** och välj den behållare som du vill arbeta med.

1. Klicka på **skala & inställningar**.

1. Ändra JSON-dokumentet för indexerings principen (se exemplen [nedan](#indexing-policy-examples))

1. Klicka på **Spara** när du är klar.

![Hantera indexering med hjälp av Azure-portalen](./media/how-to-manage-indexing-policy/indexing-policy-portal.png)

## <a name="use-the-azure-cli"></a>Använda Azure CLI

Om du vill skapa en behållare med en anpassad indexerings princip går du till [skapa en behållare med en anpassad index princip med hjälp av CLI](manage-with-cli.md#create-a-container-with-a-custom-index-policy)

## <a name="use-powershell"></a>Använd PowerShell

Om du vill skapa en behållare med en anpassad indexerings princip går du till [skapa en behållare med en anpassad index princip med hjälp av PowerShell](manage-with-powershell.md#create-container-custom-index)

## <a name="use-the-net-sdk"></a><a id="dotnet-sdk"></a>Använda .NET SDK

# <a name="net-sdk-v2"></a>[.NET SDK V2](#tab/dotnetv2)

`DocumentCollection` Objektet från [.NET SDK v2](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB/) visar en `IndexingPolicy` egenskap som gör att du kan ändra `IndexingMode` och lägga till eller ta bort `IncludedPaths` och `ExcludedPaths`.

```csharp
// Retrieve the container's details
ResourceResponse<DocumentCollection> containerResponse = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"));
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(new SpatialSpec() { Path = "/locations/*", SpatialTypes = new Collection<SpatialType>() { SpatialType.Point } } );
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> {new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending }});
// Update container with changes
await client.ReplaceDocumentCollectionAsync(containerResponse.Resource);
```

Om du vill spåra omvandlings förloppet för `RequestOptions` indexet skickar du `PopulateQuotaInfo` ett objekt `true`som anger egenskapen till.

```csharp
// retrieve the container's details
ResourceResponse<DocumentCollection> container = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("database", "container"), new RequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = container.IndexTransformationProgress;
```

# <a name="net-sdk-v3"></a>[.NET SDK V3](#tab/dotnetv3)

`ContainerProperties` Objektet från [.NET SDK v3](https://www.nuget.org/packages/Microsoft.Azure.Cosmos/) (se [den här snabb](create-sql-api-dotnet.md) starten om användningen) visar `IndexingPolicy` en egenskap som gör att du kan ändra `IndexingMode` och lägga till eller ta `IncludedPaths` bort `ExcludedPaths`och.

```csharp
// Retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Set the indexing mode to consistent
containerResponse.Resource.IndexingPolicy.IndexingMode = IndexingMode.Consistent;
// Add an included path
containerResponse.Resource.IndexingPolicy.IncludedPaths.Add(new IncludedPath { Path = "/*" });
// Add an excluded path
containerResponse.Resource.IndexingPolicy.ExcludedPaths.Add(new ExcludedPath { Path = "/name/*" });
// Add a spatial index
SpatialPath spatialPath = new SpatialPath
{
    Path = "/locations/*"
};
spatialPath.SpatialTypes.Add(SpatialType.Point);
containerResponse.Resource.IndexingPolicy.SpatialIndexes.Add(spatialPath);
// Add a composite index
containerResponse.Resource.IndexingPolicy.CompositeIndexes.Add(new Collection<CompositePath> { new CompositePath() { Path = "/name", Order = CompositePathSortOrder.Ascending }, new CompositePath() { Path = "/age", Order = CompositePathSortOrder.Descending } });
// Update container with changes
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

Om du vill spåra omvandlings förloppet för `RequestOptions` indexet skickar du `PopulateQuotaInfo` ett objekt `true`som anger egenskapen till och hämtar sedan `x-ms-documentdb-collection-index-transformation-progress` värdet från svars huvudet.

```csharp
// retrieve the container's details
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync(new ContainerRequestOptions { PopulateQuotaInfo = true });
// retrieve the index transformation progress from the result
long indexTransformationProgress = long.Parse(containerResponse.Headers["x-ms-documentdb-collection-index-transformation-progress"]);
```

När du definierar en anpassad indexerings princip när du skapar en ny behållare kan du med SDK V3's Fluent API skriva den här definitionen på ett koncist och effektivt sätt:

```csharp
await client.GetDatabase("database").DefineContainer(name: "container", partitionKeyPath: "/myPartitionKey")
    .WithIndexingPolicy()
        .WithIncludedPaths()
            .Path("/*")
        .Attach()
        .WithExcludedPaths()
            .Path("/name/*")
        .Attach()
        .WithSpatialIndex()
            .Path("/locations/*", SpatialType.Point)
        .Attach()
        .WithCompositeIndex()
            .Path("/name", CompositePathSortOrder.Ascending)
            .Path("/age", CompositePathSortOrder.Descending)
        .Attach()
    .Attach()
    .CreateIfNotExistsAsync();
```
---

## <a name="use-the-java-sdk"></a>Använda Java SDK

`DocumentCollection` Objektet från [Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb) (se [den här snabb](create-sql-api-java.md) starten om användningen) visar `getIndexingPolicy()` och `setIndexingPolicy()` metoder. Med `IndexingPolicy` objektet som de ändrar, kan du ändra indexerings läget och lägga till eller ta bort inkluderade och undantagna sökvägar.

```java
// Retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), null);
containerResponse.subscribe(result -> {
DocumentCollection container = result.getResource();
IndexingPolicy indexingPolicy = container.getIndexingPolicy();

// Set the indexing mode to consistent
indexingPolicy.setIndexingMode(IndexingMode.Consistent);

// Add an included path

Collection<IncludedPath> includedPaths = new ArrayList<>();
IncludedPath includedPath = new IncludedPath();
includedPath.setPath("/*");
includedPaths.add(includedPath);
indexingPolicy.setIncludedPaths(includedPaths);

// Add an excluded path

Collection<ExcludedPath> excludedPaths = new ArrayList<>();
ExcludedPath excludedPath = new ExcludedPath();
excludedPath.setPath("/name/*");
excludedPaths.add(excludedPath);
indexingPolicy.setExcludedPaths(excludedPaths);

// Add a spatial index

Collection<SpatialSpec> spatialIndexes = new ArrayList<SpatialSpec>();
Collection<SpatialType> collectionOfSpatialTypes = new ArrayList<SpatialType>();

SpatialSpec spec = new SpatialSpec();
spec.setPath("/locations/*");
collectionOfSpatialTypes.add(SpatialType.Point);
spec.setSpatialTypes(collectionOfSpatialTypes);
spatialIndexes.add(spec);

indexingPolicy.setSpatialIndexes(spatialIndexes);

// Add a composite index

Collection<ArrayList<CompositePath>> compositeIndexes = new ArrayList<>();
ArrayList<CompositePath> compositePaths = new ArrayList<>();

CompositePath nameCompositePath = new CompositePath();
nameCompositePath.setPath("/name");
nameCompositePath.setOrder(CompositePathSortOrder.Ascending);

CompositePath ageCompositePath = new CompositePath();
ageCompositePath.setPath("/age");
ageCompositePath.setOrder(CompositePathSortOrder.Descending);

compositePaths.add(ageCompositePath);
compositePaths.add(nameCompositePath);

compositeIndexes.add(compositePaths);
indexingPolicy.setCompositeIndexes(compositeIndexes);

// Update the container with changes

 client.replaceCollection(container, null);
});
```

Om du vill spåra omvandlings förloppet för index omvandlingen på `RequestOptions` en behållare, skickar du ett objekt som begär kvot informationen och hämtar sedan värdet från `x-ms-documentdb-collection-index-transformation-progress` svars huvudet.

```java
// set the RequestOptions object
RequestOptions requestOptions = new RequestOptions();
requestOptions.setPopulateQuotaInfo(true);
// retrieve the container's details
Observable<ResourceResponse<DocumentCollection>> containerResponse = client.readCollection(String.format("/dbs/%s/colls/%s", "database", "container"), requestOptions);
containerResponse.subscribe(result -> {
    // retrieve the index transformation progress from the response headers
    String indexTransformationProgress = result.getResponseHeaders().get("x-ms-documentdb-collection-index-transformation-progress");
});
```

## <a name="use-the-nodejs-sdk"></a>Använd Node. js SDK

`ContainerDefinition` Gränssnittet från [Node. js SDK](https://www.npmjs.com/package/@azure/cosmos) (se [den här snabb](create-sql-api-nodejs.md) starten om användningen) visar `indexingPolicy` en egenskap som gör att du kan ändra `indexingMode` och lägga till eller `includedPaths` ta `excludedPaths`bort och.

Hämta behållarens information

```javascript
const containerResponse = await client.database('database').container('container').read();
```

Ange att indexerings läget ska vara konsekvent

```javascript
containerResponse.body.indexingPolicy.indexingMode = "consistent";
```

Lägg till inkluderad sökväg inklusive ett rums index

```javascript
containerResponse.body.indexingPolicy.includedPaths.push({
    includedPaths: [
      {
        path: "/age/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.String
          },
          {
            kind: cosmos.DocumentBase.IndexKind.Range,
            dataType: cosmos.DocumentBase.DataType.Number
          }
        ]
      },
      {
        path: "/locations/*",
        indexes: [
          {
            kind: cosmos.DocumentBase.IndexKind.Spatial,
            dataType: cosmos.DocumentBase.DataType.Point
          }
        ]
      }
    ]
  });
```

Lägg till undantagen sökväg

```javascript
containerResponse.body.indexingPolicy.excludedPaths.push({ path: '/name/*' });
```

Uppdatera behållaren med ändringar

```javascript
const replaceResponse = await client.database('database').container('container').replace(containerResponse.body);
```

Om du vill spåra omvandlings förloppet för indexet i `RequestOptions` en behållare, skickar `populateQuotaInfo` du ett `true`objekt som anger egenskapen till och hämtar `x-ms-documentdb-collection-index-transformation-progress` sedan värdet från svars huvudet.

```javascript
// retrieve the container's details
const containerResponse = await client.database('database').container('container').read({
    populateQuotaInfo: true
});
// retrieve the index transformation progress from the response headers
const indexTransformationProgress = replaceResponse.headers['x-ms-documentdb-collection-index-transformation-progress'];
```

## <a name="use-the-python-sdk"></a>Använda Python SDK

# <a name="python-sdk-v3"></a>[Python SDK v3](#tab/pythonv3)

När du använder [python SDK v3](https://pypi.org/project/azure-cosmos/) (se [den här snabb](create-sql-api-python.md) starten om användningen) hanteras behållar konfigurationen som en ord lista. Från den här ord listan är det möjligt att komma åt indexerings principen och alla dess attribut.

Hämta behållarens information

```python
containerPath = 'dbs/database/colls/collection'
container = client.ReadContainer(containerPath)
```

Ange att indexerings läget ska vara konsekvent

```python
container['indexingPolicy']['indexingMode'] = 'consistent'
```

Definiera en indexerings princip med en inkluderad sökväg och ett rums index

```python
container["indexingPolicy"] = {

    "indexingMode":"consistent",
    "spatialIndexes":[
                {"path":"/location/*","types":["Point"]}
             ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Definiera en indexerings princip med en undantagen sökväg

```python
container["indexingPolicy"] = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Lägg till ett sammansatt index

```python
container['indexingPolicy']['compositeIndexes'] = [
                [
                    {
                        "path": "/name",
                        "order": "ascending"
                    },
                    {
                        "path": "/age",
                        "order": "descending"
                    }
                ]
                ]
```

Uppdatera behållaren med ändringar

```python
response = client.ReplaceContainer(containerPath, container)
```

# <a name="python-sdk-v4"></a>[Python SDK v4](#tab/pythonv4)

När du använder [python SDK v4](https://pypi.org/project/azure-cosmos/)hanteras behållar konfigurationen som en ord lista. Från den här ord listan är det möjligt att komma åt indexerings principen och alla dess attribut.

Hämta behållarens information

```python
database_client = cosmos_client.get_database_client('database')
container_client = database_client.get_container_client('container')
container = container_client.read()
```

Ange att indexerings läget ska vara konsekvent

```python
indexingPolicy = {
    'indexingMode': 'consistent'
}
```

Definiera en indexerings princip med en inkluderad sökväg och ett rums index

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "spatialIndexes":[
        {"path":"/location/*","types":["Point"]}
    ],
    "includedPaths":[{"path":"/age/*","indexes":[]}],
    "excludedPaths":[{"path":"/*"}]
}
```

Definiera en indexerings princip med en undantagen sökväg

```python
indexingPolicy = {
    "indexingMode":"consistent",
    "includedPaths":[{"path":"/*","indexes":[]}],
    "excludedPaths":[{"path":"/name/*"}]
}
```

Lägg till ett sammansatt index

```python
indexingPolicy['compositeIndexes'] = [
    [
        {
            "path": "/name",
            "order": "ascending"
        },
        {
            "path": "/age",
            "order": "descending"
        }
    ]
]
```

Uppdatera behållaren med ändringar

```python
response = database_client.replace_container(container_client, container['partitionKey'], indexingPolicy)
```
---

## <a name="next-steps"></a>Nästa steg

Läs mer om indexering i följande artiklar:

- [Indexeringsöversikt](index-overview.md)
- [Indexerings princip](index-policy.md)
