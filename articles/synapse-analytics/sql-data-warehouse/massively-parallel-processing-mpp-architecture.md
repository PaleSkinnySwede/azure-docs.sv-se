---
title: Azure Synapse Analytics-arkitektur (tidigare SQL DW)
description: Lär dig hur Azure Synapse Analytics (tidigare SQL DW) kombinerar massivt parallell bearbetning (MPP) med Azure Storage för att uppnå höga prestanda och skalbarhet.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 11/04/2019
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: d204477818ce2733d9f6d1e3dcc7455018456bcb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80884840"
---
# <a name="azure-synapse-analytics-formerly-sql-dw-architecture"></a>Azure Synapse Analytics-arkitektur (tidigare SQL DW)

Azure Synapse är en obegränsad analystjänst som sammanför informationslager i företagsklass och stordataanalys. Det ger dig friheten att fråga efter data på dina villkor, med hjälp av antingen serverlösa resurser på begäran eller etablerade resurser – i stor skala. Azure synapse ger de här två världar tillsammans med en enhetlig upplevelse för att mata in, förbereda, hantera och hantera data för omedelbara BI-och maskin inlärnings behov.

 Azure-Synapse har fyra komponenter:

- SQL Analytics: fullständig T-SQL-baserad analys

  - SQL-pool (betala per DWU etablerad) – allmänt tillgänglig
  - SQL på begäran (betala per TB bearbetat) – (för hands version)
- Spark: djupt integrerad Apache Spark (förhands granskning)
- Data integrering: hybrid data integrering (för hands version)
- Studio: enhetlig användar upplevelse.  (Förhandsversion)

