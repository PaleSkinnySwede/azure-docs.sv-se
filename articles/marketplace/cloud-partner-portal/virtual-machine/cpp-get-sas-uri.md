---
title: Hämta signatur-URI för delad åtkomst för din Microsoft Azure-baserade VM-avbildning | Azure Marketplace
description: 'Förklarar hur du hämtar URL: en för signaturen för delad åtkomst (SAS) för din VM-avbildning.'
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 10/19/2018
ms.author: dsindona
ms.openlocfilehash: 46b3764fe2a1e09875f2b92a461591b6ff08540f
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82147896"
---
# <a name="get-shared-access-signature-uri-for-your-vm-image"></a>Hämta signatur-URI för delad åtkomst för din VM-avbildning

> [!IMPORTANT]
> Med början den 13 april 2020 kommer vi att börja flytta hanteringen av dina virtuella Azure-datorer till Partner Center. Efter migreringen kommer du att skapa och hantera dina erbjudanden i Partner Center. Följ instruktionerna i [Hämta signatur-URI för delad åtkomst för din virtuella dator avbildning](https://docs.microsoft.com/azure/marketplace/partner-center-portal/common-sas-uri-issues) för att hantera dina migrerade erbjudanden.

Under publicerings processen måste du ange en URI (Uniform Resource Identifier) för varje virtuell hård disk (VHD) som är associerad med dina SKU: er. Microsoft behöver åtkomst till dessa VHD: er under certifierings processen. Den här artikeln beskriver hur du skapar en URL för signaturer för delad åtkomst (SAS) för varje virtuell hård disk. Du kommer att ange den här URI: n i fliken **SKU: er** i Cloud Partner Portal.

När du genererar SAS-URI: er för dina virtuella hård diskar följer du med följande krav:

- Endast ohanterade virtuella hård diskar stöds.
- `List`och `Read` behörigheter är tillräckliga. `Write` Ge *inte* eller `Delete` åtkomst.
- Tiden för åtkomst (*utgångs datum*) bör vara minst tre veckor från det att SAS-URI: n skapas.
- För att skydda mot UTC-tidsvariationer ställer du in start datumet på en dag före det aktuella datumet. Om det aktuella datumet är till exempel 6 oktober 2014 väljer du 10/5/2014.

## <a name="generate-the-sas-url"></a>Generera SAS-URL

SAS-URL: en kan genereras på två vanliga sätt med hjälp av följande verktyg:

- Microsoft Storage Explorer-grafiskt verktyg tillgängligt för Windows, macOS och Linux
- Microsoft Azure CLI – rekommenderas för icke-Windows OSs och automatiserade eller kontinuerliga integrerings miljöer

### <a name="azure-cli"></a>Azure CLI

Använd följande steg för att skapa en SAS-URI med Azure CLI.

1. Hämta och installera [Microsoft Azure CLI](https://azure.microsoft.com/documentation/articles/xplat-cli-install/). Versioner är tillgängliga för Windows, macOS och olika distributioner i Linux.
2. Skapa en PowerShell-fil`.ps1` (fil namns tillägg), kopiera i följande kod och spara den lokalt.

   ```azurecli-interactive
   az storage container generate-sas --connection-string 'DefaultEndpointsProtocol=https;AccountName=<account-name>;AccountKey=<account-key>;EndpointSuffix=core.windows.net' --name <vhd-name> --permissions rl --start '<start-date>' --expiry '<expiry-date>'
   ```

3. Redigera filen för att ange följande parameter värden.  Datum ska anges i UTC-format, till exempel `2016-10-25T00:00:00Z`.
   - `<account-name>`– Namnet på ditt Azure Storage-konto
   - `<account-key>`– Din lagrings konto nyckel för Azure
   - `<vhd-name>`– Ditt VHD-namn
   - `<start-date>`– Behörighetens start datum för VHD-åtkomst. Ange ett datum en dag före det aktuella datumet.
   - `<expiry-date>`-Behörighetens förfallo datum för VHD-åtkomst.  Ange ett datum minst tre veckor bortom det aktuella datumet.

   I följande exempel visas lämpliga parameter värden (vid tidpunkten för skrivning).

   ```azurecli-interactive
   az storage container generate-sas --connection-string 'DefaultEndpointsProtocol=https;AccountName=st00009;AccountKey=6L7OWFrlabs7Jn23OaR3rvY5RykpLCNHJhxsbn9ONc+bkCq9z/VNUPNYZRKoEV1FXSrvhqq3aMIDI7N3bSSvPg==;EndpointSuffix=core.windows.net' --name vhds --permissions rl --start '2017-11-06T00:00:00Z' --expiry '2018-08-20T00:00:00Z'
   ```

4. Spara ändringarna i det här PowerShell-skriptet.
5. Kör det här skriptet med administratörs behörighet för att skapa en *SAS-anslutningssträng* för åtkomst till behållar nivån.  Du kan använda två grundläggande metoder:
   - Kör skriptet från-konsolen.  Skriv till exempel i Windows och klicka på skriptet och välj **Kör som administratör**.
   - Kör skriptet från en PowerShell-skriptfil, till exempel [Windows PowerShell ISE](https://docs.microsoft.com/powershell/scripting/components/ise/introducing-the-windows-powershell-ise), med administratörs behörighet.
     Följande visar en SAS-anslutningssträng som skapas i den här redigeraren.

     ![SAS URI-generering i PowerShell ISE](./media/publishvm_032.png)

6. Kopiera den resulterande SAS-anslutningssträngen och spara den i en textfil på en säker plats.  Du redigerar den här strängen för att lägga till associerad information om VHD-platsen i den för att skapa den sista SAS-URI: n.
7. I Azure Portal navigerar du till blob-lagringen som innehåller den virtuella hård disk som är kopplad till den nyligen genererade URI: n.
8. Kopiera URL-värdet för **BLOB service slut punkten**som visas nedan.

    ![Blob Service slut punkt i Azure Portal](./media/publishvm_033.png)

9. Redigera text filen med SAS-anslutningssträngen från steg 6.  Du kommer att skapa hela SAS-URI: n med följande format:

    `<blob-service-endpoint-url>` + `/vhds/` + `<vhd-name>?` + `<sas-connection-string>`

    Om namnet på VDH till exempel är `TestRGVM2.vhd`, blir den RESULTERAnde SAS-URI: n:

    `https://catech123.blob.core.windows.net/vhds/TestRGVM2.vhd?st=2018-05-06T07%3A00%3A00Z&se=2019-08-02T07%3A00%3A00Z&sp=rl&sv=2017-04-17&sr=c&sig=wnEw9RfVKeSmVgqDfsDvC9IHhis4x0fc9Hu%2FW4yvBxk%3D`

Upprepa dessa steg för varje virtuell hård disk i de SKU: er som du planerar att publicera.

### <a name="microsoft-storage-explorer"></a>Microsoft Storage Explorer

Använd följande steg för att skapa en SAS-URI med Microsoft Azure Storage Explorer.

1. Hämta och installera [Microsoft Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/).
2. Öppna Utforskaren och klicka på ikonen **Lägg till konto** i den vänstra meny raden.  Dialog rutan **Anslut till Azure Storage** visas.
3. Välj **Lägg till ett Azure-konto** och klicka på **Logga in...**.  Fortsätt med de nödvändiga stegen för att logga in på ditt Azure-konto.
4. I **fönstret till** vänster går du till dina **lagrings konton** och expanderar den här noden.
5. Högerklicka på din virtuella hård disk och välj **Hämta signatur för resurs åtkomst** på snabb menyn.

    ![Hämta SAS-objekt i Azure Explorer](./media/publishvm_034.png)

6. Dialog rutan **signatur för delad åtkomst** visas. Ange värden för följande fält:
   - **Start tid** – behörighet start datum för VHD-åtkomst. Ange ett datum som infaller en dag före det aktuella datumet.
   - Förfallo **tid** -behörighetens förfallo datum för VHD-åtkomst.  Ange ett datum minst tre veckor bortom det aktuella datumet.
   - **Behörigheter** – Välj `Read` behörigheterna `List` och.

     ![Dialog rutan SAS i Azure Explorer](./media/publishvm_035.png)

7. Klicka på **skapa** för att skapa tillhör ande SAS-URI för den här virtuella hård disken.  Dialog rutan visar nu information om den här åtgärden.
8. Kopiera **URL** -värdet och spara det i en textfil på en säker plats.

    ![SAS-URI-skapande i Azure Explorer](./media/publishvm_036.png)

    Den här skapade SAS-URL: en är för åtkomst på container nivå.  För att göra den unik måste det associerade VHD-namnet läggas till i den.

9. Redigera text filen. Infoga VHD-namnet efter `vhds` STRÄNGEN i SAS-URL: en (inklusive inledande snedstreck).  Den sista SAS-URI: n ska ha formatet:

    `<blob-service-endpoint-url>` + `/vhds/` + `<vhd-name>?` + `<sas-connection-string>`

    Om namnet på VDH till exempel är `TestRGVM2.vhd`, blir den RESULTERAnde SAS-URI: n:

    `https://catech123.blob.core.windows.net/vhds/TestRGVM2.vhd?st=2018-05-06T07%3A00%3A00Z&se=2019-08-02T07%3A00%3A00Z&sp=rl&sv=2017-04-17&sr=c&sig=wnEw9RfVKeSmVgqDfsDvC9IHhis4x0fc9Hu%2FW4yvBxk%3D`

Upprepa dessa steg för varje virtuell hård disk i de SKU: er som du planerar att publicera.

## <a name="verify-the-sas-uri"></a>Verifiera SAS-URI: n

Granska och verifiera varje genererad SAS-URI med hjälp av följande check lista.  Kontrol lera att:

- URI: n har formatet: `<blob-service-endpoint-url>` + `/vhds/` + `<vhd-name>?` +`<sas-connection-string>`
- URI: n innehåller fil namnet för VHD-avbildningen, inklusive fil namns tillägget. VHD.
- I mitten av URI: `sp=rl` n visas. Den här strängen anger `Read` att `List` och åtkomst har angetts.
- Efter den punkten visas `sr=c` också. Den här strängen anger att åtkomst på behållare nivå har angetts.
- Kopiera och klistra in URI: n i en webbläsare för att börja ladda ned den associerade blobben.  (Du kan avbryta åtgärden innan nedladdningen är klar.)

## <a name="next-steps"></a>Nästa steg

Om du har problem med att skapa en SAS-URI kan du läsa [vanliga SAS URL-problem](./cpp-common-sas-url-issues.md).  Annars sparar du SAS-URI: erna på en säker plats för senare användning. Du måste [publicera VM-erbjudandet](./cpp-publish-offer.md) i Cloud Partner Portal.
