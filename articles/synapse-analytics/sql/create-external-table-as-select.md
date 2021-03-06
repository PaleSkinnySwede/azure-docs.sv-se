---
title: Spara frågeresultat till lagringsplats
description: I den här artikeln får du lära dig hur du lagrar frågeresultat till lagring med SQL på begäran (för hands version).
services: synapse-analytics
author: vvasic-msft
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: ''
ms.date: 04/15/2020
ms.author: vvasic
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: dd7666bb9f22214fb4701e6be9edc171912d9bf9
ms.sourcegitcommit: 366e95d58d5311ca4b62e6d0b2b47549e06a0d6d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/01/2020
ms.locfileid: "82691870"
---
# <a name="store-query-results-to-storage-using-sql-on-demand-preview-using-azure-synapse-analytics"></a>Lagra frågeresultat till lagring med SQL på begäran (för hands version) med Azure Synapse Analytics

I den här artikeln får du lära dig hur du lagrar frågeresultat till lagrings utrymmet med SQL på begäran (för hands version).

## <a name="prerequisites"></a>Krav

Ditt första steg är att granska artiklarna nedan och se till att du uppfyller kraven:

- [Installation vid första tiden](query-data-storage.md#first-time-setup)
- [Förutsättningar](query-data-storage.md#prerequisites)

## <a name="create-external-table-as-select"></a>Skapa extern tabell som Välj

Du kan använda CREATE EXTERNAL TABLE AS SELECT (CETAS)-instruktionen för att lagra frågeresultaten till lagringen.

> [!NOTE]
> Ändra den första raden i frågan, t. ex. [mydbname], så att du använder den databas som du har skapat. Om du inte har skapat en databas läser du [installations programmet för första gången](query-data-storage.md#first-time-setup). Du måste ändra plats för den externa data källan för data källan för att kunna peka på platsen som du har Skriv behörighet för. 

```sql
USE [mydbname];
GO

CREATE EXTERNAL DATA SOURCE [MyDataSource] WITH (
    LOCATION = 'https://sqlondemandstorage.blob.core.windows.net/csv'
);
GO

CREATE EXTERNAL FILE FORMAT [ParquetFF] WITH (
    FORMAT_TYPE = PARQUET,
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
);
GO

CREATE EXTERNAL TABLE [dbo].[PopulationCETAS] WITH (
        LOCATION = 'populationParquet/',
        DATA_SOURCE = [MyDataSource],
        FILE_FORMAT = [ParquetFF]
) AS
SELECT
    *
FROM
    OPENROWSET(
        BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/population-unix/population.csv',
        FORMAT='CSV'
    ) WITH (
        CountryCode varchar(4),
        CountryName varchar(64),
        Year int,
        PopulationCount int
    ) AS r;

```

## <a name="use-a-external-table-created"></a>Använd en extern tabell som skapats

Du kan använda en extern tabell som skapats via CETAS som en vanlig extern tabell.

> [!NOTE]
> Ändra den första raden i frågan, t. ex. [mydbname], så att du använder den databas som du har skapat. Om du inte har skapat en databas läser du [installations programmet för första gången](query-data-storage.md#first-time-setup).

```sql
USE [mydbname];
GO

SELECT
    country_name, population
FROM PopulationCETAS
WHERE
    [year] = 2019
ORDER BY
    [population] DESC;
```

## <a name="next-steps"></a>Nästa steg

Information om hur du frågar olika filtyper finns i artikeln [fråga en enkel CSV-fil](query-single-csv-file.md), [fråga Parquet-filer](query-parquet-files.md)och [fråga JSON-filer](query-json-files.md) .
