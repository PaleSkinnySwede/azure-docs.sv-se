---
title: Aktivera Azures resurs roller i PIM – Azure AD | Microsoft Docs
description: Lär dig hur du aktiverar Azures resurs roller i Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: d35c81f7bb478d91bd207327ea37c80aa1778142
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74023153"
---
# <a name="activate-my-azure-resource-roles-in-privileged-identity-management"></a>Aktivera mina Azure-resurs roller i Privileged Identity Management

Använd Privileged Identity Management (PIM) för att tillåta att kvalificerade roll medlemmar för Azure-resurser schemalägger aktivering för framtida datum och tid. De kan också välja en speciell aktiverings tid inom den högsta (som kon figurer ATS av administratörer).

Den här artikeln är för medlemmar som behöver aktivera Azures resurs roll i Privileged Identity Management.

## <a name="activate-a-role"></a>Aktivera en roll

När du behöver utföra en Azure-resurs roll kan du begära aktivering med hjälp av navigerings alternativet **Mina roller** i Privileged Identity Management.

1. Logga in på [Azure-portalen](https://portal.azure.com/).

1. Öppna **Azure AD Privileged Identity Management**. Information om hur du lägger till panelen Privileged Identity Management på instrument panelen finns i [börja använda Privileged Identity Management](pim-getting-started.md).

1. Välj **Mina roller**.

    ![Sidan mina roller med roller som du kan aktivera](./media/pim-resource-roles-activate-your-roles/resources-my-roles.png)

1. Välj **Azure-resurs roller** om du vill se en lista över dina kvalificerade Azure-resurs roller.

   ![Mina roller – sidan Azure-resurs roller](./media/pim-resource-roles-activate-your-roles/resources-my-roles-azure-resources.png) 

1. Leta upp den roll som du vill aktivera i listan **Azure resurs roller** .

    ![Azure resurs roller – listan Mina berättigade roller](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate.png)

1. Välj **Aktivera** för att öppna fönstret aktivera.

1. Om din roll kräver Multi-Factor Authentication väljer du **Verifiera din identitet innan du fortsätter**. Du behöver bara autentisera en gång per session.

    ![Verifiera min identitet med MFA innan du aktiverar rollen](./media/pim-resource-roles-activate-your-roles/resources-my-roles-mfa.png)

1. Välj **verifiera min identitet** och följ instruktionerna för att tillhandahålla ytterligare säkerhets verifiering.

    ![Skärm för att tillhandahålla säkerhets verifiering, till exempel en PIN-kod](./media/pim-resource-roles-activate-your-roles/resources-mfa-enter-code.png)

1. Om du vill ange ett reducerat omfång väljer du **omfång** för att öppna fönstret resurs filter.

    Det är en bra idé att bara begära åtkomst till de resurser du behöver. I fönstret resurs filter kan du ange de resurs grupper eller resurser som du behöver åtkomst till.

    ![Aktivera fönstret resurs filter för att ange omfång](./media/pim-resource-roles-activate-your-roles/resources-my-roles-resource-filter.png)

1. Vid behov kan du ange en start tid för anpassad aktivering. Medlemmen aktive ras efter den valda tiden.

1. Ange orsaken till aktiverings förfrågan i rutan **orsak** .

    ![Slutfört aktiverings fönster med omfång, start tid, varaktighet och orsak](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-done.png)

1. Välj **Aktivera**.

    Om rollen inte kräver godkännande aktive ras den och läggs till i listan över aktiva roller. Följ stegen i nästa avsnitt om du vill använda rollen.

    Om [rollen kräver godkännande](pim-resource-roles-approval-workflow.md) för att aktive ras visas ett meddelande i det övre högra hörnet i webbläsaren som informerar dig om att begäran väntar på godkännande.

    ![Begäran om aktivering väntar på godkännande](./media/pim-resource-roles-activate-your-roles/resources-my-roles-activate-notification.png)

## <a name="use-a-role-immediately-after-activation"></a>Använd en roll omedelbart efter aktiveringen

Om du har aktiverat en fördröjning efter aktiveringen följer du de här stegen när du har aktiverat för att använda Azures resurs roller direkt.

1. Öppna Azure AD Privileged Identity Management.

1. Välj **Mina roller** om du vill se en lista över dina kvalificerade Azure AD-roller och Azure-resurs roller.

1. Välj **Azure-resurs roller**.

1. Välj fliken **aktiva roller** .

1. När rollen är aktiv loggar du ut från portalen och loggar in igen.

    Rollen bör nu vara tillgänglig för användning.

## <a name="view-the-status-of-your-requests"></a>Visa status för dina begär Anden

Du kan visa statusen för väntande begär Anden att aktivera.

1. Öppna Azure AD Privileged Identity Management.

1. Välj **Mina begär Anden** om du vill se en lista över dina Azure AD-roller och Azure-resurs roll förfrågningar.

    ![Mina begär Anden – Azure-resurs sidan visar dina väntande förfrågningar](./media/pim-resource-roles-activate-your-roles/resources-my-requests.png)

1. Bläddra till höger för att visa kolumnen **status för begäran** .

## <a name="cancel-a-pending-request"></a>Avbryt en väntande begäran

Om du inte behöver aktivera en roll som kräver godkännande kan du när som helst avbryta en väntande begäran.

1. Öppna Azure AD Privileged Identity Management.

1. Välj **Mina begär Anden**.

1. För den roll som du vill avbryta väljer du länken **Avbryt** .

    När du väljer Avbryt avbryts begäran. Om du vill aktivera rollen igen måste du skicka in en ny begäran om aktivering.

   ![Listan Mina begär Anden med avbrotts åtgärd markerad](./media/pim-resource-roles-activate-your-roles/resources-my-requests-cancel.png)

## <a name="troubleshoot"></a>Felsöka

### <a name="permissions-are-not-granted-after-activating-a-role"></a>Behörigheter beviljas inte efter att en roll har aktiverats

När du aktiverar en roll i Privileged Identity Management kan aktiveringen inte genast spridas till alla portaler som kräver den privilegierade rollen. Ibland kan webbcachelagringen i en portal resultera i att ändringen inte börjar gälla omedelbart, även om ändringen har spridits. Om aktiveringen fördröjs, är det vad du ska göra.

1. Logga ut från Azure-portalen och logga sedan in igen.

    När du aktiverar en Azure-resurs roll visas stegen i aktiveringen. När alla steg har slutförts visas länken **Logga ut**. Du kan använda den här länken för att logga ut. Detta kommer att lösa de flesta fall för aktiverings fördröjning.

1. I Privileged Identity Management kontrollerar du att du är listad som medlem i rollen.

## <a name="next-steps"></a>Nästa steg

- [Utöka eller förnya Azures resurs roller i Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Aktivera mina Azure AD-roller i Privileged Identity Management](pim-how-to-activate-role.md)
