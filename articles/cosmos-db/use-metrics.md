---
title: Övervaka och felsöka med mått i Azure Cosmos DB
description: Använd mått i Azure Cosmos DB för att felsöka vanliga problem och övervaka databasen.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/18/2019
ms.openlocfilehash: b65bc6097d4841c79a68d4313ac7a3f89f6d1dbb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80065937"
---
# <a name="monitor-and-debug-with-metrics-in-azure-cosmos-db"></a>Övervaka och felsöka med mått i Azure Cosmos DB

Azure Cosmos DB tillhandahåller mått för dataflöde, lagring, konsekvens, tillgänglighet och svarstid. Azure-portalen innehåller en sammanställd vy över dessa mått. Du kan också Visa Azure Cosmos DB-mått från Azure Monitor-API:et. Information om hur du visar mått från Azure Monitor finns i artikeln [Hämta mått från Azure Monitor](cosmos-db-azure-monitor-metrics.md) . 

Den här artikeln vägleder dig genom vanliga användnings fall och hur Azure Cosmos DB mått kan användas för att analysera och felsöka problemen. Mått samlas in var femte minut och bevaras i sju dagar.

## <a name="view-metrics-from-azure-portal"></a>Visa mått från Azure Portal

1. Logga in på [Azure-portalen](https://portal.azure.com/)

1. Öppna fönstret **mått** . Som standard visar fönstret mått lagring, index, enheter för enhets mått för alla databaser i ditt Azure Cosmos-konto. Du kan filtrera dessa mått per databas, behållare eller region. Du kan också filtrera måtten vid en angiven tids kornig het. Mer information om data flöde, lagring, tillgänglighet, svars tid och konsekvens mått finns på separata flikar. 

   ![Cosmos DB prestanda mått i Azure Portal](./media/use-metrics/performance-metrics.png)

Följande mått är tillgängliga från fönstret **mått** : 

* **Data flödes mått** – det här måttet visar antalet begär Anden som för bruk ATS eller misslyckades (429 svars kod) eftersom data flödet eller lagrings kapaciteten som har allokerats för behållaren har överskridits.

* **Lagrings mått** – det här måttet visar storleken på data-och index användning.

* **Tillgänglighets mått** – det här måttet visar procent andelen lyckade förfrågningar för totalt antal begär Anden per timme. Frekvensen lyckades definieras av Azure Cosmos DB service avtal.

* **Svars tids mått** – det här måttet visar svars tiden för läsning och skrivning som observerats av Azure Cosmos db i den region där ditt konto är igång. Du kan visualisera svars tider i regioner för ett geo-replikerat konto. Detta mått representerar inte svars tiden för slut punkt till slut punkt.

* **Konsekvens mått** – det här måttet visar hur eventuell konsekvens för konsekvens modellen du väljer. För konton med flera regioner visar det här måttet även replikeringsfördröjning mellan de regioner som du har valt.

* **System mått** – det här måttet visar hur många metadata-begäranden som hanteras av huvudpartitionen. Den hjälper också till att identifiera begränsade begär Anden.

I följande avsnitt beskrivs vanliga scenarier där du kan använda Azure Cosmos DB mått. 

## <a name="understand-how-many-requests-are-succeeding-or-causing-errors"></a>Förstå hur många begär Anden som lyckas eller orsakar fel

Kom igång genom att gå till [Azure Portal](https://portal.azure.com) och navigera till **mått** bladet. På bladet hittar du * *-antalet begär Anden överskred kapaciteten per 1 minut diagram. Det här diagrammet visar antalet minuter per minut som har segmenterats av status koden. Mer information om HTTP-statuskod finns i [http status Codes för Azure Cosmos DB](https://docs.microsoft.com/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

Den vanligaste fel status koden är 429 (hastighets begränsning/begränsning). Det här felet innebär att begär anden till Azure Cosmos DB är mer än det etablerade data flödet. Den vanligaste lösningen på det här problemet är att [skala upp ru: er](./set-throughput.md) för den aktuella samlingen.

![Antal begär Anden per minut](media/use-metrics/metrics-12.png)

## <a name="determine-the-throughput-distribution-across-partitions"></a>Fastställa data flödes distributionen över partitioner

Att ha en bra kardinalitet för dina partitionstyper är viktig för alla skalbara program. Om du vill fastställa data flödes distributionen av en partitionerad behållare uppdelad efter partition, navigerar du till **bladet mått** i [Azure Portal](https://portal.azure.com). På fliken **data flöde** visas lagrings uppdelningen i den **högsta förbrukade ru/sekund av varje fysiskt partitionsschema** . Följande bild illustrerar ett exempel på en dålig data distribution som visas av den sneda partitionen längst till vänster.

![En enda partition som ser tung användning på 3:05 PM](media/use-metrics/metrics-17.png)

En ojämn distribution av data flödet kan orsaka *varma* partitioner, vilket kan leda till begränsade begär Anden och kan kräva ompartitionering. Mer information om partitionering i Azure Cosmos DB finns i [partition och skala i Azure Cosmos DB](./partition-data.md).

## <a name="determine-the-storage-distribution-across-partitions"></a>Fastställa lagrings fördelningen mellan partitioner

Att ha en bra kardinalitet för din partition är viktig för alla skalbara program. För att fastställa lagrings distributionen av en partitionerad behållare uppdelad efter partition, head to the Metrics-bladet i [Azure Portal](https://portal.azure.com). På fliken lagring visas lagrings uppdelningen i diagrammet data + index lagring som förbrukas av främsta partitionerings nycklar. Följande bild illustrerar en dålig distribution av data lagring som visas av den sneda partitionen längst till vänster.

![Exempel på dålig data distribution](media/use-metrics/metrics-07.png)

Du kan rotor Saks vilken partitionsnyckel som skevar distributionen genom att klicka på partitionen i diagrammet.

![Partitionsnyckel skevar fördelningen](media/use-metrics/metrics-05.png)

När du har identifierat vilken partitionsnyckel som orsakar snedheten i distributionen kan du behöva partitionera om behållaren med en mer distribuerad partitionsnyckel. Mer information om partitionering i Azure Cosmos DB finns i [partition och skala i Azure Cosmos DB](./partition-data.md).

## <a name="compare-data-size-against-index-size"></a>Jämför data storleken mot index storleken

I Azure Cosmos DB är den totala förbrukade lagringen kombinationen av både data storlek och index storlek. Index storleken är vanligt vis en bråkdel av data storleken. På bladet mått i [Azure Portal](https://portal.azure.com)visar fliken Storage fördelningen av lagrings förbrukning baserat på data och index.

```csharp
// Measure the document size usage (which includes the index size)  
ResourceResponse<DocumentCollection> collectionInfo = await client.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri("db", "coll"));
 Console.WriteLine("Document size quota: {0}, usage: {1}", collectionInfo.DocumentQuota, collectionInfo.DocumentUsage);
```

Om du vill spara index utrymme kan du justera [indexerings principen](index-policy.md).

## <a name="debug-why-queries-are-running-slow"></a>Felsöka varför frågor körs långsamt

Azure Cosmos DB innehåller statistik för körning av frågor i SQL API SDK: n.

```csharp
IDocumentQuery<dynamic> query = client.CreateDocumentQuery(
 UriFactory.CreateDocumentCollectionUri(DatabaseName, CollectionName),
 "SELECT * FROM c WHERE c.city = 'Seattle'",
 new FeedOptions
 {
 PopulateQueryMetrics = true,
 MaxItemCount = -1,
 MaxDegreeOfParallelism = -1,
 EnableCrossPartitionQuery = true
 }).AsDocumentQuery();
FeedResponse<dynamic> result = await query.ExecuteNextAsync();

// Returns metrics by partition key range Id
IReadOnlyDictionary<string, QueryMetrics> metrics = result.QueryMetrics;
```

*QueryMetrics* innehåller information om hur länge varje komponent i frågan ska köras. Den vanligaste rotor saken för tids krävande frågor är genomsökningar, vilket innebär att frågan inte kunde utnyttja indexen. Det här problemet kan lösas med ett bättre filter villkor.

## <a name="next-steps"></a>Nästa steg

Nu har du lärt dig hur du övervakar och felsöker problem med hjälp av de mått som anges i Azure Portal. Du kanske vill lära dig mer om hur du kan förbättra databasens prestanda genom att läsa följande artiklar:

* Information om hur du visar mått från Azure Monitor finns i artikeln [Hämta mått från Azure Monitor](cosmos-db-azure-monitor-metrics.md) . 
* [Prestanda-och skalnings testning med Azure Cosmos DB](performance-testing.md)
* [Prestandatips för Azure Cosmos DB](performance-tips.md)
