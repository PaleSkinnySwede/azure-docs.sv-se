---
title: Ändra storlek på en virtuell dator i ett labb i Azure DevTest Labs
description: Lär dig hur du ändrar storleken på en virtuell dator (VM) i Azure DevTest Labs baserat på dina behov av processor, nätverk eller disk prestanda.
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.assetid: 8460f09e-482f-48ba-a57a-c95fe8afa001
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2020
ms.author: spelluru
ms.openlocfilehash: bf7c425766a97aaa1d143133f04502a0aa3c36cb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "76756185"
---
# <a name="resize-a-vm-in-a-lab-in-azure-devtest-labs"></a>Ändra storlek på en virtuell dator i ett labb i Azure DevTest Labs
En av de viktigaste funktionerna i Azure Virtual Machines är att du kan ändra storleken på en virtuell dator (VM) utifrån dina behov av processor-, nätverks-eller disk prestanda. Azure DevTest Labs stöder den här funktionen för virtuella datorer i ett labb nu. Funktionen för storleks ändring följer Lab-principen för tillåtna VM-storlekar i labbet. Det innebär att du kan ändra storleken på en virtuell dator till endast tillåtna storlekar i labbet. 


## <a name="steps-to-resize-a-vm-in-a-lab"></a>Steg för att ändra storlek på en virtuell dator i ett labb 
Gör så här om du vill ändra storlek på en virtuell dator i ett labb i Azure DevTest Labs: 

> [!NOTE]
> Om du är ansluten till den virtuella datorn via en fjärrskrivbordssession, sparar du ditt arbete och kopplar från den virtuella datorn innan du ändrar storlek på den.

1. Logga in på [Azure-portalen](https://portal.azure.com).
2. Välj **alla tjänster**och välj sedan **DevTest Labs** i listan.
3. I listan med labb väljer du labbet som innehåller den virtuella dator som du vill ändra storlek på.  
4. I den vänstra panelen väljer du **mina Virtual Machines**. 
5. Välj en virtuell dator i listan över virtuella datorer.
6. Välj **stoppa** i verktygsfältet om den virtuella datorn körs. Kontrol lera status för åtgärden i fönstret **meddelanden** . Vänta tills den virtuella datorn har stoppats och stäng sedan fönstret **meddelanden** . 

    ![Stoppa den virtuella datorn](media/devtest-lab-resize-vm/stop-vm.png)
1. På sidan virtuell dator för den virtuella datorn väljer du **storlek** under **Inställningar** på den vänstra menyn.

    ![Storlek-menyn](media/devtest-lab-resize-vm/size-menu.png)
1. I fönstret **Välj en storlek** bläddrar du till och väljer en storlek för den virtuella datorn och klickar på **Välj**.     
1. Kontrol lera status för åtgärden ändra storlek i fönstret **meddelanden** .

    ![Ändra storlek på status](media/devtest-lab-resize-vm/resize-status.png)
10. När åtgärden ändra storlek har slutförts stänger du fönstret **meddelanden** . 
11. Välj **Översikt** i den vänstra menyn och välj **Starta** om den virtuella datorn i verktygsfältet. 

## <a name="next-steps"></a>Nästa steg
Detaljerad information om funktionen för storleks ändring som stöds av virtuella Azure-datorer finns i [ändra storlek på virtuella datorer](https://azure.microsoft.com/blog/resize-virtual-machines/).


