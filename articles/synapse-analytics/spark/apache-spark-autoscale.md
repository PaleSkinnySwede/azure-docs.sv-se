---
title: Skala automatiskt Azure-Synapse Apache Spark instanser
description: Använd funktionen för automatisk skalning i Azure-Synapse för att automatiskt skala Apache Spark instanser
author: euangMS
ms.author: euang
ms.reviewer: euang
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: a2f907384326aa887c12c293feb8c988f42bbaf1
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2020
ms.locfileid: "83210519"
---
# <a name="automatically-scale-azure-synapse-analytics-apache-spark-pools"></a>Skala automatiskt Apache Spark pooler i Azure Synapse Analytics

Funktionen för automatisk skalning i Azure Synapse Spark-poolen skalar automatiskt antalet noder i en kluster instans upp och ned. När du skapar en ny Azure Synapse Spark-pool kan du ange ett minsta och högsta antal noder när autoskalning är markerat. Autoskalning övervakar sedan resurs kraven för belastningen och skalar antalet noder uppåt eller nedåt. Den här funktionen kostar inget extra.

## <a name="metrics-monitoring"></a>Mått övervakning

Autoskalning övervakar kontinuerligt instansen av Spark-instansen och samlar in följande mått:

|Mått|Beskrivning|
|---|---|
|Total väntande processor|Det totala antalet kärnor som krävs för att starta körningen av alla väntande noder.|
|Totalt väntande minne|Det totala minne (i MB) som krävs för att starta körningen av alla väntande noder.|
|Total ledig CPU|Summan av alla oanvända kärnor på aktiva noder.|
|Totalt ledigt minne|Summan av oanvänt minne (i MB) på de aktiva noderna.|
|Använt minne per nod|Belastningen på en nod. En nod där 10 GB minne används, anses vara under mer belastning än en anställd med 2 GB använt minne.|

Ovanstående mått kontrol leras var 30: e sekund. Med autoskalning kan du skala upp och skala ned beslut baserat på dessa mått.

## <a name="load-based-scale-conditions"></a>Belastnings beroende skalnings villkor

När följande villkor upptäcks kommer autoskalning att utfärda en skalnings förfrågan:

|Skala upp|Skala ned|
|---|---|
|Den totala väntande processorn är större än den totala kostnads fria processorn i mer än 1 minut.|Den totala väntande processorn är mindre än den totala kostnads fria processorn i mer än två minuter.|
|Totalt minne som väntar är större än det totala lediga minnet i mer än 1 minut.|Totalt minne som väntar är mindre än det totala lediga minnet i mer än två minuter.|

För att skala upp beräknar Azure Synapse AutoScale service hur många nya noder som behövs för att uppfylla de aktuella processor-och minnes kraven, och skickar sedan en skalbar begäran om att lägga till antalet noder som krävs.

För nedskalning, baserat på antalet körningar, program hanterare per nod och aktuella processor-och minnes krav, kan autoskalning utfärda en begäran om att ta bort ett visst antal noder. Tjänsten identifierar också vilka noder som ska tas bort, baserat på den aktuella jobb körningen. Åtgärden för att skala ned inaktiverar först noderna och tar sedan bort dem från klustret.

## <a name="get-started"></a>Kom igång

### <a name="create-a-spark-pool-with-autoscaling"></a>Skapa en spark-pool med automatisk skalning

Aktivera funktionen för autoskalning genom att utföra följande steg som en del av processen för att skapa en normal pool:

1. På fliken **grundläggande** markerar du kryss rutan **Aktivera autoskalning** .
1. Ange önskade värden för följande egenskaper:  

    * **Minsta** antal noder.
    * **Maximalt** antal noder.

Det första antalet noder är minimivärdet. Det här värdet definierar den ursprungliga storleken på instansen när den skapas. Det minsta antalet noder får inte vara mindre än tre.

## <a name="best-practices"></a>Bästa praxis

### <a name="consider-the-latency-of-scale-up-or-scale-down-operations"></a>Beakta svars tiden för skalning upp-eller nedskalning

Det kan ta 1 till fem minuter innan skalnings åtgärden slutförs.

### <a name="preparation-for-scaling-down"></a>Förberedelse för skalning

Under processen vid skalnings processen för instanser placeras noderna i inaktive rings läge automatiskt så att inga nya körningar kan starta på noden.

De jobb som körs fortsätter att köras och avslutas. Väntande jobb väntar på att schemaläggas som normalt med färre tillgängliga noder.

## <a name="next-steps"></a>Nästa steg

Snabb start för att konfigurera en ny Spark-pool [skapa en spark-pool](..\quickstart-create-apache-spark-pool.md)