> [!VIDEO https://www.youtube.com/embed/PlyQ8yOb8kc]

## <a name="synapse-sql-mpp-architecture-components"></a>Synapse SQL MPP Architecture-komponenter

[SYNAPSE SQL](sql-data-warehouse-overview-what-is.md#synapse-sql-pool-in-azure-synapse) utnyttjar en skalbar arkitektur för att distribuera beräknings bearbetning av data över flera noder. Skalan är en abstraktion av beräknings kraften som kallas för en [informations lager enhet](what-is-a-data-warehouse-unit-dwu-cdwu.md). Compute är separat från lagring, vilket gör att du kan skala beräkningarna oberoende av data i systemet.

![Synapse SQL-arkitektur](./media/massively-parallel-processing-mpp-architecture/massively-parallel-processing-mpp-architecture.png)

SQL Analytics använder en Node-baserad arkitektur. Programmen ansluter och utfärdar T-SQL-kommandon till en Control-nod, vilket är den enda punkten i posten för SQL Analytics. Noden kontroll kör minnes minnes minnes minnes minnes motorn, som optimerar frågor för parallell bearbetning och skickar sedan åtgärder för att beräkna noder för att utföra sitt arbete parallellt.

Beräkningsnoderna lagrar alla användardata i Azure Storage och kör de parallella frågorna. Data Movement Service (DMS) är en intern tjänst på systemnivå som flyttar data mellan noder efter behov för att köra frågor parallellt och returnera korrekta resultat.

När du använder Synapse SQL-pool kan du med fristående lagrings utrymme och data bearbetning:

- Oberoende storleks beräknings kraft oberoende av dina lagrings behov.
- Öka eller minska beräknings kraften i en SQL-pool (data lager) utan att flytta data.
- Pausa beräkningskapaciteten och lämna data intakta, så att du bara betalar för lagring.
- Återuppta beräkningskapacitet under driftstimmar.

### <a name="azure-storage"></a>Azure Storage

Synapse SQL utnyttjar Azure Storage för att skydda dina användar data.  Eftersom dina data lagras och hanteras av Azure Storage, finns det en separat avgift för lagrings användningen. Data är shardade i **distributioner** för att optimera systemets prestanda. Du kan välja vilket horisontell partitionering-mönster som ska användas för att distribuera data när du definierar tabellen. Dessa horisontell partitionering-mönster stöds:

- Hash
- Resursallokering
- Replikera

### <a name="control-node"></a>Kontrollnoden

Noden kontroll är hjärna i arkitekturen. Det är den som är klientdelen som interagerar med alla program och anslutningar. MPP-motorn körs på kontrollnoden för att optimera och koordinera parallella frågor. När du skickar en T-SQL-fråga omvandlas noden Control till frågor som körs mot varje distribution parallellt.

### <a name="compute-nodes"></a>Beräkningsnoder

Beräkningsnoderna ger dataresurser. Distributioner mappar till Compute-noder för bearbetning. När du betalar för fler beräknings resurser mappas distributioner om till tillgängliga datornoder. Antalet datornoder sträcker sig från 1 till 60 och bestäms av Service nivån för Synapse SQL.

Varje Compute-nod har ett nod-ID som visas i systemvyer. Du kan se Compute Node ID genom att leta efter kolumnen node_id i systemvyer vars namn börjar med sys. pdw_nodes. En lista över dessa systemvyer finns i [MPP system views](/sql/relational-databases/system-catalog-views/sql-data-warehouse-and-parallel-data-warehouse-catalog-views?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest).

### <a name="data-movement-service"></a>Data Movement Service

Data flyttnings tjänsten (DMS) är den data transport teknik som samordnar data förflyttningen mellan Compute-noderna. Vissa frågor kräver data förflyttning för att säkerställa att de parallella frågorna returnerar korrekta resultat. När data förflyttning krävs säkerställer DMS att rätt data får rätt plats.

## <a name="distributions"></a>Distributioner

En distribution är den grundläggande lagringsenheten för parallella frågor som körs på distribuerade data. När SQL Analytics kör en fråga delas arbetet upp i 60 mindre frågor som körs parallellt.

Var och en av de 60 mindre frågorna körs på en av data distributionerna. Varje Compute-nod hanterar en eller flera av 60-distributionerna. En SQL-pool med maximala beräknings resurser har en distribution per Compute-nod. En SQL-pool med minsta beräknings resurser har alla distributioner på en Compute-nod.  

## <a name="hash-distributed-tables"></a>Hash-distribuerade tabeller

En hash-distribuerad tabell kan leverera högsta frågeprestanda för kopplingar och aggregeringar för stora tabeller.

För att Shard data till en hash-distribuerad tabell används en hash-funktion för att deterministiskt tilldela varje rad till en distribution. I tabelldefinitionen utses en av kolumnerna till distributionskolumnen. Hash-funktionen använder värdena i distributionskolumnen för att tilldela varje rad till en distribution.

Följande diagram illustrerar hur en fullständig (icke-distribuerad tabell) lagras som en hash-distribuerad tabell.

![Distribuerad tabell](./media/massively-parallel-processing-mpp-architecture/hash-distributed-table.png "Distribuerad tabell")  

- Varje rad tillhör en distribution.  
- En deterministisk hash-algoritm tilldelar varje rad till en distribution.  
- Antalet tabell rader per distribution varierar så som de visas i olika tabell storlekar.

Det finns prestanda överväganden för valet av en distributions kolumn, till exempel distinkthet, data skevning och de typer av frågor som körs i systemet.

## <a name="round-robin-distributed-tables"></a>Distribuerade tabeller med resursallokering

En Round Robin-tabell är den enklaste tabellen för att skapa och leverera snabba prestanda när den används som en mellanlagrings tabell för belastningar.

En distribuerad tabell med resursallokering distribuerar data jämnt i tabellen men utan ytterligare optimering. En distribution väljs först slumpmässigt och sedan är buffertar för rader kopplade till distributioner sekventiellt. Det går snabbt att läsa in data i en resursallokeringstabell men frågeprestanda kan ofta vara bättre med hash-distribuerade tabeller. Kopplingar i Round-Robin-tabeller kräver reshuffling-data, vilket tar ytterligare tid.

## <a name="replicated-tables"></a>Replikerade tabeller

En replikerad tabell ger snabbaste frågeprestanda för små tabeller.

En tabell som replikeras cachelagrar en fullständig kopia av tabellen på varje Compute-nod. Därför behöver du, när du replikerar en tabell, inte överföra data till beräkningsnoder innan en koppling eller aggregering. Replikerade tabeller används bäst med små tabeller. Extra lagring krävs och det finns ytterligare kostnader som uppstår när du skriver data, vilket gör stora tabeller opraktiska.  

Diagrammet nedan visar en replikerad tabell som cachelagras på den första distributionen på varje Compute-nod.  

![Replikerad tabell](./media/massively-parallel-processing-mpp-architecture/replicated-table.png "Replikerad tabell")

## <a name="next-steps"></a>Nästa steg

Nu när du vet lite om Azure-Synapse kan du lära dig hur du snabbt [skapar en SQL-pool](create-data-warehouse-portal.md) och [läser in exempel data](load-data-from-azure-blob-storage-using-polybase.md). Om du inte har erfarenhet av Azure kan [Azure-ordlistan](../../azure-glossary-cloud-terminology.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) vara till hjälp om du stöter på ny terminologi. Eller titta på några av de andra Azure Synapse-resurserna.  

- [Kundernas framgångsberättelser](https://azure.microsoft.com/case-studies/?service=sql-data-warehouse)
- [Bloggar](https://azure.microsoft.com/blog/tag/azure-sql-data-warehouse/)
- [Funktionsbegäranden](https://feedback.azure.com/forums/307516-sql-data-warehouse)
- [Videor](https://azure.microsoft.com/documentation/videos/index/?services=sql-data-warehouse)
- [Skapa ett supportärende](sql-data-warehouse-get-started-create-support-ticket.md)
- [MSDN-forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureSQLDataWarehouse)
- [Stack Overflow-forum](https://stackoverflow.com/questions/tagged/azure-sqldw)
- [Twitter](https://twitter.com/hashtag/SQLDW)
