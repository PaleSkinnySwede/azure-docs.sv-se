---
title: Felsöka kluster skapande fel med Azure HDInsight
description: Lär dig hur du felsöker problem med skapande av Apache-kluster för Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: troubleshooting
ms.date: 04/14/2020
ms.openlocfilehash: 14b449590f6ffc5e735faa26baadfcc4e526450c
ms.sourcegitcommit: 309a9d26f94ab775673fd4c9a0ffc6caa571f598
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/09/2020
ms.locfileid: "82996400"
---
# <a name="troubleshoot-cluster-creation-failures-with-azure-hdinsight"></a>Felsöka kluster skapande fel med Azure HDInsight

Följande problem är de vanligaste orsakerna till kluster skapande fel:

- Behörighets problem
- Begränsningar för resurs principer
- Brandväggar
- Resurslås
- Komponent versioner som inte stöds
- Begränsningar för lagrings konto namn
- Drift avbrott för tjänsten

## <a name="permissions-issues"></a>Problem med behörigheter

Om du använder Azure Data Lake Storage Gen2 och får `AmbariClusterCreationFailedErrorCode`felet: ":::no-loc text="Internal server error occurred while processing the request. Please retry the request or contact support.":::", öppna Azure Portal, gå till ditt lagrings konto och under Access Control (IAM) kontrollerar du att rollen **Storage BLOB-data deltagare** eller rollen **lagrings-BLOB-dataägare** har tilldelat åtkomst till den användare som tilldelats den **hanterade identiteten** för prenumerationen. Detaljerade anvisningar finns i [Konfigurera behörigheter för den hanterade identiteten i Data Lake Storage Gen2-kontot](../hdinsight-hadoop-use-data-lake-storage-gen2.md#set-up-permissions-for-the-managed-identity-on-the-data-lake-storage-gen2-account).

Om du använder Azure Data Lake Storage Gen1, se installations-och konfigurations anvisningar [här](../hdinsight-hadoop-use-data-lake-store.md). Data Lake Storage Gen1 stöds inte för HBase-kluster och stöds inte i HDInsight version 4,0.

Om du använder Azure Storage, se till att lagrings konto namnet är giltigt när klustret skapas.

## <a name="resource-policy-restrictions"></a>Begränsningar för resurs principer

Med prenumerations Azure-principer kan du inte skapa offentliga IP-adresser. Skapande av HDInsight-klustret kräver två offentliga IP-adresser.  

I allmänhet kan följande principer påverka skapandet av kluster:

* Principer förhindrar skapande av IP-adress & belastningsutjämnare i prenumerationen.
* Princip förhindrar skapande av lagrings konto.
* Princip som förhindrar borttagning av nätverks resurser (IP-/load-saldon).

## <a name="firewalls"></a>Brandväggar

Brand väggar i ditt virtuella nätverk eller lagrings konto kan neka kommunikation med hanterings-IP-adresser för HDInsight.

Tillåt trafik från IP-adresserna i tabellen nedan.

| Källans IP-adress | Mål | Riktning |
|---|---|---|
| 168.61.49.99 | *: 443 | Inkommande |
| 23.99.5.239 | *: 443 | Inkommande |
| 168.61.48.131 | *: 443 | Inkommande |
| 138.91.141.162 | *: 443 | Inkommande |

Lägg också till de IP-adresser som är speciella för den region där klustret skapas. Se [hanterings-IP-adresser för HDInsight](../hdinsight-management-ip-addresses.md) för en lista över adresser för varje Azure-region.

Om du använder en Express-väg eller din egen anpassade DNS-server kan du läsa [planera ett virtuellt nätverk för Azure HDInsight – ansluta flera nätverk](../hdinsight-plan-virtual-network-deployment.md#multinet).

## <a name="resources-locks"></a>Resurs lås  

Se till att det inte finns några [Lås på det virtuella nätverket och resurs gruppen](../../azure-resource-manager/management/lock-resources.md). Det går inte att skapa eller ta bort kluster om resurs gruppen är låst. 

## <a name="unsupported-component-versions"></a>Komponent versioner som inte stöds

Se till att du använder en [version av Azure HDInsight](../hdinsight-component-versioning.md) och Apache Hadoop- [komponenter](../hdinsight-component-versioning.md#apache-components-available-with-different-hdinsight-versions) som stöds i din lösning.  

## <a name="storage-account-name-restrictions"></a>Begränsningar för lagrings konto namn

Lagrings konto namn får inte vara längre än 24 tecken och får inte innehålla specialtecken. Dessa begränsningar gäller även för standardnamnet för containrar i lagringskontot.

Andra namngivnings begränsningar gäller även för skapande av kluster. Mer information finns i [begränsningar för kluster namn](../hdinsight-hadoop-provision-linux-clusters.md#cluster-name).

## <a name="service-outages"></a>Drift avbrott för tjänsten

Kontrol lera [Azure-status](https://status.azure.com) för eventuella problem med avbrott och tjänster.

## <a name="next-steps"></a>Nästa steg

* [Utöka Azure HDInsight med ett Azure-Virtual Network](../hdinsight-plan-virtual-network-deployment.md)
* [Använda Azure Data Lake Storage Gen2 med Azure HDInsight-kluster](../hdinsight-hadoop-use-data-lake-storage-gen2.md)  
* [Använda Azure-lagring med Azure HDInsight-kluster](../hdinsight-hadoop-use-blob-storage.md)
* [Konfigurera kluster i HDInsight med Apache Hadoop, Apache Spark, Apache Kafka med mera](../hdinsight-hadoop-provision-linux-clusters.md)
