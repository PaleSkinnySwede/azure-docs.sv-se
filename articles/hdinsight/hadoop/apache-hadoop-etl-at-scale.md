---
title: Extrahera, transformera och läsa in (ETL) i skala – Azure HDInsight
description: Lär dig hur extrahering, transformering och inläsning används i HDInsight med Apache Hadoop.
author: ashishthaps
ms.author: ashishth
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive,seoapr2020
ms.date: 04/28/2020
ms.openlocfilehash: ee989ccbb2e441256bec71781c538c7761fc7b88
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82232248"
---
# <a name="extract-transform-and-load-etl-at-scale"></a>Extrahera, transformera och läsa in (ETL) i skala

Extrahering, transformering och inläsning (ETL) är den process som data hämtas från olika källor. Samlas in på en standard plats, rensas och bearbetas. Läses slutligen in i ett data lager som det kan frågas från. Äldre ETL-processer importerar data, rengör dem på plats och lagrar dem sedan i en Relations data motor. Med HDInsight stöder en mängd olika Apache Hadoops miljö komponenter stöd för ETL i stor skala.

Användningen av HDInsight i ETL-processen kan sammanfattas av den här pipelinen:

![HDInsight ETL i skalnings översikt](./media/apache-hadoop-etl-at-scale/hdinsight-etl-at-scale-overview.png)

I följande avsnitt lär du dig var och en av de olika ETL-faserna och deras associerade komponenter.

## <a name="orchestration"></a>Orkestrering

Dirigering sträcker sig över alla faser i ETL-pipeline. ETL-jobb i HDInsight omfattar ofta flera olika produkter som arbetar tillsammans med varandra.  Du kan använda Hive för att rensa viss del av data medan gris rengör en annan del.  Du kan använda Azure Data Factory för att läsa in data i Azure SQL Database från Azure Data Lake Store.

Dirigering krävs för att köra lämpligt jobb vid lämplig tidpunkt.

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie är ett system för att koordinera arbetsflöden som hanterar Hadoop-jobb. Oozie körs i ett HDInsight-kluster och är integrerat med Hadoop-stacken. Oozie stöder Hadoop-jobb för Apache Hadoop MapReduce, Apache gris, Apache Hive och Apache Sqoop. Oozie kan också användas för att schemalägga jobb som är speciella för ett system, t. ex. Java-program eller Shell-skript.

Mer information finns i [använda Apache Oozie med Apache Hadoop för att definiera och köra ett arbets flöde i HDInsight](../hdinsight-use-oozie-linux-mac.md). Se även [operationalisera data pipeline](../hdinsight-operationalize-data-pipeline.md).

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory tillhandahåller Orchestration-funktioner i form av Platform-as-a-service. Det är en molnbaserad data integrerings tjänst som gör att du kan skapa data drivna arbets flöden i molnet. Arbets flöden för att dirigera och automatisera data förflyttning och data omvandling.

Med hjälp av Azure Data Factory kan du:

1. Skapa och schemalägga data drivna arbets flöden (kallas pipelines) som matar in data från olika data lager.
2. Bearbeta och transformera data med hjälp av beräknings tjänster som Azure HDInsight Hadoop. Eller Spark, Azure Data Lake Analytics, Azure Batch och Azure Machine Learning.
3. Publicera utdata till datalager som Azure SQL Data Warehouse och som sedan kan användas av Business Intelligence-program (BI).

Mer information om Azure Data Factory finns i [dokumentationen](../../data-factory/introduction.md).

## <a name="ingest-file-storage-and-result-storage"></a>Hämta fil lagring och resultat lagring

Källfiler för data läses vanligt vis in på en plats i Azure Storage eller Azure Data Lake Storage. Filerna kan vara i valfritt format, men är vanligt vis enkla filer som CSV: er.

### <a name="azure-storage"></a>Azure Storage

Azure Storage har speciella anpassnings mål. Se [skalbarhets-och prestanda mål för Blob Storage](../../storage/blobs/scalability-targets.md). För de flesta analytiska noder skalas Azure Storage bäst vid hantering av många mindre filer.  Azure Storage garanterar samma prestanda, oavsett hur stora filerna (så länge du befinner sig inom gränserna).  Den här garantin innebär att du kan lagra terabyte och fortfarande få konsekventa prestanda, oavsett om du använder en delmängd av data eller data.

