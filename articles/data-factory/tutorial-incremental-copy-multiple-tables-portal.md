---
title: Kopiera flera tabeller stegvis med Azure Portal
description: I den här självstudiekursen kommer du att skapa en Azure Data Factory-pipeline som kopierar deltadata stegvis från flera tabeller i en lokal SQL Server-databas till en Azure SQL-databas.
services: data-factory
ms.author: yexu
author: dearandyxu
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 01/20/2018
ms.openlocfilehash: 290ddf9a99d421bbf6303675fd544e81b637d070
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "81419280"
---
# <a name="incrementally-load-data-from-multiple-tables-in-sql-server-to-an-azure-sql-database"></a>Läs in data stegvis från flera tabeller i SQL Server till en Azure SQL-databas

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

I den här självstudiekursen kommer du att skapa en Azure-datafabrik med en pipeline som läser in deltadata från flera tabeller på en lokal SQL-server till en Azure SQL-databas.    

I den här självstudiekursen får du göra följande:

> [!div class="checklist"]
> * Förbereda käll- och måldatalager.
> * Skapa en datafabrik.
> * Skapa Integration Runtime med egen värd.
> * Installera Integration Runtime. 
> * Skapa länkade tjänster. 
> * Skapa datauppsättningar för källa, mottagare och vattenstämpel.
> * Skapa, köra och övervaka en pipeline.
> * Granska resultaten.
> * Lägga till eller uppdatera data i källtabeller.
> * Köra och övervaka pipelinen igen.
> * Granska de slutliga resultaten.

## <a name="overview"></a>Översikt
Här är några viktiga steg för att skapa den här lösningen: 

1. **Markera vattenstämpelkolumnen**.
    
    Markera en kolumn för varje tabell i källdatalagret som går att använda för att identifiera de nya eller uppdaterade posterna för varje körning. Vanligtvis ökar data i den markerade kolumnen (till exempel last_modify_time elle ID) när rader skapas eller uppdateras. Det maximala värdet i den här kolumnen används som vattenstämpel.

1. **Förbered datalagringen för att lagra värdet för vattenstämpeln**.   
    
    I den här självstudien lagrar du storleksgränsen i en SQL-databas.

1. **Skapa en pipeline med följande aktiviteter**: 
    
    a. Skapa en ForEach-aktivitet som upprepas över en lista med namn på källtabeller och som skickas som en parameter till pipelinen. För varje källtabell anropas följande aktiviteter som utför deltainläsningen för tabellen.

    b. Skapa två sökningsaktiviteter. Använd den första sökningsaktiviteten för att hämta det sista vattenstämpelvärdet. Använd den andra sökningsaktiviteten för att hämta det nya vattenstämpelvärdet. Vattenstämpelvärdena skickas till kopieringsaktiviteten.

    c. Skapa en {0}kopieringsaktivitet{0} som kopierar raderna från källdatalagringen med värdet för vattenstämpelkolumnen som är större än det gamla värdet och mindre än det nya. Sedan kopieras deltadata från källdatalagringen till Azure Blob-lagring som en ny fil.

    d. Skapa en StoredProcedure-aktivitet som uppdaterar vattenstämpelvärdet för den pipeline som körs nästa gång. 

    Här är det avancerade diagrammet: 

    ![Läsa in data stegvis](media/tutorial-incremental-copy-multiple-tables-portal/high-level-solution-diagram.png)


