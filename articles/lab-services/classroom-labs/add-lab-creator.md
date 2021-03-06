---
title: Lägg till en användare som en labb skapare i Azure Lab Services
description: Den här artikeln visar hur du lägger till en användare i labb skaparen-rollen för ett labb konto i Azure Lab Services. Labb skapare kan skapa labb i det här labb kontot.
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/14/2020
ms.author: spelluru
ms.openlocfilehash: 0538747ec639b3fab1a7b38193796d80a7736170
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77444776"
---
# <a name="add-lab-creators-to-a-lab-account-in-azure-lab-services"></a>Lägga till labb skapare i ett labb konto i Azure Lab Services
Den här artikeln visar hur du lägger till användare som labbs kapare i ett labb konto i Azure Lab Services. Dessa används sedan för att skapa klass rum labb i labb kontot. 

## <a name="add-microsoft-user-account-to-lab-creator-role"></a>Lägg till Microsoft-användarkonto till labb skapare rollen
Om du vill konfigurera ett klassrumslabb i ett labbkonto måste användaren vara medlem i rollen **Lab Creator** i labbkontot. Det konto som du använde för att skapa labbkontot läggs automatiskt till i den här rollen. Om du planerar att använda samma användarkonto för att skapa ett klassrumslabb kan du hoppa över det här steget. Om du vill använda ett annat konto för att skapa ett klassrumslabb gör du följande: 

Ge lärarna behörighet att skapa labb åt sina klasser genom att lägga till dem i rollen **Labbskapare**:

1. På sidan **Labbkonto** väljer du **Åtkomstkontroll (IAM)** och klickar på **+ Lägg till rolltilldelning** i verktygsfältet. 

    ![Åtkomstkontroll -> Lägg till rolltilldelning, knapp](../media/tutorial-setup-lab-account/add-role-assignment-button.png)
1. På sidan **Lägg till rolltilldelning** väljer du **Labbskapare** som **Roll**. Välj den användare som du vill lägga till rollen Labbskapare och välj **Spara**. 

    ![Lägg till labbuppgiftsförfattare](../media/tutorial-setup-lab-account/add-lab-creator.png)

    > [!NOTE]
    > Om du lägger till en icke-Microsoft-konto användare som en labb skapare, se avsnittet [Lägg till en icke-Microsoft-konto användare som en labb skapare](#add-a-non-microsoft-account-user-as-a-lab-creator) . 

## <a name="add-a-non-microsoft-account-user-as-a-lab-creator"></a>Lägg till en icke-Microsoft-konto användare som en labb skapare
Om du vill lägga till en användare som en labb skapare, använder du deras e-postkonton. Följande typer av e-postkonton kan användas:

- Ett e-postkonto som tillhandahålls av ditt universitets Office 365 Azure Active Directory (AAD). 
- Ett Microsoft-e-postkonto, `@outlook.com`till `@hotmail.com`exempel `@msn.com`,, `@live.com`eller.
- Ett e-postkonto som inte kommer från Microsoft, till exempel ett som tillhandahålls av Yahoo eller Google. Dessa typer av konton måste dock länkas till ett Microsoft-konto.
- Ett GitHub-konto. Kontot måste vara länkat till ett Microsoft-konto.

### <a name="using-a-non-microsoft-email-account"></a>Använda ett e-postkonto som inte kommer från Microsoft
Labb skapare/lärare kan använda e-postkonton som inte kommer från Microsoft för att registrera sig och logga in i ett klass rums labb.  Inloggningen till labb tjänst portalen kräver dock att instruktörerna först skapar en Microsoft-konto som är länkad till den e-postadress som inte kommer från Microsoft.

Många instruktörer kanske redan har ett Microsoft-konto kopplat till sina e-postadresser som inte kommer från Microsoft. Lärare har till exempel redan en Microsoft-konto om de har använt sin e-postadress med Microsofts andra produkter eller tjänster, t. ex. Office, Skype, OneDrive eller Windows.  

När lärare loggar in på labb tjänst portalen uppmanas de att ange sina e-postadresser och lösen ord. Om instruktören försöker logga in med en icke-Microsoft-konto som inte har något Microsoft-konto länkat, får instruktören följande fel meddelande: 

![Felmeddelande](../media/how-to-configure-student-usage/cant-find-account.png)

Om du vill registrera dig för en Microsoft-konto bör instruktörer gå [http://signup.live.com](http://signup.live.com)till.  


### <a name="using-a-github-account"></a>Använda ett GitHub-konto
Lärare kan också använda ett befintligt GitHub-konto för att registrera sig och logga in i ett klass rums labb. Om instruktören redan har en Microsoft-konto länkat till sitt GitHub-konto kan de logga in och ange sitt lösen ord som visas i föregående avsnitt. Om de inte har länkat sitt GitHub-konto till en Microsoft-konto bör de välja **inloggnings alternativ**:

![Länk för inloggnings alternativ](../media/how-to-configure-student-usage/signin-options.png)

På sidan **inloggnings alternativ** väljer du logga in **med GitHub**.

![Logga in med GitHub-länk](../media/how-to-configure-student-usage/signin-github.png)

Slutligen uppmanas de att skapa en Microsoft-konto som är länkad till sitt GitHub-konto. Det sker automatiskt när instruktören väljer **Nästa**.  Instruktören är sedan omedelbart inloggad och ansluten till klass rummets labb.


## <a name="next-steps"></a>Nästa steg
Se följande artiklar:

- [Skapa och hantera labb som labbägare](how-to-manage-classroom-labs.md)
- [Konfigurera och publicera mallar som labbägare](how-to-create-manage-template.md)
- [Konfigurera och kontroller användning av ett labb som labbägare](how-to-configure-student-usage.md)
- [Som labb användare, åtkomst till klass rum labb](how-to-use-classroom-lab.md)
