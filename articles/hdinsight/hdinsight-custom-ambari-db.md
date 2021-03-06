---
title: Anpassad Apache Ambari-databas på Azure HDInsight
description: Lär dig hur du skapar HDInsight-kluster med din egen anpassade Apache Ambari-databas.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 10/29/2019
ms.author: hrasheed
ms.openlocfilehash: e7351e2f39c7e4eed84f4a47e3eeb2214a062a94
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80240162"
---
# <a name="set-up-hdinsight-clusters-with-a-custom-ambari-db"></a>Konfigurera HDInsight-kluster med en anpassad Ambari-databas

Apache Ambari fören klar hanteringen och övervakningen av ett Apache Hadoop-kluster. Ambari är ett enkelt sätt att använda webb gränssnitt och REST API. Ambari ingår i HDInsight-kluster och används för att övervaka klustret och göra konfigurations ändringar.

I det normala klustret skapas, som beskrivs i andra artiklar, till exempel [Konfigurera kluster i HDInsight](hdinsight-hadoop-provision-linux-clusters.md), Ambari distribueras i en [S0 Azure SQL-databas](../sql-database/sql-database-dtu-resource-limits-single-databases.md#standard-service-tier) som hanteras av HDInsight och inte är tillgänglig för användarna.

Med den anpassade Ambari DB-funktionen kan du distribuera ett nytt kluster och konfigurera Ambari i en extern databas som du hanterar. Distributionen görs med en Azure Resource Manager-mall. Den här funktionen har följande fördelar:

- Anpassning – du väljer databasens storlek och bearbetnings kapacitet. Om du har stora kluster bearbetnings intensiva arbets belastningar kan en Ambari-databas med lägre specifikationer bli en Flask hals för hanterings åtgärder.
- Flexibilitet – du kan skala databasen efter behov för att passa dina behov.
- Kontroll – du kan hantera säkerhets kopiering och säkerhet för din databas på ett sätt som passar dina organisations krav.

I resten av den här artikeln beskrivs följande saker:

- krav för att använda funktionen för anpassad Ambari-databas
- de steg som krävs för att etablera HDInsight-kluster med din egen externa databas för Apache Ambari

## <a name="custom-ambari-db-requirements"></a>Anpassade Ambari DB-krav

Du kan distribuera en anpassad Ambari-databas med alla kluster typer och-versioner. Flera kluster kan inte använda samma Ambari-databas.

Den anpassade Ambari DB har följande andra krav:

- Du måste ha en befintlig Azure SQL DB-server och-databas.
- Den databas som du anger för Ambari-installationen måste vara tom. Det ska inte finnas några tabeller i standardvärdet för dbo-schemat.
- Den användare som används för att ansluta till databasen ska ha behörigheterna Välj, CREATE TABLE och infoga för databasen.
- Aktivera alternativet för att [tillåta åtkomst till Azure-tjänster](../sql-database/sql-database-vnet-service-endpoint-rule-overview.md#azure-portal-steps) på Azure SQL-servern där du kommer att vara värd för Ambari.
- Hantering av IP-adresser från HDInsight-tjänsten måste tillåtas i SQL Server. Se [HDInsight Management IP-adresser](hdinsight-management-ip-addresses.md) för en lista över IP-adresser som måste läggas till i SQL Server-brandväggen.

Kom ihåg följande saker när du är värd för Apache Ambari DB i en extern databas:

- Du är ansvarig för de ytterligare kostnaderna för Azure SQL DB som innehåller Ambari.
- Säkerhetskopiera din anpassade Ambari DB med jämna mellanrum. Azure SQL Database skapar säkerhets kopior automatiskt, men tiden för kvarhållning av säkerhets kopior varierar. Mer information finns i [Läs mer om automatisk SQL Database säkerhets kopiering](../sql-database/sql-database-automated-backups.md).

## <a name="deploy-clusters-with-a-custom-ambari-db"></a>Distribuera kluster med en anpassad Ambari-databas

Om du vill skapa ett HDInsight-kluster som använder din egen externa Ambari-databas använder du [snabb starts mal len anpassad AMBARI DB](https://github.com/Azure/azure-quickstart-templates/tree/master/101-hdinsight-custom-ambari-db).

Redigera parametrarna i om `azuredeploy.parameters.json` du vill ange information om det nya klustret och databasen som ska innehålla Ambari.

Du kan börja distribuera med hjälp av Azure CLI. Ersätt `<RESOURCEGROUPNAME>` med den resurs grupp där du vill distribuera klustret.

```azurecli
az group deployment create --name HDInsightAmbariDBDeployment \
    --resource-group <RESOURCEGROUPNAME> \
    --template-file azuredeploy.json \
    --parameters azuredeploy.parameters.json
```

## <a name="next-steps"></a>Nästa steg

- [Använda extern metadatalagring i Azure HDInsight](hdinsight-use-external-metadata-stores.md)