---
title: Konfigurera Data Sync
description: Den här självstudien visar hur du konfigurerar Azure-SQL Data Sync
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 01/14/2019
ms.openlocfilehash: 019ddbac1900856666b958d90b4395f25eb5ee84
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79268969"
---
# <a name="tutorial-set-up-sql-data-sync-between-azure-sql-database-and-sql-server-on-premises"></a>Självstudie: Konfigurera SQL Data Sync mellan Azure SQL Database och SQL Server lokalt

I den här självstudien får du lära dig hur du konfigurerar Azure SQL Data Sync genom att skapa en synkroniseringsresurs som innehåller både Azure SQL Database-och SQL Server instanser. Sync-gruppen är anpassad konfigurerad och synkroniseras enligt det schema du anger.

Självstudien förutsätter att du har minst en tidigare erfarenhet av SQL Database och SQL Server.

En översikt över SQL Data Sync finns i [synkronisera data över molnbaserade och lokala databaser med Azure SQL Data Sync](sql-database-sync-data.md).

PowerShell-exempel på hur du konfigurerar SQL Data Sync finns i [så här synkroniserar du mellan Azure SQL-databaser](scripts/sql-database-sync-data-between-sql-databases.md) eller [en Azure SQL Database och en SQL Server lokal databas](scripts/sql-database-sync-data-between-azure-onprem.md)

> [!IMPORTANT]
> Azure SQL Data Sync har **inte** stöd för Azure SQL Database Managed Instance just nu.

## <a name="create-sync-group"></a>Skapa sync-grupp

