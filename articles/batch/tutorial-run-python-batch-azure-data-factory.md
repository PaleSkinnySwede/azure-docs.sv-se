---
title: Köra Python-skript via Data Factory-Azure Batch python
description: Självstudie – lär dig hur du kör Python-skript som en del av en pipeline genom Azure Data Factory att använda Azure Batch.
author: mammask
ms.devlang: python
ms.topic: tutorial
ms.date: 12/11/2019
ms.author: komammas
ms.custom: mvc
ms.openlocfilehash: 6761896a6555c11d7957f923a5951641c1541012
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "82117071"
---
# <a name="tutorial-run-python-scripts-through-azure-data-factory-using-azure-batch"></a>Självstudie: köra Python-skript via Azure Data Factory med Azure Batch

I den här självstudien får du lära dig att:

> [!div class="checklist"]
> * autentisera med Batch- och Storage-konton
> * Utveckla och köra ett skript i python
> * skapa en pool med beräkningsnoder för att köra ett program
> * Schemalägg dina python-arbetsbelastningar
> * Övervaka din Analytics-pipeline
> * Komma åt dina loggfiler

Exemplet nedan kör ett Python-skript som tar emot CSV-indata från en Blob Storage-behållare, utför en data behandlings process och skriver utdata till en separat Blob Storage-behållare.

