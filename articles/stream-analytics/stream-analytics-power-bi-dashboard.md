---
title: Power BI instrument panels integrering med Azure Stream Analytics
description: Den här artikeln beskriver hur du använder en Power BI instrument panel i real tid för att visualisera data från ett Azure Stream Analytics jobb.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/05/2019
ms.openlocfilehash: 8466fbcb4325dc244551a3b84fc20581366b7071
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "78851147"
---
# <a name="stream-analytics-and-power-bi-a-real-time-analytics-dashboard-for-streaming-data"></a>Stream Analytics och Power BI: en real tids analys instrument panel för strömmande data

Med Azure Stream Analytics kan du dra nytta av ett av de ledande Business Intelligence verktygen, [Microsoft Power BI](https://powerbi.com/). I den här artikeln får du lära dig hur du skapar Business Intelligence verktyg med Power BI som utdata för dina Azure Stream Analytics jobb. Du lär dig också hur du skapar och använder en instrument panel i real tid.

Den här artikeln fortsätter från den Stream Analytics själv studie kursen om att [upptäcka bedrägerier i real tid](stream-analytics-real-time-fraud-detection.md) . Den bygger på arbets flödet som skapats i den självstudien och lägger till ett Power BI utdata så att du kan visualisera falska telefonsamtal som identifieras av ett strömmande analys jobb. 

Du kan titta på [en video](https://www.youtube.com/watch?v=SGUpT-a99MA) som illustrerar det här scenariot.


## <a name="prerequisites"></a>Krav

Se till att du har följande innan du börjar:

* Ett Azure-konto.
* Ett konto för Power BI Pro. Du kan använda ett arbets konto eller ett skol konto.
* En slutförd version av självstudien om att [upptäcka bedrägerier i real tid](stream-analytics-real-time-fraud-detection.md) . Självstudien innehåller en app som genererar fiktiva telefonsamtal. I självstudien skapar du en Event Hub och skickar samtals data från strömmade telefonsamtal till Event Hub. Du skriver en fråga som identifierar bedrägliga anrop (samtal från samma nummer vid samma tidpunkt på olika platser). 


## <a name="add-power-bi-output"></a>Lägg till Power BI utdata
I självstudien om bedrägeri identifiering i real tid skickas utdata till Azure Blob Storage. I det här avsnittet lägger du till utdata som skickar information till Power BI.

1. Öppna det strömmande analys jobb som du skapade tidigare i Azure Portal. Om du använde det föreslagna namnet namnges `sa_frauddetection_job_demo`jobbet.

2. På den vänstra menyn väljer du **utdata** under **jobb sto pol Ogin**. Välj sedan **+ Lägg till** och välj **Power BI** på list menyn.

3. Välj **+ Lägg till** > **Power BI**. Fyll sedan i formuläret med följande information och välj **Auktorisera**:

   |**Inställning**  |**Föreslaget värde**  |
   |---------|---------|
   |Utdataalias  |  CallStream – PowerBI  |
   |Namn på datauppsättning  |   sa – data uppsättning  |
   |Tabellnamn |  bedrägliga samtal  |

   ![Konfigurera Stream Analytics-utdata](media/stream-analytics-power-bi-dashboard/configure-stream-analytics-output.png)

   > [!WARNING]
   > Om Power BI har en data uppsättning och en tabell med samma namn som de som du anger i Stream Analytics jobb, skrivs befintliga befintliga.
   > Vi rekommenderar att du inte uttryckligen skapar den här data uppsättningen och tabellen i ditt Power BI-konto. De skapas automatiskt när du startar Stream Analytics-jobbet och jobbet börjar pumpa utdata i Power BI. Om din jobb fråga inte returnerar några resultat skapas inte data uppsättningen och tabellen.
   >

4. När du har valt **Auktorisera** visas ett popup-fönster och du ombeds ange autentiseringsuppgifter för att autentisera ditt Power BI-konto. När auktoriseringen är klar ska du **Spara** inställningarna.

8. Klicka på **Skapa**.

Data uppsättningen skapas med följande inställningar:

* **defaultRetentionPolicy: BasicFIFO** -data är FIFO, med högst 200 000 rader.
* **defaultMode: pushStreaming** – data uppsättningen stöder både strömmande paneler och traditionella rapportbaserade visuella objekt (kallas även push).

För närvarande kan du inte skapa data uppsättningar med andra flaggor.

Mer information om Power BI data uppsättningar finns i [Power BI REST API](https://msdn.microsoft.com/library/mt203562.aspx) referens.


## <a name="write-the-query"></a>Skriv frågan

1. Stäng bladet **utdata** och gå tillbaka till bladet jobb.

2. Klicka i rutan **fråga** . 

3. Ange följande fråga. Den här frågan liknar den själv kopplings fråga som du skapade i själv studie kursen för bedrägeri identifiering. Skillnaden är att den här frågan skickar resultat till de nya utdata du skapade (`CallStream-PowerBI`). 

    >[!NOTE]
    >Om du inte har `CallStream` angett något namn i själv studie kursen `CallStream` för bedrägeri identifiering, ersätter du ditt namn i **from** -och **Join** -satserna i frågan.

   ```SQL
   /* Our criteria for fraud:
   Calls made from the same caller to two phone switches in different locations (for example, Australia and Europe) within five seconds */

   SELECT System.Timestamp AS WindowEnd, COUNT(*) AS FraudulentCalls
   INTO "CallStream-PowerBI"
   FROM "CallStream" CS1 TIMESTAMP BY CallRecTime
   JOIN "CallStream" CS2 TIMESTAMP BY CallRecTime

   /* Where the caller is the same, as indicated by IMSI (International Mobile Subscriber Identity) */
   ON CS1.CallingIMSI = CS2.CallingIMSI

   /* ...and date between CS1 and CS2 is between one and five seconds */
   AND DATEDIFF(ss, CS1, CS2) BETWEEN 1 AND 5

   /* Where the switch location is different */
   WHERE CS1.SwitchNum != CS2.SwitchNum
   GROUP BY TumblingWindow(Duration(second, 1))
   ```

4. Klicka på **Spara**.


## <a name="test-the-query"></a>Testa frågan

Det här avsnittet är valfritt, men rekommenderas. 

1. Om TelcoStreaming-appen inte körs för tillfället startar du den genom att följa dessa steg:

    * Öppna kommando tolken.
    * Gå till mappen där filerna telcogenerator. exe och Modified telcodatagen. exe. config finns.
    * Kör följande kommando:

       `telcodatagen.exe 1000 .2 2`

2. På sidan **fråga** för ditt Stream Analytics jobb klickar du på punkterna bredvid `CallStream` indata och väljer sedan **exempel data från indata**.

3. Ange att du vill ha tre minuter med data och klicka på **OK**. Vänta tills du får ett meddelande om att data har samplats.

4. Klicka på **testa** och granska resultaten.

## <a name="run-the-job"></a>Kör jobbet

1. Kontrol lera att TelcoStreaming-appen körs.

2. Gå till sidan **Översikt** för Stream Analytics jobbet och välj **Starta**.

    ![Starta Stream Analytics jobbet](./media/stream-analytics-power-bi-dashboard/stream-analytics-sa-job-start-output.png)

Ditt strömnings analys jobb börjar söka efter bedrägliga anrop i den inkommande data strömmen. Jobbet skapar även data uppsättningen och tabellen i Power BI och börjar skicka data om bedrägliga anrop till dem.


## <a name="create-the-dashboard-in-power-bi"></a>Skapa instrument panelen i Power BI

1. Gå till [PowerBI.com](https://powerbi.com) och logga in med ditt arbets-eller skol konto. Om den Stream Analytics jobb fråga returnerar resultat, ser du att data uppsättningen redan har skapats:

    ![Strömmande data uppsättnings plats i Power BI](./media/stream-analytics-power-bi-dashboard/stream-analytics-streaming-dataset.png)

2. Klicka på ** + &nbsp;skapa**i arbets ytan.

    ![Knappen Skapa i Power BI arbets ytan](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard.png)

3. Skapa en ny instrument panel och ge `Fraudulent Calls`den namnet.

    ![Skapa en instrument panel och ge den ett namn i Power BI arbets yta](./media/stream-analytics-power-bi-dashboard/pbi-create-dashboard-name.png)

4. Längst upp i fönstret klickar du på **Lägg till panel**, väljer **anpassade strömmande data**och klickar sedan på **Nästa**.

    ![Anpassad panel för strömmande data uppsättning i Power BI](./media/stream-analytics-power-bi-dashboard/custom-streaming-data.png)

5. Under **din DATSETS**väljer du din data uppsättning och klickar sedan på **Nästa**.

    ![Din strömmande data uppsättning i Power BI](./media/stream-analytics-power-bi-dashboard/your-streaming-dataset.png)

6. Under **visualiserings typ**väljer du **kort**och i listan **fält** väljer du sedan **fraudulentcalls**.

    ![Visualiserings information för ny panel](./media/stream-analytics-power-bi-dashboard/add-fraudulent-calls-tile.png)

7. Klicka på **Nästa**.

8. Fyll i panel information som rubrik och under rubrik.

    ![Rubrik och under rubrik för den nya panelen](./media/stream-analytics-power-bi-dashboard/pbi-new-tile-details.png)

9. Klicka på **Använd**.

    Nu har du en bedrägeri räknare!

    ![Bedrägeri räknare i Power BI instrument panel](./media/stream-analytics-power-bi-dashboard/power-bi-fraud-counter-tile.png)

8. Följ stegen igen för att lägga till en panel (från och med steg 4). Den här gången gör du följande:

    * När du kommer till **visualiserings typen**väljer du **linje diagram**. 
    * Lägg till en axel och välj **windowend**. 
    * Lägg till ett värde och välj **fraudulentcalls**.
    * För **Tidsfönster att visa** väljer du de senaste 10 minuterna.

      ![Skapa en panel för linje diagrammet i Power BI](./media/stream-analytics-power-bi-dashboard/pbi-create-tile-line-chart.png)

9. Klicka på **Nästa**, Lägg till en rubrik och en under rubrik och klicka på **Använd**.

     På instrument panelen för Power BI får du nu två vyer av data om bedrägliga anrop som identifieras i strömmande data.

     ![Har Power BI instrument panelen visar två paneler för bedrägliga samtal](./media/stream-analytics-power-bi-dashboard/pbi-dashboard-fraudulent-calls-finished.png)


## <a name="learn-more-about-power-bi"></a>Läs mer om Power BI

Den här självstudien visar hur du skapar bara några typer av visualiseringar för en data uppsättning. Power BI kan hjälpa dig att skapa andra kund Business Intelligence verktyg för din organisation. Fler idéer finns i följande resurser:

* Ett annat exempel på en Power BI instrument panel finns på [komma igång med Power BI](https://youtu.be/L-Z_6P56aas?t=1m58s) video.
* Mer information om hur du konfigurerar utdata från Stream Analytics-jobb till Power BI och använder Power BI grupper finns i avsnittet [Power BI](stream-analytics-define-outputs.md#power-bi) i artikeln [Stream Analytics utdata](stream-analytics-define-outputs.md) . 
* Information om hur du använder Power BI vanligt vis finns [i instrument paneler i Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-dashboards/).


## <a name="learn-about-limitations-and-best-practices"></a>Lär dig mer om begränsningar och bästa praxis
För närvarande kan Power BI anropas ungefär en gång per sekund. Stöd paket för direkt uppspelning av visualiseringar på 15 KB. Utöver det fungerar strömmande visuella objekt (men push fortsätter att fungera). På grund av de här begränsningarna är Power BI bäst i de fall där Azure Stream Analytics gör en betydande data inläsnings minskning. Vi rekommenderar att du använder ett rullande-fönster eller ett hoppande-fönster för att säkerställa att data-push är högst en push per sekund och att frågan omfattas av data flödes kraven.

Du kan använda följande ekvation för att beräkna värdet för att ge ditt fönster på några sekunder:

![Ekvation till beräknings värde för att ge fönster på några sekunder](./media/stream-analytics-power-bi-dashboard/compute-window-seconds-equation.png)  

Ett exempel:

* Du har 1 000 enheter som skickar data med en sekunds intervall.
* Du använder den Power BI Pro SKU som stöder 1 000 000 rader per timme.
* Du vill publicera mängden Genomsnittligt antal data per enhet till Power BI.

Därför blir formeln:

![Ekvation, baserat på exempel villkor](./media/stream-analytics-power-bi-dashboard/power-bi-example-equation.png)  

Med den här konfigurationen kan du ändra den ursprungliga frågan till följande:

```SQL
    SELECT
        MAX(hmdt) AS hmdt,
        MAX(temp) AS temp,
        System.TimeStamp AS time,
        dspl
    INTO "CallStream-PowerBI"
    FROM
        Input TIMESTAMP BY time
    GROUP BY
        TUMBLINGWINDOW(ss,4),
        dspl
```

### <a name="renew-authorization"></a>Förnya auktorisering
Om lösen ordet har ändrats sedan jobbet skapades eller senast autentiserades måste du autentisera ditt Power BI-konto. Om Azure Multi-Factor Authentication har kon figurer ATS på din Azure Active Directory (Azure AD)-klient måste du också förnya Power BI auktorisering varannan vecka. Om du inte förnyar kan du se symptom som brist på utdata från jobb eller en `Authenticate user error` i åtgärds loggarna.

Om ett jobb startar efter att token har gått ut, uppstår ett fel och jobbet Miss lyckas. Lös problemet genom att stoppa jobbet som körs och gå till Power BI utdata. För att undvika data förlust väljer du länken **förnya auktorisering** och startar sedan om jobbet från den **senaste stopp tiden**.

När auktoriseringen har uppdaterats med Power BI visas en grön avisering i avsnittet Authorization som visar att problemet har lösts.

## <a name="get-help"></a>Få hjälp
Om du behöver ytterligare hjälp kan du prova vårt [Azure Stream Analytics-forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics).

## <a name="next-steps"></a>Nästa steg
* [Introduktion till Azure Stream Analytics](stream-analytics-introduction.md)
* [Komma igång med Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Skala Azure Stream Analytics-jobb](stream-analytics-scale-jobs.md)
* [Språk referens för Azure Stream Analytics-fråga](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referens för Azure Stream Analytics hanterings REST API](https://msdn.microsoft.com/library/azure/dn835031.aspx)
