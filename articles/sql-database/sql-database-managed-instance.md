---
title: Översikt över SQL-hanterad instans
description: I den här artikeln beskrivs Azure SQL Database Hanterad instans.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: bonova
ms.author: bonova
ms.reviewer: sstein, carlrab, vanto
ms.date: 04/02/2020
ms.openlocfilehash: cce871d2f38d7871bd1ba9c5cb7b086e514a6913
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2020
ms.locfileid: "83120801"
---
# <a name="what-is-azure-sql-database-managed-instance"></a>Vad är Azure SQL Database Hanterad instans?

Managed instance är ett nytt distributions alternativ för Azure SQL Database, vilket ger en nära 100% kompatibilitet med den senaste SQL Server lokala (Enterprise Edition) databas motorn, som tillhandahåller en inbyggd [virtuell nätverks implementering (VNet)](../virtual-network/virtual-networks-overview.md) som behandlar vanliga säkerhets problem och en [affärs modell](https://azure.microsoft.com/pricing/details/sql-database/) som är fördelaktig för lokala SQL Server kunder. Distributions modellen för hanterade instanser gör att befintliga SQL Server kunder kan lyfta och byta sina lokala program till molnet med minimala program-och databas ändringar. På samma gång bevarar den hanterade instansen distributions alternativet alla PaaS-operativa funktioner med avseende på automatiska korrigeringar och versions uppdateringar, [automatiserade säkerhets kopieringar](sql-database-automated-backups.md)och [hög tillgänglighet](sql-database-high-availability.md), vilket drastiskt minskar hanterings kostnader och TCO.

> [!IMPORTANT]
> För en lista över regioner där distributions alternativet för hanterade instanser är tillgängligt, se [regioner som stöds](sql-database-managed-instance-resource-limits.md#supported-regions).

Följande diagram beskriver viktiga funktioner i hanterade instanser:

![viktiga funktioner](./media/sql-database-managed-instance/key-features.png)

Distributions modellen för hanterade instanser är utformad för kunder som vill migrera ett stort antal appar från lokala eller IaaS, självbyggda eller ISV-baserade miljöer till fullständigt hanterad PaaS-moln miljö, med så liten migrering som möjligt. Med hjälp av den helt automatiserade [tjänsten för data migration (DMS)](../dms/tutorial-sql-server-to-managed-instance.md#create-an-azure-database-migration-service-instance) i Azure kan kunderna lyfta och byta lokala SQL Server till en hanterad instans som erbjuder kompatibilitet med SQL Server lokala och fullständiga isolering av kund instanser med inbyggt VNet-stöd.  Med Software Assurance kan du byta ut dina befintliga licenser för rabatterade priser på en hanterad instans med hjälp av [Azure Hybrid-förmån för SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/).  En hanterad instans är det bästa migrerings målet i molnet för SQL Server instanser som kräver hög säkerhet och en omfattande programmerings yta.

Distributions alternativet för hanterade instanser ger nära-100% Surface Area-kompatibilitet med den senaste lokala SQL Server versionen via en stegad versions plan.

För att välja mellan Azure SQL Database distributions alternativ: enkel databas, poolad databas och hanterad instans och SQL Server som finns på den virtuella datorn, se [hur du väljer rätt version av SQL Server i Azure](sql-database-paas-vs-sql-server-iaas.md).

## <a name="key-features-and-capabilities"></a>Viktiga funktioner och funktioner

Den hanterade instansen kombinerar de bästa funktionerna som finns tillgängliga både i Azure SQL Database och SQL Server databas motor.

> [!IMPORTANT]
> En hanterad instans körs med alla funktioner i den senaste versionen av SQL Server, inklusive online-åtgärder, automatiska plan korrigeringar och andra förbättringar i företags prestanda. En jämförelse av funktionerna som är tillgängliga förklaras i jämförelse av funktioner [: Azure SQL Database jämfört med SQL Server](sql-database-features.md).

| **PaaS-förmåner** | **Verksamhetskontinuitet** |
| --- | --- |
|Ingen maskin varu inköp och-hantering <br>Ingen hanterings kostnad för hantering av underliggande infrastruktur <br>Snabb etablering och tjänst skalning <br>Automatiserad uppdatering och versions uppgradering <br>Integrering med andra PaaS Data Services |SLA för 99,99% drift tid  <br>Inbyggd [hög tillgänglighet](sql-database-high-availability.md) <br>Data som skyddas med [automatiserade säkerhets kopieringar](sql-database-automated-backups.md) <br>Kvarhållning av kundens bevarande period för säkerhets kopiering <br>Användarinitierad [säkerhets kopieringar](https://docs.microsoft.com/sql/t-sql/statements/backup-transact-sql?view=azuresqldb-mi-current) <br>[Återställnings funktion för tidpunkts databas](sql-database-recovery-using-backups.md#point-in-time-restore) |
|**Säkerhet och efterlevnad** | **Hantering**|
|Isolerad miljö ([VNet-integrering](sql-database-managed-instance-connectivity-architecture.md), enskild klient tjänst, dedikerad beräkning och lagring) <br>[Transparent data kryptering (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql)<br>[Azure AD-autentisering](sql-database-aad-authentication.md), stöd för enkel inloggning <br> <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">Azure AD server-Huvudkonton (inloggningar)</a>  <br>Följer standarderna för efterlevnad på samma sätt som Azure SQL Database <br>[SQL-granskning](sql-database-managed-instance-auditing.md) <br>[Advanced Threat Protection](sql-database-managed-instance-threat-detection.md) |Azure Resource Manager-API för automatisering av tjänst etablering och skalning <br>Azure Portal funktioner för manuell tjänst etablering och skalning <br>Datamigreringstjänst

> [!IMPORTANT]
> Azure SQL Database (alla distributions alternativ) har certifierats mot ett antal efterlevnads standarder. Mer information finns i [Microsoft Azure Trust Center](https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942) där du hittar den mest aktuella listan med SQL Database certifierings certifieringar.

Huvud funktionerna i hanterade instanser visas i följande tabell:

|Funktion | Beskrivning|
|---|---|
| SQL Server version/build | SQL Server databas motor (senaste stabila) |
| Hanterade automatiserade säkerhets kopieringar | Ja |
| Inbyggd instans och databas övervakning och mått | Ja |
| Automatisk uppdatering av program vara | Ja |
| De senaste databas motor funktionerna | Ja |
| Antal datafiler (rader) per databas | Flera |
| Antal loggfiler (logg) per databas | 1 |
| VNet – Azure Resource Manager distribution | Ja |
| VNet – klassisk distributions modell | Inga |
| Portal stöd | Ja|
| Inbyggd integrerings tjänst (SSIS) | No-SSIS är en del av [Azure Data Factory PaaS](https://docs.microsoft.com/azure/data-factory/tutorial-deploy-ssis-packages-azure) |
| Inbyggd Analysis Service (SSAS) | No-SSAS är separat [PaaS](https://docs.microsoft.com/azure/analysis-services/analysis-services-overview) |
| Inbyggd repor ting service (SSRS) | Använd [Power BI sid färdiga rapporter](https://docs.microsoft.com/power-bi/paginated-reports/paginated-reports-report-builder-power-bi) i stället för att vara värd för SSRS på virtuella Azure-datorer. Även om den hanterade instansen inte kan köra SSRS som en tjänst, kan den vara värd för SSRS 2019-katalog databaser för en extern rapport server med SQL Server autentisering. |
|||

## <a name="vcore-based-purchasing-model"></a>Köpmodell baserad på virtuell kärna

Den [vCore-baserade inköps modellen](sql-database-service-tiers-vcore.md) för hanterade instanser ger dig flexibilitet, kontroll, transparens och ett enkelt sätt att översätta lokala arbets belastnings krav till molnet. Med den här modellen kan du ändra beräkning, minne och lagring baserat på dina arbets belastnings behov. VCore-modellen är också kvalificerad för upp till 55 procent besparingar med [Azure Hybrid-förmån för SQL Server](https://azure.microsoft.com/pricing/hybrid-benefit/).

I vCore-modellen kan du välja mellan generationens maskin vara.

- **Gen4** Logiska processorer baseras på Intel E5-2673 v3 (Haswell) 2,4-GHz-processorer, anslutna SSD, fysiska kärnor, 7 GB RAM-minne per kärna och beräknings storlekar mellan 8 och 24 virtuella kärnor.
- **Gen5** Logiska processorer baseras på Intel E5-2673 v4 (Broadwell) 2,3-GHz-och Intel SP-8160-processorer (Skylake), fast NVMe SSD, Hyper-threaded Logical Core och beräknings storlekar mellan 4 och 80 kärnor.

Hitta mer information om skillnaden mellan maskin varu generationer i [resurs gränser för hanterade instanser](sql-database-managed-instance-resource-limits.md#hardware-generation-characteristics).

## <a name="managed-instance-service-tiers"></a>Tjänstnivåer för hanterad instans

Hanterad instans finns tillgänglig på två tjänst nivåer:

- **Generell användning**: utformad för program med krav på standard prestanda och IO-latens.
- **Verksamhets kritisk**: utformad för program med krav på låg IO-latens och minimal påverkan på underliggande underhålls åtgärder på arbets belastningen.

Båda tjänst nivåerna garanterar 99,99% tillgänglighet och gör att du kan välja lagrings storlek och beräknings kapacitet separat. Mer information om arkitekturen för hög tillgänglighet för Azure SQL Database finns i [hög tillgänglighet och Azure SQL Database](sql-database-high-availability.md).

### <a name="general-purpose-service-tier"></a>Tjänst nivå för generell användning

I följande lista beskrivs viktiga egenskaper för Generell användning tjänst nivån:

- Design för de flesta affärs program med typiska prestanda krav
- Azure Blob Storage med höga prestanda (8 TB)
- Inbyggd [hög tillgänglighet](sql-database-high-availability.md#basic-standard-and-general-purpose-service-tier-availability) baserat på tillförlitlig Azure Blob Storage och [Azure Service Fabric](../service-fabric/service-fabric-overview.md)

Mer information finns i [lagrings skikt i generell användnings nivå](https://medium.com/azure-sqldb-managed-instance/file-layout-in-general-purpose-azure-sql-managed-instance-cf21fff9c76c) och [metod tips för lagrings prestanda och överväganden för hanterade instanser (generell användning)](https://blogs.msdn.microsoft.com/sqlcat/2018/07/20/storage-performance-best-practices-and-considerations-for-azure-sql-db-managed-instance-general-purpose/).

Hitta mer information om skillnaden mellan tjänst nivåer i [resurs gränser för hanterade instanser](sql-database-managed-instance-resource-limits.md#service-tier-characteristics).

### <a name="business-critical-service-tier"></a>Affärskritisk tjänst nivå

Affärskritisk Service Tier är byggd för program med höga IO-krav. Den ger högsta möjliga återhämtning till problem med flera isolerade repliker.

I följande lista beskrivs viktiga egenskaper för Affärskritisk tjänst nivå:

- Utformad för företags program med högsta prestanda och HA krav
- Levereras med super-fast lokal SSD-lagring (upp till 1 TB på Gen4 och upp till 4 TB på Gen5)
- Inbyggd [hög tillgänglighet](sql-database-high-availability.md#premium-and-business-critical-service-tier-availability) baserat på [tillgänglighets grupper](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server) och [Azure-Service Fabric](../service-fabric/service-fabric-overview.md)som alltid är tillgängliga.
- Inbyggd ytterligare [skrivskyddad databas replik](sql-database-read-scale-out.md) som kan användas för rapportering och andra skrivskyddade arbets belastningar
- [Minnes intern OLTP](sql-database-in-memory.md) som kan användas för arbets belastning med höga prestanda krav  

Hitta mer information om skillnaden mellan tjänst nivåer i [resurs gränser för hanterade instanser](sql-database-managed-instance-resource-limits.md#service-tier-characteristics).


## <a name="managed-instance-management-operations"></a>Hanterings åtgärder för hanterade instanser

Azure SQL Database innehåller hanteringsåtgärder som du kan använda för att automatiskt distribuera nya hanterade instanser, uppdatera instansegenskaper och ta bort instanser när de inte längre behövs. Det här avsnittet innehåller information om hanterings åtgärder och deras normala varaktighet.

För att stödja [distributioner inom virtuella Azure-nätverk (virtuella nätverk)](../virtual-network/virtual-network-for-azure-services.md) och tillhandahålla isolering och säkerhet för kunder använder den hanterade instansen på [virtuella kluster](sql-database-managed-instance-connectivity-architecture.md#high-level-connectivity-architecture), som representerar en dedikerad uppsättning isolerade virtuella datorer som distribueras i kundens virtuella nätverks under nät. Varje distribution av hanterade instanser i ett tomt undernät resulterar i grunden i en ny version av det virtuella klustret.

Efterföljande åtgärder på distribuerade hanterade instanser kan också ha effekter på det underliggande virtuella klustret. Detta påverkar varaktigheten för hanterings åtgärder, eftersom distribution av ytterligare virtuella datorer levereras med en kostnad som måste beaktas när du planerar nya distributioner eller uppdateringar till befintliga hanterade instanser.

Alla hanteringsåtgärder kan kategoriseras på följande sätt:

- Instans distribution (ny instans skapas). 
- Instans uppdatering (ändring av instans egenskaper, till exempel virtuella kärnor eller reserverad lagring.
- Borttagning av instans.

Åtgärder i virtuella kluster tar vanligt vis längst. Varaktigheten för de virtuella klustren varierar – nedan visas de värden som du normalt förväntar dig, baserat på befintliga telemetridata för tjänsten:

- Skapa virtuellt kluster. Det här är ett synkront steg i instans hanterings åtgärder. **90% av åtgärderna har slutförts på 4 timmar**.
- Storleks ändring av virtuellt kluster (expandering eller krympning). Expansion är ett synkront steg, medan krympningen utförs asynkront (utan inverkan på instans hanterings åtgärdernas varaktighet). **90% av kluster utökningar slutförs på mindre än 2,5 timmar**.
- Borttagning av virtuellt kluster. Borttagning är ett asynkront steg, men det kan också [initieras manuellt](sql-database-managed-instance-delete-virtual-cluster.md) på ett tomt virtuellt kluster, vilket innebär att det körs synkront. **90% av de virtuella kluster borttagningarna slutförs om 1,5 timmar**.

Dessutom kan hantering av instanser också innehålla en av åtgärderna på värdbaserade databaser, vilket leder till längre varaktigheter:

- Bifogar databasfiler från Azure Storage. Det här är ett synkront steg, till exempel Compute (vCore) eller skalning av lagrings utrymme på Generell användning tjänst nivå. **90% av dessa åtgärder har slutförts på 5 minuter**.
- Dirigerar alltid om tillgänglighets gruppens dirigering. Det här är ett synkront steg, till exempel Compute (vCore) eller lagrings skalning på Affärskritisk tjänst nivå samt i ändra tjänst nivån från Generell användning till Affärskritisk (eller vice versa). Den här åtgärdens varaktighet är proportionerlig till den totala databas storleken samt den aktuella databas aktiviteten (antal aktiva transaktioner). Databas aktivitet vid uppdatering av en instans kan medföra betydande varianser för den totala varaktigheten. **90% av dessa åtgärder körs vid 220 GB/timme eller högre**.

I följande tabell sammanfattas åtgärder och typiska övergripande varaktigheter:

|Kategori  |Åtgärd  |Tids krävande segment  |Uppskattad varaktighet  |
|---------|---------|---------|---------|
|**Distribution** |Första instansen i ett tomt undernät|Skapa virtuellt kluster|90% av åtgärderna har slutförts på 4 timmar|
|Distribution |Första instansen av en annan maskin varu generation i ett undernät som inte är tomt (till exempel första generation 5-instansen i ett undernät med generation 4 instanser)|Skapa virtuellt kluster *|90% av åtgärderna har slutförts på 4 timmar|
|Distribution |Första instans skapandet av 4 virtuella kärnor, i ett tomt eller icke-tomt undernät|Skapa virtuellt kluster * *|90% av åtgärderna har slutförts på 4 timmar|
|Distribution |Efterföljande instans skapas i det icke-tomma under nätet (andra, tredje osv.)|Storleks ändring av virtuellt kluster|90% av åtgärderna har slutförts om 2,5 timmar|
|**Uppdatera** |Ändring av instans egenskap (administratörs lösen ord, AAD-inloggning, Azure Hybrid-förmån flagga)|E.t.|Upp till 1 minut|
|Uppdatera |Skalning av instans lagring upp/ned (Generell användning tjänst nivå)|Bifoga databasfiler|90% av åtgärderna har slutförts på 5 minuter|
|Uppdatera |Skalning av instans lagring upp/ned (Affärskritisk tjänst nivå)|-Storleks ändring av virtuellt kluster<br>-Always on-tillgänglighets grupps dirigering|90% av åtgärderna slutförs om 2,5 timmar + tid för att dirigera alla databaser (220 GB/timme)|
|Uppdatera |Virtuella kärnor (Instance Compute) skalar upp och ned (Generell användning)|-Storleks ändring av virtuellt kluster<br>-Bifoga databasfiler|90% av åtgärderna har slutförts om 2,5 timmar|
|Uppdatera |Virtuella kärnor (Instance Compute) skalar upp och ned (Affärskritisk)|-Storleks ändring av virtuellt kluster<br>-Always on-tillgänglighets grupps dirigering|90% av åtgärderna slutförs om 2,5 timmar + tid för att dirigera alla databaser (220 GB/timme)|
|Uppdatera |Instans skala ned till 4 virtuella kärnor (Generell användning)|-Storleks ändring av virtuellt kluster (om det är färdigt för första gången kan det krävas att skapa virtuella kluster * *)<br>-Bifoga databasfiler|90% av åtgärderna har slutförts på 4 timmar 5 min * *|
|Uppdatera |Instans skala ned till 4 virtuella kärnor (Affärskritisk)|-Storleks ändring av virtuellt kluster (om det är färdigt för första gången kan det krävas att skapa virtuella kluster * *)<br>-Always on-tillgänglighets grupps dirigering|90% av åtgärderna har slutförts på 4 timmar + tid för att dirigera alla databaser (220 GB/timme)|
|Uppdatera |Instans tjänst nivå ändring (Generell användning till Affärskritisk och vice versa)|-Storleks ändring av virtuellt kluster<br>-Always on-tillgänglighets grupps dirigering|90% av åtgärderna slutförs om 2,5 timmar + tid för att dirigera alla databaser (220 GB/timme)|
|**Borttagning**|Borttagning av instans|Logg säkerhets kopiering för alla databaser|90% åtgärder har slutförts på upp till 1 minut.<br>OBS! om den sista instansen i under nätet tas bort, kommer den här åtgärden att schemalägga borttagning av virtuellt kluster efter 12 timmar * * *|
|Borttagning|Borttagning av virtuellt kluster (som användarinitierad åtgärd)|Borttagning av virtuellt kluster|90% av åtgärderna har slutförts på upp till 1,5 timmar|

\*Det virtuella klustret skapas per maskin varu generation.

\*\*Distributions alternativet 4 virtuella kärnor släpptes i juni 2019 och kräver en ny virtuell kluster version. Om du har instanser i mål under nätet som alla skapades före 12 juni, distribueras ett nytt virtuellt kluster automatiskt till värd 4 vCore-instanser.

\*\*\*12 timmar är den aktuella konfigurationen men som kan ändras i framtiden, så ta inte ett hårt beroende av den. Om du behöver ta bort ett virtuellt kluster tidigare (för att frigöra under nätet till exempel), se [ta bort ett undernät när du har tagit bort en Azure SQL Database Hanterad instans](sql-database-managed-instance-delete-virtual-cluster.md).

### <a name="instance-availability-during-management-operations"></a>Tillgänglighet för instanser under hanterings åtgärder

Den hanterade instansen är inte tillgänglig för klient program under distributions-och borttagnings åtgärder.

Den hanterade instansen är tillgänglig under uppdaterings åtgärder förutom ett kort stillestånd som orsakas av redundansväxlingen som inträffar i slutet av uppdateringen. Det tar vanligt vis upp till 10 sekunder, även om tids krävande transaktioner har avbrutits, tack vare den [påskyndade databas återställningen](sql-database-accelerated-database-recovery.md).

> [!TIP]
> Uppdateringen av det reserverade lagrings utrymmet i Generell användning tjänst nivån påverkar inte redundans eller påverkar inte instans tillgänglighet.

> [!IMPORTANT]
> Det rekommenderas inte att skala beräkning eller lagring av Azure SQL Database hanterade instanser eller ändra tjänst nivån samtidigt med tids krävande transaktioner (data import, data bearbetnings jobb, index återuppbyggnad osv.). Redundansväxling av databasen som utförs i slutet av åtgärden avbryter alla pågående transaktioner.


### <a name="management-operations-cross-impact"></a>Hanterings åtgärder över-påverkan

Hanterings åtgärder för hanterade instanser kan påverka andra hanterings åtgärder som placerats i samma virtuella kluster. Detta omfattar följande:

- **Tids krävande återställnings åtgärder** i ett virtuellt kluster sätter igång andra instanser att skapa eller skala i samma undernät.<br/>**Exempel:** om det finns tids krävande återställnings åtgärd och det finns en begäran om att skapa eller skala i samma undernät, tar det längre tid att slutföra den här begäran eftersom den väntar på att återställningen ska slutföras innan den fortsätter.
    
- **Efterföljande instans skapande eller skalnings** åtgärd spärras av tidigare initierad instans skapande eller instans skala som initierade storlek för virtuella kluster.<br/>**Exempel:** om det finns flera skapande-och/eller skalnings begär anden i samma undernät under samma virtuella kluster, och en av dem initierar storleks ändring för virtuella kluster, kommer alla begär Anden som har skickats 5 + minuter efter den som krävde storleken på virtuellt kluster att ändra storlek att bli längre än förväntat.

- **Åtgärder för att skapa/skala som skickats in i fönstret för 5 minuter** kommer att grupperas och köras parallellt.<br/>**Exempel:** Det går bara att ändra storlek på ett virtuellt kluster för alla åtgärder som skickats i fönstret 5 minuter (mätning från tidpunkten för att köra den första åtgärden). Om en annan begäran skickas mer än 5 minuter efter att du har skickat det första, väntar det tills det virtuella klustret har ändrats innan körningen påbörjas.

> [!IMPORTANT]
> Hanterings åtgärder som spärras på grund av att en annan åtgärd pågår kommer att återupptas automatiskt när villkoren för att fortsätta är uppfyllda. Det finns ingen användar åtgärd som krävs för att återuppta en tillfälligt pausad hanterings åtgärd.

### <a name="canceling-management-operations"></a>Avbryta hanterings åtgärder

I följande tabell sammanfattas möjligheten att avbryta vissa hanterings åtgärder och typiska övergripande varaktigheter:

Kategori  |Åtgärd  |Avbrytbar  |Beräknad tids längd för avbrott  |
|---------|---------|---------|---------|
|Distribution |Skapa instans |Inga |  |
|Uppdatera |Skalning av instans lagring upp/ned (Generell användning) |Inga |  |
|Uppdatera |Skalning av instans lagring upp/ned (Affärskritisk) |Ja |90% av åtgärderna har slutförts på 5 minuter |
|Uppdatera |Virtuella kärnor (Instance Compute) skalar upp och ned (Generell användning) |Ja |90% av åtgärderna har slutförts på 5 minuter |
|Uppdatera |Virtuella kärnor (Instance Compute) skalar upp och ned (Affärskritisk) |Ja |90% av åtgärderna har slutförts på 5 minuter |
|Uppdatera |Instans tjänst nivå ändring (Generell användning till Affärskritisk och vice versa) |Ja |90% av åtgärderna har slutförts på 5 minuter |
|Ta bort |Borttagning av instans |Inga |  |
|Ta bort |Borttagning av virtuellt kluster (som användarinitierad åtgärd) |Inga |  |

Om du vill avbryta hanterings åtgärden går du till bladet översikt och klickar på meddelande rutan för pågående åtgärder. En skärm med kontinuerlig åtgärd visas från höger sida och det kommer att finnas en knapp för att avbryta åtgärden. När du har klickat på uppmanas du att klicka igen och bekräfta att du vill avbryta åtgärden.

[![](./media/sql-database-managed-instance/canceling-operation.png)](./media/sql-database-managed-instance/canceling-operation.png#lightbox)

När Cancel-begäran har skickats och bearbetats får du ett meddelande om att avbrottet har slutförts eller inte. 

Om åtgärden avbryts, avbryts hanterings åtgärden på några minuter, vilket resulterar i ett haveri.

![avbryter åtgärds resultat](./media/sql-database-managed-instance/canceling-operation-result.png)

Om Avbryt förfrågan Miss lyckas eller om Avbryt-knappen inte är aktiv, innebär det att hanterings åtgärden inte har gått att avbryta och att den slutförs på några minuter. Hanterings åtgärden fortsätter att köras tills den har slutförts.

> [!IMPORTANT]
> Det finns för närvarande bara stöd för att avbryta åtgärden i portalen.

## <a name="advanced-security-and-compliance"></a>Avancerad säkerhet och efterlevnad

Alternativet för distribution av hanterade instanser kombinerar avancerade säkerhetsfunktioner som tillhandahålls av Azure-molnet och SQL Server databas motor.

### <a name="managed-instance-security-isolation"></a>Säkerhets isolering för hanterad instans

En hanterad instans ger ytterligare säkerhets isolering från andra klienter i Azure-molnet. Säkerhets isolering innehåller:

- [Inbyggd implementering av virtuella nätverk](sql-database-managed-instance-connectivity-architecture.md) och anslutning till din lokala miljö med hjälp av Azure Express Route eller VPN gateway.
- I en standard distribution exponeras SQL-slutpunkten bara via en privat IP-adress, vilket ger säker anslutning från privata Azure-eller hybrid nätverk.
- En-klient med dedikerad underliggande infrastruktur (beräkning, lagring).

I följande diagram beskrivs olika anslutnings alternativ för dina program:

![hög tillgänglighet](./media/sql-database-managed-instance/application-deployment-topologies.png)  

Om du vill veta mer om VNet-integrering och nätverks princip tillämpning på under näts nivån, se [VNet-arkitektur för hanterade instanser](sql-database-managed-instance-connectivity-architecture.md) och [Anslut ditt program till en hanterad instans](sql-database-managed-instance-connect-app.md).

> [!IMPORTANT]
> Placera flera hanterade instanser i samma undernät, där de tillåts av dina säkerhets krav, eftersom de ger dig ytterligare förmåner. Collocating-instanser i samma undernät fören klar avsevärt nätverks infrastrukturen och minskar instansen av etablerings tiden, eftersom varaktigheten för lång etablering är kopplad till kostnaden för att distribuera den första hanterade instansen i ett undernät.

### <a name="azure-sql-database-security-features"></a>Azure SQL Database säkerhetsfunktioner

Azure SQL Database innehåller en uppsättning avancerade säkerhetsfunktioner som kan användas för att skydda dina data.

- [Granskning av hanterade instanser](sql-database-managed-instance-auditing.md) spårar databas händelser och skriver dem till en Gransknings logg fil som placeras i ditt Azure Storage-konto. Granskning kan hjälpa till att upprätthålla regelefterlevnad, förstå databas aktivitet och få insikt i avvikelser och avvikelser som kan tyda på affärs problem eller misstänkta säkerhets överträdelser.
- Data kryptering i rörelse – en hanterad instans skyddar dina data genom att tillhandahålla kryptering för data i rörelse med hjälp av Transport Layer Security. Förutom Transport Layer Security ger distributions alternativet hanterad instans skydd av känsliga data i Flight, i vila och under bearbetning av frågor med [Always Encrypted](/sql/relational-databases/security/encryption/always-encrypted-database-engine). Always Encrypted är först i branschen med att erbjuda oslagbar datasäkerhet mot överträdelser, inklusive stöld av viktiga data. Med Always Encrypted lagras exempelvis kreditkorts nummer krypterade i databasen alltid, även under frågekörning, vilket tillåter dekryptering vid användnings punkten av behörig personal eller program som behöver behandla dessa data.
- [Avancerat skydd](sql-database-managed-instance-threat-detection.md) kompletterar [granskning](sql-database-managed-instance-auditing.md) genom att tillhandahålla ett extra lager av säkerhets information som är inbyggt i tjänsten och som identifierar ovanliga och potentiellt skadliga försök att komma åt eller utnyttja databaser. Du får varningar om misstänkta aktiviteter, potentiella sårbarheter och SQL-injektering, samt avvikande databas åtkomst mönster. Aviseringar för avancerat skydd kan visas från [Azure Security Center](https://azure.microsoft.com/services/security-center/) och ger information om misstänkt aktivitet och rekommenderar åtgärder för att undersöka och minimera hotet.  
- [Dynamisk data maskning](/sql/relational-databases/security/dynamic-data-masking) begränsar känslig data exponering genom att maskera den till icke-privilegierade användare. Dynamisk datamaskering bidrar till att förhindra obehörig åtkomst till känsliga data genom att göra det möjligt att ange hur mycket av känsliga data som ska visas med minimal påverkan på program nivån. Det är en principbaserad säkerhetsfunktion som döljer känsliga data i resultat uppsättningen för en fråga över angivna databas fält, medan data i databasen inte ändras.
- [Säkerhet på radnivå](/sql/relational-databases/security/row-level-security) gör att du kan styra åtkomsten till rader i en databas tabell baserat på egenskaperna för användaren som kör en fråga (t. ex. efter grupp medlemskap eller körnings kontext). Säkerheten på radnivå (RLS) förenklar design och kodning av säkerheten i ditt program. RLS låter dig implementera begränsningar för dataåtkomst för raden. Du kan till exempel se till att anställda endast kan komma åt de data rader som är relevanta för deras avdelning eller begränsa en data åtkomst till enbart relevanta data.
- [Transparent data kryptering (TDE)](https://docs.microsoft.com/sql/relational-databases/security/encryption/transparent-data-encryption-azure-sql) krypterar hanterade instans data filer, som kallas att kryptera data i vila. TDE utför I/O-kryptering i real tid och dekryptering av data-och loggfiler. Krypteringen använder en databas krypterings nyckel (DEK), som lagras i databasens start post för tillgänglighet under återställningen. Du kan skydda alla dina databaser i en hanterad instans med transparent data kryptering. TDE är SQL Server en beprövad krypterings-till-rest-teknik som krävs av många kompatibla standarder för att skydda mot stöld av lagrings medier.

Migrering av en krypterad databas till en hanterad instans stöds via Azure Database Migration Service (DMS) eller intern återställning. Om du planerar att migrera en krypterad databas med intern återställning är det ett obligatoriskt steg att migrera det befintliga TDE-certifikatet från SQL Server lokalt eller SQL Server i en virtuell dator till en hanterad instans. Mer information om alternativ för migrering finns i [SQL Server instans migrering till hanterad instans](sql-database-managed-instance-migrate.md).

## <a name="azure-active-directory-integration"></a>Azure Active Directory-integrering

Distributions alternativet för hanterade instanser stöder traditionella inloggningar och inloggningar för SQL Server-databasmotorn som är integrerade med Azure Active Directory (AAD). Azure AD server-Huvudkonton (inloggningar) (**offentlig för hands**version) är Azure Cloud-versioner av lokala databas inloggningar som du använder i din lokala miljö. Med Azure AD-serverns huvud namn (inloggningar) kan du ange användare och grupper från din Azure Active Directory klient som sann huvud namn för instans – som kan utföra alla instanser på instans nivå, inklusive frågor över flera databaser inom samma hanterade instans.

En ny syntax introduceras för att skapa Azure AD server-huvudobjekt (inloggningar), **från extern provider**. Mer information om syntaxen finns i <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">Skapa inloggning</a>och granska [en Azure Active Directory administratör för den hanterade instansen](sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-managed-instance) av artikeln.

### <a name="azure-active-directory-integration-and-multi-factor-authentication"></a>Azure Active Directory-integrering och multifaktorautentisering

Med alternativet för distribution av hanterade instanser kan du centralt hantera identiteter för databas användare och andra Microsoft-tjänster med [Azure Active Directory-integrering](sql-database-aad-authentication.md). Den här funktionen förenklar hanteringen av behörighet och ger ökad säkerhet. Azure Active Directory stöder [multifaktorautentisering](sql-database-ssms-mfa-authentication-configure.md) (MFA) för att öka säkerheten för data och program med stöd för en process med enkel inloggning.

### <a name="authentication"></a>Autentisering

Autentisering med hanterade instanser avser hur användare bekräftar sin identitet när de ansluter till databasen. SQL Database stöder två typer av autentisering:  

- **SQL-autentisering**:

  Den här autentiseringsmetoden använder ett användar namn och lösen ord.
- **Azure Active Directory autentisering**:

  Den här autentiseringsmetoden använder identiteter som hanteras av Azure Active Directory och stöds för hanterade och integrerade domäner. Använd Active Directory-autentisering (integrerad säkerhet) [närhelst det går](https://docs.microsoft.com/sql/relational-databases/security/choose-an-authentication-mode).

### <a name="authorization"></a>Auktorisering

Auktorisering syftar på vad en användare kan göra inom en Azure SQL Database och styrs av ditt användar kontos databas roll medlemskap och behörigheter på objekt nivå. En hanterad instans har samma Authorization-funktioner som SQL Server 2017.

## <a name="database-migration"></a>Databasmigrering

Distributions alternativet för hanterade instanser riktar sig mot användar scenarier med en Mass databas migrering från lokala eller IaaS databas implementeringar. Hanterad instans har stöd för flera alternativ för migrering av databas:

### <a name="back-up-and-restore"></a>Säkerhetskopiera och återställa  

Migreringsprocessen utnyttjar SQL-säkerhetskopieringar till Azure Blob Storage. Säkerhets kopior som lagras i Azure Storage BLOB kan återställas direkt till en hanterad instans med hjälp av [kommandot T-SQL REstore](https://docs.microsoft.com/sql/t-sql/statements/restore-statements-transact-sql?view=azuresqldb-mi-current).

- En snabb start som visar hur du återställer Wide World-importörer – standard säkerhets kopierings filen för databasen finns i [återställa en säkerhets kopia till en hanterad instans](sql-database-managed-instance-get-started-restore.md). Den här snabb starten visar att du måste ladda upp en säkerhets kopia till Azure Blob Storage och skydda den med en SAS-nyckel (signatur för delad åtkomst).
- Information om återställning från URL finns i [intern återställning från URL](sql-database-managed-instance-migrate.md#native-restore-from-url).

> [!IMPORTANT]
> Säkerhets kopieringar från en hanterad instans kan bara återställas till en annan hanterad instans. De kan inte återställas till en lokal SQL Server eller till en enskild databas/elastisk pool.

### <a name="data-migration-service"></a>Datamigreringstjänst

Azure Database Migration Service är en fullständigt hanterad tjänst som är utformad för att möjliggöra sömlös migrering från flera databas källor till Azure-dataplattformar med minimal stillestånds tid. Den här tjänsten effektiviserar de uppgifter som krävs för att flytta befintliga tredje parter och SQL Server databaser till Azure SQL Database (enstaka databaser, pooler databaser i elastiska pooler och instans databaser i en hanterad instans) och SQL Server i Azure VM. Se [hur du migrerar din lokala databas till en hanterad instans med DMS](https://aka.ms/migratetoMIusingDMS).

## <a name="sql-features-supported"></a>SQL-funktioner som stöds

Distributions alternativet för hanterade instanser syftar på att leverera nära till 100% Surface Area-kompatibilitet med lokala SQL Server kommer i stadier tills tjänsten är allmänt tillgänglig. För en funktion och jämförelse lista, se [SQL Database funktions jämförelse](sql-database-features.md)och för en lista över t-SQL-skillnader i hanterade instanser och SQL Server, se [hanterade instanser t-SQL-skillnader från SQL Server](sql-database-managed-instance-transact-sql-information.md).

Distributions alternativet för hanterade instanser stöder bakåtkompatibilitet till SQL 2008-databaser. Direkt migrering från SQL 2005-databas servrar stöds, kompatibilitetsnivå för migrerade SQL 2005-databaser uppdateras till SQL 2008.
  
Följande diagram visar kompatibiliteten för Surface Area i den hanterade instansen:  

![migreringsarkivet](./media/sql-database-managed-instance/migration.png)

### <a name="key-differences-between-sql-server-on-premises-and-in-a-managed-instance"></a>Viktiga skillnader mellan SQL Server lokalt och i en hanterad instans

Distributions alternativet för hanterade instanser är inte alltid uppdaterat i molnet, vilket innebär att vissa funktioner i lokala SQL Server kan vara antingen föråldrade, föråldrade eller ha alternativ. Det finns särskilda fall när verktyg måste identifiera att en viss funktion fungerar på ett något annorlunda sätt eller att tjänsten körs i en miljö som du inte har fullständig kontroll över:

- Hög tillgänglighet är inbyggd och förkonfigurerad med teknik som liknar [Always on-tillgänglighetsgrupper](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server).
- Automatiserade säkerhets kopieringar och tidpunkts återställning. Kunden kan initiera `copy-only` säkerhets kopieringar som inte stör automatisk säkerhets kopierings kedja.
- DEN hanterade instansen tillåter inte att fullständiga fysiska sökvägar anges, vilket innebär att alla motsvarande scenarier måste stödjas på olika sätt: Restore DB stöder inte flytt, CREATE DB tillåter inte fysiska sökvägar, BULK INSERT bara fungerar med Azure-blobbar, osv.
- Den hanterade instansen stöder [Azure AD-autentisering](sql-database-aad-authentication.md) som moln alternativ till Windows-autentisering.
- Hanterad instans hanterar automatiskt XTP-filgrupp och filer för databaser som innehåller InMemory OLTP-objekt
- Den hanterade instansen stöder SQL Server Integration Services (SSIS) och kan vara värd för SSIS Catalog (SSISDB) som lagrar SSIS-paket, men de körs på en hanterad Azure-SSIS Integration Runtime (IR) i Azure Data Factory (ADF) finns i [skapa Azure-SSIS IR i ADF](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime). Information om hur du jämför SSIS-funktionerna i SQL Database finns i [jämföra en Azure SQL Database enkel databas, elastisk pool och hanterad instans](../data-factory/create-azure-ssis-integration-runtime.md#comparison-of-a-sql-database-single-database-elastic-pool-and-managed-instance).

### <a name="managed-instance-administration-features"></a>Administrations funktioner för hanterade instanser

Med distributions alternativet för hanterade instanser kan system administratören ägna mindre tid åt administrativa uppgifter eftersom SQL Database tjänsten utför dem antingen åt dig eller avsevärt fören klar dessa aktiviteter. Till exempel [installation och uppdatering av OS/RDBMS](sql-database-high-availability.md), [dynamisk storleks ändring och konfiguration av instanser](sql-database-single-database-scale.md), [säkerhets kopieringar](sql-database-automated-backups.md), [databasreplikering](replication-with-sql-database-managed-instance.md) (inklusive system databaser), [konfiguration av hög tillgänglighet](sql-database-high-availability.md)och konfiguration av data strömmar för hälso-och [prestanda övervakning](../azure-monitor/insights/azure-sql.md) .

> [!IMPORTANT]
> En lista över funktioner som stöds delvis och som inte stöds finns i [SQL Database funktioner](sql-database-features.md). En lista över T-SQL-skillnader i hanterade instanser och SQL Server finns i [hanterade instanser t-SQL-skillnader från SQL Server](sql-database-managed-instance-transact-sql-information.md)

### <a name="how-to-programmatically-identify-a-managed-instance"></a>Så här identifierar du en hanterad instans program mässigt

I följande tabell visas flera egenskaper, som är tillgängliga via Transact SQL, som du kan använda för att identifiera att programmet fungerar med en hanterad instans och hämta viktiga egenskaper.

|Egenskap|Värde|Kommentar|
|---|---|---|
|`@@VERSION`|Microsoft SQL Azure (RTM) – 12.0.2000.8 2018-03-07 Copyright (C) 2018 Microsoft Corporation.|Värdet är samma som i SQL Database. Detta indikerar **inte** SQL-motor version 12 (SQL Server 2014). Den hanterade instansen kör alltid den senaste säkra versionen av SQL-motorn, som är lika med eller högre än den senaste tillgängliga RTM-versionen av SQL Server.  |
|`SERVERPROPERTY ('Edition')`|SQL Azure|Värdet är samma som i SQL Database.|
|`SERVERPROPERTY('EngineEdition')`|8|Det här värdet identifierar en hanterad instans unikt.|
|`@@SERVERNAME`, `SERVERPROPERTY ('ServerName')`|Fullständigt instans-DNS-namn i följande format: `<instanceName>` . `<dnsPrefix>` . database.windows.net, där `<instanceName>` är namnet som tillhandahålls av kunden, medan `<dnsPrefix>` är automatiskt genererad del av namnet som garanterar unikt DNS-namn ("wcus17662feb9ce98", till exempel)|Exempel: my-managed-instance.wcus17662feb9ce98.database.windows.net|

## <a name="next-steps"></a>Nästa steg

- Information om hur du skapar din första hanterade instans finns i [snabb starts guide](sql-database-managed-instance-get-started.md).
- En funktion och en jämförelse lista finns i [vanliga SQL-funktioner i SQL](sql-database-features.md).
- Mer information om VNet-konfiguration finns i [konfiguration av VNet-instansen VNet](sql-database-managed-instance-connectivity-architecture.md).
- En snabb start som skapar en hanterad instans och återställer en databas från en säkerhets kopia finns i [skapa en hanterad instans](sql-database-managed-instance-get-started.md).
- En själv studie kurs om hur du använder Azure Database Migration Service (DMS) för migrering finns i [migrering av hanterade instanser med DMS](../dms/tutorial-sql-server-to-managed-instance.md).
- Avancerad övervakning av hanterade instanser av databas prestanda med inbyggd fel söknings information finns i [övervaka Azure SQL Database att använda Azure SQL-analys](../azure-monitor/insights/azure-sql.md).
- Pris information finns i [priser för SQL Database Managed instance](https://azure.microsoft.com/pricing/details/sql-database/managed/).
