---
title: Så här skapar du en support förfrågan för Azure | Microsoft Docs
description: Kunder som behöver hjälp kan använda Azure Portal för att hitta självbetjänings lösningar och för att skapa och hantera support förfrågningar.
services: Azure Supportability
author: ganganarayanan
manager: scotthit
ms.assetid: fd6841ea-c1d5-4bb7-86bd-0c708d193b89
ms.service: azure-supportability
ms.topic: article
ms.date: 03/31/2020
ms.author: kfollis
ms.openlocfilehash: 2b68b6ddf19d6b07475e7009b47e162bfb2d0d2f
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80478986"
---
# <a name="how-to-create-an-azure-support-request"></a>Skapa en supportbegäran för Azure

## <a name="overview"></a>Översikt

Med Azure kan du skapa och hantera support förfrågningar, även kallade support biljetter. Du kan skapa och hantera begär anden i [Azure Portal](https://portal.azure.com), som beskrivs i den här artikeln. Du kan också skapa och hantera förfrågningar program mässigt med hjälp av [Azures support biljett REST API](/rest/api/support).

> [!NOTE]
> Azure Portal-URL: en är unik för det Azure-moln där din organisation har distribuerats.
>
>* Azure Portal för kommersiellt bruk är:[https://portal.azure.com](https://portal.azure.com)
>* Azure Portal för Tyskland:[https://portal.microsoftazure.de](https://portal.microsoftazure.de)
>* Azure Portal för USA myndigheter är:[https://portal.azure.us](https://portal.azure.us)
>
>

Baserat på kundfeedback har vi uppdaterat support ärende upplevelsen för att fokusera på tre huvudsakliga mål:

* **Effektiviserat**: gör support och fel sökning lätt att hitta och förenkla hur du skickar en supportbegäran.
* **Integrerad**: du kan enkelt öppna en support förfrågan när du felsöker ett problem med en Azure-resurs, utan att växla sammanhang.
* **Effektiv**: samla in viktig information som support agenten behöver för att effektivt lösa problemet.

## <a name="getting-started"></a>Komma igång

Du kan få **Hjälp + Support** i Azure Portal. Den finns på Azure Portal-menyn, i den globala rubriken eller på resurs-menyn för en tjänst. Innan du kan skicka en förfrågan till en supportbegäran måste du ha rätt behörighet.

### <a name="role-based-access-control"></a>Rollbaserad åtkomstkontroll

Om du vill skapa en supportbegäran måste du vara [ägare](../../role-based-access-control/built-in-roles.md#owner), [deltagare](../../role-based-access-control/built-in-roles.md#contributor) eller tilldelad rollen [Support Request Contributor](../../role-based-access-control/built-in-roles.md#support-request-contributor) på prenumerations nivå. Om du vill skapa en supportbegäran utan en prenumeration, till exempel ett Azure Active Directory (AAD)-scenario, måste du vara [administratör](../../active-directory/users-groups-roles/directory-assign-admin-roles.md).

### <a name="go-to-help--support-from-the-global-header"></a>Gå till hjälp + support från den globala rubriken

Så här startar du en supportbegäran var som helst i Azure Portal:

1. Välj **?** i den globala rubriken. Välj sedan **Hjälp + Support**.

   ![Hjälp och support](./media/how-to-create-azure-support-request/helpandsupportnewlower.png)

2. Välj **ny supportbegäran**. Följ anvisningarna för att ge oss information om ditt problem. Vi föreslår några möjliga lösningar, samlar in information om problemet och hjälper dig att skicka och spåra support förfrågan.

   ![Ny supportbegäran](./media/how-to-create-azure-support-request/newsupportrequest2lower.png)

### <a name="go-to-help--support-from-a-resource-menu"></a>Gå till hjälp + support från en resurs meny

För att starta en support förfrågan i kontexten för resursen arbetar du för närvarande med:

1. På resurs-menyn i avsnittet **support och fel sökning** väljer du **ny supportbegäran**.

   ![I sammanhang](./media/how-to-create-azure-support-request/incontext2lower.png)

2. Följ anvisningarna för att ge oss information om problemet. När du startar support förfrågnings processen från resursen väljs vissa alternativ i förväg.

## <a name="create-a-support-request"></a>Skapa en supportbegäran

Vi vägleder dig genom några steg för att samla in information om ditt problem och hjälpa dig att lösa det. Varje steg beskrivs i följande avsnitt.

### <a name="basics"></a>Grundläggande inställningar

Det första steget i support förfrågnings processen samlar in grundläggande information om ditt problem och ditt support avtal.

På fliken **grundläggande** i den **nya support förfrågan**använder du selectrs för att tala om för oss om problemet. Först ska du identifiera några allmänna kategorier för ärende typen och välja den relaterade prenumerationen. Välj tjänsten, till exempel **virtuell dator som kör Windows**. Välj resursen, till exempel namnet på den virtuella datorn. Beskriv problemet med dina egna ord och välj sedan **problem typ** för att få mer information.

![Bladet Grundläggande inställningar](./media/how-to-create-azure-support-request/basics2lower.png)

> [!NOTE]
> Azure ger obegränsad support för prenumerations hantering, bland annat fakturering, kvot justeringar och konto överföringar. För teknisk support behöver du en Support plan. [Läs mer om support planer](https://azure.microsoft.com/support/plans).
>
>

### <a name="solutions"></a>Lösningar

När du har samlat in grundläggande information visar vi lösningar som du kan prova på egen hand. I vissa fall kan vi till och med köra en snabb diagnostik. Lösningar skrivs av Azure-tekniker och kommer att lösa de vanligaste problemen.

### <a name="details"></a>Information

Sedan samlar vi in ytterligare information om problemet. Att tillhandahålla grundlig och detaljerad information i det här steget hjälper oss att skicka support förfrågan till rätt agent.

Om möjligt ska du meddela oss när problemet har inletts och hur du kan återskapa det. Du kan ladda upp en fil, till exempel en loggfil eller utdata från diagnostiken.

När vi har all information om problemet väljer du hur du får support. I avsnittet **support metod** i **detalj**väljer du allvarlighets graden för påverkan. Ange önskad kontakt metod, en bra tid att kontakta dig och ditt support språk.

Fyll sedan i avsnittet **kontakt information** så att vi kan kontakta dig.

### <a name="review--create"></a>Granska + skapa

Slutför all nödvändig information på varje flik och välj sedan **Granska + skapa**. Kontrol lera den information som du skickar till supporten. Gå tillbaka till en flik och gör en ändring om det behövs. När du är nöjd med att support förfrågan är klar väljer du **skapa**.

En support agent kontaktar dig genom att använda den metod du angav. Information om första svars tid finns i [support omfattning och svars tider](https://azure.microsoft.com/support/plans/response/).

## <a name="all-support-requests"></a>Alla support förfrågningar

Du kan visa information och status för support förfrågningar genom att gå till **Hjälp + Support** >  förfrågningar**alla support förfrågningar**.

![Alla support förfrågningar](./media/how-to-create-azure-support-request/allrequestslower.png)

På den här sidan kan du filtrera support förfrågningar efter **prenumeration**, **skapat** datum (UTC) och **status**. Dessutom kan du sortera och söka efter support förfrågningar på den här sidan.

Välj en supportbegäran om du vill visa information, inklusive allvarlighets grad och den förväntade tiden det tar för en support agent att svara.

Om du vill ändra allvarlighets graden för begäran väljer du **affärs påverkan**. Välj från en lista över allvarlighets grader som ska tilldelas.

> [!NOTE]
> Den högsta allvarlighets graden beror på support avtalet. [Läs mer om support planer](https://azure.microsoft.com/support/plans).
>
>
Titta på den här videon om du vill veta mer om support alternativ för självhjälp i Azure:

> [!VIDEO https://www.youtube.com/embed/gNhzR5FE9DY]

## <a name="next-steps"></a>Nästa steg

* [Skicka feedback och förslag till oss](https://feedback.azure.com/forums/266794-support-feedback)
* Engagera oss på [Twitter](https://twitter.com/azuresupport)
* Få hjälp från dina kollegor i MSDN- [forumen](https://social.msdn.microsoft.com/Forums/azure)
* Läs mer i [vanliga frågor och svar om support för Azure](https://azure.microsoft.com/support/faq)
