---
title: Hanterade instanser T-SQL-skillnader
description: I den här artikeln beskrivs skillnader i T-SQL mellan en hanterad instans i Azure SQL Database och SQL Server
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein, carlrab, bonova, danil
ms.date: 03/11/2020
ms.custom: seoapril2019
ms.openlocfilehash: e01c61ca4f415ffbb46c86034d4b7441bc2617d9
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80365487"
---
# <a name="managed-instance-t-sql-differences-and-limitations"></a>Hanterade instanser i T-SQL-skillnader och begränsningar

Den här artikeln sammanfattar och förklarar skillnaderna i syntax och beteende mellan Azure SQL Database Hanterad instans och lokala SQL Server databas motor. Distributions alternativet för hanterade instanser ger hög kompatibilitet med lokala SQL Server databas motor. De flesta av funktionerna i SQL Server-databasmotorn stöds i en hanterad instans.

![Migrering](./media/sql-database-managed-instance/migration.png)

Det finns vissa PaaS-begränsningar som introduceras i den hanterade instansen och vissa beteende ändringar jämfört med SQL Server. Skillnaderna är indelade i följande kategorier:<a name="Differences"></a>

- [Tillgänglighet](#availability) inkluderar skillnaderna i [Always on-tillgänglighetsgrupper](#always-on-availability-groups) och [säkerhets kopior](#backup).
- [Säkerhet](#security) omfattar skillnaderna i [granskning](#auditing), [certifikat](#certificates), [autentiseringsuppgifter](#credential), [kryptografiproviders](#cryptographic-providers), [inloggningar och användare](#logins-and-users)samt [tjänst nyckeln och tjänstens huvud nyckel](#service-key-and-service-master-key).
- [Konfigurationen](#configuration) inkluderar skillnaderna i [tillägg för buffertpooltillägget](#buffer-pool-extension), [sortering](#collation), [kompatibilitetsnivå](#compatibility-levels), [databas spegling](#database-mirroring), [databas alternativ](#database-options), [SQL Server Agent](#sql-server-agent)och [tabell alternativ](#tables).
- [Funktionerna](#functionalities) omfattar [bulk INSERT/OpenRowSet](#bulk-insert--openrowset), [CLR](#clr), [DBCC](#dbcc), [distribuerade transaktioner](#distributed-transactions), [utökade händelser](#extended-events), [externa bibliotek](#external-libraries), [FILESTREAM och FileTable](#filestream-and-filetable), [hel texts semantisk sökning](#full-text-semantic-search), [länkade servrar](#linked-servers), [PolyBase](#polybase), [replikering](#replication), [återställning](#restore-statement), [Service Broker](#service-broker), [lagrade procedurer, funktioner och utlösare](#stored-procedures-functions-and-triggers).
- [Miljö inställningar](#Environment) som virtuella nätverk och under näts konfiguration.

De flesta av dessa funktioner är arkitektur begränsningar och representerar tjänst funktioner.

Tillfälliga kända problem som upptäcks i den hanterade instansen och som kommer att lösas i framtiden beskrivs på [sidan viktig information](sql-database-release-notes.md).

## <a name="availability"></a>Tillgänglighet

### <a name="always-on-availability-groups"></a><a name="always-on-availability-groups"></a>Always on-tillgänglighetsgrupper

[Hög tillgänglighet](sql-database-high-availability.md) är inbyggt i en hanterad instans och kan inte styras av användare. Följande uttryck stöds inte:

- [SKAPA SLUT PUNKT... FÖR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql)
- [SKAPA TILLGÄNGLIGHETS GRUPP](/sql/t-sql/statements/create-availability-group-transact-sql)
- [ÄNDRA TILLGÄNGLIGHETS GRUPP](/sql/t-sql/statements/alter-availability-group-transact-sql)
- [SLÄPP TILLGÄNGLIGHETS GRUPP](/sql/t-sql/statements/drop-availability-group-transact-sql)
- [Set hadr](/sql/t-sql/statements/alter-database-transact-sql-set-hadr) -satsen i [Alter Database](/sql/t-sql/statements/alter-database-transact-sql) -instruktionen

### <a name="backup"></a>Backup

Hanterade instanser har automatiska säkerhets kopieringar, så användare kan `COPY_ONLY` skapa fullständiga säkerhets kopior av databasen. Säkerhets kopiering av differentiella, loggade och ögonblicks bilder stöds inte.

- Med en hanterad instans kan du bara säkerhetskopiera en instans databas till ett Azure Blob Storage-konto:
  - Stöds `BACKUP TO URL` endast.
  - `FILE`, `TAPE`-och säkerhets kopierings enheter stöds inte.
- De flesta allmänna `WITH` alternativ stöds.
  - `COPY_ONLY`är obligatoriskt.
  - `FILE_SNAPSHOT`stöds inte.
  - Band alternativ: `REWIND`, `NOREWIND` `UNLOAD`, och `NOUNLOAD` stöds inte.
  - Användarspecifika alternativ: `NORECOVERY`, `STANDBY`, och `NO_TRUNCATE` stöds inte.

Begränsningar: 

- Med en hanterad instans kan du säkerhetskopiera en instans databas till en säkerhets kopia med upp till 32 Stripes, vilket är tillräckligt för databaser upp till 4 TB om komprimering av säkerhets kopia används.
- Det går inte `BACKUP DATABASE ... WITH COPY_ONLY` att köra på en databas som är krypterad med service-hanterad transparent DATAKRYPTERING (TDE). Service-Managed TDE tvingar säkerhets kopieringarna att krypteras med en intern TDE-nyckel. Det går inte att exportera nyckeln, så du kan inte återställa säkerhets kopian. Använd automatisk säkerhets kopiering och återställning av tidpunkter, eller Använd [kundhanterad (BYOK) TDE](transparent-data-encryption-azure-sql.md#customer-managed-transparent-data-encryption---bring-your-own-key) i stället. Du kan också inaktivera kryptering på databasen.
- Den största storleken för säkerhets kopierings stripe `BACKUP` med kommandot i en hanterad instans är 195 GB, vilket är den maximala BLOB-storleken. Öka antalet ränder i säkerhets kopierings kommandot för att minska storleken på enskilda stripe-volymer och håll dig inom den här gränsen.

    > [!TIP]
    > För att undvika den här begränsningen när du säkerhetskopierar en databas från antingen SQL Server i en lokal miljö eller på en virtuell dator kan du:
    >
    > - Säkerhetskopiera till `DISK` i stället för att säkerhetskopiera till `URL`.
    > - Överför säkerhetskopieringsfilerna till Blob Storage.
    > - Återställ till den hanterade instansen.
    >
    > `Restore` Kommandot i en hanterad instans har stöd för större blob-storlekar i säkerhetskopieringsfilerna eftersom en annan Blob-typ används för lagring av de uppladdade säkerhets kopiorna.

Information om säkerhets kopior med T-SQL finns i [säkerhets kopiering](/sql/t-sql/statements/backup-transact-sql).

## <a name="security"></a>Säkerhet

### <a name="auditing"></a>Granskning

De viktigaste skillnaderna mellan granskning i databaser i Azure SQL Database och databaser i SQL Server är:

- Med alternativet för distribution av hanterad instans i Azure SQL Database fungerar granskning på server nivå. `.xel` Loggfilerna lagras i Azure Blob Storage.
- Med distributions alternativen enkel databas och elastisk pool i Azure SQL Database, fungerar granskning på databas nivå.
- I SQL Server lokala eller virtuella datorer fungerar granskning på server nivå. Händelser lagras i fil systemet eller i Windows-händelseloggen.
 
XEvent-granskning i hanterade instanser stöder Azure Blob Storage-mål. Fil-och Windows-loggar stöds inte.

Viktiga skillnader i `CREATE AUDIT` syntaxen för granskning till Azure Blob Storage är:

- Det finns en `TO URL` ny syntax som du kan använda för att ange URL: en för Azure Blob storage-behållaren `.xel` där filerna placeras.
- Syntaxen `TO FILE` stöds inte eftersom en hanterad instans inte kan komma åt Windows-filresurser.

Mer information finns i: 

- [SKAPA SERVER GRANSKNING](/sql/t-sql/statements/create-server-audit-transact-sql) 
- [ALTER SERVER AUDIT](/sql/t-sql/statements/alter-server-audit-transact-sql)
- [Granskning](/sql/relational-databases/security/auditing/sql-server-audit-database-engine)

### <a name="certificates"></a>Certifikat

En hanterad instans kan inte komma åt fil resurser och Windows-mappar, så följande villkor gäller:

- `CREATE FROM` / `BACKUP TO`
- `CREATE` / Certifikatet från `ASSEMBLY` stöds inte. `FILE` / `BACKUP` Det går inte att använda filer för privata nycklar. 

Se [Skapa certifikat](/sql/t-sql/statements/create-certificate-transact-sql) -och [säkerhets kopierings certifikat](/sql/t-sql/statements/backup-certificate-transact-sql). 
 
**Lösning**: i stället för att skapa en säkerhets kopia av certifikatet och återställa säkerhets kopian [hämtar du det binära innehållet och den privata nyckeln, lagrar det som. SQL-fil och skapar från binär](/sql/t-sql/functions/certencoded-transact-sql#b-copying-a-certificate-to-another-database):

```sql
CREATE CERTIFICATE  
   FROM BINARY = asn_encoded_certificate
WITH PRIVATE KEY (<private_key_options>)
```

### <a name="credential"></a>Autentiseringsuppgift

Endast Azure Key Vault och `SHARED ACCESS SIGNATURE` identiteter stöds. Windows-användare stöds inte.

Se [skapa autentiseringsuppgifter](/sql/t-sql/statements/create-credential-transact-sql) och [ändra autentiseringsuppgifter](/sql/t-sql/statements/alter-credential-transact-sql).

### <a name="cryptographic-providers"></a>Kryptografiproviders

En hanterad instans kan inte komma åt filer, så det går inte att skapa kryptografiska providers:

- `CREATE CRYPTOGRAPHIC PROVIDER`stöds inte. Se [skapa kryptografiprovider](/sql/t-sql/statements/create-cryptographic-provider-transact-sql).
- `ALTER CRYPTOGRAPHIC PROVIDER`stöds inte. Se [Alter CRYPTOGRAPHIC Provider](/sql/t-sql/statements/alter-cryptographic-provider-transact-sql).

### <a name="logins-and-users"></a>Inloggningar och användare

- SQL-inloggningar som skapats `FROM CERTIFICATE`med `FROM ASYMMETRIC KEY`hjälp av `FROM SID` , och stöds. Se [Skapa inloggning](/sql/t-sql/statements/create-login-transact-sql).
- Azure Active Directory (inloggnings uppgifter för Azure AD) som skapats med syntaxen [create login](/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current) eller [create User from login [Azure AD login]](/sql/t-sql/statements/create-user-transact-sql?view=azuresqldb-mi-current) stöds. Dessa inloggningar skapas på server nivå.

    Den hanterade instansen stöder Azure AD Database- `CREATE USER [AADUser/AAD group] FROM EXTERNAL PROVIDER`huvudobjekt med syntaxen. Den här funktionen kallas även för Azure AD-inneslutna databas användare.

- Windows-inloggningar som skapats `CREATE LOGIN ... FROM WINDOWS` med syntaxen stöds inte. Använd Azure Active Directory inloggningar och användare.
- Den Azure AD-användare som skapade instansen har [obegränsade administratörs privilegier](sql-database-manage-logins.md).
- Användare som inte har administratörs behörighet på Azure AD kan skapas med hjälp av `CREATE USER ... FROM EXTERNAL PROVIDER` syntaxen. Se [skapa användare... FRÅN extern PROVIDER](sql-database-aad-authentication-configure.md#create-contained-database-users-in-your-database-mapped-to-azure-ad-identities).
- Azure AD server-Huvudkonton (inloggningar) stöder endast SQL-funktioner inom en hanterad instans. Funktioner som kräver interaktion mellan olika instanser, oavsett om de ligger inom samma Azure AD-klient eller olika klienter, stöds inte för Azure AD-användare. Exempel på sådana funktioner är:

  - SQL-transaktionell replikering.
  - Länka Server.

- Att ange en Azure AD-inloggning som är mappad till en Azure AD-grupp eftersom databas ägaren inte stöds.
- Personifiering av Azure AD-huvudobjekt på server nivå med hjälp av andra Azure AD-huvudobjekt stöds, till exempel [execute as](/sql/t-sql/statements/execute-as-transact-sql) -satsen. Kör som-begränsningar är:

  - Kör som-användare stöds inte för Azure AD-användare när namnet skiljer sig från inloggnings namnet. Ett exempel är när användaren skapas med syntaxen CREATE USER [myAadUser] FROM LOGIn [john@contoso.com] och personifiering görs i exec as User = _myAadUser_. När du skapar en **användare** från ett Azure AD server-huvudobjekt (inloggning) anger du user_name som samma Login_name från **inloggningen**.
  - Endast SQL Server nivå huvud konton (inloggningar) som är en del av `sysadmin` rollen kan köra följande åtgärder som är riktade till Azure AD-huvud konton:

    - KÖRA SOM ANVÄNDARE
    - KÖRA SOM INLOGGNING

- Databas export/import med BACPAC-filer stöds för Azure AD-användare i hanterade instanser med antingen [SSMS v 18.4 eller senare](/sql/ssms/download-sql-server-management-studio-ssms), eller [SQLPackage. exe](/sql/tools/sqlpackage-download).
  - Följande konfigurationer stöds med hjälp av databasen BACPAC-fil: 
    - Exportera/importera en databas mellan olika hanterings instanser i samma Azure AD-domän.
    - Exportera en databas från en hanterad instans och importera till SQL Database inom samma Azure AD-domän. 
    - Exportera en databas från SQL Database och importera till en hanterad instans inom samma Azure AD-domän.
    - Exportera en databas från en hanterad instans och importera till SQL Server (version 2012 eller senare).
      - I den här konfigurationen skapas alla Azure AD-användare som SQL Database-Huvudkonton (användare) utan inloggningar. Typ av användare visas som SQL (synlig som SQL_USER i sys. database_principals). Deras behörigheter och roller finns kvar i SQL Server metadata för databasen och kan användas för personifiering. De kan dock inte användas för att komma åt och logga in på SQL Server med sina autentiseringsuppgifter.

- Endast huvud inloggningen på server nivå, som skapas av den hanterade instans etablerings processen, medlemmar i Server rollerna, till exempel `securityadmin` eller `sysadmin`eller andra INLOGGNINGar med ändra inloggnings behörighet på server nivå, kan skapa Azure AD server-Huvudkonton (inloggningar) i Master-databasen för hanterad instans.
- Om inloggningen är ett SQL-huvud kan endast inloggningar som är en del `sysadmin` av rollen använda kommandot Skapa för att skapa inloggningar för ett Azure AD-konto.
- Azure AD-inloggningen måste vara medlem i en Azure AD i samma katalog som används för Azure SQL Database Hanterad instans.
- Azure AD server-Huvudkonton (inloggningar) visas i Object Explorer som börjar med SQL Server Management Studio 18,0 Preview 5.
- Överlappande Azure AD server-huvudobjekt (inloggningar) med ett administratörs konto för Azure AD tillåts. Azure AD server-Huvudkonton (inloggningar) prioriteras över Azure AD-administratören när du löser huvud kontot och tillämpar behörigheter på den hanterade instansen.
- Under autentiseringen används följande sekvens för att lösa det autentiserande huvudobjektet:

    1. Om Azure AD-kontot finns som direkt mappat till Azure AD server-huvudobjektet (inloggning) som finns i sys. server_principals som Skriv "E", bevilja åtkomst och tillämpa behörigheter för Azure AD server-huvudobjektet (inloggning).
    2. Om Azure AD-kontot är medlem i en Azure AD-grupp som är mappad till Azure AD server-huvudobjektet (inloggning) som finns i sys. server_principals som typen "X", bevilja åtkomst och tillämpa behörigheter för Azure AD-gruppinloggningen.
    3. Om Azure AD-kontot är en särskild Portal-konfigurerad Azure AD-administratör för en hanterad instans som inte finns i systemvyer för hanterade instanser, ska du använda särskilda fasta behörigheter för Azure AD-administratören för en hanterad instans (bakåtkompatibelt läge).
    4. Om Azure AD-kontot finns som direkt mappat till en Azure AD-användare i en databas, som finns i sys. database_principals som Skriv "E", bevilja åtkomst och tillämpa behörigheter för Azure AD Database-användaren.
    5. Om Azure AD-kontot är medlem i en Azure AD-grupp som är mappad till en Azure AD-användare i en databas som finns i sys. database_principals som typen "X", bevilja åtkomst och tillämpa behörigheter för Azure AD-gruppinloggningen.
    6. Om det finns en Azure AD-inloggning som är mappad till antingen ett Azure AD-användarkonto eller ett Azure AD-gruppkonto, som matchar den användare som autentiseras, används alla behörigheter från den här Azure AD-inloggningen.

### <a name="service-key-and-service-master-key"></a>Tjänst nyckel och tjänstens huvud nyckel

- [Säkerhets kopiering av huvud nycklar](/sql/t-sql/statements/backup-master-key-transact-sql) stöds inte (hanteras av SQL Database tjänsten).
- [Master Key Restore](/sql/t-sql/statements/restore-master-key-transact-sql) stöds inte (hanteras av SQL Database tjänsten).
- [Säkerhets kopiering av tjänstens huvud nyckel](/sql/t-sql/statements/backup-service-master-key-transact-sql) stöds inte (hanteras av SQL Database tjänsten).
- Det finns inte stöd för [återställning av tjänstens huvud nyckel](/sql/t-sql/statements/restore-service-master-key-transact-sql) (hanteras av SQL Database tjänsten).

## <a name="configuration"></a>Konfiguration

### <a name="buffer-pool-extension"></a>Buffertpooltillägget

- [Buffertpooltillägget](/sql/database-engine/configure-windows/buffer-pool-extension) stöds inte.
- `ALTER SERVER CONFIGURATION SET BUFFER POOL EXTENSION`stöds inte. Se [Alter Server Configuration](/sql/t-sql/statements/alter-server-configuration-transact-sql).

### <a name="collation"></a>Sortering

Standard instans sorteringen är `SQL_Latin1_General_CP1_CI_AS` och kan anges som en skapande parameter. Se [sorteringar](/sql/t-sql/statements/collations).

### <a name="compatibility-levels"></a>Efterlevnadsnivåer

- De kompatibilitetsnivå som stöds är 100, 110, 120, 130, 140 och 150.
- Kompatibilitetsnivån under 100 stöds inte.
- Standard kompatibilitetsnivån för nya databaser är 140. För återställda databaser förblir kompatibilitetsnivån oförändrad om den var 100 och högre.

Se [ändra kompatibilitetsnivån för databas](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level).

### <a name="database-mirroring"></a>Databasspegling

Databas spegling stöds inte.

- `ALTER DATABASE SET PARTNER`och `SET WITNESS` -alternativ stöds inte.
- `CREATE ENDPOINT … FOR DATABASE_MIRRORING`stöds inte.

Mer information finns i [ändra databas uppsättnings partner och ange vittne](/sql/t-sql/statements/alter-database-transact-sql-database-mirroring) och [skapa slut punkt... FÖR DATABASE_MIRRORING](/sql/t-sql/statements/create-endpoint-transact-sql).

### <a name="database-options"></a>Databas alternativ

- Flera loggfiler stöds inte.
- InMemory-objekt stöds inte i Generell användning tjänst nivå. 
- Det finns en gräns på 280 filer per Generell användning instans, vilket innebär högst 280 filer per databas. Både data-och loggfiler på Generell användning nivån räknas mot den här gränsen. [Affärskritisk nivån stöder 32 767-filer per databas](https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance-resource-limits#service-tier-characteristics).
- Databasen får inte innehålla fil grupper som innehåller FILESTREAM-data. Återställningen Miss lyckas om `FILESTREAM` . bak innehåller data. 
- Varje fil placeras i Azure Blob Storage. I/o och data flöde per fil beror på storleken på varje enskild fil.

#### <a name="create-database-statement"></a>SKAPA databas uttryck

Följande begränsningar gäller för `CREATE DATABASE`:

- Det går inte att definiera filer och fil grupper. 
- `CONTAINMENT` Alternativet stöds inte. 
- `WITH`alternativ stöds inte. 
   > [!TIP]
   > Som en lösning använder `ALTER DATABASE` du efter `CREATE DATABASE` för att ange databas alternativ för att lägga till filer eller för att ange inne slutning. 

- `FOR ATTACH` Alternativet stöds inte.
- `AS SNAPSHOT OF` Alternativet stöds inte.

Mer information finns i [skapa databas](/sql/t-sql/statements/create-database-sql-server-transact-sql).

#### <a name="alter-database-statement"></a>Instruktionen ALTER DATABASE

Vissa fil egenskaper kan inte ställas in eller ändras:

- Det går inte att ange en fil Sök `ALTER DATABASE ADD FILE (FILENAME='path')` väg i t-SQL-instruktionen. Ta `FILENAME` bort från skriptet eftersom filerna placeras automatiskt i en hanterad instans. 
- Ett fil namn kan inte ändras med `ALTER DATABASE` instruktionen.

Följande alternativ är inställda som standard och kan inte ändras:

- `MULTI_USER`
- `ENABLE_BROKER ON`
- `AUTO_CLOSE OFF`

Följande alternativ kan inte ändras:

- `AUTO_CLOSE`
- `AUTOMATIC_TUNING(CREATE_INDEX=ON|OFF)`
- `AUTOMATIC_TUNING(DROP_INDEX=ON|OFF)`
- `DISABLE_BROKER`
- `EMERGENCY`
- `ENABLE_BROKER`
- `FILESTREAM`
- `HADR`
- `NEW_BROKER`
- `OFFLINE`
- `PAGE_VERIFY`
- `PARTNER`
- `READ_ONLY`
- `RECOVERY BULK_LOGGED`
- `RECOVERY_SIMPLE`
- `REMOTE_DATA_ARCHIVE` 
- `RESTRICTED_USER`
- `SINGLE_USER`
- `WITNESS`

Mer information finns i [Alter Database](/sql/t-sql/statements/alter-database-transact-sql-file-and-filegroup-options).

### <a name="sql-server-agent"></a>SQL Server Agent

- Det finns för närvarande inte stöd för att aktivera och inaktivera SQL Server Agent i en hanterad instans. SQL Agent körs alltid.
- SQL Server Agent inställningarna är skrivskyddade. Proceduren `sp_set_agent_properties` stöds inte i hanterade instanser. 
- Jobb
  - Steg för T-SQL-jobb stöds.
  - Följande migreringsjobb stöds:
    - Transaktions logg läsare
    - Ögonblicksbild
    - Möjligheter
  - SSIS jobb steg stöds.
  - Andra typer av jobb-steg stöds inte för närvarande:
    - Jobb steget för Sammanslagningsreplikering stöds inte. 
    - Queue Reader stöds inte. 
    - Kommando gränssnittet stöds inte ännu.
  - Hanterade instanser kan inte komma åt externa resurser, till exempel nätverks resurser via Robocopy. 
  - SQL Server Analysis Services stöds inte.
- Meddelanden stöds delvis.
- E-postavisering stöds, även om det krävs att du konfigurerar en Database Mail-profil. SQL Server Agent kan bara använda en Database Mail profil och den måste anropas `AzureManagedInstance_dbmail_profile`. 
  - Pager stöds inte.
  - Netsend stöds inte.
  - Aviseringar stöds inte än.
  - Proxyservrar stöds inte.
- EventLog stöds inte.

Följande SQL Agent-funktioner stöds för närvarande inte:

- Proxy
- Schemalägga jobb på en inaktiv processor
- Aktivera eller inaktivera en agent
- Aviseringar

Information om SQL Server Agent finns i [SQL Server Agent](/sql/ssms/agent/sql-server-agent).

### <a name="tables"></a>Tabeller

Följande tabell typer stöds inte:

- [-](/sql/relational-databases/blob/filestream-sql-server)
- [FILETABLE](/sql/relational-databases/blob/filetables-sql-server)
- [Extern tabell](/sql/t-sql/statements/create-external-table-transact-sql) (PolyBase)
- [MEMORY_OPTIMIZED](/sql/relational-databases/in-memory-oltp/introduction-to-memory-optimized-tables) (stöds inte endast i generell användning-nivå)

Information om hur du skapar och ändrar tabeller finns i [CREATE TABLE](/sql/t-sql/statements/create-table-transact-sql) och [ändra tabell](/sql/t-sql/statements/alter-table-transact-sql).

## <a name="functionalities"></a>Funktioner

### <a name="bulk-insert--openrowset"></a>Mass infogning/OpenRowSet

En hanterad instans kan inte komma åt fil resurser och Windows-mappar, så filerna måste importeras från Azure Blob Storage:

- `DATASOURCE`krävs i `BULK INSERT` kommandot när du importerar filer från Azure Blob Storage. Se [bulk INSERT](/sql/t-sql/statements/bulk-insert-transact-sql).
- `DATASOURCE`krävs i `OPENROWSET` funktionen när du läser innehållet i en fil från Azure Blob Storage. Se [OpenRowSet](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENROWSET`kan användas för att läsa data från andra Azure SQL-databaser, hanterade instanser eller SQL Server instanser. Andra källor som Oracle-databaser eller Excel-filer stöds inte.

### <a name="clr"></a>CLR

En hanterad instans kan inte komma åt fil resurser och Windows-mappar, så följande villkor gäller:

- Stöds `CREATE ASSEMBLY FROM BINARY` endast. Se [skapa assem bly från Binary](/sql/t-sql/statements/create-assembly-transact-sql). 
- `CREATE ASSEMBLY FROM FILE`stöds inte. Se [skapa sammansättning från fil](/sql/t-sql/statements/create-assembly-transact-sql).
- `ALTER ASSEMBLY`Det går inte att referera till filer. Se [Alter Assembly](/sql/t-sql/statements/alter-assembly-transact-sql).

### <a name="database-mail-db_mail"></a>Database Mail (db_mail)
 - `sp_send_dbmail`Det går inte att @file_attachments skicka bilagor med hjälp av parametern. Lokalt fil system och externa resurser eller Azure Blob Storage kan inte nås från den här proceduren.
 - Se kända problem som `@query` rör parametrar och autentisering.
 
### <a name="dbcc"></a>DBCC

Inte dokumenterade DBCC-instruktioner som är aktiverade i SQL Server stöds inte i hanterade instanser.

- Endast ett begränsat antal globala spårnings flaggor stöds. Session-Level `Trace flags` stöds inte. Se [spårnings flaggor](/sql/t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql).
- [DBCC TRACEOFF](/sql/t-sql/database-console-commands/dbcc-traceoff-transact-sql) och [DBCC TRACEON](/sql/t-sql/database-console-commands/dbcc-traceon-transact-sql) fungerar med det begränsade antalet globala spårnings flaggor.
- Det går inte att använda [DBCC CHECKDB](/sql/t-sql/database-console-commands/dbcc-checkdb-transact-sql) med alternativ REPAIR_ALLOW_DATA_LOSS, REPAIR_FAST och REPAIR_REBUILD eftersom databasen inte kan anges `SINGLE_USER` i läge – se [Alter Database-skillnader](#alter-database-statement). Potentiella databas fel hanteras av support teamet för Azure. Kontakta Azure-supporten om du är märker databas skada som bör åtgärdas.

### <a name="distributed-transactions"></a>Distribuerade transaktioner

MSDTC-och [elastiska transaktioner](sql-database-elastic-transactions-overview.md) stöds för närvarande inte i hanterade instanser.

### <a name="extended-events"></a>Extended Events

Vissa Windows-/regionsspecifika mål för utökade händelser (XEvents) stöds inte:

- `etw_classic_sync` Målet stöds inte. Lagra `.xel` filer i Azure Blob Storage. Se [etw_classic_sync mål](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#etw_classic_sync_target-target).
- `event_file` Målet stöds inte. Lagra `.xel` filer i Azure Blob Storage. Se [event_file mål](/sql/relational-databases/extended-events/targets-for-extended-events-in-sql-server#event_file-target).

### <a name="external-libraries"></a>Externa bibliotek

I Database R och python stöds inte externa bibliotek ännu. Se [SQL Server Machine Learning Services](/sql/advanced-analytics/r/sql-server-r-services).

### <a name="filestream-and-filetable"></a>FILESTREAM och FileTable

- FILESTREAM-data stöds inte.
- Databasen får inte innehålla fil grupper `FILESTREAM` med data.
- `FILETABLE`stöds inte.
- Tabeller kan inte `FILESTREAM` ha typer.
- Följande funktioner stöds inte:
  - `GetPathLocator()`
  - `GET_FILESTREAM_TRANSACTION_CONTEXT()`
  - `PathName()`
  - `GetFileNamespacePat)`
  - `FileTableRootPath()`

Mer information finns i [FILESTREAM](/sql/relational-databases/blob/filestream-sql-server) och [FileTable](/sql/relational-databases/blob/filetables-sql-server).

### <a name="full-text-semantic-search"></a>Semantisk sökning i full text

[Semantisk sökning](/sql/relational-databases/search/semantic-search-sql-server) stöds inte.

### <a name="linked-servers"></a>Länkade servrar

Länkade servrar i hanterade instanser har stöd för ett begränsat antal mål:

- Mål som stöds är hanterade instanser, enskilda databaser och SQL Server instanser. 
- Länkade servrar har inte stöd för distribuerade skrivbara transaktioner (MS DTC).
- Mål som inte stöds är filer, Analysis Services och andra RDBMS. Försök att använda intern CSV-import från Azure Blob Storage `BULK INSERT` att `OPENROWSET` använda eller som ett alternativ för fil import.

Åtgärder

- Skriv transaktioner över instanser stöds inte.
- `sp_dropserver`stöds för att släppa en länkad server. Se [sp_dropserver](/sql/relational-databases/system-stored-procedures/sp-dropserver-transact-sql).
- `OPENROWSET` Funktionen kan endast användas för att köra frågor på SQL Server instanser. De kan antingen hanteras, lokalt eller på virtuella datorer. Se [OpenRowSet](/sql/t-sql/functions/openrowset-transact-sql).
- `OPENDATASOURCE` Funktionen kan endast användas för att köra frågor på SQL Server instanser. De kan antingen hanteras, lokalt eller på virtuella datorer. Endast `SQLNCLI`-, `SQLNCLI11`-och `SQLOLEDB` -värden stöds som en provider. Ett exempel är `SELECT * FROM OPENDATASOURCE('SQLNCLI', '...').AdventureWorks2012.HumanResources.Employee`. Se [OpenDataSource](/sql/t-sql/functions/opendatasource-transact-sql).
- Det går inte att använda länkade servrar för att läsa filer (Excel, CSV) från nätverks resurserna. Försök att använda [bulk INSERT](/sql/t-sql/statements/bulk-insert-transact-sql#e-importing-data-from-a-csv-file) eller [OpenRowSet](/sql/t-sql/functions/openrowset-transact-sql#g-accessing-data-from-a-csv-file-with-a-format-file) som läser CSV-filer från Azure Blob Storage. Spåra dessa förfrågningar på [feedback-objektet för hanterade instanser](https://feedback.azure.com/forums/915676-sql-managed-instance/suggestions/35657887-linked-server-to-non-sql-sources)|

### <a name="polybase"></a>PolyBase

Externa tabeller som refererar till filerna i HDFS eller Azure Blob Storage stöds inte. Information om PolyBase finns i [PolyBase](/sql/relational-databases/polybase/polybase-guide).

### <a name="replication"></a>Replikering

- Ögonblicks bilder och dubbelriktade typer av replikering stöds. Sammanslagningsreplikering, peer-to-peer-replikering och uppdaterings bara prenumerationer stöds inte.
- [Transaktionell replikering](sql-database-managed-instance-transactional-replication.md) är tillgänglig för offentlig för hands version på en hanterad instans med vissa begränsningar:
    - Alla typer av replikeringspartner (utgivare, distributör, pull-prenumerant och push-prenumerant) kan placeras på hanterade instanser, men utgivaren och distributören måste antingen vara både i molnet eller både lokalt.
    - Hanterade instanser kan kommunicera med de senaste versionerna av SQL Server. Mer information finns i [matrisen med versioner som stöds](sql-database-managed-instance-transactional-replication.md#supportability-matrix-for-instance-databases-and-on-premises-systems) .
    - Transaktionsreplikering har vissa [ytterligare nätverks krav](sql-database-managed-instance-transactional-replication.md#requirements).

Mer information om hur du konfigurerar Transaktionsreplikering finns i följande Självstudier:
- [Replikering mellan en MI-utgivare och prenumerant](replication-with-sql-database-managed-instance.md)
- [Replikering mellan en MI-utgivare, MI-distributör och SQL Server prenumerant](sql-database-managed-instance-configure-replication-tutorial.md)

### <a name="restore-statement"></a>Instruktionen Restore 

- Syntax som stöds:
  - `RESTORE DATABASE`
  - `RESTORE FILELISTONLY ONLY`
  - `RESTORE HEADER ONLY`
  - `RESTORE LABELONLY ONLY`
  - `RESTORE VERIFYONLY ONLY`
- Syntax som inte stöds:
  - `RESTORE LOG ONLY`
  - `RESTORE REWINDONLY ONLY`
- Källa: 
  - `FROM URL`(Azure Blob Storage) är det enda alternativ som stöds.
  - `FROM DISK`/`TAPE`/backup-enheten stöds inte.
  - Säkerhets kopierings uppsättningar stöds inte.
- `WITH`alternativen stöds inte, till exempel Nej `DIFFERENTIAL` eller `STATS`.
- `ASYNC RESTORE`: Restore fortsätter även om klient anslutningen bryts. Om anslutningen bryts, kan du kontrol lera `sys.dm_operation_status` status för en återställnings åtgärd och för att skapa och släppa en databas. Se [sys. dm_operation_status](/sql/relational-databases/system-dynamic-management-views/sys-dm-operation-status-azure-sql-database). 

Följande databas alternativ anges eller åsidosätts och kan inte ändras senare: 

- `NEW_BROKER`om Service Broker inte är aktive rad i. bak-filen. 
- `ENABLE_BROKER`om Service Broker inte är aktive rad i. bak-filen. 
- `AUTO_CLOSE=OFF`om en databas i. bak-filen har `AUTO_CLOSE=ON`. 
- `RECOVERY FULL`om en databas i. bak-filen har `SIMPLE` eller `BULK_LOGGED` återställnings läge.
- En minnesoptimerade fil grupp läggs till och anropas XTP om den inte finns i filen Source. bak. 
- Alla befintliga minnesoptimerade fil grupper byter namn till XTP. 
- `SINGLE_USER`och `RESTRICTED_USER` alternativ konverteras till `MULTI_USER`.

Begränsningar: 

- Säkerhets kopior av skadade databaser kan återställas beroende på typen av skada, men automatiska säkerhets kopieringar görs inte förrän skadan har åtgärd ATS. Kontrol lera att du kör `DBCC CHECKDB` på käll instansen och Använd `WITH CHECKSUM` säkerhets kopiering för att förhindra det här problemet.
- Återställning av `.BAK` filen för en databas som innehåller en begränsning som beskrivs i det här dokumentet (till `FILESTREAM` exempel `FILETABLE` eller objekt) kan inte återställas på den hanterade instansen.
- `.BAK`Det går inte att återställa filer som innehåller flera säkerhets kopierings uppsättningar. 
- `.BAK`filer som innehåller flera loggfiler kan inte återställas.
- Säkerhets kopior som innehåller databaser som är större än 8 TB, aktiva InMemory OLTP-objekt eller antal filer som skulle överskrida 280 filer per instans kan inte återställas på en Generell användning instans. 
- Säkerhets kopior som innehåller databaser som är större än 4 TB eller InMemory OLTP-objekt med Total storlek som är större än den storlek som beskrivs i [resurs gränser](sql-database-managed-instance-resource-limits.md) kan inte återställas affärskritisk-instansen.
Information om Restore-instruktioner finns i [restore Statements](/sql/t-sql/statements/restore-statements-transact-sql).

 > [!IMPORTANT]
 > Samma begränsningar gäller för inbyggd återställnings åtgärd för tidpunkter. Generell användning databas som är större än 4 TB kan till exempel inte återställas på Affärskritisk instansen. Affärskritisk databas med InMemory OLTP-filer eller mer än 280 filer kan inte återställas på Generell användning instansen.

### <a name="service-broker"></a>Service Broker

Service Broker för överinstans stöds inte:

- `sys.routes`: Som krav måste du välja adressen från sys. routes. Adressen måste vara lokal på varje väg. Se [sys. routes](/sql/relational-databases/system-catalog-views/sys-routes-transact-sql).
- `CREATE ROUTE`: Du kan inte `CREATE ROUTE` använda `ADDRESS` med andra `LOCAL`än. Se [skapa väg](/sql/t-sql/statements/create-route-transact-sql).
- `ALTER ROUTE`: Du kan inte `ALTER ROUTE` använda `ADDRESS` med andra `LOCAL`än. Se [Alter Route](/sql/t-sql/statements/alter-route-transact-sql). 

### <a name="stored-procedures-functions-and-triggers"></a>Lagrade procedurer, funktioner och utlösare

- `NATIVE_COMPILATION`stöds inte i Generell användning nivån.
- Följande [sp_configure](/sql/relational-databases/system-stored-procedures/sp-configure-transact-sql) alternativ stöds inte: 
  - `allow polybase export`
  - `allow updates`
  - `filestream_access_level`
  - `remote access`
  - `remote data archive`
  - `remote proc trans`
- `sp_execute_external_scripts`stöds inte. Se [sp_execute_external_scripts](/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql#examples).
- `xp_cmdshell`stöds inte. Se [xp_cmdshell](/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql).
- `Extended stored procedures`stöds inte, vilket inkluderar `sp_addextendedproc`  och `sp_dropextendedproc`. Se [utökade lagrade procedurer](/sql/relational-databases/system-stored-procedures/general-extended-stored-procedures-transact-sql).
- `sp_attach_db`, `sp_attach_single_file_db`, och `sp_detach_db` stöds inte. Se [sp_attach_db](/sql/relational-databases/system-stored-procedures/sp-attach-db-transact-sql), [sp_attach_single_file_db](/sql/relational-databases/system-stored-procedures/sp-attach-single-file-db-transact-sql)och [sp_detach_db](/sql/relational-databases/system-stored-procedures/sp-detach-db-transact-sql).

### <a name="system-functions-and-variables"></a>System funktioner och variabler

Följande variabler, funktioner och vyer returnerar olika resultat:

- `SERVERPROPERTY('EngineEdition')`Returnerar värdet 8. Den här egenskapen identifierar en hanterad instans unikt. Se [SERVERPROPERTY](/sql/t-sql/functions/serverproperty-transact-sql).
- `SERVERPROPERTY('InstanceName')`returnerar NULL eftersom begreppet instans som det finns för SQL Server inte gäller för en hanterad instans. Se [SERVERPROPERTY (' instancename ')](/sql/t-sql/functions/serverproperty-transact-sql).
- `@@SERVERNAME`Returnerar ett fullständigt DNS "anslutnings bara" namn, till exempel my-managed-instance.wcus17662feb9ce98.database.windows.net. Se [@@SERVERNAME](/sql/t-sql/functions/servername-transact-sql). 
- `SYS.SERVERS`Returnerar ett fullständigt DNS "anslutnings bara" namn, t. `myinstance.domain.database.windows.net` ex. för egenskaperna "name" och "data_source". Se [sys. SERVRAR](/sql/relational-databases/system-catalog-views/sys-servers-transact-sql).
- `@@SERVICENAME`returnerar NULL eftersom begreppet tjänst som finns för SQL Server inte gäller för en hanterad instans. Se [@@SERVICENAME](/sql/t-sql/functions/servicename-transact-sql).
- `SUSER_ID`stöds. Den returnerar NULL om Azure AD-inloggningen inte finns i sys. syslogins. Se [SUSER_ID](/sql/t-sql/functions/suser-id-transact-sql). 
- `SUSER_SID`stöds inte. Felaktiga data returneras, vilket är ett tillfälligt känt problem. Se [SUSER_SID](/sql/t-sql/functions/suser-sid-transact-sql). 

## <a name="environment-constraints"></a><a name="Environment"></a>Miljö begränsningar

### <a name="subnet"></a>Undernät
-  Du kan inte placera andra resurser (till exempel virtuella datorer) i under nätet där du har distribuerat din hanterade instans. Distribuera de här resurserna med ett annat undernät.
- Under nätet måste ha tillräckligt många tillgängliga [IP-adresser](sql-database-managed-instance-connectivity-architecture.md#network-requirements). Minimivärdet är 16 och rekommendationen måste ha minst 32 IP-adresser i under nätet.
- [Tjänstens slut punkter kan inte kopplas till under nätet för den hanterade instansen](sql-database-managed-instance-connectivity-architecture.md#network-requirements). Kontrol lera att alternativet tjänst slut punkter är inaktiverat när du skapar det virtuella nätverket.
- Antalet virtuella kärnor och typer av instanser som du kan distribuera i en region har vissa [begränsningar och begränsningar](sql-database-managed-instance-resource-limits.md#regional-resource-limitations).
- Det finns vissa [säkerhets regler som måste tillämpas på under nätet](sql-database-managed-instance-connectivity-architecture.md#network-requirements).

### <a name="vnet"></a>VNET
- VNet kan distribueras med hjälp av resurs modellen – den klassiska modellen för VNet stöds inte.
- När en hanterad instans har skapats går det inte att flytta den hanterade instansen eller det virtuella nätverket till en annan resurs grupp eller prenumeration.
- Vissa tjänster, till exempel App Service miljöer, Logic Apps och hanterade instanser (som används för geo-replikering, Transaktionsreplikering eller via länkade servrar) kan inte komma åt hanterade instanser i olika regioner om deras virtuella nätverk är anslutna med [Global peering](../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers). Du kan ansluta till dessa resurser via ExpressRoute eller VNet-till-VNet via VNet-gatewayer.

### <a name="tempdb"></a>TEMPDB

Den maximala fil storleken på `tempdb` får inte vara större än 24 GB per kärna på en generell användning nivå. Den maximala `tempdb` storleken på en affärskritisk nivå begränsas av instans lagrings storleken. `Tempdb`logg filens storlek är begränsad till 120 GB på Generell användning nivån. Vissa frågor kan returnera ett fel om de behöver mer än 24 GB per kärna i `tempdb` eller om de producerar mer än 120 GB loggdata.

### <a name="msdb"></a>MSDB

Följande MSDB-scheman i den hanterade instansen måste ägas av deras respektive fördefinierade roller:

- Allmänna roller
  - TargetServersRole
- [Fasta databas roller](https://docs.microsoft.com/sql/ssms/agent/sql-server-agent-fixed-database-roles?view=sql-server-ver15)
  - SQLAgentUserRole
  - SQLAgentReaderRole
  - SQLAgentOperatorRole
- [DatabaseMail-roller](https://docs.microsoft.com/sql/relational-databases/database-mail/database-mail-configuration-objects?view=sql-server-ver15#DBProfile):
  - DatabaseMailUserRole
- [Integration Services-roller](https://docs.microsoft.com/sql/integration-services/security/integration-services-roles-ssis-service?view=sql-server-ver15):
  - db_ssisadmin
  - db_ssisltduser
  - db_ssisoperator
  
> [!IMPORTANT]
> Att ändra de fördefinierade roll namnen, schema namnen och schema ägarna av kunderna påverkar den normala driften av tjänsten. Ändringar som görs i dessa kommer att återställas till de fördefinierade värdena så snart som identifierats, eller vid nästa tjänst uppdatering senast för att säkerställa en normal tjänst åtgärd.

### <a name="error-logs"></a>Felloggar

En hanterad instans placerar utförlig information i fel loggarna. Det finns många interna system händelser som loggas i fel loggen. Använd en anpassad procedur för att läsa fel loggar som filtrerar bort vissa irrelevanta poster. Mer information finns i [hanterad instans – sp_readmierrorlog](https://blogs.msdn.microsoft.com/sqlcat/2018/05/04/azure-sql-db-managed-instance-sp_readmierrorlog/) eller [hanterade instans tillägg (för hands version)](/sql/azure-data-studio/azure-sql-managed-instance-extension#logs) för Azure Data Studio.

## <a name="next-steps"></a>Nästa steg

- Mer information om hanterade instanser finns i [Vad är en hanterad instans?](sql-database-managed-instance.md)
- En funktion och en jämförelse lista finns i [Azure SQL Database jämförelse](sql-database-features.md)av funktioner.
- Information om versions uppdateringar och kända problem finns i [SQL Database viktig information](sql-database-release-notes.md)
- En snabb start som visar hur du skapar en ny hanterad instans finns i [skapa en hanterad instans](sql-database-managed-instance-get-started.md).
