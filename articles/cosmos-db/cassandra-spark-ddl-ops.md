---
title: DDL-åtgärder i Azure Cosmos DB API för Cassandra från Spark
description: Den här artikeln innehåller information om åtgärder för åtgärds utrymme och tabell-DDL mot Azure Cosmos DB API för Cassandra från Spark.
author: kanshiG
ms.author: govindk
ms.reviewer: sngun
ms.service: cosmos-db
ms.subservice: cosmosdb-cassandra
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: c0df05eff5dc84ef24e1ed5afcaf705d99f447ef
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77622565"
---
# <a name="ddl-operations-in-azure-cosmos-db-cassandra-api-from-spark"></a>DDL-åtgärder i Azure Cosmos DB API för Cassandra från Spark

Den här artikeln innehåller information om åtgärder för åtgärds utrymme och tabell-DDL mot Azure Cosmos DB API för Cassandra från Spark.

## <a name="cassandra-api-related-configuration"></a>API för Cassandra-relaterad konfiguration 

```scala
import org.apache.spark.sql.cassandra._

//Spark connector
import com.datastax.spark.connector._
import com.datastax.spark.connector.cql.CassandraConnector

//CosmosDB library for multiple retry
import com.microsoft.azure.cosmosdb.cassandra

//Connection-related
spark.conf.set("spark.cassandra.connection.host","YOUR_ACCOUNT_NAME.cassandra.cosmosdb.azure.com")
spark.conf.set("spark.cassandra.connection.port","10350")
spark.conf.set("spark.cassandra.connection.ssl.enabled","true")
spark.conf.set("spark.cassandra.auth.username","YOUR_ACCOUNT_NAME")
spark.conf.set("spark.cassandra.auth.password","YOUR_ACCOUNT_KEY")
spark.conf.set("spark.cassandra.connection.factory", "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")

//Throughput-related...adjust as needed
spark.conf.set("spark.cassandra.output.batch.size.rows", "1")
spark.conf.set("spark.cassandra.connection.connections_per_executor_max", "10")
spark.conf.set("spark.cassandra.output.concurrent.writes", "1000")
spark.conf.set("spark.cassandra.concurrent.reads", "512")
spark.conf.set("spark.cassandra.output.batch.grouping.buffer.size", "1000")
spark.conf.set("spark.cassandra.connection.keep_alive_ms", "600000000")
```

## <a name="keyspace-ddl-operations"></a>DDL-åtgärder för disk utrymme

### <a name="create-a-keyspace"></a>Skapa ett blank steg

```scala
//Cassandra connector instance
val cdbConnector = CassandraConnector(sc)

// Create keyspace
cdbConnector.withSessionDo(session => session.execute("CREATE KEYSPACE IF NOT EXISTS books_ks WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1 } "))
```

#### <a name="validate-in-cqlsh"></a>Validera i cqlsh

Kör följande kommando i cqlsh och se det tecken utrymme som du skapade tidigare.

```bash
DESCRIBE keyspaces;
```

### <a name="drop-a-keyspace"></a>Ta bort ett blank steg

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP KEYSPACE books_ks"))
```

#### <a name="validate-in-cqlsh"></a>Validera i cqlsh

```bash
DESCRIBE keyspaces;
```
## <a name="table-ddl-operations"></a>Tabell-DDL-åtgärder

**Angående**  

- Data flödet kan tilldelas på tabell nivå med instruktionen CREATE TABLE.  
- En partitionsnyckel kan lagra 20 GB data.  
- En post kan lagra högst 2 MB data.  
- Ett nyckel intervall för partition kan lagra flera partitionsnyckel.

### <a name="create-a-table"></a>Skapa en tabell

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("CREATE TABLE IF NOT EXISTS books_ks.books(book_id TEXT PRIMARY KEY,book_author TEXT, book_name TEXT,book_pub_year INT,book_price FLOAT) WITH cosmosdb_provisioned_throughput=4000 , WITH default_time_to_live=630720000;"))
```

#### <a name="validate-in-cqlsh"></a>Validera i cqlsh

Kör följande kommando i cqlsh och du bör se tabellen med namnet "Books: 

```bash
USE books_ks;
DESCRIBE books;
```

Etablerade data flöden och standard TTL-värden visas inte i utdata från föregående kommando, du kan hämta dessa värden från portalen.

### <a name="alter-table"></a>Ändra tabell

Du kan ändra följande värden med kommandot ALTER TABLE:

* allokerat data flöde 
* TTL-värde (Time to Live)
<br>Kolumn ändringar stöds inte för närvarande.

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("ALTER TABLE books_ks.books WITH cosmosdb_provisioned_throughput=8000, WITH default_time_to_live=0;"))
```

### <a name="drop-table"></a>Ta bort tabell

```scala
val cdbConnector = CassandraConnector(sc)
cdbConnector.withSessionDo(session => session.execute("DROP TABLE IF EXISTS books_ks.books;"))
```

#### <a name="validate-in-cqlsh"></a>Validera i cqlsh

Kör följande kommando i cqlsh och se att tabellen "böcker" inte längre är tillgänglig:

```bash
USE books_ks;
DESCRIBE tables;
```

## <a name="next-steps"></a>Nästa steg

När du har skapat tecken utrymmet och tabellen fortsätter du till följande artiklar för CRUD-åtgärder med mera:
 
* [Skapa/infoga åtgärder](cassandra-spark-create-ops.md)  
* [Läs åtgärder](cassandra-spark-read-ops.md)  
* [Upsert-åtgärder](cassandra-spark-upsert-ops.md)  
* [Ta bort åtgärder](cassandra-spark-delete-ops.md)  
* [Sammansättningsåtgärder](cassandra-spark-aggregation-ops.md)  
* [Åtgärder för tabell kopiering](cassandra-spark-table-copy-ops.md)  