Azure Storage har flera olika typer av blobbar.  En *tilläggs-BLOB* är ett bra alternativ för att lagra webb loggar eller sensor data.  

Flera blobbar kan distribueras på många servrar för att skala ut åtkomsten till dem. Men en enda BLOB kan bara hanteras av en enda server. Blobar kan grupperas logiskt i BLOB-behållare, men det finns inga konsekvenser för partitionering från den här grupperingen.

Azure Storage har också ett WebHDFS API-lager för Blob Storage.  Alla tjänster i HDInsight kan komma åt filer i Azure Blob Storage för data rensning och data bearbetning. På liknande sätt som de här tjänsterna använder Hadoop-distribuerade fil system (HDFS).

Data matas vanligt vis in i Azure Storage antingen med PowerShell, Azure Storage SDK eller AZCopy.

### <a name="azure-data-lake-storage"></a>Azure Data Lake Storage

Azure Data Lake Storage (ADLS) är en hanterad, storskalig lagrings plats. En lagrings plats för analys data som är kompatibla med HDFS.  ADLS använder ett design paradigm som liknar HDFS. ADLS erbjuder obegränsad anpassning för total kapacitet och storleken på enskilda filer. ADLS är användbart när du arbetar med stora filer, eftersom en stor fil kan lagras på flera noder.  Partitionering av data i ADLS görs i bakgrunden.  Du får enorma data flöden för att köra analys jobb med tusentals samtidiga körningar som effektivt läser och skriver hundratals terabyte data.

Data matas vanligt vis in i ADLS med hjälp av Azure Data Factory. Eller ADLS SDK: er, AdlCopy service, Apache DistCp eller Apache Sqoop.  Vilken av dessa tjänster som ska användas i stor utsträckning beror på var datan finns.  Om data för närvarande finns i ett befintligt Hadoop-kluster kan du använda Apache DistCp, AdlCopy service eller Azure Data Factory.  För data i Azure Blob Storage kan du använda Azure Data Lake Storage .NET SDK, Azure PowerShell eller Azure Data Factory.

ADLS är också optimerad för händelse inmatning med Azure Event Hub eller Apache Storm.

#### <a name="considerations-for-both-storage-options"></a>Att tänka på för båda lagrings alternativen

För att ladda upp data uppsättningar i terabyte-intervallet kan nätverks fördröjningen vara ett större problem, särskilt om data kommer från en lokal plats.  I sådana fall kan du använda alternativen nedan:

* Azure ExpressRoute: med Azure ExpressRoute kan du skapa privata anslutningar mellan Azure-datacenter och din lokala infrastruktur. Dessa anslutningar ger ett tillförlitligt alternativ för överföring av stora mängder data. Mer information finns i [dokumentationen om Azure ExpressRoute](../../expressroute/expressroute-introduction.md).

* "Frånkopplad" överföring av data. Du kan använda [Azure import/export-tjänsten](../../storage/common/storage-import-export-service.md) för att leverera hård diskar med dina data till ett Azure-datacenter. Dina data laddas först upp till Azure Storage blobbar. Du kan sedan använda Azure Data Factory eller AdlCopy-verktyget för att kopiera data från Azure Storage blobbar till Data Lake Storage.

### <a name="azure-sql-data-warehouse"></a>Azure SQL Data Warehouse

Azure SQL DW är ett bra alternativ för att lagra för beredda resultat.  Azure HDInsight kan användas för att utföra dessa tjänster för Azure SQL DW.

Azure SQL Data Warehouse (SQL DW) är ett Relations databas lager som är optimerat för analytiska arbets belastningar.  Azure SQL DW-skalningar baserade på partitionerade tabeller.  Tabeller kan partitioneras över flera noder.  Azure SQL DW-noder väljs vid tidpunkten för skapandet.  De kan skalas efter faktumet, men det är en aktiv process som kan kräva data förflyttning. Mer information finns i [SQL Data Warehouse-Manage Compute](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md).

### <a name="apache-hbase"></a>Apache HBase

Apache HBase är ett nyckel värdes lager som är tillgängligt i Azure HDInsight.  Apache HBase är en NoSQL-databas med öppen källkod som har skapats på Hadoop och modellerats efter Google BigTable. HBase tillhandahåller slumpmässig åtkomst och stark konsekvens för stora mängder ostrukturerade och semistrukturerade data. Data i en schema lös databas som organiseras av kolumn familjer.