Om du inte har en Azure-prenumeration kan du skapa ett [kostnads fritt](https://azure.microsoft.com/free/) konto innan du börjar.

## <a name="prerequisites"></a>Krav
* **SQL Server**. Du använder en lokal SQL Server-databas som källdatalager i den här självstudien. 
* **Azure SQL Database**. Du använder en SQL-databas som måldatalager. Om du inte har någon SQL Database kan du läsa om hur du skapar en i [Skapa en Azure SQL-databas](../sql-database/sql-database-get-started-portal.md). 

### <a name="create-source-tables-in-your-sql-server-database"></a>Skapa källtabeller i din SQL Server-databas

1. Öppna SQL Server Management Studio och anslut till din lokala SQL-serverdatabas.

1. I **Server Explorer**högerklickar du på databasen och väljer **ny fråga**.

1. Kör följande SQL-kommando mot databasen för att skapa tabeller med namnen `customer_table` och `project_table`:

    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );
        
    INSERT INTO customer_table
    (PersonID, Name, LastModifytime)
    VALUES
    (1, 'John','9/1/2017 12:56:00 AM'),
    (2, 'Mike','9/2/2017 5:23:00 AM'),
    (3, 'Alice','9/3/2017 2:36:00 AM'),
    (4, 'Andy','9/4/2017 3:21:00 AM'),
    (5, 'Anny','9/5/2017 8:06:00 AM');
    
    INSERT INTO project_table
    (Project, Creationtime)
    VALUES
    ('project1','1/1/2015 0:00:00 AM'),
    ('project2','2/2/2016 1:23:00 AM'),
    ('project3','3/4/2017 5:16:00 AM');
    
    ```

### <a name="create-destination-tables-in-your-azure-sql-database"></a>Skapa måltabeller i Azure SQL-databasen
1. Öppna SQL Server Management Studio och anslut till din Azure SQL-databas.

1. I **Server Explorer**högerklickar du på databasen och väljer **ny fråga**.

1. Kör följande SQL-kommando mot din Azure SQL-databas för att skapa `customer_table` tabeller `project_table`med namnen och:  
    
    ```sql
    create table customer_table
    (
        PersonID int,
        Name varchar(255),
        LastModifytime datetime
    );
    
    create table project_table
    (
        Project varchar(255),
        Creationtime datetime
    );

    ```

### <a name="create-another-table-in-the-azure-sql-database-to-store-the-high-watermark-value"></a>Skapa en annan tabell i Azure SQL-databasen för att lagra värdet för högvattenmärket
1. Kör följande SQL-kommando mot din Azure SQL-databas för att skapa en `watermarktable` tabell med namnet för att lagra värdet för vattenstämpeln: 
    
    ```sql
    create table watermarktable
    (
    
        TableName varchar(255),
        WatermarkValue datetime,
    );
    ```
1. Infoga inledande värden för högvattenmärket för båda källtabellerna i vattenmärkestabellen.

    ```sql

    INSERT INTO watermarktable
    VALUES 
    ('customer_table','1/1/2010 12:00:00 AM'),
    ('project_table','1/1/2010 12:00:00 AM');
    
    ```

### <a name="create-a-stored-procedure-in-the-azure-sql-database"></a>Skapa en lagrad procedur i Azure SQL-databasen 

Kör följande kommando för att skapa en lagrad procedur i din Azure SQL-databas. Den här lagrade proceduren uppdaterar vattenmärkets värde efter varje pipelinekörning. 

```sql
CREATE PROCEDURE usp_write_watermark @LastModifiedtime datetime, @TableName varchar(50)
AS

BEGIN

    UPDATE watermarktable
    SET [WatermarkValue] = @LastModifiedtime 
WHERE [TableName] = @TableName

END

```

### <a name="create-data-types-and-additional-stored-procedures-in-azure-sql-database"></a>Skapa datatyper och ytterligare lagrade procedurer i Azure SQL-databasen
Kör följande fråga för att skapa två lagrade procedurer och två data typer i din Azure SQL-databas. De används för att slå samman data från källtabellerna till måltabellerna.

För att göra resan lätt att börja med, använder vi direkt dessa lagrade procedurer som skickar delta data i via en tabell variabel och sedan sammanfogar dem till mål lagret. Var försiktig med att det inte förväntar sig ett "stort" antal delta rader (mer än 100) som ska lagras i tabell variabeln.  

Om du behöver slå samman ett stort antal delta rader i mål lagret, rekommenderar vi att du använder kopierings aktivitet för att kopiera alla delta data till en tillfällig "mellanlagrings tabell" i mål lagret först och sedan skapa en egen lagrad procedur utan att använda tabell variabel för att sammanfoga dem från tabellen "mellanlagring" till den "slutgiltiga" tabellen. 


```sql
CREATE TYPE DataTypeforCustomerTable AS TABLE(
    PersonID int,
    Name varchar(255),
    LastModifytime datetime
);

GO

CREATE PROCEDURE usp_upsert_customer_table @customer_table DataTypeforCustomerTable READONLY
AS

BEGIN
  MERGE customer_table AS target
  USING @customer_table AS source
  ON (target.PersonID = source.PersonID)
  WHEN MATCHED THEN
      UPDATE SET Name = source.Name,LastModifytime = source.LastModifytime
  WHEN NOT MATCHED THEN
      INSERT (PersonID, Name, LastModifytime)
      VALUES (source.PersonID, source.Name, source.LastModifytime);
END

GO

