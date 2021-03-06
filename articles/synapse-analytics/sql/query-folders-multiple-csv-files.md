---
title: Fråga mappar och flera CSV-filer med SQL på begäran (för hands version)
description: SQL på begäran (för hands version) stöder läsning av flera filer/mappar med jokertecken, som liknar de jokertecken som används i Windows OS.
services: synapse analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: ''
ms.date: 04/15/2020
ms.author: v-stazar
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 8f8af7fab7113e38b91c3f5f1bcc41b4e4fba2c1
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81457373"
---
# <a name="query-folders-and-multiple-csv-files"></a>Fråga mappar och flera CSV-filer  

I den här artikeln får du lära dig hur du skriver en fråga med SQL på begäran (för hands version) i Azure Synapse Analytics.

SQL på begäran stöder läsning av flera filer/mappar med jokertecken som liknar de jokertecken som används i Windows OS. Det finns dock större flexibilitet eftersom flera jokertecken är tillåtna.

## <a name="prerequisites"></a>Krav

Innan du läser resten av den här artikeln ser du till att granska artiklarna i listan nedan:

- [Installation vid första tiden](query-data-storage.md#first-time-setup)
- [Förutsättningar](query-data-storage.md#prerequisites)

## <a name="read-multiple-files-in-folder"></a>Läsa flera filer i mappen

Du använder mappen *CSV/taxi* för att följa exempel frågorna. Den innehåller NYC taxi-Yellow taxi-resa registrerar data från 2016 juli till 2018 juni.

Filerna i *CSV/taxi* har namnet efter år och månad:

- yellow_tripdata_2016 -07. csv
- yellow_tripdata_2016 -08. csv
- yellow_tripdata_2016 -09. csv
- ...
- yellow_tripdata_2018 -04. csv
- yellow_tripdata_2018 -05. csv
- yellow_tripdata_2018 -06. csv

Varje fil har följande struktur:
        
    [First 10 rows of the CSV file](./media/querying-folders-and-multiple-csv-files/nyc-taxi.png)

## <a name="read-all-files-in-folder"></a>Läsa alla filer i mappen
    
Exemplet nedan läser alla NYC gula taxi-datafiler från *CSV/taxi-* mappen och returnerar det totala antalet passagerare och sina val per år. Den visar också användningen av mängd funktioner.

```sql
SELECT 
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
    BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/taxi/*.*',
        FORMAT = 'CSV', 
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
           trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
           payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Alla filer som används med den enda OpenRowSet-rad uppsättningen måste ha samma struktur (d.v.s. antalet kolumner och deras data typer).

### <a name="read-subset-of-files-in-folder"></a>Läs delmängd av filer i mappen

Exemplet nedan läser de 2017 NYC gula taxi-datafilerna från *CSV/taxi-* mappen med ett jokertecken och returnerar det totala avgifts beloppet per betalnings typ.

```sql
SELECT 
    payment_type,  
    SUM(fare_amount) AS fare_total
FROM OPENROWSET(
    BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/taxi/yellow_tripdata_2017-*.csv',
        FORMAT = 'CSV', 
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY payment_type
ORDER BY payment_type;
```

> [!NOTE]
> Alla filer som används med den enda OpenRowSet-rad uppsättningen måste ha samma struktur (d.v.s. antalet kolumner och deras data typer).

## <a name="read-folders"></a>Läsa mappar

Den sökväg som du anger till OpenRowSet kan också vara en sökväg till en mapp. Följande avsnitt innehåller dessa frågetyper.

### <a name="read-all-files-from-specific-folder"></a>Läs alla filer från en speciell mapp

Du kan läsa alla filer i en mapp med hjälp av jokertecken på filnivå som visas i [mappen Läs alla filer i mappen](#read-all-files-in-folder). Men det finns ett sätt att fråga en mapp och använda alla filer i mappen.

Om sökvägen i OpenRowSet pekar på en mapp, kommer alla filer i mappen att användas som källa för frågan. Följande fråga läser alla filer i mappen *CSV/taxi* .

> [!NOTE]
> Observera att det finns en/i slutet av sökvägen i frågan nedan. Den anger en mapp. Om/utelämnas, kommer frågan att rikta en fil med namnet *taxi* i stället.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
    BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/taxi/',
        FORMAT = 'CSV', 
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Alla filer som används med den enda OpenRowSet-rad uppsättningen måste ha samma struktur (d.v.s. antalet kolumner och deras data typer).

### <a name="read-all-files-from-multiple-folders"></a>Läs alla filer från flera mappar

Det går att läsa filer från flera mappar med hjälp av ett jokertecken. Följande fråga läser alla filer från alla mappar som finns i *CSV* -mappen med namn som börjar med *t* och slutar med *i*.

> [!NOTE]
> Observera att det finns en/i slutet av sökvägen i frågan nedan. Den anger en mapp. Om/utelämnas, kommer frågan att skicka filer med namnet *t&ast;i* stället.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
    BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/t*i/', 
        FORMAT = 'CSV', 
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Alla filer som används med den enda OpenRowSet-rad uppsättningen måste ha samma struktur (d.v.s. antalet kolumner och deras data typer).

Eftersom du bara har en mapp som matchar villkoren är frågeresultatet detsamma som [i mappen läsa alla filer i mappen](#read-all-files-in-folder).

## <a name="multiple-wildcards"></a>Flera jokertecken

Du kan använda flera jokertecken på olika Sök vägs nivåer. Du kan till exempel utöka tidigare fråga till att endast läsa filer med 2017-data från alla mappar som namnen börjar med *t* och slutar med *i*.

> [!NOTE]
> Observera att det finns en/i slutet av sökvägen i frågan nedan. Den anger en mapp. Om/utelämnas, kommer frågan att skicka filer med namnet *t&ast;i* stället.
> Det finns en övre gräns på 10 jokertecken per fråga.

```sql
SELECT
    YEAR(pickup_datetime) as [year],
    SUM(passenger_count) AS passengers_total,
    COUNT(*) AS [rides_total]
FROM OPENROWSET(
    BULK 'https://sqlondemandstorage.blob.core.windows.net/csv/t*i/yellow_tripdata_2017-*.csv',
        FORMAT = 'CSV', 
        FIRSTROW = 2
    )
    WITH (
        vendor_id VARCHAR(100) COLLATE Latin1_General_BIN2, 
        pickup_datetime DATETIME2, 
        dropoff_datetime DATETIME2,
        passenger_count INT,
        trip_distance FLOAT,
        rate_code INT,
        store_and_fwd_flag VARCHAR(100) COLLATE Latin1_General_BIN2,
        pickup_location_id INT,
        dropoff_location_id INT,
        payment_type INT,
        fare_amount FLOAT,
        extra FLOAT,
        mta_tax FLOAT,
        tip_amount FLOAT,
        tolls_amount FLOAT,
        improvement_surcharge FLOAT,
        total_amount FLOAT
    ) AS nyc
GROUP BY
    YEAR(pickup_datetime)
ORDER BY
    YEAR(pickup_datetime);
```

> [!NOTE]
> Alla filer som används med den enda OpenRowSet-rad uppsättningen måste ha samma struktur (d.v.s. antalet kolumner och deras data typer).

Eftersom du bara har en mapp som matchar villkoren, är frågeresultatet detsamma som att [läsa delmängd av filer i mappen](#read-subset-of-files-in-folder) och [läsa alla filer från en viss mapp](#read-all-files-from-specific-folder). Mer komplexa scenarier för användning av jokertecken beskrivs i [fråga Parquet-filer](query-parquet-files.md).

## <a name="next-steps"></a>Nästa steg

Mer information finns i artikeln om [speciella filer i frågor](query-specific-files.md) .