Data lagras i tabellens rader och data i raderna grupperas per kolumnfamilj. HBase är en schema lös databas. De kolumner och data typer som lagras i dem behöver inte definieras innan de används. Den öppna källkoden skalas linjärt för att hantera petabyte med data på tusentals noder. HBase kan förlita sig på dataredundans, batchbearbetning och andra funktioner som tillhandahålls av distribuerade program i Hadoop-miljön.

HBase är ett utmärkt mål för sensor-och loggdata för framtida analys.

HBase anpassning är beroende av antalet noder i HDInsight-klustret.

### <a name="azure-sql-database-and-azure-database"></a>Azure SQL Database och Azure Database

Azure erbjuder tre olika Relations databaser som PAAS (Platform-as-a-Service).

* [Azure SQL Database](../../sql-database/sql-database-technical-overview.md) är en implementering av Microsoft SQL Server. Mer information om prestanda finns [i justera prestanda i Azure SQL Database](../../sql-database/sql-database-performance-guidance.md).
* [Azure Database for MySQL](../../mysql/overview.md) är en implementering av Oracle MySQL.
* [Azure Database for PostgreSQL](../../postgresql/quickstart-create-server-database-portal.md) är en implementering av postgresql.

Dessa produkter skalas upp, vilket innebär att de skalas genom att lägga till mer processor och minne.  Du kan också välja att använda Premium diskar med produkterna för bättre I/O-prestanda.

## <a name="azure-analysis-services"></a>Azure Analysis Services

Azure Analysis Services (AAS) är en analys data motor som används i besluts support och affärs analys. AAS tillhandahåller analys data för företags rapporter och klient program som Power BI. Excel, repor ting Services-rapporter och andra verktyg för data visualisering.

Analysis-kuber kan skalas genom att ändra nivåer för varje enskild kub.  Mer information finns i [Azure Analysis Services prissättning](https://azure.microsoft.com/pricing/details/analysis-services/).

## <a name="extract-and-load"></a>Extrahera och Läs in

När data finns i Azure kan du använda många tjänster för att extrahera och läsa in dem i andra produkter.  HDInsight stöder Sqoop och FLUME.

### <a name="apache-sqoop"></a>Apache Sqoop

Apache Sqoop är ett verktyg som utformats för att effektivt överföra data mellan strukturerade, halv strukturerade och ostrukturerade data källor.

Sqoop använder MapReduce för att importera och exportera data för att tillhandahålla parallell drift och fel tolerans.

### <a name="apache-flume"></a>Apache FLUME

`Apache Flume`är en distribuerad, tillförlitlig och tillgänglig tjänst för att effektivt samla in, aggregera och flytta stora mängder logg data. FLUME har en flexibel arkitektur baserad på strömmande data flöden. FLUME är robust och feltolerant med justerbara Tillförlitlighets metoder och många funktioner för redundans och återställning. FLUME använder en enkel utöknings bar data modell som möjliggör online analys program.

Apache FLUME kan inte användas med Azure HDInsight.  En lokal Hadoop-installation kan använda FLUME för att skicka data till antingen Azure Storage blobbar eller Azure Data Lake Storage.  Mer information finns i [using Apache FLUME med HDInsight](https://web.archive.org/web/20190217104751/https://blogs.msdn.microsoft.com/bigdatasupport/2014/03/18/using-apache-flume-with-hdinsight/).

## <a name="transform"></a>Transformering

När data finns på den valda platsen måste du rensa den, kombinera den eller förbereda den för ett särskilt användnings mönster.  Hive, gris och Spark SQL är alla lämpliga alternativ för den typen av arbete.  De stöds i HDInsight.

## <a name="next-steps"></a>Nästa steg

* [Använda Apache Hive som ETL-verktyg](apache-hadoop-using-apache-hive-as-an-etl-tool.md)
* [Använda Azure Data Lake Storage Gen2 med Azure HDInsight-kluster](../hdinsight-hadoop-use-data-lake-storage-gen2.md)
* [Flytta data från Azure SQL Database till Apache Hive tabell](./apache-hadoop-use-sqoop-mac-linux.md)
