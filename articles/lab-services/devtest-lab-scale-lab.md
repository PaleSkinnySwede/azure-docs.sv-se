---
title: Skala kvoter och begränsningar i din labb Azure DevTest Labs | Microsoft Docs
description: I den här artikeln beskrivs hur du kan skala ditt labb i Azure DevTest Labs. Visa användnings kvoter och-gränser och begär en ökning.
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.assetid: ae624155-9181-45fa-bd2b-1983339b7e0e
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2020
ms.author: spelluru
ms.openlocfilehash: f3299c24bc751263ccd284a4fddb492fba5dd1e0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "76761193"
---
# <a name="scale-quotas-and-limits-in-devtest-labs"></a>Skalnings kvoter och begränsningar i DevTest Labs
När du arbetar i DevTest Labs kanske du märker att det finns vissa standard gränser för vissa Azure-resurser, vilket kan påverka DevTest Labs-tjänsten. Dessa gränser kallas **kvoter**.

> [!NOTE]
> DevTest Labs-tjänsten tillämpar inga kvoter. Alla kvoter som du kan stöta på är standard begränsningar för den övergripande Azure-prenumerationen.

Du kan använda varje Azure-resurs tills du når kvoten. Varje prenumeration har separata kvoter och användningen spåras per prenumeration.

Varje prenumeration har till exempel en standard kvot på 20 kärnor. Så om du skapar virtuella datorer i labbet med fyra kärnor var och en, kan du bara skapa fem virtuella datorer.

[Azure-prenumerations-och tjänst begränsningar](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits) listar några av de vanligaste kvoterna för Azure-resurser. De resurser som används oftast i ett labb, och där du kan stöta på kvoter, inkludera VM-kärnor, offentliga IP-adresser, nätverks gränssnitt, hanterade diskar, RBAC-roll tilldelning och ExpressRoute-kretsar.

## <a name="view-your-usage-and-quotas"></a>Visa användning och kvoter
De här stegen visar hur du visar de aktuella kvoterna i din prenumeration för vissa Azure-resurser och hur många procent av varje kvot du har använt.

1. Logga in på [Azure-portalen](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Välj **fler tjänster**och välj sedan **fakturering** i listan.
1. Välj en prenumeration på bladet fakturering.
4. Välj **användning + kvoter**.

   ![Knappen användning och kvoter](./media/devtest-lab-scale-lab/devtestlab-usage-and-quotas.png)

   Bladet användning och kvoter visas och visar olika tillgängliga resurser i den prenumerationen och den procent andel av kvoten som används per resurs.

   ![Kvoter och användning](./media/devtest-lab-scale-lab/devtestlab-view-quotas.png)

## <a name="requesting-more-resources-in-your-subscription"></a>Begära fler resurser i din prenumeration
Om du når en kvot gräns kan standard gränsen för en resurs i en prenumeration ökas till en maxgräns, enligt beskrivningen i [Azure-prenumeration och tjänst begränsningar](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits).

De här stegen visar hur du begär en kvot ökning via [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Välj **fler tjänster**, Välj **fakturering**och välj sedan **användning + kvoter**.
1. På bladet användning och kvoter väljer du knappen för att **öka begäran** .

   ![Begär öknings knapp](./media/devtest-lab-scale-lab/devtestlab-request-increase.png)

1. Slutför och skicka begäran genom att fylla i den information som krävs på alla tre flikarna i formuläret för den **nya support förfrågningen** .

   ![Begär öknings formulär](./media/devtest-lab-scale-lab/devtestlab-support-form.png)

Att [förstå Azure-gränser och ökningar](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/) ger mer information om hur du kontaktar Azure-supporten för att begära en kvot ökning.



[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

### <a name="next-steps"></a>Nästa steg
* Utforska [DevTest Labs Azure Resource Manager snabb starts galleriet](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
