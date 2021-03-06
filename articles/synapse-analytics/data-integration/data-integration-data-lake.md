---
title: Mata in i Azure Data Lake Storage Gen2
description: Lär dig hur du matar in data i Azure Data Lake Storage Gen2 i Azure Synapse Analytics
services: synapse-analytics
author: djpmsft
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: ''
ms.date: 04/15/2020
ms.author: daperlov
ms.reviewer: jrasnick
ms.openlocfilehash: 7b844bcf45417fefc7dd78a26d5dae0b2ce03249
ms.sourcegitcommit: 999ccaf74347605e32505cbcfd6121163560a4ae
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2020
ms.locfileid: "82982912"
---
# <a name="ingesting-data-into-azure-data-lake-storage-gen2"></a>Mata in data i Azure Data Lake Storage Gen2 

I den här artikeln får du lära dig hur du matar in data från en plats till en annan i ett Azure Data Lake gen 2-lagrings konto (Azure Data Lake gen 2) med Azure Synapse Analytics.

## <a name="prerequisites"></a>Krav

* **Azure-prenumeration**: om du inte har en Azure-prenumeration kan du skapa ett [kostnads fritt Azure-konto](https://azure.microsoft.com/free/) innan du börjar.
* **Azure Storage konto**: du använder Azure Data Lake gen 2 som *käll* data lager. Om du inte har ett lagrings konto kan du läsa [skapa ett Azure Storage-konto](../../storage/blobs/data-lake-storage-quickstart-create-account.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) för att skapa ett.

## <a name="create-linked-services"></a>Skapa länkade tjänster

I Azure Synapse Analytics är en länkad tjänst där du definierar din anslutnings information till andra tjänster. I det här avsnittet ska du lägga till Azure Synapse Analytics och Azure Data Lake gen 2 som länkade tjänster.

1. Öppna Azure Synapse Analytics-UX och gå till fliken **Hantera** .
1. Under **externa anslutningar**väljer du **länkade tjänster**.
1. Klicka på **ny**om du vill lägga till en länkad tjänst.
1. Välj panelen Azure Data Lake Storage Gen2 i listan och klicka på **Fortsätt**.
1. Ange autentiseringsuppgifterna för autentisering. Konto nyckel, tjänstens huvud namn och hanterad identitet stöds för närvarande autentiseringstyper som stöds. Verifiera att dina autentiseringsuppgifter är korrekta genom att klicka på Testa anslutning. 
1. Klicka på **skapa** när du är färdig.

## <a name="create-pipeline"></a>Skapa pipeline

En pipeline innehåller det logiska flödet för en körning av en uppsättning aktiviteter. I det här avsnittet ska du skapa en pipeline som innehåller en kopierings aktivitet som matar in data från Azure Data Lake gen 2 i en SQL-pool.

1. Gå till fliken **dirigera** . Klicka på plus ikonen bredvid pipelinens huvud och välj **pipeline**.
1. Under **flytta och transformera** i fönstret aktiviteter drar du **Kopiera data** till pipeline-arbetsytan.
1. Klicka på kopierings aktiviteten och gå till fliken **källa** . Klicka på **nytt** om du vill skapa en ny käll data uppsättning.
1. Välj Azure Data Lake Storage Gen2 som data lager och klicka på Fortsätt.
1. Välj DelimitedText som format och klicka på Fortsätt.
1. I rutan Ange egenskaper väljer du den länkade tjänsten ADLS som du skapade. Ange fil Sök vägen till dina källdata och ange om den första raden har ett sidhuvud. Du kan importera schemat från fil arkivet eller en exempel fil. Klicka på OK när du är färdig.
1. Gå till fliken **mottagare** . Klicka på **ny** för att skapa en ny data uppsättning för mottagare.
1. Välj Azure Data Lake Storage Gen2 som data lager och klicka på Fortsätt.
1. Välj DelimitedText som format och klicka på Fortsätt.
1. I rutan Ange egenskaper väljer du den länkade tjänsten ADLS som du skapade. Ange sökvägen till den mapp där du vill skriva data. Klicka på OK när du är färdig.

## <a name="debug-and-publish-pipeline"></a>Felsöka och publicera pipeline

När du har konfigurerat din pipeline kan du köra en fel söknings körning innan du publicerar dina artefakter för att kontrol lera att allt är korrekt.

1. Välj **Felsöka** i verktygsfält för att felsöka pipelinen. Du ser status för pipelinekörningen på fliken **Utdata** längst ned i fönstret. 
1. När pipelinen har körts kan du gå till det övre verktygsfältet och välja **publicera alla**. Den här åtgärden publicerar entiteter (data uppsättningar och pipeliner) som du skapade i Synapse Analytics-tjänsten.
1. Vänta tills du ser meddelandet om att entiteterna **har publicerats**. Om du vill visa meddelanden klickar du på klock knappen längst upp till höger. 


## <a name="trigger-and-monitor-the-pipeline"></a>Utlös och övervaka pipelinen

I det här steget aktiverar du pipelinen som publicerades i föregående steg manuellt. 

1. Välj **Lägg till utlösare** i verktygsfältet och välj sedan **Utlös nu**. På sidan **Pipeline Run** (Pipelinekörning) väljer du **Slutför**.  
1. Gå till fliken **övervakare** som finns i den vänstra panelen. Du ser en pipelinekörning som är utlöst av en manuell utlösare. Du kan använda länkar i kolumnen **åtgärder** för att Visa aktivitets information och köra pipelinen igen.
1. Om du vill se aktivitetskörningar som är associerade med pipelinekörningen, väljer du länken **View Activity Runs** (Visa aktivitetskörningar) i kolumnen **Actions** (Åtgärder). I det här exemplet finns det bara en aktivitet, så du ser bara en post i listan. Om du vill se mer information om kopieringsåtgärden väljer du länken för **detaljer** (glasögonikonen) i kolumnen **Actions** (Åtgärder). Välj **pipeline-körningar** överst för att gå tillbaka till vyn pipelines-körningar. Välj **Uppdatera** för att uppdatera vyn.
1. Kontrol lera att dina data är korrekt skrivna i SQL-poolen.


## <a name="next-steps"></a>Nästa steg

Mer information om data integrering för Synapse-analys finns i artikeln om att mata [in data i en SQL-pool](data-integration-sql-pool.md) .
