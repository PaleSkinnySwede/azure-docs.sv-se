---
title: Säkerhetskontroller
description: En check lista över säkerhets kontroller för utvärdering av Azure SQL Database
services: sql-database
author: msmbaldwin
manager: rkalrin
ms.service: load-balancer
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: ce7f3eafa57cbd993be98f4a2da3d89cb312f9b7
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77190693"
---
# <a name="security-controls-for-azure-sql-database"></a>Säkerhets kontroller för Azure SQL Database

I den här artikeln dokumenteras de säkerhets kontroller som är inbyggda i Azure SQL Database.

[!INCLUDE [Security controls Header](../../includes/security-controls-header.md)]

SQL Database innehåller både [en enkel databas och en](sql-database-single-index.yml) [hanterad instans](sql-database-managed-instance.md). Följande poster gäller för båda erbjudandena utom i de fall där annat anges.

## <a name="network"></a>Nätverk

| Säkerhets kontroll | Ja/nej | Obs! |
|---|---|--|
| Stöd för tjänst slut punkt| Ja | Gäller endast för [en enskild databas](sql-database-single-index.yml) . |
| Stöd för Azure Virtual Network-injektering| Ja | Gäller endast för [hanterade instanser](sql-database-managed-instance.md) . |
| Nätverks isolering och brand Väggs stöd| Ja | Brand väggen på både databas nivå och server nivå. Nätverks isolering är endast för [hanterad instans](sql-database-managed-instance.md) . |
| Stöd för Tvingad tunnel trafik| Ja | [Hanterad instans](sql-database-managed-instance.md) via en [ExpressRoute](../expressroute/index.yml) VPN. |

## <a name="monitoring--logging"></a>Övervaka & loggning

| Säkerhets kontroll | Ja/nej | Obs!|
|---|---|--|
| Stöd för Azure-övervakning, till exempel Log Analytics eller Application Insights| Ja | SecureSphere, SIEM-lösningen från Imperva, stöds också via [Azure Event Hubs](../event-hubs/index.yml) integration via [SQL-granskning](sql-database-auditing.md). |
| Kontroll – plan och hantering – plan loggning och granskning| Ja | Ja endast för vissa händelser |
| Data Plans loggning och granskning | Ja | Via [SQL audit](sql-database-auditing.md) |

## <a name="identity"></a>Identitet

| Säkerhets kontroll | Ja/nej | Obs!|
|---|---|--|
| Autentisering| Ja | Azure Active Directory (Azure AD) |
| Auktorisering| Ja | Inga |

## <a name="data-protection"></a>Dataskydd

| Säkerhets kontroll | Ja/nej | Obs! |
|---|---|--|
| Kryptering på Server sidan i vila: Microsoft-hanterade nycklar | Ja | Kallas "kryptering i användning", enligt beskrivningen i artikeln [Always Encrypted](sql-database-always-encrypted.md). Kryptering på Server sidan använder [transparent data kryptering](transparent-data-encryption-azure-sql.md).|
| Kryptering under överföring:<ul><li>Azure ExpressRoute-kryptering</li><li>Kryptering i ett virtuellt nätverk</li><li>Kryptering mellan virtuella nätverk</ul>| Ja | Använda HTTPS. |
| Kryptering – nyckel hantering, till exempel CMK eller BYOK| Ja | Både hanterad och kundhanterad nyckel hantering erbjuds. Den senare erbjuds via [Azure Key Vault](../key-vault/index.yml). |
| Kryptering på kolumn nivå som tillhandahålls av Azure Data Services| Ja | Via [Always Encrypted](sql-database-always-encrypted.md). |
| Krypterade API-anrop| Ja | Använda HTTPS/TLS. |

## <a name="configuration-management"></a>Konfigurationshantering

| Säkerhets kontroll | Ja/nej | Obs!|
|---|---|--|
| Konfiguration – hanterings stöd, till exempel konfiguration av versioner| Nej  | Inga |

## <a name="additional-security-controls-for-sql-database"></a>Ytterligare säkerhets kontroller för SQL Database

| Säkerhets kontroll | Ja/nej | Obs!|
|---|---|--|
| Förebyggande: sårbarhets bedömning | Ja | Se [SQL sårbarhet Assessment service hjälper dig att identifiera databas sårbarheter](sql-vulnerability-assessment.md). |
| Förebyggande: data identifiering och klassificering  | Ja | Se [Azure SQL Database och SQL Data Warehouse data identifiering & klassificering](sql-database-data-discovery-and-classification.md). |
| Identifiering: Hot identifiering | Ja | Se [Avancerat skydd för Azure SQL Database](sql-database-threat-detection-overview.md). |

## <a name="next-steps"></a>Nästa steg

- Lär dig mer om de [inbyggda säkerhets kontrollerna i Azure-tjänster](../security/fundamentals/security-controls.md).
