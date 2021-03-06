---
title: Utforma ELT i stället för ETL
description: Implementera flexibla data inläsnings strategier för Synapse SQL-pool i Azure Synapse Analytics
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 02/19/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: e99fd898956e11a4827d023691111a47e5a790c0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80744953"
---
# <a name="data-loading-strategies-for-synapse-sql-pool"></a>Data inläsnings strategier för Synapse SQL-pool

Traditionella SMP SQL-pooler använder en process för extrahering, transformering och inläsning (ETL) för att läsa in data. Synapse SQL-pool i Azure Synapse Analytics har en arkitektur för massivt parallell bearbetning (MPP) som drar nytta av skalbarheten och flexibiliteten i beräknings-och lagrings resurser.

Om du använder en process för att extrahera, läsa in och transformera (ELT) används MPP och eliminerar de resurser som krävs för dataomvandling innan inläsningen.

SQL-poolen har stöd för många inläsnings metoder, inklusive populära SQL Server alternativ som [BCP](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) och [SqlBulkCopy-API](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json), och det snabbaste och mest skalbara sättet att läsa in data är genom PolyBase externa tabeller och [kopierings instruktionen](/sql/t-sql/statements/copy-into-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) (för hands version).

Med PolyBase och KOPIERINGs instruktionen kan du komma åt externa data som lagras i Azure Blob Storage eller Azure Data Lake Store via T-SQL-språket. Vi rekommenderar att du använder COPY-instruktionen för största möjliga flexibilitet vid inläsning.

> [!NOTE]  
> KOPIERINGs instruktionen är för närvarande en offentlig för hands version. Skicka e-post till följande distributions lista för att ge sqldwcopypreview@service.microsoft.comfeedback:.

