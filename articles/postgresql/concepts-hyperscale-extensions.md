---
title: Tillägg – storskalig (citus) – Azure Database for PostgreSQL
description: Beskriver möjligheten att utöka funktionaliteten i databasen med hjälp av tillägg i Azure Database for PostgreSQL-storskalig (citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/16/2020
ms.openlocfilehash: ba8f4591782a4e34fbde26d9669ef01f24450486
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82146421"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql--hyperscale-citus"></a>PostgreSQL-tillägg i Azure Database for PostgreSQL – storskalig (citus)

PostgreSQL ger möjlighet att utöka funktionaliteten i databasen med hjälp av tillägg. Tillägg tillåter flera relaterade SQL-objekt tillsammans i ett enda paket som kan läsas in eller tas bort från databasen med ett enda kommando. När de har lästs in i databasen kan tillägg fungera som inbyggda funktioner. Mer information om PostgreSQL-tillägg finns i [paket relaterade objekt till ett tillägg](https://www.postgresql.org/docs/current/static/extend-extensions.html).

## <a name="use-postgresql-extensions"></a>Använd PostgreSQL-tillägg

PostgreSQL-tillägg måste vara installerade i databasen innan du kan använda dem. Om du vill installera ett visst tillägg kör du kommandot [skapa tillägg](https://www.postgresql.org/docs/current/static/sql-createextension.html) från psql-verktyget för att läsa in de paketerade objekten i databasen.

Azure Database for PostgreSQL-Scale (citus) stöder för närvarande en delmängd av nyckel tillägg som visas här. Andra tillägg än de som listas stöds inte. Du kan inte skapa ditt eget tillägg med Azure Database for PostgreSQL.

## <a name="extensions-supported-by-azure-database-for-postgresql"></a>Tillägg som stöds av Azure Database for PostgreSQL

I följande tabeller visas de standard PostgreSQL-tillägg som för närvarande stöds av Azure Database for PostgreSQL. Den här informationen är också tillgänglig genom `SELECT * FROM pg_available_extensions;`att köra.

### <a name="data-types-extensions"></a>Tillägg för data typer

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [citext](https://www.postgresql.org/docs/current/static/citext.html) | Innehåller en Skift läges okänslig tecken sträng typ. |
> | [kuben](https://www.postgresql.org/docs/current/static/cube.html) | Tillhandahåller en datatyp för flerdimensionella kuber. |
> | [hstore](https://www.postgresql.org/docs/current/static/hstore.html) | Tillhandahåller en datatyp för lagring av uppsättningar med nyckel/värde-par. |
> | [hll](https://github.com/citusdata/postgresql-hll) | Tillhandahåller en HyperLogLog-datastruktur. |
> | [inte är](https://www.postgresql.org/docs/current/static/isn.html) | Tillhandahåller data typer för internationella standarder för produkt nummer. |
> | [Lo](https://www.postgresql.org/docs/current/lo.html) | Stort objekt underhåll. |
> | [ltree](https://www.postgresql.org/docs/current/static/ltree.html) | Tillhandahåller en datatyp för hierarkiska träd strukturer. |
> | [seg](https://www.postgresql.org/docs/current/seg.html) | Datatyp som representerar linje segment eller flytt ALS intervall. |
> | [TOPN](https://github.com/citusdata/postgresql-topn/) | Typ för Top-n-JSONB. |

### <a name="full-text-search-extensions"></a>Full texts öknings tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [dict\_int](https://www.postgresql.org/docs/current/static/dict-int.html) | Innehåller en mall för text Sök lexikon för heltal. |
> | [dict\_xsyn](https://www.postgresql.org/docs/current/dict-xsyn.html) | Mall för text Sök lexikon för utökad synonym bearbetning. |
> | [avaccent](https://www.postgresql.org/docs/current/static/unaccent.html) | En text Sök ord lista som tar bort accenttecken (dia kritiska tecken) från lexemes. |

### <a name="functions-extensions"></a>Funktions tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [autoinc](https://www.postgresql.org/docs/current/contrib-spi.html#id-1.11.7.45.7) | Funktioner för att autoöka fält. |
> | [earthdistance](https://www.postgresql.org/docs/current/static/earthdistance.html) | Ger ett sätt att beräkna fantastiska avstånd på jordens yta. |
> | [fuzzystrmatch](https://www.postgresql.org/docs/current/static/fuzzystrmatch.html) | Innehåller flera funktioner som avgör likheter och avstånd mellan strängar. |
> | [infoga\_användar namn](https://www.postgresql.org/docs/current/contrib-spi.html#id-1.11.7.45.8) | Funktioner för att spåra vem som har ändrat en tabell. |
> | [intagg](https://www.postgresql.org/docs/current/intagg.html) | Heltals Aggregator och uppräknare (föråldrad). |
> | [intarray](https://www.postgresql.org/docs/current/static/intarray.html) | Tillhandahåller funktioner och operatorer för att ändra null-fria matriser med heltal. |
> | [moddatetime](https://www.postgresql.org/docs/current/contrib-spi.html#id-1.11.7.45.9) | Funktioner för att spåra senaste ändrings tid. |
> | [pgcrypto](https://www.postgresql.org/docs/current/static/pgcrypto.html) | Tillhandahåller kryptografiska funktioner. |
> | [PG\_part](https://pgxn.org/dist/pg_partman/doc/pg_partman.html) | Hanterar partitionerade tabeller efter tid eller ID. |
> | [PG\_-trgm](https://www.postgresql.org/docs/current/static/pgtrgm.html) | Tillhandahåller funktioner och operatorer för att fastställa likheten mellan alfanumerisk text baserat på trigram matchning. |
> | [refint](https://www.postgresql.org/docs/current/contrib-spi.html#id-1.11.7.45.5) | Funktioner för att implementera referens integritet (föråldrad). |
> | analys\_av sessioner | Funktioner för att skicka frågor till hstore-matriser. |
> | [tablefunc](https://www.postgresql.org/docs/current/static/tablefunc.html) | Innehåller funktioner som ändrar hela tabeller, inklusive kors. |
> | [tcn](https://www.postgresql.org/docs/current/tcn.html) | Utlösta ändrings meddelanden. |
> | [timetravel](https://www.postgresql.org/docs/current/contrib-spi.html#id-1.11.7.45.6) | Funktioner för att implementera res tid. |
> | [UUID – ossp](https://www.postgresql.org/docs/current/static/uuid-ossp.html) | Genererar universellt unika identifierare (UUID). |

### <a name="hyperscale-extensions"></a>Skalnings tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [citus](https://github.com/citusdata/citus) | Citus distribuerad databas. |
> | Shard\_-ombalanser | På ett säkert sätt balansera om data i en Server grupp i händelse av nod tillägg eller borttagning. |

### <a name="index-types-extensions"></a>Tillägg för index typer

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [hinna](https://www.postgresql.org/docs/current/bloom.html) | Anslutnings metod för blomma – Signature filbaserat index. |
> | [b\_gin](https://www.postgresql.org/docs/current/static/btree-gin.html) | Tillhandahåller exempel på GIN operator klasser som implementerar B-träd-liknande beteenden för vissa data typer. |
> | [b\_register](https://www.postgresql.org/docs/current/static/btree-gist.html) | Tillhandahåller klasser för registrerings index operatorer som implementerar B-träd. |

### <a name="language-extensions"></a>Språk tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [plpgsql](https://www.postgresql.org/docs/current/static/plpgsql.html) | PL/pgSQL-inbelastnings förfarande språk. |

### <a name="miscellaneous-extensions"></a>Diverse tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [adminpaket](https://www.postgresql.org/docs/current/adminpack.html) | Administrativa funktioner för PostgreSQL. |
> | [amcheck](https://www.postgresql.org/docs/current/amcheck.html) | Funktioner för att verifiera Relations integritet. |
> | [fil\_FDW](https://www.postgresql.org/docs/current/file-fdw.html) | Sekundärt data omslag för Flat-filåtkomst. |
> | [pageinspect](https://www.postgresql.org/docs/current/pageinspect.html) | Granska innehållet på databas sidor på en låg nivå. |
> | [PG\_-buffercache](https://www.postgresql.org/docs/current/static/pgbuffercache.html) | Ger ett sätt att undersöka vad som händer i det delade cacheminnet i real tid. |
> | [PG\_-cron](https://github.com/citusdata/pg_cron) | Jobb schema för PostgreSQL. |
> | [PG\_-freespacemap](https://www.postgresql.org/docs/current/pgfreespacemap.html) | Kontrol lera kartan över ledigt utrymme (FSM). |
> | [PG\_-förvarm](https://www.postgresql.org/docs/current/static/pgprewarm.html) | Är ett sätt att läsa in relations data i bufferten. |
> | [rapporter\_om\_PG stat](https://www.postgresql.org/docs/current/static/pgstatstatements.html) | Ger ett sätt att spåra körnings statistik för alla SQL-uttryck som körs av en server. Mer information om det här tillägget finns i avsnittet pg_stat_statements. |
> | [PG\_-synlighet](https://www.postgresql.org/docs/current/pgvisibility.html) | Granska Synlighets kartan (VM) och Synlighets information på sidnivå. |
> | [pgrowlocks](https://www.postgresql.org/docs/current/static/pgrowlocks.html) | Ger möjlighet att Visa lås information på radnivå. |
> | [pgstattuple](https://www.postgresql.org/docs/current/static/pgstattuple.html) | Ger ett sätt att visa statistik på tuple-nivå. |
> | [postgres\_FDW](https://www.postgresql.org/docs/current/static/postgres-fdw.html) | Sekundärt data omslag som används för att komma åt data som lagras i externa PostgreSQL-servrar. Mer information om det här tillägget finns i avsnittet "dbLink och postgres_fdw".|
> | [sslinfo](https://www.postgresql.org/docs/current/sslinfo.html) | Information om TLS/SSL-certifikat. |
> | [TSM\_system\_rader](https://www.postgresql.org/docs/current/tsm-system-rows.html) | TABLESAMPLE-metoden som accepterar antalet rader som en gräns. |
> | [TSM\_system\_tid](https://www.postgresql.org/docs/current/tsm-system-time.html) | TABLESAMPLE-metoden som accepterar tiden i millisekunder som en gräns. |
> | [hypopg](https://hypopg.readthedocs.io/en/latest/) | Ger ett sätt att skapa hypotetiska index som inte kostar CPU eller disk. |
> | [dblink](https://www.postgresql.org/docs/current/dblink.html) | En modul som stöder anslutningar till andra PostgreSQL-databaser inifrån en databas-session. Mer information om det här tillägget finns i avsnittet "dbLink och postgres_fdw". |
> | [xml2](https://www.postgresql.org/docs/current/xml2.html) | XPath-frågor och XSLT. |


### <a name="postgis-extensions"></a>PostGIS-tillägg

> [!div class="mx-tableFixed"]
> | **Anknytning** | **Beskrivning** |
> |---|---|
> | [Postgis](https://www.postgis.net/), postgis\_-topologi,\_postgis\_Tiger-kod, postgis\_sfcgal | Spatialdata och geografiska objekt för PostgreSQL. |
> | adress\_standardiserare, adress\_standardiserade\_data\_US | Används för att parsa en adress till komponent element. Används för att stödja kod normaliserings steg för den här adressen. |
> | postgis\_sfcgal | PostGIS SFCGAL-funktioner. |
> | postgis\_Tiger\_-kod | PostGIS Tiger hårdkodad och omvänd landskod. |
> | postgis\_-topologi | Spatiala typer och funktioner för PostGIS-topologi. |


## <a name="pg_stat_statements"></a>pg_stat_statements
[Tillägget för\_PG\_stat-satser](https://www.postgresql.org/docs/current/pgstatstatements.html) är förinstallerat på alla Azure Database for postgresql server för att ge dig ett sätt att spåra körnings statistik för SQL-uttryck.

Inställningen `pg_stat_statements.track` styr vilka instruktioner som räknas av tillägget. Den används som `top`standard, vilket innebär att alla instruktioner som utfärdas direkt av klienter spåras. De två andra spårnings nivåerna `none` är `all`och. Den här inställningen kan konfigureras som en server parameter via [Azure Portal](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-portal) eller [Azure CLI](https://docs.microsoft.com/azure/postgresql/howto-configure-server-parameters-using-cli).

Det är en kompromiss mellan information om körning av fråga pg_stat_statements tillhandahåller och påverkan på Server prestanda när varje SQL-sats loggas. Om du inte aktivt använder pg_stat_statements-tillägget rekommenderar vi att du ställer in `pg_stat_statements.track` på `none`. Vissa övervaknings tjänster från tredje part kan vara beroende av pg_stat_statements för att leverera frågor om prestanda insikter, så kontrol lera om detta är fallet för dig eller inte.

## <a name="dblink-and-postgres_fdw"></a>dbLink och postgres_fdw
Du kan använda dbLink och postgres_fdw för att ansluta från en PostgreSQL-server till en annan, eller till en annan databas på samma server. Den mottagande servern måste tillåta anslutningar från den sändande servern via brand väggen. Om du vill använda dessa tillägg för att ansluta mellan Azure Database for PostgreSQL-servrar anger du **Tillåt åtkomst till Azure-tjänster** till på. Du måste också aktivera den här inställningen om du vill använda tilläggen för att återgå till samma server. Du hittar inställningen **Tillåt åtkomst till Azure-tjänster** på Azure Portal sidan för postgres-servern under **anslutnings säkerhet**. Aktivera **Tillåt åtkomst till Azure-tjänster** på whitelists alla Azure IP-adresser.

För närvarande stöds inte utgående anslutningar från Azure Database for PostgreSQL, förutom för anslutningar till andra Azure Database for PostgreSQL-servrar.
