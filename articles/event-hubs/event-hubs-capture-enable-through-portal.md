---
title: Event Hubs samla in strömmande händelser med Azure Portal
description: Den här artikeln beskriver hur du aktiverar avbildning av händelser som strömmas via Azure Event Hubs med hjälp av Azure-portalen.
services: event-hubs
documentationcenter: ''
author: spelluru
editor: ''
ms.assetid: ''
ms.service: event-hubs
ms.workload: na
ms.tgt_pltfrm: na
ms.custom: seodec18
ms.devlang: na
ms.topic: conceptual
ms.date: 02/12/2020
ms.author: spelluru
ms.openlocfilehash: 8a6d9456b00e5520e6f4fbb9ccb77b0260731ddd
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77187420"
---
# <a name="enable-capturing-of-events-streaming-through-azure-event-hubs"></a>Aktivera avbildning av händelser som strömmas via Azure Event Hubs

Med Azure [Event Hubs Capture][capture-overview] kan du automatiskt leverera strömmande data i Event Hubs till ett [Azure Blob storage](https://azure.microsoft.com/services/storage/blobs/) -eller [Azure Data Lake Storage gen1-eller gen 2](https://azure.microsoft.com/services/data-lake-store/) -konto som du väljer.

Du kan konfigurera avbildningsfunktionen med [Azure Portal](https://portal.azure.com) när du skapar händelsehubben. Du kan antingen samla in data till en Azure [Blob Storage](https://azure.microsoft.com/services/storage/blobs/) -behållare eller till ett [Azure Data Lake Storage gen 1-eller gen 2](https://azure.microsoft.com/services/data-lake-store/) -konto.

Mer information finns i [Översikt över Event Hubs Capture][capture-overview].

## <a name="capture-data-to-azure-storage"></a>Samla in data till Azure Storage

När du skapar en händelsehubb kan du aktivera avbildningsfunktionen genom att klicka på knappen **På** på portalskärmen **Skapa händelsehubb**. Sedan anger du ett lagringskonto och en container genom att klicka på **Azure Storage** i rutan **Capture Provider** (Avbildningsprovider). Eftersom tjänst-till-tjänst-autentisering används för lagring i avbildningsfunktionen i Event Hubs behöver du inte ange någon anslutningssträng för lagringen. Resursväljaren väljer automatiskt rätt resurs-URI för ditt lagringskonto. Om du använder Azure Resource Manager måste du ange den här URI:n uttryckligen som en sträng.

Standardvärdet för Tidsperiod är 5 minuter. Lägsta värde är 1 och högsta är 15. Du kan ange intervallet 10–500 MB för **Storlek**.

![Tidsperiod för avbildning][1]

> [!NOTE]
> Du kan aktivera eller inaktivera tomma filer om inga händelser inträffar i fönstret Hämta. 

## <a name="capture-data-to-azure-data-lake-storage-gen-2"></a>Samla in data till Azure Data Lake Storage gen 2 

1. Följ avsnittet [skapa en lagrings konto](../storage/common/storage-account-create.md?tabs=azure-portal#create-a-storage-account) för att skapa ett Azure Storage-konto. Ange **hierarkiskt namn område** till **aktiverat** på fliken **Avancerat** för att göra det till ett Azure Data Lake Storage gen 2-konto.
2. Utför följande steg när du skapar en Event Hub: 

    1. Välj **på** för **avbildning**. 
    2. Välj **Azure Storage** som avbildnings leverantör. **Azure Data Lake Store** alternativ som du ser för **inspelnings leverantören** är för gen 1 i Azure Data Lake Storage. Om du vill använda en gen 2 Azure Data Lake Storage väljer du **Azure Storage**.
    2. Välj knappen **Välj behållare** . 

        ![Aktivera avbildning till Data Lake Storage gen 2](./media/event-hubs-capture-enable-through-portal/data-lake-storage-gen2.png)
3. Välj kontot **Azure Data Lake Storage gen 2** i listan. 

    ![Välj Data Lake Storage gen 2](./media/event-hubs-capture-enable-through-portal/select-data-lake-storage-gen2.png)
4. Välj **behållare** (fil system i data Lake Storage gen 2).

    ![Välj fil system i lagringen](./media/event-hubs-capture-enable-through-portal/select-file-system-data-lake-storage.png)
5. På sidan **skapa händelsehubben** väljer du **skapa**. 

    ![Välj knappen Skapa](./media/event-hubs-capture-enable-through-portal/create-event-hub-data-lake-storage.png)

    > [!NOTE]
    > Den behållare som du skapar i ett Azure Data Lake Storage gen 2 med hjälp av det här användar gränssnittet (UI) visas under **fil system** i **Storage Explorer**. På samma sätt visas fil systemet som du skapar i ett Data Lake Storage gen 2-konto som en behållare i det här användar gränssnittet. 


## <a name="capture-data-to-azure-data-lake-storage-gen-1"></a>Samla in data till Azure Data Lake Storage gen 1 

Om du vill samla in data till Azure Data Lake Storage gen 1 skapar du ett Data Lake Storage gen 1-konto och en Event Hub:

### <a name="create-an-azure-data-lake-storage-gen-1-account-and-folders"></a>Skapa ett Azure Data Lake Storage gen 1-konto och mappar

1. Skapa ett Data Lake Storage konto genom att följa anvisningarna i [Kom igång med Azure Data Lake Storage gen 1 med hjälp av Azure Portal](../data-lake-store/data-lake-store-get-started-portal.md).
2. Följ instruktionerna i avsnittet [tilldela behörigheter till Event Hubs](../data-lake-store/data-lake-store-archive-eventhub-capture.md#assign-permissions-to-event-hubs) för att skapa en mapp i data Lake Storage gen 1-konto där du vill samla in data från Event Hubs och tilldela behörigheter till Event Hubs så att den kan skriva data till ditt data Lake Storage gen 1-konto.  


### <a name="create-an-event-hub"></a>Skapa en händelsehubb

1. Händelsehubben måste finnas i samma Azure-prenumeration som Azure Data Lake Storage gen 1-konto som du skapade. Skapa en händelsehubb genom att klicka på knappen **På** under **Capture** (Avbilda) på portalsidan **Skapa händelsehubb**. 
2. Välj **Azure Data Lake Store** i rutan **Capture Provider** (Avbildningsprovider) på portalsidan **Skapa händelsehubb**.
3. I **Välj Arkiv** bredvid list rutan **data Lake Store** anger du det data Lake Storage gen 1-konto som du skapade tidigare och i fältet **data Lake sökväg** anger du sökvägen till den datamapp som du skapade.

    ![Välja Data Lake Storage-konto][3]


## <a name="add-or-configure-capture-on-an-existing-event-hub"></a>Lägga till eller konfigurera avbildningsfunktionen i en befintlig händelsehubb

Du kan konfigurera avbildningsfunktionen i befintliga händelsehubbar som finns i namnområden för Event Hubs. Om du vill aktivera avbildningsfunktionen i en befintlig händelsehubb eller om du vill ändra inställningarna för avbildningsfunktionen, klickar du på namnområdet för att läsa in översiktsskärmen. Klicka sedan på den händelsehubb som du vill aktivera eller ändra avbildningsinställningen för. Klicka slutligen på alternativet **Avbilda** på vänster sida på sidan som är öppen och redigera inställningarna enligt följande bilder:

### <a name="azure-blob-storage"></a>Azure Blob Storage

![Konfigurera Azure Blob Storage][2]

### <a name="azure-data-lake-storage-gen-2"></a>Azure Data Lake Storage Gen 2

![Konfigurera Azure Data Lake Storage gen 2](./media/event-hubs-capture-enable-through-portal/configure-data-lake-storage-gen2.png)

### <a name="azure-data-lake-storage-gen-1"></a>Azure Data Lake Storage gen 1 

![Konfigurera Azure Data Lake Storage][4]

[1]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture1.png
[2]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture2.png
[3]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture3.png
[4]: ./media/event-hubs-capture-enable-through-portal/event-hubs-capture4.png

## <a name="next-steps"></a>Nästa steg

- Läs mer om Event Hubs Capture i [Översikt över Event Hubs Capture][capture-overview].
- Du kan också konfigurera avbildningsfunktionen i Event Hubs med hjälp av Azure Resource Manager-mallar. Mer information finns i [Skapa ett namnområde för Event Hubs med en händelsehubb och aktivera avbildning med hjälp av en Azure Resource Manager-mall](event-hubs-resource-manager-namespace-event-hub-enable-capture.md).
- [Lär dig hur du skapar en Azure Event Grid-prenumeration med en Event Hubs-namnrymd som dess källa](store-captured-data-data-warehouse.md)
- [Kom igång med Azure Data Lake Store med hjälp av Azure Portal](../data-lake-store/data-lake-store-get-started-portal.md)

[capture-overview]: event-hubs-capture-overview.md
