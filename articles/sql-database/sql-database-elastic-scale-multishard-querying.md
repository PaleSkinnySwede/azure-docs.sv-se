---
title: Fråga shardade-databaser
description: Kör frågor över Shards med klient biblioteket för Elastic Database.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.date: 01/25/2019
ms.openlocfilehash: ae14a9fd8fc8479eac596fb694e12e3e0a9027f5
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80067302"
---
# <a name="multi-shard-querying-using-elastic-database-tools"></a>Shard frågor med Elastic Database-verktyg

## <a name="overview"></a>Översikt

Med [Elastic Database verktyg](sql-database-elastic-scale-introduction.md)kan du skapa shardade-databas lösningar. **Multi-Shard-frågor** används för uppgifter som data insamling/rapportering som kräver körning av en fråga som sträcker sig över flera Shards. (Kontrast till [data beroende routning](sql-database-elastic-scale-data-dependent-routing.md), som utför allt arbete på en enda Shard.)

1. Hämta en **RangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) eller **ListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) med hjälp av **TryGetRangeShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), **TryGetListShardMap** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)) eller **GetShardMap** -metoden ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)). Se [konstruera en ShardMapManager](sql-database-elastic-scale-shard-map-management.md#constructing-a-shardmapmanager) och [Hämta en RangeShardMap eller ListShardMap](sql-database-elastic-scale-shard-map-management.md#get-a-rangeshardmap-or-listshardmap).
2. Skapa ett **MultiShardConnection** -objekt ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardconnection), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardconnection)).
3. Skapa en **MultiShardStatement eller MultiShardCommand** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
4. Ange **CommandText-egenskapen** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)) till ett T-SQL-kommando.
5. Kör kommandot genom att anropa metoden **ExecuteQueryAsync eller ExecuteReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardstatement.executeQueryAsync), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multishardcommand)).
6. Visa resultaten med hjälp av klassen **MultiShardResultSet eller MultiShardDataReader** ([Java](/java/api/com.microsoft.azure.elasticdb.query.multishard.multishardresultset), [.net](https://docs.microsoft.com/dotnet/api/microsoft.azure.sqldatabase.elasticscale.query.multisharddatareader)).

## <a name="example"></a>Exempel

Följande kod illustrerar användningen av multi-Shard-frågor med en angiven **ShardMap** med namnet *myShardMap*.

```csharp
using (MultiShardConnection conn = new MultiShardConnection(myShardMap.GetShards(), myShardConnectionString))
{
    using (MultiShardCommand cmd = conn.CreateCommand())
    {
        cmd.CommandText = "SELECT c1, c2, c3 FROM ShardedTable";
        cmd.CommandType = CommandType.Text;
        cmd.ExecutionOptions = MultiShardExecutionOptions.IncludeShardNameColumn;
        cmd.ExecutionPolicy = MultiShardExecutionPolicy.PartialResults;

        using (MultiShardDataReader sdr = cmd.ExecuteReader())
        {
            while (sdr.Read())
            {
                var c1Field = sdr.GetString(0);
                var c2Field = sdr.GetFieldValue<int>(1);
                var c3Field = sdr.GetFieldValue<Int64>(2);
            }
        }
    }
}
```

En viktig skillnad är att bygga anslutningar med flera Shard. När **SQLConnection** körs på en enskild databas tar **MultiShardConnection** en ***samling Shards*** som indata. Fyll i samlingen av Shards från en Shard-karta. Frågan körs sedan på samlingen Shards med **union all** semantik för att sammanställa ett enda övergripande resultat. Alternativt kan namnet på Shard där raden kommer från kan läggas till i utdata med hjälp av egenskapen **ExecutionOptions** i kommandot.

Notera anropet till **myShardMap. GetShards ()**. Den här metoden hämtar alla Shards från Shard-kartan och ger ett enkelt sätt att köra en fråga i alla relevanta databaser. Mängden Shards för en Shard-fråga kan förfinas ytterligare genom att utföra en LINQ-fråga över samlingen som returneras från anropet till **myShardMap. GetShards ()**. I kombination med den partiella resultat principen har den aktuella funktionen i multi-Shard-frågor utformats för att fungera bra i upp till hundratals Shards.

En begränsning med multi-Shard-frågor är för närvarande brist på verifiering för Shards och shardletar som efter frågas. Medan data beroende routning verifierar att en specifik Shard är en del av Shard-kartan vid tidpunkten för frågan, utför inte Multi-Shard-frågor den här kontrollen. Detta kan leda till att flera Shard-frågor körs på databaser som har tagits bort från Shard-kartan.

## <a name="multi-shard-queries-and-split-merge-operations"></a>Shard frågor och dela sammanslagnings åtgärder

Frågor med multi-Shard verifierar inte om shardletar på den efterfrågade databasen deltar i pågående delnings sammanslagnings åtgärder. (Se [skalning med Elastic Database dela-merge-verktyget](sql-database-elastic-scale-overview-split-and-merge.md).) Detta kan leda till inkonsekvenser där rader från samma shardlet visas för flera databaser i samma multi-Shard-fråga. Tänk på dessa begränsningar och Överväg att tömma pågående delnings sammanslagnings åtgärder och ändringar i Shard-kartan samtidigt som du utför frågor med flera Shard.

[!INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]