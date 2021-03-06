---
title: Använda .NET för Apache Spark med Azure Synapse Analytics
description: Lär dig mer om att använda .NET och Apache Spark för batchbearbetning, real tids strömning, maskin inlärning och att skriva ad hoc-frågor i Azure Synapse Analytics-anteckningsböcker.
author: mamccrea
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: mamccrea
ms.reviewer: jrasnick
ms.openlocfilehash: 6943be033653d88b272817d2dcf58042aaaab616
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81430518"
---
# <a name="use-net-for-apache-spark-with-azure-synapse-analytics"></a>Använda .NET för Apache Spark med Azure Synapse Analytics

[.Net för Apache Spark](https://dot.net/spark) är kostnads fritt, öppen källkod och plattforms oberoende .net-support för Spark. .NET för Apache Spark tillhandahåller .NET-bindningar för Spark som gör att du kan komma åt Spark-API: er via C# och F #. Med .NET för Apache Spark har du möjlighet att skriva och köra användardefinierade funktioner för Spark med hjälp av .NET. Med .NET-API: erna för Spark kan du komma åt alla aspekter av Spark som hjälper dig att analysera dina data, inklusive Spark SQL och strukturerad strömning.

Du kan analysera data med .NET för Apache Spark genom Spark-batchjobb eller med interaktiva Azure Synapse Analytics-anteckningsböcker. I den här artikeln får du lära dig hur du använder .NET för Apache Spark med Azure Synapse med hjälp av båda metoderna. 

## <a name="submit-batch-jobs-using-the-spark-job-definition"></a>Skicka batch-jobb med hjälp av Spark-jobbets definition

Besök själv studie kursen och lär dig hur du använder Azure Synapse Analytics för att [skapa Apache Spark jobb definitioner för Synapse Spark-pooler](apache-spark-job-definitions.md). Om du inte har paketerat appen för att skicka till Azure-Synapse utför du följande steg.

1. Kör följande kommandon för att publicera din app. Se till att ersätta *mySparkApp* med sökvägen till din app.

   **I Windows:**

   ```dotnetcli
   cd mySparkApp
   dotnet publish -c Release -f netcoreapp3.0 -r ubuntu.16.04-x64
   ```

   **I Linux:**

### <a name="net-for-apache-spark-in-azure-synapse-analytics-notebooks"></a>.NET för Apache Spark i Azure Synapse Analytics-anteckningsböcker

När du skapar en ny antecknings bok väljer du en språk kärna som du vill uttrycka affärs logiken för. Det finns stöd för flera språk i kernel, inklusive C#.

Om du vill använda .NET för Apache Spark i din Azure Synapse Analytics-anteckningsbok väljer du **.net Spark (C#)** som kernel och ansluter antecknings boken till en befintlig Spark-pool.

   ```bash
   zip -r publish.zip
   ```

## <a name="net-for-apache-spark-in-azure-synapse-analytics-notebooks"></a>.NET för Apache Spark i Azure Synapse Analytics-anteckningsböcker 

Bärbara datorer är ett bra alternativ för prototypering av .NET för Apache Spark pipelines och scenarier. Du kan börja arbeta med, förstå, filtrera, Visa och visualisera dina data snabbt och effektivt. Data tekniker, data forskare, affärsanalytiker och maskin inlärnings tekniker kan samar beta över ett delat, interaktivt dokument. Du ser omedelbara resultat från data utforskning och kan visualisera dina data i samma antecknings bok.

### <a name="how-to-use-notebooks"></a>Använda antecknings böcker

När du skapar en ny antecknings bok väljer du en språk kärna som du vill uttrycka affärs logiken för. Det finns stöd för flera språk i kernel, inklusive C#. 

Om du vill använda .NET för Apache Spark i din Azure Synapse Analytics-anteckningsbok väljer du **.net Spark (C#)** som kernel och ansluter antecknings boken till en befintlig Spark-pool. 

.NET Spark-anteckningsboken baseras på de interaktiva funktionerna i .NET och ger interaktiva C#-upplevelser möjlighet att använda .NET för Spark med en spark-sessionsvariabel som `spark` redan är fördefinierad.

### <a name="sparknet-c-kernel-features"></a>Spark.NET C# kernel-funktioner

Följande funktioner är tillgängliga när du använder .NET för Apache Spark i antecknings boken för Azure Synapse Analytics:

* Deklarativ HTML: generera utdata från dina celler med HTML-syntax, till exempel sidhuvuden, punkt listor och till och med visning av bilder.
* Enkla C#-uttryck (till exempel tilldelningar, utskrift till konsol, Utlös ande undantag och så vidare).
* Kod block i flera rader i C# (till exempel IF-instruktioner, förgrunds slingor, klass definitioner och så vidare).
* Åtkomst till standard biblioteket i C# (till exempel system, LINQ, Enumerables och så vidare).
* Stöd för [språk funktionerna i C# 8,0](/dotnet/csharp/whats-new/csharp-8?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
* "Spark" som en fördefinierad variabel för att ge dig åtkomst till din Apache Spark-session.
* Stöd för [att definiera .net-användardefinierade funktioner som kan köras i Apache Spark](https://github.com/dotnet/spark/blob/master/examples/Microsoft.Spark.CSharp.Examples/Sql).
* Stöd för visualisering av utdata från Spark-jobb med olika diagram (till exempel linje, stapel eller histogram) och layouter (till exempel en enskild, överanvändning osv.) med hjälp `XPlot.Plotly` av biblioteket.
* Möjlighet att inkludera NuGet-paket i din C#-anteckningsbok.

## <a name="next-steps"></a>Nästa steg

* [Dokumentation om .NET för Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
* [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
* [.NET-interaktiv](https://devblogs.microsoft.com/dotnet/creating-interactive-net-documentation/)