---
title: Testa ett Azure Stream Analytics jobb lokalt med exempel data med Visual Studio Code
description: Den här artikeln beskriver hur du testar frågor lokalt med exempel data med hjälp av Azure Stream Analytics verktyg för Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/10/2019
ms.topic: conceptual
ms.openlocfilehash: c29d0d9ecd856ee9611df21d23b1b2b763e24652
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75486474"
---
# <a name="test-stream-analytics-queries-locally-with-sample-data-using-visual-studio-code"></a>Testa Stream Analytics frågor lokalt med exempel data med Visual Studio Code

Du kan använda Azure Stream Analytics verktyg för Visual Studio Code för att testa dina Stream Analytics-jobb lokalt med exempel data. Du kan hitta frågeresultaten i JSON-filer i mappen **LocalRunOutputs** i projektet.

## <a name="prerequisites"></a>Krav

* Installera [.net Core SDK](https://dotnet.microsoft.com/download) och starta om Visual Studio Code.

* Använd den här [snabb](quick-create-vs-code.md) starten för att lära dig hur du skapar ett Stream Analytics jobb med Visual Studio Code.

## <a name="prepare-sample-data"></a>Förbereda exempel data

Du måste förbereda exempelfilerna med indata först. Om du redan har några exempel data filer på datorn kan du hoppa över det här steget och gå vidare till nästa.

1. Klicka på **Förhandsgranska data** i konfigurations filen för indata på den översta raden. Vissa indata hämtas från IoT Hub och visas i förhands gransknings fönstret. Observera att det kan ta en stund.

2. När data har visats klickar du på **Spara som** för att spara data till en lokal fil.

 ![Förhandsgranska Live-ingångar](./media/quick-create-vs-code/preview-live-input.png)

## <a name="define-a-local-input"></a>Definiera en lokal indatamängd

1. Klicka på **input. JSON** under indata-mappen i Stream Analytics-projektet. Välj sedan **Lägg till lokal inström** från den översta raden.

    ![Lägg till lokal Indatatyp från projekt](./media/quick-create-vs-code/add-input-from-project.png)

    Du kan också använda **Ctrl + Shift + P** för att öppna kommando paletten och ange **ASA: Lägg till information**.

   ![Lägg till Stream Analytics in VS Code](./media/quick-create-vs-code/add-input.png)

2. Välj **lokal indatamängd**.

    ![Lägg till en lokal indatatyp i ASA i Visual Studio Code](./media/vscode-local-run/add-local-input.png)

3. Välj **+ ny lokal indatamängd**.

    ![Lägga till en ny information lokalt i Visual Studio Code](./media/vscode-local-run/add-new-local-input.png)

4. Ange samma kolumnalias som du använde i frågan.

    ![Lägg till ett nytt lokalt indataports-alias](./media/vscode-local-run/new-local-input-alias.png)

5. I den nyligen genererade **LocalInput_Input. JSON** -filen anger du sökvägen till filen där den lokala data filen finns.

    ![Ange lokal fil Sök väg i Visual Studio](./media/vscode-local-run/local-file-path.png)

6. Välj **Förhandsgranska data** om du vill förhandsgranska indata. Serialiserings typen för dina data identifieras automatiskt om dess JSON eller CSV. Använd väljaren för att visa dina data i **tabell** -eller **RAW** -format. Följande tabell är ett exempel på data i **tabell format**:

     ![Förhandsgranska lokala data i tabell format](./media/vscode-local-run/local-file-preview-table.png)

    Följande tabell är ett exempel på data i RAW- **format**:

    ![Förhandsgranska lokala data i RAW-format](./media/vscode-local-run/local-file-preview-raw.png)

## <a name="run-queries-locally"></a>Kör frågor lokalt

Återgå till Frågeredigeraren och välj **Kör lokalt**. Välj sedan **Använd lokal indatamängd** i list rutan.

![Välj Kör lokalt i Frågeredigeraren](./media/vscode-local-run/run-locally.png)

![Använd lokal indatamängd](./media/vscode-local-run/run-locally-use-local-input.png)

Resultatet visas i det högra fönstret. Du kan klicka på **Kör** för att testa igen. Du kan också välja **Öppna i mapp** för att se resultatet i Utforskaren och öppna dem ytterligare med andra verktyg. Observera att resultat-filerna bara är tillgängliga i JSON-format.

![Visa lokalt körnings resultat](./media/vscode-local-run/run-locally-result.png)

## <a name="next-steps"></a>Nästa steg

* [Testa Azure Stream Analytics jobb lokalt med Live-indatamängden med Visual Studio Code](visual-studio-code-local-run-live-input.md)

* [Utforska Azure Stream Analytics-jobb med Visual Studio Code (för hands version)](visual-studio-code-explore-jobs.md)
