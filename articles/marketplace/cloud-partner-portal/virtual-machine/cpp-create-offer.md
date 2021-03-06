---
title: Skapa erbjudande för virtuella datorer på Azure Marketplace
description: Visar de steg som krävs för att skapa ett nytt erbjudande för virtuell dator (VM) för Azure Marketplace.
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 10/19/2018
ms.author: dsindona
ms.openlocfilehash: a25f6877f1fb4940fb1de127b81d83975c8e835c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82142712"
---
# <a name="create-virtual-machine-offer"></a>Skapa erbjudande för virtuell dator

> [!IMPORTANT]
> Med början den 13 april 2020 kommer vi att börja flytta hanteringen av dina virtuella Azure-datorer till Partner Center. Efter migreringen kommer du att skapa och hantera dina erbjudanden i Partner Center. Följ instruktionerna i [skapa ett erbjudande för virtuell Azure-dator](https://docs.microsoft.com/azure/marketplace/partner-center-portal/azure-vm-create-offer) för att hantera dina migrerade erbjudanden.

Det här avsnittet innehåller de steg som krävs för att skapa en ny virtuell dator (VM)-erbjudande förfrågan för Azure Marketplace.  Varje erbjudande visas som sin egen enhet i Azure Marketplace och är associerad med en eller flera SKU: er.  Ett virtuellt dator erbjudande består av följande grupperingar av till gångar och stöd tjänster: 

![Till gångar för ett virtuellt dator erbjudande](./media/publishvm_002.png)

där:

|  **Till gångs grupp**   |  **Beskrivning**  |
|  ---------------   |  ---------------  |
|    SKU: er            |  Den minsta köpbara enheten för ett erbjudande. Ett enda erbjudande (produkt klass) kan ha flera SKU: er kopplade till sig för att skilja mellan vilka funktioner som stöds, avbildnings typer för virtuella datorer och fakturerings modeller. |
|  Marketplace       | Innehåller till gångar och specifikationer för marknadsföring, juridiska och ledande hantering.  <ul><li> Marknadsförings till gångar innehåller namn, beskrivning och logo typer för erbjudandet</li> <li> Juridiska till gångar innehåller en sekretess policy, användnings villkor och annan juridisk dokumentation</li>  <li> Med princip för hantering av lead kan du ange hur du vill hantera leads från slut användar portalen för Azure Marketplace.</li> </ul> |
| Support            | Innehåller support kontakt och princip information |
| Test Drive         | Definierar till gångar som gör det möjligt för slutanvändare att testa ditt erbjudande innan de köper det |
|  |  |


## <a name="new-offer-form"></a>Nytt erbjudande formulär

När du har loggat in på [Cloud Partner Portal](https://cloudpartner.azure.com/)klickar du på **+ nytt erbjudande** objekt på den vänstra meny raden. I den resulterande menyn klickar du på **Virtual Machines** för att visa det **nya erbjudande** formuläret och starta processen för att definiera till gångar för ett nytt erbjudande för virtuella datorer. 
<!-- not all publishers see corevm or azure apps test, you need to be whitelisted to see them. we should hide those in these images. -->

![Nytt val av användar gränssnitt för virtuell dator](./media/publishvm_003.png)

> [!WARNING]
> Om alternativet **Virtual Machines** inte visas eller inte är aktiverat har ditt konto inte behörighet att skapa den här erbjudande typen.  Kontrol lera att du har uppfyllt alla [krav](./cpp-prerequisites.md) för den här erbjudande typen, inklusive registrering för ett utvecklar konto.


## <a name="next-steps"></a>Nästa steg

Följande avsnitt i det här avsnittet speglar flikarna på sidan **ny erbjudande** (för en typ av virtuell dator erbjudande).  I varje artikel förklaras hur du använder den associerade fliken för att definiera till gångs grupper och stöd tjänster för det nya VM-erbjudandet.

- [Fliken Erbjudandeinställningar](./cpp-offer-settings-tab.md)
- [Fliken SKU:er](./cpp-skus-tab.md)
- [Fliken Test Drive](./cpp-test-drive-tab.md)
- [Fliken Marketplace](./cpp-marketplace-tab.md)
- [Fliken Stöd](./cpp-support-tab.md)
