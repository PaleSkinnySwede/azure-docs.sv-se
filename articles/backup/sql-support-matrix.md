---
title: Azure Backup support mat ris för SQL Server säkerhets kopiering på virtuella Azure-datorer
description: Innehåller en översikt över support inställningar och begränsningar när du säkerhetskopierar SQL Server i virtuella Azure-datorer med tjänsten Azure Backup.
ms.topic: conceptual
ms.date: 03/05/2020
ms.openlocfilehash: 79a7e30ab9240c489a66b547ff85bea7887131b1
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79410004"
---
# <a name="support-matrix-for-sql-server-backup-in-azure-vms"></a>Support mat ris för SQL Server säkerhets kopiering på virtuella Azure-datorer

Du kan använda Azure Backup för att säkerhetskopiera SQL Server databaser i virtuella Azure-datorer som finns på Microsoft Azure moln plattformen. Den här artikeln sammanfattar allmänna support inställningar och begränsningar för scenarier och distributioner av SQL Server säkerhets kopiering på virtuella Azure-datorer.

## <a name="scenario-support"></a>Scenariostöd

**Support** | **Information**
--- | ---
**Distributioner som stöds** | Virtuella SQL Marketplace Azure-datorer och virtuella icke-Marketplace-datorer (manuellt installerat SQL Server) stöds.
**Geografiska områden som stöds** | Sydöstra Australien (ASE), östra Australien (AE), Australien, centrala (AC), Australien, centrala 2 (AC) <br> Brasilien, södra (BRS)<br> Kanada, centrala (CNC), Kanada, östra (CE)<br> Asien, sydöstra (SEA), Asien, östra (EA) <br> Östra USA (EUS), östra USA 2 (EUS2), västra centrala USA (WCUS), västra USA (WUS); Västra USA 2 (WUS 2) norra centrala USA (NCUS) centrala USA (CUS) södra centrala USA (SCUS) <br> Indien, centrala (INC), Indien, södra (moduler), västra Indien <br> Japan, östra (JPE), Japan, väst (JPW) <br> Korea, centrala (KRC), Korea, södra (KRS) <br> Nord Europa (NE), Västeuropa <br> Storbritannien, södra (UKS), Storbritannien, västra (UKW) <br> US Gov, Arizona, US Gov, Virginia, US Gov, Texas, US DoD, centrala, US DoD, östra <br> Tyskland, norra Tyskland, västra centrala <br> Schweiz, norra Schweiz, västra <br> Frankrike, centrala <br> Kina, östra, Kina, östra 2, Kina, norra, Kina, norra 2
**Operativsystem som stöds** | Windows Server 2019, Windows Server 2016, Windows Server 2012, Windows Server 2008 R2 SP1 <br/><br/> Linux stöds inte för närvarande.
**SQL Server-versioner som stöds** | SQL Server 2019, SQL Server 2017 som beskrivs på [sidan Sök produktens livs cykel](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202017), SQL Server 2016 och SPS enligt beskrivningen på [sidan Sök produkt livs cykel](https://support.microsoft.com/lifecycle/search?alpha=SQL%20server%202016%20service%20pack)SQL Server 2014, SQL Server 2012, SQL Server 2008 R2, SQL Server 2008 <br/><br/> Enterprise, Standard, Web, Developer, Express.
**.NET-versioner som stöds** | .NET Framework 4.5.2 eller senare installerat på den virtuella datorn

## <a name="feature-consideration-and-limitations"></a>Funktions överväganden och begränsningar

* SQL Server säkerhets kopiering kan konfigureras i Azure Portal eller **PowerShell**. Vi stöder inte CLI.
* Lösningen stöds på båda typerna av [distributioner](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-deployment-model) – Azure Resource Manager virtuella datorer och klassiska virtuella datorer.
* Virtuell dator som kör SQL Server kräver Internet anslutning för att få åtkomst till offentliga Azure-IP-adresser.
* SQL Server **-redundanskluster (FCI)** stöds inte.
* Säkerhets kopierings-och återställnings åtgärder för speglings databaser och ögonblicks bilder av databasen stöds inte.
* Om du använder fler än en säkerhets kopierings lösning för att säkerhetskopiera din fristående SQL Server instans eller SQL Always on-tillgänglighetsgruppen kan det leda till säkerhets kopierings problem. avstå från att göra det.
* Att säkerhetskopiera två noder i en tillgänglighets grupp individuellt med samma eller olika lösningar, kan också leda till att säkerhets kopieringen Miss lyckas.
* Azure Backup stöder endast fullständig och endast kopiering av fullständiga säkerhets kopierings typer för **skrivskyddade** databaser
* Databaser med ett stort antal filer kan inte skyddas. Det maximala antalet filer som stöds är **~ 1000**.  
* Du kan säkerhetskopiera till **~ 2000** SQL Server databaser i ett valv. Du kan skapa flera valv om du har ett större antal databaser.
* Du kan konfigurera säkerhets kopiering för upp till **50** databaser i en go; den här begränsningen hjälper till att optimera säkerhets kopierings belastning.
* Vi har stöd för databaser upp till **2 TB** i storlek. för större storlekar kan prestanda problem uppstå.
* Om du vill ha en uppfattning om hur många databaser som kan skyddas per server bör du tänka på faktorer som bandbredd, VM-storlek, säkerhets kopierings frekvens, databas storlek och så vidare. [Ladda ned](https://download.microsoft.com/download/A/B/5/AB5D86F0-DCB7-4DC3-9872-6155C96DE500/SQL%20Server%20in%20Azure%20VM%20Backup%20Scale%20Calculator.xlsx) resurs planeraren för att beräkna det ungefärliga antalet databaser som du kan ha per server baserat på de virtuella dator resurserna och säkerhets kopierings principen.
* När tillgänglighets grupper har kon figurer ATS tas säkerhets kopiorna från de olika noderna utifrån några faktorer. Säkerhets kopierings beteendet för en tillgänglighets grupp sammanfattas nedan.

### <a name="back-up-behavior-with-always-on-availability-groups"></a>Säkerhetskopiera beteende med Always on-tillgänglighetsgrupper

Vi rekommenderar att säkerhets kopian bara konfigureras på en nod i en tillgänglighets grupp (AG). Konfigurera alltid säkerhets kopiering i samma region som den primära noden. Med andra ord behöver du alltid den primära noden för att finnas i den region där du konfigurerar säkerhets kopian. Om alla noder i AG finns i samma region där säkerhets kopian har kon figurer ATS, finns det inga problem.

#### <a name="for-cross-region-ag"></a>För en AG i flera regioner

* Oavsett säkerhets kopierings inställningen körs säkerhets kopieringarna bara från noderna i samma region där säkerhets kopian har kon figurer ATS. Detta beror på att säkerhets kopiering mellan regioner inte stöds. Om du bara har två noder och den sekundära noden finns i den andra regionen fortsätter säkerhets kopieringarna att köras från den primära noden (om inte säkerhets kopierings inställningen är sekundär).
* Om en nod växlar över till en annan region än den där säkerhets kopian har kon figurer ATS, Miss lyckas säkerhets kopieringarna på noderna i den felande regionen.

Beroende på vilka säkerhets kopierings-och säkerhets kopierings typer (fullständig/differentiell/Logga/kopiera endast fullständig), tas säkerhets kopiorna från en viss nod (primär/sekundär).

#### <a name="backup-preference-primary"></a>Säkerhets kopierings inställning: primär

**Typ av säkerhetskopiering** | **Node**
--- | ---
Fullständig | Primär
Differentiell | Primär
Logga |  Primär
Fullständig kopia |  Primär

#### <a name="backup-preference-secondary-only"></a>Inställningar för säkerhets kopiering: endast sekundär

**Typ av säkerhetskopiering** | **Node**
--- | ---
Fullständig | Primär
Differentiell | Primär
Logga |  Sekundär
Fullständig kopia |  Sekundär

#### <a name="backup-preference-secondary"></a>Säkerhets kopierings inställning: sekundär

**Typ av säkerhetskopiering** | **Node**
--- | ---
Fullständig | Primär
Differentiell | Primär
Logga |  Sekundär
Fullständig kopia |  Sekundär

#### <a name="no-backup-preference"></a>Ingen säkerhets kopierings inställning

**Typ av säkerhetskopiering** | **Node**
--- | ---
Fullständig | Primär
Differentiell | Primär
Logga |  Sekundär
Fullständig kopia |  Sekundär

## <a name="next-steps"></a>Nästa steg

Lär dig hur du [säkerhetskopierar en SQL Server databas](backup-azure-sql-database.md) som körs på en virtuell Azure-dator.
