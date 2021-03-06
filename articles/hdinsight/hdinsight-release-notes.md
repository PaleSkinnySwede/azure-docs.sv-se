---
title: Viktig information om Azure HDInsight
description: Senaste viktig information för Azure HDInsight. Få utvecklings tips och information för Hadoop, Spark, R Server, Hive med mera.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: 458925dd9f7f7386a9159256fdb024d027f7016c
ms.sourcegitcommit: a6d477eb3cb9faebb15ed1bf7334ed0611c72053
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2020
ms.locfileid: "82929323"
---
# <a name="release-notes"></a>Viktig information

Den här artikeln innehåller information om de **senaste** versionerna av Azure HDInsight-utgåvor. Information om tidigare versioner finns i avsnittet om [versions anmärkningar för HDInsight](hdinsight-release-notes-archive.md).

## <a name="summary"></a>Sammanfattning

Azure HDInsight är en av de populäraste tjänsterna mellan företags kunder för analys med öppen källkod på Azure.

## <a name="release-date-01092020"></a>Utgivnings datum: 01/09/2020

Den här versionen gäller både för HDInsight 3,6 och 4,0. HDInsight-versionen görs tillgänglig för alla regioner över flera dagar. Lanserings datumet här anger den första regionens utgivnings datum. Om du inte ser ändringarna nedan väntar du tills lanseringen är aktiv i din region under flera dagar.

> [!IMPORTANT]  
> Linux är det enda operativsystemet som används med HDInsight version 3.4 och senare. Mer information finns i [artikeln om versions hantering i HDInsight](hdinsight-component-versioning.md).