> [!VIDEO https://www.youtube.com/embed/l9-wP7OdhDk]

## <a name="what-is-elt"></a>Vad är ELT?

Extrahera, läsa in och transformera (ELT) är en process genom vilken data extraheras från ett käll system, läses in i en SQL-pool och omvandlas sedan.

De grundläggande stegen för att implementera ELT är:

1. Extrahera källdata i textfiler.
2. Landa data i Azure Blob Storage eller Azure Data Lake Store.
3. Förbered data för inläsning.
4. Läs in data i mellanlagrings tabeller med PolyBase eller kommandot COPY.
5. Transformera data.
6. Infoga data i produktionstabeller.

En inläsnings kurs för PolyBase finns i [använda PolyBase för att läsa in data från Azure Blob Storage](load-data-from-azure-blob-storage-using-polybase.md).

Mer information finns i [blogg om inläsnings mönster](https://blogs.msdn.microsoft.com/sqlcat/20../../azure-sql-data-warehouse-loading-patterns-and-strategies/).

## <a name="1-extract-the-source-data-into-text-files"></a>1. extrahera källdata i textfiler

Att hämta data från käll systemet beror på lagrings platsen.  Målet är att flytta data till PolyBase och KOPIAn som stöder avgränsade text-eller CSV-filer.

### <a name="polybase-and-copy-external-file-formats"></a>PolyBase och kopiera externa fil format

Med PolyBase och KOPIERINGs instruktionen kan du läsa in data från UTF-8-och UTF-16-kodade avgränsade text-eller CSV-filer. Förutom avgränsade text-eller CSV-filer läses de in från Hadoop-filformat som ORC och Parquet. PolyBase och KOPIERINGs instruktionen kan också läsa in data från gzip och fästa komprimerade filer.

Utökad ASCII, fast bredds format och kapslade format som WinZip eller XML stöds inte. Om du exporterar från SQL Server kan du använda [kommando rads verktyget BCP](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) för att exportera data till avgränsade textfiler.

## <a name="2-land-the-data-into-azure-blob-storage-or-azure-data-lake-store"></a>2. landa data i Azure Blob Storage eller Azure Data Lake Store

Om du vill landa data i Azure Storage kan du flytta dem till [Azure Blob Storage](../../storage/blobs/storage-blobs-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) eller [Azure Data Lake Store Gen2](../../data-lake-store/data-lake-store-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). På någon av platserna ska data lagras i textfiler. PolyBase och KOPIERINGs instruktionen kan läsas in från vilken plats som helst.

Verktyg och tjänster som du kan använda för att flytta data till Azure Storage:

- [Azure ExpressRoute](../../expressroute/expressroute-introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) service förbättrar nätverks data flöde, prestanda och förutsägbarhet. ExpressRoute är en tjänst som dirigerar dina data via en dedikerad privat anslutning till Azure. ExpressRoute-anslutningar dirigerar inte data via det offentliga Internet. Anslutningarna ger högre tillförlitlighet, snabbare hastighet, lägre fördröjning och högre säkerhet än vanliga anslutningar via det offentliga Internet.
- [AzCopy-verktyget](../../storage/common/storage-choose-data-transfer-solution.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) flyttar data till Azure Storage över det offentliga Internet. Detta fungerar om data storlekarna är mindre än 10 TB. Om du vill utföra belastningen regelbundet med AZCopy testar du nätverks hastigheten för att se om den är acceptabel.
- [Azure Data Factory (ADF)](../../data-factory/introduction.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) har en gateway som du kan installera på den lokala servern. Sedan kan du skapa en pipeline för att flytta data från din lokala server upp till Azure Storage. Information om hur du använder Data Factory med SQL Analytics finns i [inläsning av data för SQL Analytics](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="3-prepare-the-data-for-loading"></a>3. Förbered data för inläsning

Du kan behöva förbereda och rensa data i ditt lagrings konto innan du läser in. Data förberedelse kan utföras medan dina data är i källan, när du exporterar data till textfiler, eller när data har Azure Storage.  Det är enklast att arbeta med data så tidigt i processen som möjligt.  

### <a name="define-external-tables"></a>Definiera externa tabeller

Om du använder PolyBase måste du definiera externa tabeller i SQL-poolen innan du läser in. Externa tabeller krävs inte av KOPIERINGs instruktionen. PolyBase använder externa tabeller för att definiera och komma åt data i Azure Storage.

En extern tabell liknar en Database-vy. Den externa tabellen innehåller tabellens schema och pekar på data som lagras utanför SQL-poolen.

Definiera externa tabeller innebär att du anger data källan, formatet på textfilerna och tabell definitionerna. Referens artiklar för T-SQL-syntax som du behöver:

- [SKAPA EXTERN DATA KÄLLA](/sql/t-sql/statements/create-external-data-source-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [CREATE EXTERNAL FILE FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
- [SKAPA EXTERN TABELL](/sql/t-sql/statements/create-external-table-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)

Vid inläsning av Parquet är SQL Data Type-mappningen:

| **Data typen Parquet** | **SQL-datatyp** |
| :-------------------: | :---------------: |
|        tinyint        |      tinyint      |
|       smallint        |     smallint      |
|          int          |        int        |
|        bigint         |      bigint       |
|        boolean        |        bit        |
|        double         |       float       |
|         float         |       real        |
|        double         |       money       |
|        double         |    smallmoney     |
|        sträng         |       nchar       |
|        sträng         |     nvarchar      |
|        sträng         |       char        |
|        sträng         |      varchar      |
|        binary         |      binary       |
|        binary         |     varbinary     |
|       timestamp       |       date        |
|       timestamp       |   smalldatetime   |
|       timestamp       |     datetime2     |
|       timestamp       |     datetime      |
|       timestamp       |       time        |
|         date          |       date        |
|        decimal        |      decimal      |

Ett exempel på hur du skapar externa objekt finns i steget [skapa externa tabeller](load-data-from-azure-blob-storage-using-polybase.md#create-external-tables-for-the-sample-data) i inläsnings kursen.

### <a name="format-text-files"></a>Formatera textfiler

Om du använder PolyBase måste de externa objekt som definieras justera raderna i textfilerna med den externa tabell-och fil formats definitionen. Data i varje rad i text filen måste överensstämma med tabell definitionen.
Formatera textfilerna:

- Om dina data kommer från en icke-relationell källa måste du omvandla den till rader och kolumner. Oavsett om data kommer från en Relations källa eller icke-relationell källa måste data omvandlas för att anpassas efter kolumn definitionerna för den tabell som du planerar att läsa in data i.
- Formatera data i text filen så att de överensstämmer med kolumnerna och data typerna i mål tabellen. Fel justering mellan data typer i externa textfiler och tabellen SQL-pool gör att rader avvisas under inläsningen.
- Separera fält i text filen med en avslutning.  Se till att du använder ett Character eller en teckensekvens som inte finns i dina källdata. Använd den avslutnings fil som du angav med [Skapa externt fil format](/sql/t-sql/statements/create-external-file-format-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="4-load-the-data-using-polybase-or-the-copy-statement"></a>4. Läs in data med PolyBase eller kopiera instruktionen

Vi rekommenderar att du läser in data i en mellanlagringsplats. Med mellanlagrings tabeller kan du hantera fel utan att störa produktions tabellerna. Med en mellanlagringsplats får du också möjlighet att använda SQL-poolens MPP för data transformationer innan du infogar data i produktions tabeller.

Tabellen måste skapas i förväg när den läses in i en mellanlagringsplats med COPY.

### <a name="options-for-loading-with-polybase-and-copy-statement"></a>Alternativ för att läsa in med PolyBase-och COPY-instruktion

Om du vill läsa in data med PolyBase kan du använda något av följande inläsnings alternativ:

- [PolyBase med T-SQL](load-data-from-azure-blob-storage-using-polybase.md) fungerar bra när dina data finns i Azure Blob storage eller Azure Data Lake Store. Du får störst kontroll över inläsnings processen, men du måste också definiera externa data objekt. De andra metoderna definierar dessa objekt bakom scenerna när du mappar käll tabeller till mål tabeller.  För att dirigera T-SQL-inläsningar kan du använda Azure Data Factory, SSIS eller Azure Functions.
- [PolyBase med SSIS](/sql/integration-services/load-data-to-sql-data-warehouse?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) fungerar bra när dina källdata är i SQL Server, antingen SQL Server lokalt eller i molnet. SSIS definierar källa till mål tabell mappningar och dirigerar även belastningen. Om du redan har SSIS-paket kan du ändra paketen så att de fungerar med det nya informations lager målet.
- [PolyBase-och Copy-uttryck med Azure Data Factory (ADF)](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) är ett annat Orchestration-verktyg.  Den definierar en pipeline och schemalägger jobb.
- [PolyBase med Azure Databricks](../../azure-databricks/databricks-extract-load-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) överför data från en tabell till en Databricks-dataframe och/eller skriver data från en Databricks-dataframe till en tabell med PolyBase.

### <a name="other-loading-options"></a>Andra inläsnings alternativ

Förutom PolyBase och KOPIERINGs instruktionen kan du använda [BCP](/sql/tools/bcp-utility?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest) -eller SqlBulkCopy- [API: et](/dotnet/api/system.data.sqlclient.sqlbulkcopy?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). BCP läser in direkt till databasen utan att gå via Azure Blob Storage och är endast avsedd för små belastningar.

> [!NOTE]
> Belastnings prestandan för de här alternativen är långsammare än PolyBase och KOPIERINGs instruktionen.

## <a name="5-transform-the-data"></a>5. transformera data

När data finns i mellanlagringsplatsen ska du utföra omvandlingar som din arbets belastning kräver. Flytta sedan data till en produktions tabell.

## <a name="6-insert-the-data-into-production-tables"></a>6. infoga data i produktions tabeller

INSERT INTO... SELECT-instruktionen flyttar data från mellanlagrings tabellen till den permanenta tabellen.

När du utformar en ETL-process kan du prova att köra processen på ett litet test exempel. Försök att extrahera 1000 rader från tabellen till en fil, flytta den till Azure och försök sedan att läsa in den i en mellanlagringsplats.

## <a name="partner-loading-solutions"></a>Lösningar för partner inläsning

Många av våra partner har inläsnings lösningar. Om du vill veta mer kan du se en lista över våra [lösnings partner](sql-data-warehouse-partner-business-intelligence.md).

## <a name="next-steps"></a>Nästa steg

Hjälp med inläsningen hittar du i [Vägledning för att läsa in data](guidance-for-loading-data.md).
