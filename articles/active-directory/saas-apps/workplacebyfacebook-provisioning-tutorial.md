---
title: 'Självstudie: Konfigurera arbets ytan efter Facebook för automatisk användar etablering med Azure Active Directory | Microsoft Docs'
description: Lär dig hur du konfigurerar enkel inloggning mellan Azure Active Directory och Workplace by Facebook.
services: active-directory
documentationCenter: na
author: jeevansd
manager: daveba
ms.assetid: 6341e67e-8ce6-42dc-a4ea-7295904a53ef
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/28/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 99103c9994b240e2f45b66acf269b320c90e5135
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82231738"
---
# <a name="tutorial-configure-workplace-by-facebook-for-automatic-user-provisioning"></a>Självstudie: Konfigurera arbets ytan efter Facebook för automatisk användar etablering

I den här självstudien beskrivs de steg du behöver utföra i båda arbets platserna av Facebook och Azure Active Directory (Azure AD) för att konfigurera automatisk användar etablering. När Azure AD konfigureras, etablerar och avetablerar Azure AD automatiskt användare och grupper i [arbets ytan efter Facebook](https://work.workplace.com/) med Azure AD Provisioning-tjänsten. Viktig information om vad den här tjänsten gör, hur det fungerar och vanliga frågor finns i [Automatisera användar etablering och avetablering för SaaS-program med Azure Active Directory](../manage-apps/user-provisioning.md).

## <a name="migrating-to-the-new-workplace-by-facebook-application"></a>Migrera till den nya arbets ytan efter Facebook-program
Om du har en befintlig integrering med arbets ytan av Facebook kan du läsa avsnittet nedan om vilka ändringar som kommer. Om du konfigurerar arbets ytan efter Facebook för första gången kan du hoppa över det här avsnittet och flytta till de funktioner som stöds. 

#### <a name="whats-changing"></a>Vad förändras?
* Ändringar på Azure AD-sidan: autentiseringsmetoden för att etablera användare i arbets platsen har tidigare varit en hemlig hemlig token. Snart kommer du att se den auktoriseringsregler som ändrades till beviljande av OAuth-auktorisering. 
* Ändringar på arbets plats sidan: tidigare var Azure AD-appen en anpassad integrering på arbets platsen av Facebook. Nu kommer du att se Azure AD i katalogen för arbets plats integrering som ett program från tredje part. 

#### <a name="what-do-i-need-to-do-to-migrate-my-existing-custom-integration-to-the-new-application"></a>Vad behöver jag för att migrera min befintliga anpassade integrering till det nya programmet?
Om du har en befintlig arbets plats integration med en giltig token krävs ingen åtgärd. **Från och med 04/28/2020 har vi automatiskt migrerat alla program som inte finns i karantän på grund av ogiltiga autentiseringsuppgifter.**
 
#### <a name="how-can-i-tell-if-my-application-has-been-migrated"></a>Hur ser jag om mitt program har migrerats? 
* I Azure Portal: när ditt program migreras tas banderollen i avsnittet Authorization om kommande ändringar bort och fältet hemligt token ersätts med en blå behörighets knapp. 
* På arbets ytan av Facebook-portalen: granska Azure AD-appen för att kontrol lera att den är godkänd.  

#### <a name="the-admin-credentials-section-is-greyed-out-on-my-application-and-i-cant-save-why"></a>Avsnittet admin credentials är nedtonat i mitt program och jag kan inte spara. Varför det?
Vi har låst avsnittet admin credentials för arbets plats kunder som inte har migrerats. Använd följande URL om avsnittet admin-autentiseringsuppgifter är nedtonat och du måste auktorisera åtkomsten igen. **? Microsoft_AAD_IAM_userProvisioningEnableCredentialsOverride = sant** (https://portal.azure.com/?Microsoft_AAD_IAM_userProvisioningEnableCredentialsOverride=true)


## <a name="capabilities-supported"></a>Funktioner som stöds
> [!div class="checklist"]
> * Skapa användare på arbets ytan efter Facebook
> * Ta bort användare i arbets ytan efter Facebook när de inte behöver åtkomst längre
> * Behåll användarattribut synkroniserade mellan Azure AD och arbets platsen på Facebook
> * [Enkel inloggning](https://docs.microsoft.com/azure/active-directory/saas-apps/workplacebyfacebook-tutorial) till arbets plats efter Facebook (rekommenderas)

## <a name="prerequisites"></a>Krav

Det scenario som beskrivs i den här självstudien förutsätter att du redan har följande krav:

* [En Azure AD-klient](https://docs.microsoft.com/azure/active-directory/develop/quickstart-create-new-tenant) 
* Ett användar konto i Azure AD med [behörighet](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-assign-admin-roles) att konfigurera etablering (t. ex. program administratör, moln program administratör, program ägare eller global administratör)
* En arbets plats per Facebook-aktiverad prenumeration med enkel inloggning

> [!NOTE]
> Vi rekommenderar att du inte använder en produktionsmiljö för att testa stegen i den här självstudien.

Du bör följa de här rekommendationerna när du testar stegen i självstudien:

- Använd inte din produktionsmiljö om det inte behövs.
- Om du inte har en utvärderings miljö för Azure AD kan du få en månads utvärderings version [här](https://azure.microsoft.com/pricing/free-trial/).

## <a name="step-1-plan-your-provisioning-deployment"></a>Steg 1. Planera etablerings distributionen
1. Läs om [hur etablerings tjänsten fungerar](https://docs.microsoft.com/azure/active-directory/manage-apps/user-provisioning).
2. Ta reda på vem som kommer att vara inom [omfånget för etablering](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts).
3. Ta reda på vilka data som ska [mappas mellan Azure AD och arbets ytan på Facebook](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes).

## <a name="step-2-configure-workplace-by-facebook-to-support-provisioning-with-azure-ad"></a>Steg 2. Konfigurera arbets ytan efter Facebook för att stödja etablering med Azure AD

Innan du konfigurerar och aktiverar etablerings tjänsten måste du bestämma vilka användare och/eller grupper i Azure AD som representerar de användare som behöver åtkomst till din arbets plats av Facebook-appen. När du har bestämt dig kan du tilldela dessa användare till din arbets plats av Facebook-appen genom att följa anvisningarna här:

*   Vi rekommenderar att en enda Azure AD-användare tilldelas arbets platsen av Facebook för att testa etablerings konfigurationen. Ytterligare användare och/eller grupper kan tilldelas senare.

*   När du tilldelar en användare till arbets ytan per Facebook måste du välja en giltig användar roll. Rollen "standard åtkomst" fungerar inte för etablering.

## <a name="step-3-add-workplace-by-facebook-from-the-azure-ad-application-gallery"></a>Steg 3. Lägg till arbets plats från Facebook från Azure AD-programgalleriet

Lägg till arbets ytan från Facebook från Azure AD-programgalleriet för att börja hantera etablering till arbets platsen av Facebook. Om du tidigare har konfigurerat arbets ytan på Facebook för SSO kan du använda samma program. Vi rekommenderar dock att du skapar en separat app när du testar integreringen från början. Lär dig mer om att lägga till ett program från galleriet [här](https://docs.microsoft.com/azure/active-directory/manage-apps/add-gallery-app).

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Steg 4. Definiera vem som ska finnas inom omfånget för etablering 

Med Azure AD Provisioning-tjänsten kan du definiera omfång som ska tillhandahållas baserat på tilldelning till programmet och eller baserat på attribut för användaren/gruppen. Om du väljer att omfånget som ska tillhandahållas till din app baserat på tilldelning kan du använda följande [steg](../manage-apps/assign-user-or-group-access-portal.md) för att tilldela användare och grupper till programmet. Om du väljer att omfånget som endast ska tillhandahållas baserat på attribut för användaren eller gruppen kan du använda ett omfångs filter enligt beskrivningen [här](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

* När du tilldelar användare och grupper till arbets ytan per Facebook måste du välja en annan roll än **standard åtkomst**. Användare med standard åtkomst rollen undantas från etablering och markeras som inte faktiskt berättigade i etablerings loggarna. Om den enda rollen som är tillgänglig i programmet är standard åtkomst rollen kan du [Uppdatera applikations manifestet](https://docs.microsoft.com/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps) för att lägga till ytterligare roller. 

* Starta litet. Testa med en liten uppsättning användare och grupper innan de distribueras till alla. När omfång för etablering har angetts till tilldelade användare och grupper kan du styra detta genom att tilldela en eller två användare eller grupper till appen. När omfång är inställt på alla användare och grupper kan du ange ett [omfångs filter för attribut](https://docs.microsoft.com/azure/active-directory/manage-apps/define-conditional-rules-for-provisioning-user-accounts). 

1. Logga in på [Azure-portalen](https://portal.azure.com). Välj **företags program**och välj sedan **alla program**.

    ![Bladet Företagsprogram](common/enterprise-applications.png)

2. I listan program väljer du **arbets plats efter Facebook**.

    ![Länken för Workplace by Facebook i listan med program](common/all-applications.png)

3. Välj fliken **etablering** .

    ![Fliken etablering](common/provisioning.png)

4. Ställ in **etablerings läget** på **automatiskt**.

    ![Fliken etablering](common/provisioning-automatic.png)

5. Under avsnittet **admin credentials** klickar du på **auktorisera**. Du kommer att omdirigeras till sidan för arbets plats på Facebook-sidan. Mata in din arbets plats efter Facebook-användarnamn och klicka på knappen **Fortsätt** . Klicka på **Testa anslutning** för att se till att Azure AD kan ansluta till arbets ytan på Facebook. Om anslutningen Miss lyckas ser du till att din arbets plats av Facebook-kontot har administratörs behörighet och försöker igen.

    ![etablerings](./media/workplacebyfacebook-provisioning-tutorial/provisioning.png)

    ![auktorisera](./media/workplacebyfacebook-provisioning-tutorial/workplacelogin.png)

6. I fältet **e-postavisering** anger du e-postadressen till den person eller grupp som ska få etablerings fel meddelanden och markerar kryss rutan **Skicka ett e-postmeddelande när ett fel inträffar** .

    ![E-postmeddelande](common/provisioning-notification-email.png)

7. Välj **Spara**.

8. Under avsnittet **mappningar** väljer du **Synkronisera Azure Active Directory användare till arbets ytan efter Facebook**.

9. Granska de användarattribut som synkroniseras från Azure AD till arbets ytan efter Facebook i avsnittet **attribut-mappning** . De attribut som väljs som **matchande** egenskaper används för att matcha användar kontona på arbets platsen av Facebook för uppdaterings åtgärder. Om du väljer att ändra [matchande målattribut](https://docs.microsoft.com/azure/active-directory/manage-apps/customize-application-attributes)måste du se till att arbets ytan av Facebook API stöder filtrering av användare baserat på detta attribut. Välj knappen **Spara** för att spara ändringarna.

   |Attribut|Typ|
   |---|---|
   |userName|Sträng|
   |displayName|Sträng|
   |aktiv|Boolesk|
   |title|Boolesk|
   |e-postmeddelanden [typ EQ "Work"]. värde|Sträng|
   |Name. givenName|Sträng|
   |Name. familyName|Sträng|
   |namn. formaterad|Sträng|
   |adresser [Type EQ "Work"]. formaterad|Sträng|
   |adresser [Type EQ "Work"]. streetAddress|Sträng|
   |adresser [Type EQ "Work"]. plats|Sträng|
   |adresser [Type EQ "Work"]. region|Sträng|
   |adresser [Type EQ "Work"]. land|Sträng|
   |adresser [Type EQ "Work"]. Postnr|Sträng|
   |adresser [Type EQ "other"]. formaterad|Sträng|
   |phoneNumbers [typ EQ "Work"]. värde|Sträng|
   |phoneNumbers [Type EQ "Mobile"]. värde|Sträng|
   |phoneNumbers [Type EQ "fax"]. värde|Sträng|
   |externalId|Sträng|
   |preferredLanguage|Sträng|
   |urn: IETF: params: scim: schemas: tillägg: Enterprise: 2.0: användare: Manager|Sträng|
   |urn: IETF: params: scim: schemas: tillägg: Enterprise: 2.0: användare: avdelning|Sträng|

10. Information om hur du konfigurerar omfångs filter finns i följande instruktioner i [kursen omfångs filter](../manage-apps/define-conditional-rules-for-provisioning-user-accounts.md).

11. Om du vill aktivera Azure AD Provisioning-tjänsten för arbets ytan av Facebook ändrar du **etablerings statusen** till **på** i avsnittet **Inställningar** .

    ![Etablerings status växlad på](common/provisioning-toggle-on.png)

12. Definiera de användare och/eller grupper som du vill etablera till arbets ytan på Facebook genom att välja önskade värden i **omfång** i avsnittet **Inställningar** .

    ![Etablerings omfång](common/provisioning-scope.png)

13. När du är redo att etablera klickar du på **Spara**.

    ![Etablerings konfigurationen sparas](common/provisioning-configuration-save.png)

Den här åtgärden startar den första synkroniseringen av alla användare och grupper som definierats i **omfånget** i avsnittet **Inställningar** . Den första cykeln tar längre tid att utföra än efterföljande cykler, vilket inträffar ungefär var 40: e minut, förutsatt att Azure AD Provisioning-tjänsten körs. 

## <a name="step-6-monitor-your-deployment"></a>Steg 6. Övervaka distributionen
När du har konfigurerat etableringen använder du följande resurser för att övervaka distributionen:

1. Använd [etablerings loggarna](https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-provisioning-logs) för att avgöra vilka användare som har etablerats eller har misslyckats
2. Kontrol lera [förlopps indikatorn](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-when-will-provisioning-finish-specific-user) för att se status för etablerings cykeln och hur nära den är att slutföras
3. Om etablerings konfigurationen verkar vara i ett ohälsosamt tillstånd, kommer programmet att placeras i karantän. Lär dig mer om karantän tillstånd [här](https://docs.microsoft.com/azure/active-directory/manage-apps/application-provisioning-quarantine-status).

## <a name="troubleshooting-tips"></a>Felsökningstips
*  Om du ser en användare som inte har skapats utan problem och det finns en Gransknings logg händelse med koden "1789003" innebär det att användaren är från en overifierad domän.

## <a name="additional-resources"></a>Ytterligare resurser

* [Hantera användar konto etablering för företags program](../manage-apps/configure-automatic-user-provisioning-portal.md)
* [Vad är program åtkomst och enkel inloggning med Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Nästa steg

* [Lär dig hur du granskar loggar och hämtar rapporter om etablerings aktivitet](../manage-apps/check-status-user-account-provisioning.md)