1. Gå till [Azure Portal](https://portal.azure.com) för att hitta din SQL-databas. Sök efter och välj **SQL-databaser**.

    ![Sök efter SQL-databaser Microsoft Azure-portalen](media/sql-database-get-started-sql-data-sync/search-for-sql-databases.png)

1. Välj den databas som du vill använda som hubb databas för datasynkronisering.

    ![Välj från SQL Database-listan Microsoft Azure-portalen](media/sql-database-get-started-sql-data-sync/select-sql-database.png)

    > [!NOTE]
    > Hub-databasen är en central slut punkt för synkronisering, där en synkroniserad grupp har flera databas slut punkter. Alla andra medlems databaser med slut punkter i Sync-gruppen synkroniseras med Hub-databasen.

1. På **SQL Database** -menyn för den valda databasen väljer du **synkronisera till andra databaser**.

    ![Synkronisera till andra databaser, SQL Database, Microsoft Azure-portalen](media/sql-database-get-started-sql-data-sync/sync-to-other-databases.png)

1. På sidan **synkronisera till andra databaser** väljer du **ny Sync-grupp**. Sidan **ny synkroniseringsresurs** öppnas med **Skapa sync-grupp (steg 1)** markerat.

   ![Steg 1-inställningar](media/sql-database-get-started-sql-data-sync/stepone.png)

   På sidan **skapa data Sync-grupp** ändrar du följande inställningar:

   | Inställningen                        | Beskrivning |
   | ------------------------------ | ------------------------------------------------- |
   | **Sync-gruppens namn** | Ange ett namn för den nya Sync-gruppen. Namnet skiljer sig från namnet på själva databasen. |
   | **Synka metadata-databas** | Välj att skapa en databas (rekommenderas) eller Använd en befintlig databas.<br/><br/>Om du väljer **ny databas**väljer du **Skapa ny databas.** På sidan **SQL Database** namnger och konfigurerar du den nya databasen och väljer **OK**.<br/><br/>Om du väljer **Använd befintlig databas**väljer du databasen i listan. |
   | **Automatisk synkronisering** | Välj **på** eller **av**.<br/><br/>Om du väljer **på**anger du ett nummer och väljer **sekunder**, **minuter**, **timmar**eller **dagar** i avsnittet **Synkroniseringsfrekvens** . |
   | **Konflikt lösning** | Välj **hubben Win** eller **member Win**.<br/><br/>**Hubben Win** innebär att data i Hub-databasen skriver över motstridiga data i medlems databasen när konflikter uppstår.<br/><br/>**Medlemmen Win** innebär att om konflikter uppstår skriver data i medlems databasen över motstridiga data i Hubbs databasen. |

   > [!NOTE]
   > Microsoft rekommenderar att du skapar en ny, tom databas för användning som **databasen för synkronisering av metadata**. Datasynkronisering skapar tabeller i den här databasen och kör en frekvent arbets belastning. Den här databasen delas som **databasen för synkronisering av metadata** för alla Sync-grupper i en vald region och du kan inte ändra databasen eller dess namn utan att ta bort alla synkroniserade grupper och synkronisera agenter i regionen.

   Välj **OK** och vänta tills Sync-gruppen har skapats och distribuerats.

## <a name="add-sync-members"></a>Lägg till medlemmar i synkronisering

När den nya synkroniseringsresursen har skapats och distribuerats lägger du **till Sync-medlemmar (steg 2)** på sidan **ny Sync** -grupp.

I avsnittet **Hubbs databas** anger du befintliga autentiseringsuppgifter för den SQL Database Server där Hub-databasen finns. Ange inte *nya* autentiseringsuppgifter i det här avsnittet.

![Steg 2 inställningar](media/sql-database-get-started-sql-data-sync/steptwo.png)

### <a name="to-add-an-azure-sql-database"></a>Lägga till en Azure SQL Database

I avsnittet **medlems databas** lägger du till en Azure SQL Database till Sync-gruppen genom att välja **lägg till en Azure SQL Database**. Sidan **konfigurera Azure SQL Database** öppnas.

  ![Steg 2 – Konfigurera databas](media/sql-database-get-started-sql-data-sync/steptwo-configure.png)

  Ändra följande inställningar på sidan **konfigurera Azure SQL Database** :

  | Inställningen                       | Beskrivning |
  | ----------------------------- | ------------------------------------------------- |
  | **Synkronisera medlems namn** | Ange ett namn för den nya synkroniserings medlemmen. Namnet skiljer sig från själva databas namnet. |
  | **Prenumeration** | Välj den associerade Azure-prenumerationen för fakturerings syfte. |
  | **Azure-SQL Server** | Välj den befintliga SQL Database-servern. |
  | **Azure SQL Database** | Välj den befintliga SQL-databasen. |
  | **Anvisningar för synkronisering** | Välj **dubbelriktad synkronisering**, **till hubben**eller **från hubben**. |
  | **Användar namn** och **lösen ord** | Ange de befintliga autentiseringsuppgifterna för den SQL Database Server där medlems databasen finns. Ange inte *nya* autentiseringsuppgifter i det här avsnittet. |

  Välj **OK** och vänta tills den nya synkroniserings medlemmen har skapats och distribuerats.

<a name="add-on-prem"></a>
### <a name="to-add-an-on-premises-sql-server-database"></a>Lägga till en lokal SQL Server databas

I avsnittet **medlems databas** lägger du till en lokal SQL Server till Sync-gruppen genom att välja **Lägg till en lokal databas**. Sidan **Konfigurera lokalt** öppnas där du kan göra följande saker:

1. Välj **Välj Sync agent-Gateway**. Sidan **Välj Sync-agent** öppnas.

   ![Skapa en Sync-agent](media/sql-database-get-started-sql-data-sync/steptwo-agent.png)

1. På sidan **Välj agent för synkronisering** väljer du om du vill använda en befintlig agent eller skapa en agent.

   Om du väljer **befintliga agenter**väljer du den befintliga agenten i listan.

   Om du väljer **skapa en ny agent**gör du följande:

   1. Ladda ned agenten för data synkronisering från den angivna länken och installera den på den dator där SQL Server finns. Du kan också hämta agenten direkt från [SQL Azure Data Sync-agenten](https://www.microsoft.com/download/details.aspx?id=27693).

      > [!IMPORTANT]
      > Du måste öppna utgående TCP-port 1433 i brand väggen så att klient agenten kan kommunicera med servern.

   1. Ange ett namn för agenten.

   1. Välj **skapa och generera nyckel** och kopiera Agent nyckeln till Urklipp.

   1. Välj **OK** för att stänga sidan **Välj Sync-agent** .

1. Leta upp och kör Klientsynkroniseringsagent-appen på SQL Server dator.

   ![Appen för klient agent för data synkronisering](media/sql-database-get-started-sql-data-sync/datasync-preview-clientagent.png)

    1. I appen Sync-agent väljer du **Skicka agent nyckel**. Dialog rutan **konfiguration av databas för synkronisering av metadata** öppnas.

    1. I dialog rutan **konfiguration av databas för synkronisering av metadata** klistrar du in Agent nyckeln som kopierats från Azure Portal. Ange även de befintliga autentiseringsuppgifterna för den Azure SQL Database-Server där metadata-databasen finns. (Om du har skapat en metadata-databas finns den här databasen på samma server som Hub-databasen.) Välj **OK** och vänta tills konfigurationen har slutförts.

        ![Ange agent nyckel och autentiseringsuppgifter för servern](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-enterkey.png)

        > [!NOTE]
        > Om du får ett brand Väggs fel skapar du en brand Väggs regel på Azure för att tillåta inkommande trafik från den SQL Server datorn. Du kan skapa regeln manuellt i portalen eller i SQL Server Management Studio (SSMS). I SSMS ansluter du till Hub-databasen på Azure genom att ange dess namn som <hub_database_name>. database.windows.net.

    1. Välj **Registrera** för att registrera en SQL Server databas med agenten. Dialog rutan **SQL Server konfiguration** öppnas.

        ![Lägga till och konfigurera en SQL Server databas](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-adddb.png)

    1. I dialog rutan **SQL Server konfiguration** väljer du att ansluta med SQL Server autentisering eller Windows-autentisering. Om du väljer SQL Server autentisering anger du de befintliga autentiseringsuppgifterna. Ange SQL Server namn och namnet på den databas som du vill synkronisera och välj **Testa anslutning** för att testa inställningarna. Välj sedan **Spara** och den registrerade databasen visas i listan.

        ![SQL Server Database har nu registrerats](media/sql-database-get-started-sql-data-sync/datasync-preview-agent-dbadded.png)

    1. Stäng Klientsynkroniseringsagent-appen.

1. I portalen på sidan **Konfigurera lokalt** väljer **du Välj databas**.

1. På sidan **Välj databas** i fältet **Synkronisera medlems namn** anger du ett namn för den nya synkroniserings medlemmen. Namnet skiljer sig från namnet på själva databasen. Välj databasen i listan. I fältet **Synkronisera vägvisningar** väljer du **dubbelriktad synkronisering**, **till hubben**eller **från hubben**.

    ![Välj den lokala databasen](media/sql-database-get-started-sql-data-sync/datasync-preview-selectdb.png)

1. Klicka på **OK** för att stänga sidan **Välj databas** . Välj **OK** för att stänga sidan **Konfigurera lokalt** och vänta tills den nya synkroniserings medlemmen skapas och distribueras. Välj slutligen **OK** för att stänga sidan **Välj synkronisera medlemmar** .

> [!NOTE]
> Om du vill ansluta till SQL Data Sync och den lokala agenten lägger du till ditt användar namn i roll *DataSync_Executor*. DataSync skapar den här rollen på SQL Server-instansen.

## <a name="configure-sync-group"></a>Konfigurera Sync-grupp

När de nya medlemmarna i Sync-gruppen har skapats och distribuerats, är **Konfigurera Sync-grupp (steg 3)** markerat på sidan **ny Sync** -grupp.

![Steg 3 inställningar](media/sql-database-get-started-sql-data-sync/stepthree.png)

1. På sidan **tabeller** väljer du en databas i listan över synkroniserade grupp medlemmar och väljer **Uppdatera schema**.

1. I listan väljer du de tabeller som du vill synkronisera. Som standard är alla kolumner markerade, så inaktivera kryss rutan för de kolumner som du inte vill synkronisera. Se till att lämna kolumnen primär nyckel vald.

1. Välj **Spara**.

1. Som standard synkroniseras inte databaser förrän schemalagda eller manuellt körs. Om du vill köra en manuell synkronisering går du till din SQL-databas i Azure Portal, väljer **synkronisera till andra databaser**och väljer Sync-gruppen. Sidan **datasynkronisering** öppnas. Välj **Synkronisera**.

    ![Manuell synkronisering](media/sql-database-get-started-sql-data-sync/datasync-sync.png)

## <a name="faq"></a>VANLIGA FRÅGOR OCH SVAR

**Hur ofta kan datasynkronisering synkronisera mina data?**

Den minsta varaktigheten mellan synkroniseringar är fem minuter.

**Skapar SQL Data Sync tabeller helt?**

Om tabeller i synkroniseringsschemat saknas i destinationsdatabasen skapar SQL Data Sync dem med de kolumner du har valt. Detta resulterar dock inte i ett schema med fullständig åter givning av följande orsaker:

- Endast de kolumner som du väljer skapas i måltabellen. Kolumner som inte valts ignoreras.
- Endast markerade kolumnindex skapas i måltabellen. Index för kolumner som inte valts ignoreras.
- Index för kolumner med XML-typ skapas inte.
- KONTROLL begränsningar skapas inte.
- Utlösare i käll tabellerna skapas inte.
- Vyer och lagrade procedurer skapas inte.

På grund av dessa begränsningar rekommenderar vi följande:

- För produktions miljöer skapar du schemat full Fidelity själv.
- Använd funktionen för automatisk etablering när du experimenterar med tjänsten.

**Varför visas tabeller som jag inte har skapat?**

Datasynkronisering skapar ytterligare tabeller i databasen för ändrings spårning. Ta inte bort dessa eller så slutar datasynkroniseringen att fungera.

**Är mina data Convergent efter en synkronisering?**

Inte nödvändigt vis. Ta en Sync-grupp med en hubb och tre ekrar (A, B och C) där synkroniseringarna är hubb till en, hubb till B och hubb till C. Om en ändring görs i databasen A *efter* navet till en synkronisering, skrivs den ändringen inte till databas B eller databas C förrän nästa synkroniseringsuppgift.

**Hur gör jag för att få schema ändringar till en Sync-grupp?**

Gör och sprid alla schema ändringar manuellt.

1. Replikera schema ändringarna manuellt till hubben och till alla synkroniserade medlemmar.
1. Uppdatera synkroniseringsschemat.

För att lägga till nya tabeller och kolumner:

Nya tabeller och kolumner påverkar inte den aktuella synkroniseringen och datasynkroniseringen ignorerar dem tills de läggs till i synkroniseringsschemat. När du lägger till nya databas objekt följer du sekvensen:

1. Lägg till nya tabeller eller kolumner i hubben och alla synkroniserade medlemmar.
1. Lägg till nya tabeller eller kolumner i synkroniseringsschemat.
1. Börja infoga värden i nya tabeller och kolumner.

För att ändra data typen för en kolumn:

När du ändrar data typen för en befintlig kolumn fortsätter data synkroniseringen att fungera så länge de nya värdena passar den ursprungliga data typen som definierats i synkroniseringsschemat. Om du till exempel ändrar typen i käll databasen från **int** till **bigint**fortsätter datasynkroniseringen att fungera tills du sätter in värdet för stor för data typen **int** . Du slutför ändringen genom att replikera schema ändringen manuellt till hubben och till alla synkroniserade medlemmar och sedan uppdatera synkroniseringsschemat.

**Hur kan jag exportera och importera en databas med datasynkronisering?**

När du har exporterat en databas som en *. bacpac* -fil och importera filen för att skapa en databas, gör du följande för att använda datasynkronisering i den nya databasen:

1. Rensa DataSync-objekten och ytterligare tabeller i den nya databasen med hjälp av [det här skriptet](https://github.com/vitomaz-msft/DataSyncMetadataCleanup/blob/master/Data%20Sync%20complete%20cleanup.sql). Skriptet tar bort alla nödvändiga data synkroniseringsobjekt från databasen.
1. Återskapa Sync-gruppen med den nya databasen. Om du inte längre behöver den gamla Sync-gruppen tar du bort den.

**Var kan jag hitta information om klient agenten?**

Vanliga frågor och svar om klient agenten finns i [vanliga frågor och svar om agent](sql-database-data-sync-agent.md#agent-faq).

## <a name="next-steps"></a>Nästa steg

Grattis! Du har skapat en Sync-grupp som innehåller både en SQL Database-instans och en SQL Server-databas.

Mer information om SQL Data Sync finns i:

- [Data Sync-agent för Azure SQL Data Sync](sql-database-data-sync-agent.md)
- [Metod tips](sql-database-best-practices-data-sync.md) och [fel sökning av problem med Azure SQL Data Sync](sql-database-troubleshoot-data-sync.md)
- [Övervaka SQL Data Sync med Azure Monitor loggar](sql-database-sync-monitor-oms.md)
- [Uppdatera synkroniseringsschemat med Transact-SQL](sql-database-update-sync-schema.md) eller [PowerShell](scripts/sql-database-sync-update-schema.md)

Mer information om SQL Database finns i:

- [Översikt över SQL Database](sql-database-technical-overview.md)
- [Livscykelhantering för databas](https://msdn.microsoft.com/library/jj907294.aspx)
