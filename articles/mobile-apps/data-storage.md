---
title: Lagra, hantera och spara program data i molnet med Visual Studio App Center-och Azure-tjänster
description: Lär dig mer om tjänster som Visual Studio App Center som gör att du kan lagra, hantera och bevara mobil program data i molnet.
author: codemillmatt
ms.assetid: 12344321-0123-4678-8588-ccff02097224
ms.service: vs-appcenter
ms.topic: article
ms.date: 03/24/2020
ms.author: masoucou
ms.openlocfilehash: 166847325fa9094136f1c2a143f1751420f05f66
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80240916"
---
# <a name="store-sync-and-query-mobile-application-data-from-the-cloud"></a>Lagra, synkronisera och fråga mobil program data från molnet
Oavsett vilken typ av program du skapar kommer du troligen att generera och bearbeta data. Ditt programs användare har höga förväntningar. De vill att programmet ska fungera snabbt och smidigt, under alla omständigheter. De flesta program fungerar också på flera enheter. Du kan komma åt ditt program från en stationär eller en mobil enhet. Flera användare kan använda programmet samtidigt och dela data med förväntad åtkomst till data som direkt och i real tid.

Dina program användare har inte alltid Internet anslutning. Program är utformade och förväntas fungera med eller utan en Internet anslutning. Utvecklare måste välja rätt lösning för att lagra och synkronisera data till molnet för att tillhandahålla en bra kund upplevelse för sitt program.

Microsoft tillhandahåller en mängd olika tjänster som eliminerar behovet av att skapa servrar, välja din databas eller oroa dig för skalning eller säkerhet för att ge så omfattande erfarenheter som möjligt. Dessa tjänster ger en bra utvecklare som gör det möjligt att lagra program data i molnet med hjälp av SQL-eller NoSQL-API: er. Du kan också synkronisera data på alla enheter och göra det möjligt för programmet att fungera med eller utan en nätverks anslutning för att skapa skalbara och robusta program.

Använd följande tjänster för att hantera och lagra data i mobila program i molnet.

## <a name="visual-studio-app-center"></a>Visual Studio App Center
[App Center data](/appcenter/data/) är en data hanterings tjänst. Den gör det möjligt för program att hantera, bevara och synkronisera program data i molnet över olika enheter och plattformar i scenarier med online-och offline-läge. App Center data skalningar som bygger på Azure Cosmos DB, och är i takt med att användar basen och antalet program växer. Det garanterar låg latens, hög tillgänglighet och hög skalbarhet för alla dina data.

**Huvudfunktioner**
- Enkel etablering till en ny Azure Cosmos DB databas eller en befintlig Azure Cosmos DB databas från Visual Studio App Center-portalen.
- NoSQL Database-stöd för att enkelt lagra, synkronisera och fråga program data.
- Den här tjänsten är byggd ovanpå Azure Cosmos DB och ärver de flesta viktiga funktioner som erbjuds av Azure Cosmos DB och kan skalas globalt för att möta dina affärs behov.
- Synkronisering online och offline för att synkronisera data mellan enheter.
- Klient-SDK: er för mobila klienter som du kan använda för att enkelt hantera privata program data.
- Plattforms stöd för iOS, Android, Xamarin och reagera inbyggd.