CREATE TYPE DataTypeforProjectTable AS TABLE(
    Project varchar(255),
    Creationtime datetime
);

GO

CREATE PROCEDURE usp_upsert_project_table @project_table DataTypeforProjectTable READONLY
AS

BEGIN
  MERGE project_table AS target
  USING @project_table AS source
  ON (target.Project = source.Project)
  WHEN MATCHED THEN
      UPDATE SET Creationtime = source.Creationtime
  WHEN NOT MATCHED THEN
      INSERT (Project, Creationtime)
      VALUES (source.Project, source.Creationtime);
END

```

## <a name="create-a-data-factory"></a>Skapa en datafabrik

1. Starta webbläsaren **Microsoft Edge** eller **Google Chrome**. Användargränssnittet för Data Factory stöds för närvarande bara i webbläsarna Microsoft Edge och Google Chrome.
2. På den vänstra menyn väljer du **skapa en resurs** > **analys** > **Data Factory**: 
   
   ![Valet Data Factory i fönstret Nytt](./media/doc-common-process/new-azure-data-factory-menu.png)

3. På sidan **Ny datafabrik** anger du **ADFMultiIncCopyTutorialDF** som **namn**. 
 
   Namnet på Azure Data Factory måste vara **globalt unikt**. Om du ser ett rött utropstecken med följande fel ändrar du namnet på datafabriken (till exempel dittnamnADFIncCopyTutorialDF) och provar att skapa fabriken igen. Se artikeln [Data Factory – namnregler](naming-rules.md) för namnregler för Data Factory-artefakter.
  
   `Data factory name "ADFIncCopyTutorialDF" is not available`

4. Välj den Azure-**prenumeration** som du vill skapa den nya datafabriken i. 
5. För **resursgruppen** utför du något av följande steg:
     
    - Välj **Använd befintlig**och välj en befintlig resurs grupp i den nedrullningsbara listan. 
    - Välj **Skapa ny**och ange namnet på en resurs grupp.   
    Mer information om resursgrupper finns i [Använda resursgrupper till att hantera Azure-resurser](../azure-resource-manager/management/overview.md).  
6. Välj **V2** för **versionen**.
7. Välj **plats** för datafabriken. Endast platser som stöds visas i listrutan. Datalagren (Azure Storage, Azure SQL Database osv.) och beräkningarna (HDInsight osv.) som används i Data Factory kan finnas i andra regioner.
8. Klicka på **Skapa**.      
9. När datafabriken har skapats visas sidan **Datafabrik** som på bilden.
   
   ![Datafabrikens startsida](./media/doc-common-process/data-factory-home-page.png)
10. Klicka på rutan **Författare och övervakare** för att starta användargränssnittet för Azure Data Factory på en separat flik.

## <a name="create-self-hosted-integration-runtime"></a>Skapa Integration Runtime med lokal installation
När du flyttar data från ett datalager i ett privat nätverk (lokalt) till ett Azure-datalager ska du installera en lokal Integration Runtime (IR) i din lokala miljö. Lokalt installerad IR flyttar data mellan ditt privata nätverk och Azure. 

1. Klicka på **Anslutningar** längst ned i det vänstra fönstret och växla till **Integration Runtimes** i fönstret **Anslutningar**. 

1. På fliken **Integration Runtimes** klickar du på **+ Ny**. 

1. I fönstret **integration runtime installation** väljer **du utför data flytt och skicka aktiviteter till externa beräkningar**och klickar på **Fortsätt**. 

1. Välj **egen värd**och klicka på **Fortsätt**. 
1. Ange **MySelfHostedIR** som **namn**och klicka på **skapa**. 

1. Klicka på **Click here to launch the express setup for this computer** (Klicka här för att starta expressinstallation för den här datorn) i avsnittet **Option 1: Express setup** (Alternativ 1: Expressinstallation). 

   ![Klicka på länken för expressinstallation](./media/tutorial-incremental-copy-multiple-tables-portal/click-express-setup.png)
1. Klicka på **Stäng** i fönstret **Snabbinstallation av Integration Runtime (lokal installation)**. 

   ![Integration Runtime har installerats](./media/tutorial-incremental-copy-multiple-tables-portal/integration-runtime-setup-successful.png)
1. Klicka på **Slutför** i webbläsaren för att stänga installationsfönstret för **Integration Runtime**. 

 
1. Bekräfta att du ser **MySelfHostedIR** i listan över Integration Runtimes.

## <a name="create-linked-services"></a>Skapa länkade tjänster
Du kan skapa länkade tjänster i en datafabrik för att länka ditt datalager och beräkna datafabrik-tjänster. I det här avsnittet får du skapa länkade tjänster till din lokala SQL Server-databas och din Azure SQL-databas. 

### <a name="create-the-sql-server-linked-service"></a>Skapa länkad tjänst till SQL Server
I det här steget länkar du din lokala SQL Serverdatabas till datafabriken.

1. I fönstret **Anslutningar** växlar du från fliken **Integration Runtimes** till fliken med **länkade tjänster** och klickar på **+ Ny**.

1. I fönstret **New Linked Service** (Ny länkad tjänst) väljer du **SQL Server** och klickar på **Fortsätt**. 

1. Utför följande steg i fönstret **New Linked Service** (Ny länkad tjänst):

    1. Ange **SqlServerLinkedService** som **namn**. 
    1. Välj **MySelfHostedIR** för **Connect via integration runtime** (Anslut via Integration Runtime). Detta är ett **viktigt** steg. Integration Runtime i standardversionen kan inte ansluta till ett lokalt datalager. Använda den lokalt installerade Integration Runtime som du skapade tidigare. 
    1. För **Servernamn** anger du namnet på den dator som har SQL Server-databasen.
    1. För **Databasnamn** anger du namnet på databasen i SQL Server som innehåller källdata. Du skapade en tabell och infogade data i den här databasen som en del av förberedelserna. 
    1. För **Autentiseringstyp** väljer du vilken **typ av autentisering** du vill använda för att ansluta till databasen. 
    1. För **Användarnamn** anger du namnet på en användare som har åtkomst till SQL Server-databasen. Om du behöver använda ett snedstreck (`\`) i ditt användarkonto eller användarnamn använder du escape-tecknet (`\`). Ett exempel är `mydomain\\myuser`.
    1. För **lösen ord**anger du användarens **lösen ord** . 
    1. Om du vill testa om Data Factory kan ansluta till SQL Server-databasen, klickar du på **Testa anslutning**. Åtgärda eventuella fel tills anslutningen lyckas. 
    1. Klicka på **Slutför**om du vill spara den länkade tjänsten.

### <a name="create-the-azure-sql-database-linked-service"></a>Skapa länkad tjänst för Azure SQL Database
I det sista steget skapar du en länkad tjänst för att länka SQL Server-databasen till datafabriken. I det här steget länkar du din (destination/mottagare) Azure SQL-databas till datafabriken. 

1. I fönstret **Anslutningar** växlar du från fliken **Integration Runtimes** till fliken med **länkade tjänster** och klickar på **+ Ny**.
1. I fönstret **New Linked Service** (Ny länkad tjänst) väljer du **Azure SQL Database** och klickar på **Fortsätt**. 
1. Utför följande steg i fönstret **New Linked Service** (Ny länkad tjänst):

    1. Ange **AzureSqlDatabaseLinkedService** som **namn**. 
    1. För **Servernamn** väljer du namnet på din Azure SQL-server från den nedrullningsbara listan. 
    1. För **Databasnamn** väljer du den Azure SQL-databas där du skapade customer_table och project_table som en del av förberedelserna. 
    1. För **Användarnamn** anger du namnet på en användare som har åtkomst till Azure SQL-databasen. 
    1. För **lösen ord**anger du användarens **lösen ord** . 
    1. Om du vill testa om Data Factory kan ansluta till SQL Server-databasen, klickar du på **Testa anslutning**. Åtgärda eventuella fel tills anslutningen lyckas. 
    1. Klicka på **Slutför**om du vill spara den länkade tjänsten.

1. Bekräfta att du ser två länkade tjänster i listan. 
   
    ![Två länkade tjänster](./media/tutorial-incremental-copy-multiple-tables-portal/two-linked-services.png) 

## <a name="create-datasets"></a>Skapa datauppsättningar
I det här steget skapar du datauppsättningar som representerar datakällan, datamålet och platsen för vattenstämpeln.

### <a name="create-a-source-dataset"></a>Skapa en källdatauppsättning

1. I den vänstra rutan klickar du på **+ (plus)** och sedan på **Datauppsättning**.

1. I fönstret **ny data uppsättning** väljer du **SQL Server**, klicka på **Fortsätt**. 

1. Du ser en ny flik öppnas i webbläsaren för att konfigurera datauppsättningen. Du ser också en data uppsättning i trädvyn. Längst ned på fliken **Allmänt** i fönstret Egenskaper skriver du **SourceDataset** som **namn**. 

1. Byt till fliken **Connection** (Anslutning) i fönstret Egenskaper och välj **SqlServerLinkedService** för **Linked service** (Länkad tjänst). Du väljer inte en tabell här. Kopieringsaktivitet i pipelinen använder en SQL-fråga till att läsa in data, snarare än att läsa in hela tabellen.

   ![Källdatauppsättning – anslutning](./media/tutorial-incremental-copy-multiple-tables-portal/source-dataset-connection.png)


### <a name="create-a-sink-dataset"></a>Skapa en källdatauppsättning
1. I den vänstra rutan klickar du på **+ (plus)** och sedan på **Datauppsättning**.

1. I fönstret **ny data uppsättning** väljer du **Azure SQL Database**och klickar på **Fortsätt**. 

1. Du ser en ny flik öppnas i webbläsaren för att konfigurera datauppsättningen. Du ser också en data uppsättning i trädvyn. Längst ned på fliken **Allmänt** i fönstret Egenskaper skriver du **SinkDataset** som **namn**.

1. Växla till fliken **Parameters** (Parametrar) i fönstret Egenskaper och utför följande steg: 

    1. Klicka på **+ Ny** i avsnittet för att **skapa/uppdatera parametrar**. 
    1. Ange **SinkTableName** som **namn**, och **String** som **typ**. Den här datauppsättningen tar **SinkTableName** som en parameter. Parametern SinkTableName anges dynamiskt vid körning av pipelinen. ForEach-aktiviteten i pipelinen upprepas över en lista med tabellnamn och skickar tabellnamnet till datamängden i varje iteration.
   
    ![Mottagardatauppsättning – egenskaper](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-parameters.png)
1. Växla till fliken **anslutning** i fönstret Egenskaper och välj **AzureSqlDatabaseLinkedService** för **länkad tjänst**. För egenskapen **Table** (Tabell) klickar du på **Add dynamic content** (Lägg till dynamiskt innehåll).   
    
1. I fönstret **Lägg till dynamiskt innehåll** väljer du **SinkTableName** i avsnittet **parametrar** . 
 
1. När du **har**klickat på Slutför@datasetvisas "(). SinkTableName "som tabell namn.

   ![Datauppsättning för mottagare – anslutning](./media/tutorial-incremental-copy-multiple-tables-portal/sink-dataset-connection-completion.png)

### <a name="create-a-dataset-for-a-watermark"></a>Skapa en datauppsättning för en vattenstämpel
I det här steget skapar du en datauppsättning för att lagra ett värde för ett högvattenmärke. 

1. I den vänstra rutan klickar du på **+ (plus)** och sedan på **Datauppsättning**.

1. I fönstret **ny data uppsättning** väljer du **Azure SQL Database**och klickar på **Fortsätt**. 

1. Längst ned på fliken **Allmänt** i fönstret Egenskaper skriver du **WatermarkDataset** som **namn**.
1. Välj fliken **Anslutning** och gör följande: 

    1. Välj **AzureSqlDatabaseLinkedService** som **Länkad tjänst**.
    1. Välj **[dbo].[watermarktable]** för **Tabell**.

    ![Vattenmärkesdatauppsättning – anslutning](./media/tutorial-incremental-copy-multiple-tables-portal/watermark-dataset-connection.png)

## <a name="create-a-pipeline"></a>Skapa en pipeline
Den här pipelinen tar en lista med tabellnamn som en parameter. ForEach-aktiviteten upprepas över listan med tabellnamn och utför följande åtgärder: 

1. Använd sökningsaktiviteten till att hämta det gamla vattenmärkesvärdet (startvärdet eller det som användes i den senaste iterationen).

1. Använd sökningsaktiviteten för att hämta det nya högvattenmärket (högsta värdet i kolumnen vattenmärke i källtabellen).

1. Använd kopieringsaktiviteten till att kopiera data mellan de två vattenmärkesvärdena från källdatabasen till måldatabasen.

1. Använd StoredProcedure-aktiviteten för att uppdatera det gamla vattenmärket som ska användas i det första steget i nästa iteration. 

### <a name="create-the-pipeline"></a>Skapa pipelinen

1. I den vänstra rutan klickar du på **+ (plus)** och sedan på **Pipeline**.

1. På fliken **Allmänt** anger du **IncrementalCopyPipeline** som **namn**. 

1. Gör så här på fliken **parametrar** : 

    1. Klicka på **+ Ny**. 
    1. Ange **tableList** som parameterns **namn**. 
    1. Välj **matris** för parameter **typen**.

1. I verktygslådan **Aktiviteter** expanderar du **Iteration & Conditions** (Iteration och villkor) och drar och släpper aktiviteten **ForEach** till pipelinedesignytan. På fliken **Allmänt** i fönstret Egenskaper skriver du **IterateSQLTables** som **namn**. 

1. Växla till fliken **Inställningar** och ange `@pipeline().parameters.tableList` för **Objekt**. Aktiviteten ForEach upprepas över listan med tabeller och utför följande inkrementella kopieringsåtgärd. 

    ![Aktiviteten ForEach – inställningar](./media/tutorial-incremental-copy-multiple-tables-portal/foreach-settings.png)

1. Markera aktiviteten **ForEach** i pipelinen om det inte redan är markerat. Klicka på knappen **Redigera (pennikonen)**.

1. I verktygslådan **Aktiviteter** expanderar du **Allmänt** och drar och släpper **sökningen** på pipelinedesignytan. Ange **LookupOldWaterMarkActivity** som **Namn**.

1. Växla till fliken **Settings** (Inställningar) i fönstret **Egenskaper** och utför följande steg: 

    1. Markera **WatermarkDataset** för **Källdatauppsättning**.
    1. Välj **Fråga** för **Använd fråga**. 
    1. Ange följande SQL-fråga för **Fråga**. 

        ```sql
        select * from watermarktable where TableName  =  '@{item().TABLE_NAME}'
        ```

        ![Första Lookup-aktiviteten – inställningar](./media/tutorial-incremental-copy-multiple-tables-portal/first-lookup-settings.png)
1. Dra och släpp aktiviteten **Lookup** från verktygslådan **Aktiviteter** och ange **LookupNewWaterMarkActivity** som **namn**.
        
1. Växla till fliken **Inställningar** .

    1. Markera **SourceDataset** för **Källdatauppsättning**. 
    1. Välj **Fråga** för **Använd fråga**.
    1. Ange följande SQL-fråga för **Fråga**.

        ```sql    
        select MAX(@{item().WaterMark_Column}) as NewWatermarkvalue from @{item().TABLE_NAME}
        ```
    
        ![Andra Lookup-aktiviteten – inställningar](./media/tutorial-incremental-copy-multiple-tables-portal/second-lookup-settings.png)
1. Dra och släpp aktiviteten **Copy** (Kopiera) från verktygslådan **Aktiviteter** och ange **IncrementalCopyActivity** som **namn**. 

1. Koppla aktiviteterna **Lookup** (Sökning) till aktiviteten **Copy** (Kopiering), en i taget. Koppla genom att börja dra den **gröna** rutan som hör till **Lookup**-aktiviteten och släpp den på **Copy**-aktiviteten. Släpp musknappen när du ser att kantlinjefärgen för kopieringsaktiviteten ändras till **blått**.

    ![Koppla sökningsaktiviteter till kopieringsaktivitet](./media/tutorial-incremental-copy-multiple-tables-portal/connect-lookup-to-copy.png)
1. Markera **Copy**-aktiviteten i pipeline. Växla till fliken **Source** (Käll) i **egenskapsfönstret**. 

    1. Markera **SourceDataset** för **Källdatauppsättning**. 
    1. Välj **Fråga** för **Använd fråga**. 
    1. Ange följande SQL-fråga för **Fråga**.

        ```sql
        select * from @{item().TABLE_NAME} where @{item().WaterMark_Column} > '@{activity('LookupOldWaterMarkActivity').output.firstRow.WatermarkValue}' and @{item().WaterMark_Column} <= '@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}'        
        ```

        ![Kopiera aktivitet – källinställningar](./media/tutorial-incremental-copy-multiple-tables-portal/copy-source-settings.png)
1. Växla till fliken **Sink** (Mottagare) och markera **SinkDataset** för **Sink Dataset** (Datauppsättning för mottagare). 
        
1. Gör så här:

    1. I **egenskaperna för data uppsättningen**, för parametern **SinkTableName** , `@{item().TABLE_NAME}`anger du.
    1. Som egenskap för **lagrad procedur namn** anger `@{item().StoredProcedureNameForMergeOperation}`du.
    1. För egenskapen **tabell typ** anger `@{item().TableType}`du.
    1. För **tabell typ parameter namn**anger `@{item().TABLE_NAME}`du.

    ![Kopieringsaktiviteten – parametrar](./media/tutorial-incremental-copy-multiple-tables-portal/copy-activity-parameters.png)
1. Dra och släpp aktiviteten **Lagrad procedur** från verktygslådan **Aktiviteter** till pipelinedesignytan. Koppla aktiviteten **Copy** (Kopiera) till aktiviteten **Lagrad procedur**. 

1. Välj aktiviteten **Lagrad procedur** i pipelinen och ange **StoredProceduretoWriteWatermarkActivity** för **namn** på fliken **Allmänt** i fönstret **Egenskaper**. 

1. Växla till fliken **SQL-konto** och välj **AzureSqlDatabaseLinkedService** för **länkad tjänst**.

    ![Lagrad proceduraktivitet – SQL-konto](./media/tutorial-incremental-copy-multiple-tables-portal/sproc-activity-sql-account.png)
1. Växla till fliken **Lagrad procedur** och gör följande:

    1. Som **Namn på lagrad procedur** väljer du `[dbo].[usp_write_watermark]`. 
    1. Välj **Importera parameter**. 
    1. Ange följande värden för parametrarna: 

        | Name | Typ | Värde | 
        | ---- | ---- | ----- |
        | LastModifiedtime | DateTime | `@{activity('LookupNewWaterMarkActivity').output.firstRow.NewWatermarkvalue}` |
        | TableName | Sträng | `@{activity('LookupOldWaterMarkActivity').output.firstRow.TableName}` |
    
        ![Lagrad proceduraktivitet – inställningar för lagrad procedur](./media/tutorial-incremental-copy-multiple-tables-portal/sproc-activity-sproc-settings.png)
1. Välj **publicera alla** för att publicera de entiteter som du skapade i Data Factory-tjänsten. 

1. Vänta tills du ser meddelandet om att entiteterna **har publicerats**. Klicka på länken **Visa meddelanden** om du vill se dem. Stäng meddelandefönstret genom att klicka på **X**.

 
## <a name="run-the-pipeline"></a>Köra en pipeline

1. I verktygsfältet för pipelinen klickar du på **Lägg till utlösare**och klickar sedan på **Utlös nu**.     

1. I fönstret **Pipelinekörning** anger du följande värde för parametern **tableList**. Klicka på **Slutför**. 

    ```
    [
        {
            "TABLE_NAME": "customer_table",
            "WaterMark_Column": "LastModifytime",
            "TableType": "DataTypeforCustomerTable",
            "StoredProcedureNameForMergeOperation": "usp_upsert_customer_table"
        },
        {
            "TABLE_NAME": "project_table",
            "WaterMark_Column": "Creationtime",
            "TableType": "DataTypeforProjectTable",
            "StoredProcedureNameForMergeOperation": "usp_upsert_project_table"
        }
    ]
    ```

    ![Pipelinekörningsargument](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-run-arguments.png)

## <a name="monitor-the-pipeline"></a>Övervaka pipeline

1. Växla till fliken **Övervaka** till vänster. Du kan se den pipelinekörning som utlöstes av den **manuella utlösaren**. Om du vill uppdatera listan klickar du på **Uppdatera**. Med länkarna i kolumnen **Action** (Åtgärd) kan du visa de aktivitetskörningar som är associerade med pipelinekörningar och köra pipelinen på nytt. 

    ![Pipelinekörningar](./media/tutorial-incremental-copy-multiple-tables-portal/pipeline-runs.png)
1. Klicka på länken **View Activity Runs** (Visa aktivitetskörningar) i kolumnen **Åtgärder**. Du ser alla aktivitetskörningar som är associerade med den valda pipelinekörningen. 

## <a name="review-the-results"></a>Granska resultaten
Kör följande frågor mot SQL-måldatabasen i SQL Server Management Studio för att verifiera att data har kopierats från källtabellerna till måltabellerna: 

**Söka i data** 
```sql
select * from customer_table
```

**Resultat**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           Alice   2017-09-03 02:36:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

**Söka i data**

```sql
select * from project_table
```

**Resultat**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
```

