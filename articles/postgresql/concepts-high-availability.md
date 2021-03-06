---
title: Hög tillgänglighet – Azure Database for PostgreSQL-enskild server
description: Den här artikeln innehåller information om hög tillgänglighet i Azure Database for PostgreSQL-enskild server.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 80229ff78c4570db583f1218d5d2f72da2dec388
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74768579"
---
# <a name="high-availability-concepts-in-azure-database-for-postgresql---single-server"></a>Koncept med hög tillgänglighet i Azure Database for PostgreSQL-enskild server
Tjänsten Azure Database for PostgreSQL ger en garanterad hög tillgänglighets nivå. Det ekonomiskt åtgärdade service nivå avtalet (SLA) är 99,99% vid allmän tillgänglighet. Det finns praktiskt taget ingen programs otillgänglig tid när du använder den här tjänsten.

## <a name="high-availability"></a>Hög tillgänglighet
Modellen med hög tillgänglighet (HA) baseras på inbyggda funktioner för redundansväxling när ett avbrott uppstår på nod-nivå. En avbrotts tid på radnivå kan inträffa på grund av ett maskin varu fel eller som svar på en tjänst distribution.

Vid alla tillfällen sker ändringar som gjorts i en Azure Database for PostgreSQL databas server i samband med en transaktion. Ändringar registreras synkront i Azure Storage när transaktionen har genomförts. Om ett avbrott på radnivå uppstår skapar databas servern automatiskt en ny nod och kopplar data lagring till den nya noden. Alla aktiva anslutningar släpps och eventuella synlighetssekvensnummer transaktioner är inte allokerade.

## <a name="application-retry-logic-is-essential"></a>Logik för omprövning av program är viktigt
Det är viktigt att PostgreSQL Database-program har skapats för att identifiera och försöka ansluta som har släppts, och transaktioner som misslyckades. När programmet försöker igen omdirigeras programmets anslutning transparent till den nyss skapade instansen, som tar över för den misslyckade instansen.

Internt i Azure används en gateway för att omdirigera anslutningarna till den nya instansen. Vid en avbrotts process tar hela processen normalt på flera sekunder. Eftersom omdirigeringen hanteras internt av gatewayen, förblir den externa anslutnings strängen densamma för klient programmen.

## <a name="scaling-up-or-down"></a>Skala upp eller ned
Precis som för HA-modellen skapas en ny Server instans med den angivna storleken när en Azure Database for PostgreSQL skalas upp eller ned. Den befintliga data lagringen kopplas från den ursprungliga instansen och kopplas till den nya instansen.

Under skalnings åtgärden sker ett avbrott i databas anslutningarna. Klient programmen är frånkopplade och öppna ej allokerade transaktioner avbryts. När klient programmet försöker ansluta igen eller skapar en ny anslutning, dirigerar gatewayen anslutningen till den nya instansen. 

## <a name="next-steps"></a>Nästa steg
- Lär dig mer om [hantering av tillfälliga anslutnings fel](concepts-connectivity.md)
- Lär dig hur du [replikerar dina data med Läs repliker](howto-read-replicas-portal.md)