**Referenser**
- [Registrera dig med Visual Studio App Center](https://appcenter.ms/signup?utm_source=Mobile%20Development%20Docs&utm_medium=Azure&utm_campaign=New%20azure%20docs)
- [Kom igång med App Center data](/appcenter/data/getting-started)

## <a name="azure-cosmos-db"></a>Azure Cosmos DB
[Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) är en globalt distribuerad databas tjänst för flera modeller. Du kan använda den för att bygga program med plan skala. Med Azure Cosmos DB kan du elastisk och oberoende skala data flöde och lagring i valfritt antal Azure-regioner över hela världen. Du kan dra nytta av snabb, ensiffriga millisekunder för data åtkomst med hjälp av dina favorit-API-ytor. Dessa ytor är SQL, MongoDB, Cassandra, tables eller Gremlin. Azure Cosmos DB unikt tillhandahåller omfattande service avtal (service avtal) för data flöde, svars tid, tillgänglighet och konsekvens.

**Huvudfunktioner**
- Har stöd för ett brett utbud av API: er, som omfattar SQL (Core) API, API för Cassandra, MongoDB-API, Gremlin-API och Tabell-API.
- Nyckel färdig global distribution replikerar dina data oavsett var dina användare befinner sig. Dina användare kan interagera med en replik av de data som är närmast dem.
- Ingen schema-eller index hantering eftersom databas motorn är fullständigt schema oberoende.
- Allmänt förekommande regional närvaro eftersom Azure Cosmos DB är tillgänglig i alla Azure-regioner i hela världen, som innehåller 54 + regioner i det offentliga molnet.
- Ett exakt definierat alternativ, flera konsekvens val eftersom Azure Cosmos DBs protokoll för multi-master-replikering är noggrant utformade för att erbjuda fem väldefinierade konsekvens val. De här fem valen är starka, begränsat föråldrad, session, konsekvent prefix och eventuell.
- 99,999% tillgänglighet för både läsning och skrivning.
- Program mässigt (eller via Azure Portal) anropar den regionala redundansväxlingen av ditt Azure Cosmos DB-konto för att säkerställa att ditt program är utformat för att motstå en regional katastrof.
- Garanterad låg latens i 99-percentilen över hela världen.

**Referenser**
- [Azure Portal](https://portal.azure.com) 
- [Azure Cosmos DB dokumentation](/azure/cosmos-db/introduction)

## <a name="azure-sql-database"></a>Azure SQL Database
 [Azure SQL Database](https://azure.microsoft.com/services/sql-database/) är en hanterad tjänst för allmän Relations databas. Du kan använda den för att skapa ett hög tillgängligt och högpresterande data lagrings lager för program och lösningar i Azure-molnet.

**Huvudfunktioner**
- **Elastic Database-modeller och-verktyg:** Med en elastisk databas kan utvecklare pool resurser bland en grupp databaser för skalning. För att administrativt hantera dessa resurser skickar du ett skript som ett jobb. SQL-databasen utför sedan skriptet över databaserna.
- **Hög prestanda:** Program med stora data flöden kan dra nytta av den senaste versionen. Den ger 25% mer Premium databas kraft.
- **Säkerhets kopieringar, replikering och hög tillgänglighet:** Inbyggd replikering och ett Microsoft-baserat service avtal på databas nivå ger program kontinuitet och skydd mot katastrof händelser. Med aktiv geo-replikering kan du konfigurera redundans och självbetjänings återställning, vilket ger fullständig kontroll över "Hoppsan Recovery". Data återställning är tillgängligt för säkerhets kopiering av data på upp till 35 dagar.
- **Underhåll i nära noll:** Automatisk program vara är en del av tjänsten. Inbyggda system repliker hjälper till att leverera inbyggt data skydd, databas drift tid och system stabilitet. System repliker flyttas automatiskt till nya datorer. De är etablerade i farten som tidigare.
- **Säkerhet:** Azure SQL Database erbjuder en portfölj med säkerhetsfunktioner som uppfyller organisationens eller branschledande efterlevnadsprinciper:
    - Med granskning får utvecklare möjlighet att utföra efterlevnad av aktiviteter och få kunskap om aktiviteter.
    - Utvecklare och kan implementera principer på databas nivå för att begränsa åtkomsten till känsliga data med säkerhet på radnivå, dynamisk data maskning och transparent data kryptering för Azure SQL Database.
    - Azure SQL Database verifieras av nyckel moln granskare som en del av omfånget för viktiga Azure-certifieringar och-godkännanden, till exempel HIPAA BAA, ISO/IEC 27001:2005, FedRAMP och EU: s modell klausuler.

**Referenser**
- [Azure Portal](https://portal.azure.com) 
- [Dokumentation om Azure SQL Database](/azure/sql-database/) 
