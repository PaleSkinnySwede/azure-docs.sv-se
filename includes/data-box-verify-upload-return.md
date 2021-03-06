---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 09/19/2019
ms.author: alkohli
ms.openlocfilehash: a23b0b2c71207bf84a4938d54a78a62efb6cbcbd
ms.sourcegitcommit: c2065e6f0ee0919d36554116432241760de43ec8
ms.translationtype: HT
ms.contentlocale: sv-SE
ms.lasthandoff: 03/26/2020
ms.locfileid: "71172685"
---
När Microsoft tar emot och genomsöker enheten uppdateras orderstatusen till **mottagen**. Sedan görs en fysisk kontroll av skador eller tecken på manipulation.

När kontrollen har slutförts ansluts Data Box till nätverket i Azure-datacentret. Datakopieringen startar automatiskt. Beroende på datastorleken kan kopieringen ta några timmar upp till dagar att slutföra. Du kan övervaka kopieringsförloppet i portalen.

När kopieringen är slutförd uppdateras statusen till **slutförd**.

Försäkra dig om att dina data har överförts till Azure innan du tar bort dem från källan. Dina data kan finnas i:

- Ditt Azure Storage-konto. När du kopierar data till Data Box laddas data beroende på typ upp till någon av följande sökvägar i ditt Azure Storage-konto.

  - För blockblobar och sidblobar: `https://<storage_account_name>.blob.core.windows.net/<containername>/files/a.txt`
  - För Azure Files: `https://<storage_account_name>.file.core.windows.net/<sharename>/files/a.txt`

    Du kan också gå till ditt Azure-lagringskonto i Azure-portalen och navigera därifrån.

- Dina resursgrupper för hanterade diskar. När du skapar hanterade diskar laddas de virtuella hårddiskarna upp som sidblobar och konverteras sedan till hanterade diskar. De hanterade diskarna ansluts till de resursgrupper som anges när ordern skapas. 

    - Om kopieringen till hanterade diskar i Azure lyckades kan du gå till **Beställningsinformation** på Azure-portalen och kontrollera vilka resursgrupper som har angetts för hanterade diskar.

        ![Identifiera resursgrupper för hanterade diskar](media/data-box-verify-upload-return/order-details-managed-disk-resource-groups.png)

        Gå till resursgruppen och leta upp dina hanterade diskar.

        ![Hanterad disk ansluten till resursgrupper](media/data-box-verify-upload-return/managed-disks-resource-group.png)

    - Om du har kopierat en VHDX, eller en dynamisk/differentierad virtuell hårddisk, överförs VHDX/VHD till mellanlagringskontot som en sidblob, men konverteringen av de virtuella hårddiskarna till hanterade diskar misslyckas. Gå till ditt **mellanlagringskonto > Blobar** och välj sedan lämplig container – Standard SSD, Standard HDD eller Premium SSD. De virtuella hårddiskarna laddas upp som sidblobar i mellanlagringskontot.


## <a name="erasure-of-data-from-data-box"></a>Radera data från Data Box
 
När uppladdningen till Azure är klar raderar Data Box-enheten data på diskarna enligt [NIST SP 800-88 Revision 1-riktlinjerna](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).


## <a name="next-steps"></a>Nästa steg

I den här kursen har du lärt dig om Azure Data Box-ämnen som att:

> [!div class="checklist"]
> * Förutsättningar
> * Förbereda för att skicka
> * Skicka Data Box till Microsoft
> * Kontrollera datauppladdning till Azure
> * Radera data från Data Box

Gå vidare till följande artikeln om hur du hanterar Data Box via det lokala webbgränssnittet.

> [!div class="nextstepaction"]
> [Använda lokalt webbgränssnitt för att administrera Azure Data Box](../articles/databox/data-box-local-web-ui-admin.md)