**Söka i data**

```sql
select * from watermarktable
```

**Resultat**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-05 08:06:00.000
project_table   2017-03-04 05:16:00.000
```

Observera att vattenstämpelvärdena för båda tabellerna har uppdaterats. 

## <a name="add-more-data-to-the-source-tables"></a>Lägga till mer data i källtabellerna

Kör följande fråga mot SQL Servers källdatabas för att uppdatera en befintlig rad i customer_table. Infoga en ny rad i project_table. 

```sql
UPDATE customer_table
SET [LastModifytime] = '2017-09-08T00:00:00Z', [name]='NewName' where [PersonID] = 3

INSERT INTO project_table
(Project, Creationtime)
VALUES
('NewProject','10/1/2017 0:00:00 AM');
``` 

## <a name="rerun-the-pipeline"></a>Kör pipelinen igen
1. Växla till fliken **Redigera** till vänster i webbläsarfönstret. 
1. I verktygsfältet för pipelinen klickar du på **Lägg till utlösare**och klickar sedan på **Utlös nu**.   
1. I fönstret **Pipelinekörning** anger du följande värde för parametern **tableList**. Klicka på **Slutför**. 

    ```
    [
        {
            "TABLE_NAME": "customer_table",
            "WaterMark_Column": "LastModifytime",
            "TableType": "DataTypeforCustomerTable",
            "StoredProcedureNameForMergeOperation": "usp_upsert_customer_table"
        },
        {
            "TABLE_NAME": "project_table",
            "WaterMark_Column": "Creationtime",
            "TableType": "DataTypeforProjectTable",
            "StoredProcedureNameForMergeOperation": "usp_upsert_project_table"
        }
    ]
    ```

## <a name="monitor-the-pipeline-again"></a>Övervaka pipelinen igen

1. Växla till fliken **Övervaka** till vänster. Du kan se den pipelinekörning som utlöstes av den **manuella utlösaren**. Om du vill uppdatera listan klickar du på **Uppdatera**. Med länkarna i kolumnen **Action** (Åtgärd) kan du visa de aktivitetskörningar som är associerade med pipelinekörningar och köra pipelinen på nytt. 

1. Klicka på länken **View Activity Runs** (Visa aktivitetskörningar) i kolumnen **Åtgärder**. Du ser alla aktivitetskörningar som är associerade med den valda pipelinekörningen. 

## <a name="review-the-final-results"></a>Granska de slutliga resultaten
I SQL Server Management Studio kör du följande frågor mot mål-SQL-databasen för att kontrol lera att uppdaterade/nya data har kopierats från käll tabellerna till mål tabellerna. 

**Söka i data** 
```sql
select * from customer_table
```

**Resultat**
```
===========================================
PersonID    Name    LastModifytime
===========================================
1           John    2017-09-01 00:56:00.000
2           Mike    2017-09-02 05:23:00.000
3           NewName 2017-09-08 00:00:00.000
4           Andy    2017-09-04 03:21:00.000
5           Anny    2017-09-05 08:06:00.000
```

Lägg märke till de nya värdena för **Name** och **LastModifytime** för **PersonID** för nummer 3. 

**Söka i data**

```sql
select * from project_table
```

**Resultat**

```
===================================
Project     Creationtime
===================================
project1    2015-01-01 00:00:00.000
project2    2016-02-02 01:23:00.000
project3    2017-03-04 05:16:00.000
NewProject  2017-10-01 00:00:00.000
```

Observera att posten **NewProject** har lagts till i project_table. 

**Söka i data**

```sql
select * from watermarktable
```

**Resultat**

```
======================================
TableName       WatermarkValue
======================================
customer_table  2017-09-08 00:00:00.000
project_table   2017-10-01 00:00:00.000
```

Observera att vattenstämpelvärdena för båda tabellerna har uppdaterats.
     
## <a name="next-steps"></a>Nästa steg
I den här självstudiekursen fick du: 

> [!div class="checklist"]
> * Förbereda käll- och måldatalager.
> * Skapa en datafabrik.
> * Skapa Integration Runtime (IR) med egen värd.
> * Installera Integration Runtime.
> * Skapa länkade tjänster. 
> * Skapa datauppsättningar för källa, mottagare och vattenstämpel.
> * Skapa, köra och övervaka en pipeline.
> * Granska resultaten.
> * Lägga till eller uppdatera data i källtabeller.
> * Köra och övervaka pipelinen igen.
> * Granska de slutliga resultaten.

Fortsätt till följande självstudie och lär dig att transformera data med ett Spark-kluster på Azure:

> [!div class="nextstepaction"]
>[Läs in data stegvis från Azure SQL Database till Azure Blob Storage med ändringsspårningsteknik](tutorial-incremental-copy-change-tracking-feature-portal.md)


