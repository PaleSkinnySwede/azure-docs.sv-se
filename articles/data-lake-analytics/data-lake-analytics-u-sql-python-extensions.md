---
title: Utöka U-SQL-skript med python i Azure Data Lake Analytics
description: Lär dig hur du kör python-kod i U-SQL-skript med hjälp av Azure Data Lake Analytics
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: c1c74e5e-3e4a-41ab-9e3f-e9085da1d315
ms.topic: conceptual
ms.date: 06/20/2017
ms.openlocfilehash: 0a49cbdb4caf474d0628fea3679ce712d37886e7
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "60813415"
---
# <a name="extend-u-sql-scripts-with-python-code-in-azure-data-lake-analytics"></a>Utöka U-SQL-skript med python-kod i Azure Data Lake Analytics

## <a name="prerequisites"></a>Krav

Innan du börjar ska du se till att python-tilläggen är installerade i ditt Azure Data Lake Analytics-konto.

* Navigera till Data Lake Analytics konto i Azure Portal
* På den vänstra menyn, under **komma igång** klickar du på **exempel skript**
* Klicka på **Installera U-SQL-tillägg** och sedan **OK**

## <a name="overview"></a>Översikt 

Python-tillägg för U-SQL gör det möjligt för utvecklare att utföra massivt parallell körning av python-kod. I följande exempel visas de grundläggande stegen:

* Använd `REFERENCE ASSEMBLY` instruktionen för att aktivera python-tillägg för U-SQL-skriptet
* Använda `REDUCE` åtgärden för att partitionera indata på en nyckel
* Python-tilläggen för U-SQL innehåller en inbyggd minskning (`Extension.Python.Reducer`) som kör python-kod på varje hörn som har tilldelats till minskningen
* U-SQL-skriptet innehåller den inbäddade python-kod som har en funktion `usqlml_main` som kallas för att acceptera en Pandas-DataFrame som indata och returnerar en Pandas DataFrame som utdata.

--

    REFERENCE ASSEMBLY [ExtPython];

    DECLARE @myScript = @"
    def get_mentions(tweet):
        return ';'.join( ( w[1:] for w in tweet.split() if w[0]=='@' ) )

    def usqlml_main(df):
        del df['time']
        del df['author']
        df['mentions'] = df.tweet.apply(get_mentions)
        del df['tweet']
        return df
    ";

    @t  = 
        SELECT * FROM 
           (VALUES
               ("D1","T1","A1","@foo Hello World @bar"),
               ("D2","T2","A2","@baz Hello World @beer")
           ) AS 
               D( date, time, author, tweet );

    @m  =
        REDUCE @t ON date
        PRODUCE date string, mentions string
        USING new Extension.Python.Reducer(pyScript:@myScript);

    OUTPUT @m
        TO "/tweetmentions.csv"
        USING Outputters.Csv();

## <a name="how-python-integrates-with-u-sql"></a>Hur python integreras med U-SQL

### <a name="datatypes"></a>Data typer

* Sträng och numeriska kolumner från U-SQL konverteras som-är mellan Pandas och U-SQL
* U-SQL-nullvärden konverteras till och från Pandas `NA` -värden

### <a name="schemas"></a>Scheman

* Index vektorer i Pandas stöds inte i U-SQL. Alla indata-ramar i python-funktionen har alltid ett 64-bitars numeriskt index från 0 till och med antalet rader minus 1. 
* U-SQL-datauppsättningar får inte ha dubbletter av kolumn namn
* Kolumn namn för U-SQL-datauppsättningar som inte är strängar. 

### <a name="python-versions"></a>Python-versioner
Endast python 3.5.1 (kompilerade för Windows) stöds. 

### <a name="standard-python-modules"></a>Vanliga python-moduler
Alla standard-python-moduler ingår.

### <a name="additional-python-modules"></a>Ytterligare python-moduler
Förutom standard python-biblioteken ingår flera vanliga python-bibliotek:

    pandas
    numpy
    numexpr

### <a name="exception-messages"></a>Undantags meddelanden
För närvarande visas ett undantag i python-koden som ett allmänt hörn fel. I framtiden kommer U-SQL-jobbets fel meddelanden att visa fel meddelandet python.

### <a name="input-and-output-size-limitations"></a>Storleks begränsningar för indata och utdata
Varje hörn har en begränsad mängd minne som tilldelats till den. För närvarande är gränsen 6 GB för en AU. Eftersom in-och utdata-DataFrames måste finnas i minnet i python-koden får den totala storleken för indata och utdata inte överstiga 6 GB.

## <a name="see-also"></a>Se även
* [Översikt över Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Utveckla U-SQL-skript med hjälp av Data Lake-verktyg för Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Använda U-SQL-fönstrets funktioner för Azure Data Lake Analytics jobb](data-lake-analytics-use-window-functions.md)
* [Använda Azure Data Lake Tools för Visual Studio Code](data-lake-analytics-data-lake-tools-for-vscode.md)
