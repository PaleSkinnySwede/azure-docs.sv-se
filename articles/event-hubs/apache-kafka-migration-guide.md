---
title: Migrera till Azure Event Hubs för Apache Kafka
description: Den här artikeln visar hur konsumenter och producenter som använder olika protokoll (AMQP, Apache Kafka och HTTPS) kan utbyta händelser när de använder Azure Event Hubs.
services: event-hubs
documentationcenter: ''
author: ShubhaVijayasarathy
manager: ''
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/01/2020
ms.author: shvija
ms.openlocfilehash: 32b08e565b86af8f6373c9848211646128bb346d
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81677355"
---
# <a name="migrate-to-azure-event-hubs-for-apache-kafka-ecosystems"></a>Migrera till Azure Event Hubs för Apache Kafka eko system
Azure Event Hubs exponerar en Apache Kafka-slutpunkt, som gör att du kan ansluta till Event Hubs med Kafka-protokollet. Genom att göra minimala ändringar i ditt befintliga Kafka-program kan du ansluta till Azure Event Hubs och dra nytta av fördelarna med Azures eko system. Event Hubs för Kafka-support [Apache Kafka version 1,0](https://kafka.apache.org/10/documentation.html) och senare.

## <a name="pre-migration"></a>Före migrering 

### <a name="create-an-azure-account"></a>Skapa ett Azure-konto
Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) konto innan du börjar.

### <a name="create-an-event-hubs-namespace"></a>Skapa ett Event Hubs-namnområde
Följ steg-för-steg-instruktionerna i artikeln [skapa en Event Hub](event-hubs-create.md) om du vill skapa ett Event Hubs-namnområde och en Event Hub. 

### <a name="connection-string"></a>Anslutningssträng
Följ stegen i artikeln [Hämta anslutnings sträng från portalen](event-hubs-get-connection-string.md#get-connection-string-from-the-portal) . Och anteckna anslutnings strängen för senare användning. 

### <a name="fully-qualified-domain-name-fqdn"></a>Fullständigt kvalificerat domän namn (FQDN)
Du kan också behöva det fullständiga domän namnet som pekar på ditt Event Hub-namnområde. Du hittar det fullständiga domän namnet i anslutnings strängen enligt följande:

`Endpoint=sb://`**`mynamespace.servicebus.windows.net`**`/;SharedAccessKeyName=XXXXXX;SharedAccessKey=XXXXXX`

Om ditt Event Hubs namn område har distribuerats i ett icke-offentligt moln kan ditt domän namn skilja sig åt (till \*exempel. ServiceBus.chinacloudapi.CN \*,. ServiceBus.usgovcloudapi.net eller \*. ServiceBus.cloudapi.de).

## <a name="migration"></a>Migrering 

### <a name="update-your-kafka-client-configuration"></a>Uppdatera klient konfigurationen för Kafka

Om du vill ansluta till en Kafka-aktiverad Händelsehubben måste du uppdatera Kafka-klientens konfigurationer. Om du har problem med att hitta din kan du prova att `bootstrap.servers` söka efter var har angetts i ditt program.

Infoga följande konfigurationer var du än är i ditt program. Se till att uppdatera värdena `bootstrap.servers` och `sasl.jaas.config` för att dirigera klienten till din Event Hubs Kafka-slutpunkt med rätt autentisering. 

```
bootstrap.servers={MYNAMESPACE}.servicebus.windows.net:9093
request.timeout.ms=60000
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{CONNECTION STRING TO YOUR NAMESPACE}";
``` 

Om `sasl.jaas.config` inte är en konfiguration som stöds i ramverket hittar du de konfigurationer som används för att ange sasl användar namn och lösen ord och använder dem i stället. Ange användar namnet `$ConnectionString` och lösen ordet till Event Hubs anslutnings strängen.

## <a name="post-migration"></a>Efter migreringen
Kör ditt Kafka-program som skickar händelser till händelsehubben. Kontrol lera sedan att händelsehubben tar emot händelserna med hjälp av Azure Portal. På sidan **Översikt** i Event Hubs namn området växlar du till vyn **meddelanden** i avsnittet **mått** . Uppdatera sidan för att uppdatera diagrammet. Det kan ta några sekunder innan det visar att meddelandena har mottagits. 

[![Verifiera att händelsehubben tog emot meddelanden](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)


## <a name="next-steps"></a>Nästa steg
Mer information om Event Hubs och Event Hubs för Kafka finns i följande artiklar:  

- [Apache Kafka fel söknings guide för Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Vanliga frågor och svar – Event Hubs för Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Apache Kafka Developer Guide för Azure Event Hubs](apache-kafka-developer-guide.md)
- [Rekommenderade konfigurationer](https://github.com/Azure/azure-event-hubs-for-kafka/blob/master/CONFIGURATION.md)