Om du inte har en Azure-prenumeration kan du skapa ett [kostnads fritt konto](https://azure.microsoft.com/free/) innan du börjar.

## <a name="prerequisites"></a>Krav

* En installerad [python](https://www.python.org/downloads/) -distribution för lokal testning.
* [Azure](https://pypi.org/project/azure/) `pip` -paketet.
* Ett Azure Batch-konto och ett länkat Azure Storage-konto. Mer information om hur du skapar och länkar batch-konton till lagrings konton finns i [skapa ett batch-konto](quick-create-portal.md#create-a-batch-account) .
* Ett Azure Data Factory konto. Se [skapa en data fabrik](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory) för mer information om hur du skapar en data fabrik via Azure Portal.
* [Batch Explorer](https://azure.github.io/BatchExplorer/).
* [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/).

## <a name="sign-in-to-azure"></a>Logga in på Azure

Logga in på Azure Portal på [https://portal.azure.com](https://portal.azure.com).

[!INCLUDE [batch-common-credentials](../../includes/batch-common-credentials.md)]

## <a name="create-a-batch-pool-using-batch-explorer"></a>Skapa en batch-pool med Batch Explorer

I det här avsnittet ska du använda Batch Explorer för att skapa batch-poolen som din Azure Data Factory-pipeline ska använda. 

1. Logga in för att Batch Explorer med dina Azure-autentiseringsuppgifter.
1. Välj ditt batch-konto
1. Skapa en pool genom att välja **pooler** i det vänstra fältet och sedan knappen **Lägg till** ovanför Sök formuläret. 
    1. Välj ett ID och visnings namn. Vi ska använda `custom-activity-pool` det här exemplet.
    1. Ange skalnings typen till **fast storlek**och ange antalet dedikerade noder till 2.
    1. Under **data vetenskap**väljer du **Dsvm Windows** som operativ system.
    1. Välj `Standard_f2s_v2` som storlek på den virtuella datorn.
    1. Aktivera start uppgiften och Lägg till kommandot `cmd /c "pip install pandas"`. Användar identiteten kan vara som standard **användare av poolen**.
    1. Välj **OK**.

## <a name="create-blob-containers"></a>Skapa BLOB-behållare

Här skapar du Blob-behållare som lagrar dina indata och utdatafiler för OCR-batchjobbet.

1. Logga in för att Storage Explorer med dina Azure-autentiseringsuppgifter.
1. Skapa två BLOB-behållare (en för indatafiler, en för utdatafiler) med hjälp av det lagrings konto som är kopplat till ditt batch-konto genom att följa stegen i [skapa en BLOB-behållare](../vs-azure-tools-storage-explorer-blobs.md#create-a-blob-container).
    * I det här exemplet ska vi anropa vår container för `input`indata och vår behållare `output`för utdata.
1. Ladda `main.py` upp `iris.csv` och till din indatamängd `input` med Storage Explorer genom att följa stegen i [Hantera blobbar i en BLOB-behållare](../vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container)


## <a name="develop-a-script-in-python"></a>Utveckla ett skript i python

Följande Python-skript läser in `iris.csv` data uppsättningen från `input` din behållare, utför en data behandlings process och sparar tillbaka resultatet till `output` behållaren.

``` python
# Load libraries
from azure.storage.blob import BlockBlobService
import pandas as pd

# Define parameters
storageAccountName = "<storage-account-name>"
storageKey         = "<storage-account-key>"
containerName      = "output"

# Establish connection with the blob storage account
blobService = BlockBlobService(account_name=storageAccountName,
                               account_key=storageKey
                               )

# Load iris dataset from the task node
df = pd.read_csv("iris.csv")

# Subset records
df = df[df['Species'] == "setosa"]

# Save the subset of the iris dataframe locally in task node
df.to_csv("iris_setosa.csv", index = False)

# Upload iris dataset
blobService.create_blob_from_text(containerName, "iris_setosa.csv", "iris_setosa.csv")
```

Spara skriptet som `main.py` och ladda upp det till **Azure Storage** -behållaren. Se till att testa och verifiera dess funktioner lokalt innan du laddar upp den till BLOB-behållaren:

``` bash
python main.py
```

## <a name="set-up-an-azure-data-factory-pipeline"></a>Konfigurera en Azure Data Factory pipeline

I det här avsnittet ska du skapa och validera en pipeline med hjälp av python-skriptet.

1. Följ stegen för att skapa en data fabrik under avsnittet "skapa en data fabrik" i [den här artikeln](../data-factory/quickstart-create-data-factory-portal.md#create-a-data-factory).
1. I rutan **fabriks resurser** väljer du knappen + (plus) och väljer sedan **pipeline**
1. På fliken **Allmänt** anger du namnet på pipelinen som "kör python"

    ![](./media/run-python-batch-azure-data-factory/create-pipeline.png)

1. I rutan **aktiviteter** expanderar du **batch-tjänsten**. Dra den anpassade aktiviteten från **aktivitets** verktygs lådan till pipelinens designer-yta.
1. På fliken **Allmänt** anger du **testPipeline** som namn

    ![](./media/run-python-batch-azure-data-factory/create-custom-task.png)
1. På fliken **Azure Batch** lägger du till det **Batch-konto** som skapades i föregående steg och **testar anslutningen** för att kontrol lera att det lyckades

    ![](./media/run-python-batch-azure-data-factory/integrate-pipeline-with-azure-batch.png)

1. På fliken **Inställningar** anger du kommandot `python main.py`.
1. För den **länkade resurs tjänsten**lägger du till det lagrings konto som skapades i föregående steg. Testa anslutningen för att säkerställa att den lyckas.
1. I **mappsökvägen**väljer du namnet på den **Azure Blob Storage** -behållare som innehåller python-skriptet och tillhör ande indata. Detta laddar ned de valda filerna från behållaren till poolens noder innan python-skriptet körs.

    ![](./media/run-python-batch-azure-data-factory/create-custom-task-py-script-command.png)
1. Verifiera pipelineinställningarna genom att klicka på **Verifiera** i verktygsfältet för pipelinen. Bekräfta att pipelinen har verifierats. Du stänger utdata från verifieringen genom att välja &gt;&gt; (högerpil).
1. Klicka på **Felsök** för att testa pipelinen och se till att den fungerar korrekt.
1. Klicka på **publicera** för att publicera pipelinen.
1. Klicka på **Utlös** för att köra python-skriptet som en del av en batch-process.

    ![](./media/run-python-batch-azure-data-factory/create-custom-task-py-success-run.png)

### <a name="monitor-the-log-files"></a>Övervaka loggfilerna

Om varningar eller fel skapas vid körning av skriptet kan du checka ut `stdout.txt` eller `stderr.txt` för mer information om utdata som loggats.

1. Välj **jobb** från vänster sida av batch Explorer.
1. Välj det jobb som skapats av din data fabrik. Förutsatt att du har namngett din `custom-activity-pool`pool väljer `adfv2-custom-activity-pool`du.
1. Klicka på den aktivitet som hade slut på stängnings kod.
1. Visa `stdout.txt` och `stderr.txt` undersöka och diagnostisera problemet.

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du utforskat ett exempel som visar hur du kör Python-skript som en del av en pipeline genom Azure Data Factory att använda Azure Batch.

Mer information om Azure Data Factory finns i:

> [!div class="nextstepaction"]
> [Azure Data Factory](../data-factory/introduction.md)
> [pipelines och aktiviteter](../data-factory/concepts-pipelines-activities.md)
> [anpassade aktiviteter](../data-factory/transform-data-using-dotnet-custom-activity.md)