## <a name="new-features"></a>Nya funktioner
### <a name="tls-12-enforcement"></a>Tvingande TLS 1.2
Transport Layer Security (TLS) och Secure Sockets Layer (SSL) är krypterings protokoll som ger kommunikations säkerhet i ett dator nätverk. Läs mer om [TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security#SSL_1.0.2C_2.0_and_3.0). HDInsight använder TLS 1,2 på offentliga HTTPs-slutpunkter, men TLS 1,1 stöds fortfarande för bakåtkompatibilitet. 

I den här versionen kan kunder bara välja TLS 1,2 för alla anslutningar via den offentliga kluster slut punkten. För att stödja detta introduceras den nya egenskapen **minSupportedTlsVersion** och kan anges när klustret skapas. Om egenskapen inte har angetts har klustret fortfarande stöd för TLS 1,0, 1,1 och 1,2, som är samma som dagens beteende. Kunder kan ange värdet för den här egenskapen till "1,2", vilket innebär att klustret bara har stöd för TLS 1,2 och senare. Mer information finns i [Transport Layer Security](./transport-layer-security.md).

### <a name="bring-your-own-key-for-disk-encryption"></a>Ta med din egen nyckel för disk kryptering
Alla hanterade diskar i HDInsight skyddas med Azure Storage Service Encryption (SSE). Data på diskarna krypteras av Microsoft-hanterade nycklar som standard. Från och med den här versionen kan du Bring Your Own Key (BYOK) för disk kryptering och hantera den med hjälp av Azure Key Vault. BYOK-kryptering är en konfiguration i ett steg när klustret skapas utan extra kostnad. Registrera bara HDInsight som en hanterad identitet med Azure Key Vault och Lägg till krypterings nyckeln när du skapar klustret. Mer information finns i [kund hanterad nyckel disk kryptering](https://docs.microsoft.com/azure/hdinsight/disk-encryption).

## <a name="deprecation"></a>Utfasning
Inga utfasningar för den här versionen. För att komma igång med kommande inaktuella ändringar, se [kommande ändringar](#upcoming-changes).

## <a name="behavior-changes"></a>Beteende ändringar
Inga beteende ändringar för den här versionen. För att komma igång med kommande ändringar, se [kommande ändringar](#upcoming-changes).

## <a name="upcoming-changes"></a>Kommande ändringar
Följande ändringar sker i kommande versioner. 

### <a name="deprecate-spark-21-and-22-in-hdinsight-36-spark-cluster"></a>Föråldra Spark 2,1 och 2,2 i HDInsight 3,6 Spark-kluster
Från och med juli 1 2020 kommer kunderna inte att kunna skapa nya Spark-kluster med Spark 2,1 och 2,2 på HDInsight 3,6. Befintliga kluster kommer att köras i befintligt skick utan support från Microsoft. Överväg att gå vidare till Spark 2,3 i HDInsight 3,6 och 30 2020 för att undvika potentiell system/support-avbrott.

### <a name="deprecate-spark-23-in-hdinsight-40-spark-cluster"></a>Inaktuell Spark 2,3 i HDInsight 4,0 Spark-kluster
Från och med juli 1 2020 kommer kunderna inte att kunna skapa nya Spark-kluster med Spark 2,3 på HDInsight 4,0. Befintliga kluster kommer att köras i befintligt skick utan support från Microsoft. Överväg att flytta till Spark 2,4 på HDInsight 4,0 senast i juni 30 2020 för att undvika system/support-avbrott.

### <a name="deprecate-kafka-11-in-hdinsight-40-kafka-cluster"></a>Föråldrad Kafka 1,1 i HDInsight 4,0 Kafka-kluster
Från och med juli 1 2020 kommer kunderna inte att kunna skapa nya Kafka-kluster med Kafka 1,1 på HDInsight 4,0. Befintliga kluster kommer att köras i befintligt skick utan support från Microsoft. Överväg att flytta till Kafka 2,1 på HDInsight 4,0 senast i juni 30 2020 för att undvika system/support-avbrott.

### <a name="hbase-20-to-216"></a>HBase 2,0 till 2.1.6
I den kommande HDInsight 4,0-versionen kommer HBase-versionen att uppgraderas från version 2,0 till 2.1.6

### <a name="spark-240-to-244"></a>Spark-2.4.0 till 2.4.4
I den kommande HDInsight 4,0-versionen kommer Spark-versionen att uppgraderas från version 2.4.0 till 2.4.4

### <a name="a-minimum-4-core-vm-is-required-for-head-node"></a>En minsta 4-kärn virtuell dator krävs för Head-noden 
En minsta 4-kärn virtuell dator krävs för Head-noden för att säkerställa hög tillgänglighet och tillförlitlighet för HDInsight-kluster. Från och med april 6 2020 kan kunder bara välja 4 kärnor eller över VM som Head-nod för de nya HDInsight-klustren. Befintliga kluster kommer att fortsätta köras som förväntat. 

### <a name="esp-spark-cluster-node-size-change"></a>Ändra storlek på ESP Spark-klusternod 
I den kommande versionen kommer den minsta tillåtna nodtypen för ESP Spark-kluster att ändras till Standard_D13_V2. Virtuella datorer i A-serien kan orsaka problem med ESP-kluster på grund av relativt låg processor-och minnes kapacitet. Virtuella datorer i A-serien är inaktuella för att skapa nya ESP-kluster.

### <a name="moving-to-azure-virtual-machine-scale-sets"></a>Flytta till skalnings uppsättningar för virtuella Azure-datorer
HDInsight använder nu virtuella Azure-datorer för att etablera klustret. I den kommande versionen används skalnings uppsättningar för virtuella Azure-datorer istället i HDInsight. Läs mer om skalnings uppsättningar för virtuella Azure-datorer.

## <a name="bug-fixes"></a>Felkorrigeringar
HDInsight fortsätter att göra kluster tillförlitlighet och prestanda förbättringar. 

## <a name="component-version-change"></a>Komponent versions ändring
Ingen komponent versions ändring för den här versionen. Du hittar de aktuella komponent versionerna för HDInsight 4,0 AD HDInsight 3,6 här.

