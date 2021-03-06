---
title: 'Självstudie: Azure Active Directory integrering med Evidence.com | Microsoft Docs'
description: Lär dig hur du konfigurerar enkel inloggning mellan Azure Active Directory och Evidence.com.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: f9a7cb7c-ff67-40dc-872c-1fa35f9dd03b
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/24/2020
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4b2e04c49b05173c6eef27da30c96e2fd8b40b5d
ms.sourcegitcommit: 4499035f03e7a8fb40f5cff616eb01753b986278
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/03/2020
ms.locfileid: "82734696"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-evidencecom"></a>Självstudie: Azure Active Directory integration med enkel inloggning (SSO) med Evidence.com

I den här självstudien får du lära dig hur du integrerar Evidence.com med Azure Active Directory (Azure AD). När du integrerar Evidence.com med Azure AD kan du:

* Kontroll i Azure AD som har åtkomst till Evidence.com.
* Gör det möjligt för användarna att logga in automatiskt till Evidence.com med sina Azure AD-konton.
* Hantera dina konton på en central plats – Azure Portal.

Mer information om SaaS app integration med Azure AD finns i [Vad är program åtkomst och enkel inloggning med Azure Active Directory](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on).

## <a name="prerequisites"></a>Krav

För att komma igång behöver du följande objekt:

* En Azure AD-prenumeration. Om du inte har någon prenumeration kan du få ett [kostnads fritt konto](https://azure.microsoft.com/free/).
* Evidence.com för enkel inloggning (SSO) aktive rad.

## <a name="scenario-description"></a>Scenariobeskrivning

I den här självstudien konfigurerar och testar du Azure AD SSO i en test miljö.

* Evidence.com har stöd för **SP**-initierad enkel inloggning
* När du har konfigurerat Evidence.com kan du framtvinga sessionshantering, som skyddar exfiltrering och intrånget för organisationens känsliga data i real tid. Kontroll av sessionen utökas från villkorlig åtkomst. [Lär dig hur du tvingar fram en session med Microsoft Cloud App Security](https://docs.microsoft.com/cloud-app-security/proxy-deployment-any-app).

## <a name="adding-evidencecom-from-the-gallery"></a>Lägga till Evidence.com från galleriet

För att konfigurera integrering av Evidence.com i Azure AD behöver du lägga till Evidence.com från galleriet till din lista över hanterade SaaS-appar.

1. Logga in på [Azure Portal](https://portal.azure.com) med antingen ett arbets-eller skol konto eller en personlig Microsoft-konto.
1. I det vänstra navigerings fönstret väljer du tjänsten **Azure Active Directory** .
1. Navigera till **företags program** och välj sedan **alla program**.
1. Välj **nytt program**om du vill lägga till ett nytt program.
1. I avsnittet **Lägg till från galleriet** , skriver du **Evidence.com** i sökrutan.
1. Välj **Evidence.com** från resultat panelen och Lägg sedan till appen. Vänta några sekunder medan appen läggs till i din klient organisation.

## <a name="configure-and-test-azure-ad-single-sign-on-for-evidencecom"></a>Konfigurera och testa enkel inloggning med Azure AD för Evidence.com

Konfigurera och testa Azure AD SSO med Evidence.com med hjälp av en test användare som heter **B. Simon**. För att SSO ska fungera måste du upprätta en länk relation mellan en Azure AD-användare och den relaterade användaren i Evidence.com.

Om du vill konfigurera och testa Azure AD SSO med Evidence.com, slutför du följande Bygg stenar:

1. **[Konfigurera Azure AD SSO](#configure-azure-ad-sso)** – så att användarna kan använda den här funktionen.
    1. **[Skapa en Azure AD-test](#create-an-azure-ad-test-user)** för att testa enkel inloggning med Azure AD med B. Simon.
    1. **[Tilldela Azure AD-testuser](#assign-the-azure-ad-test-user)** -för att aktivera B. Simon för att använda enkel inloggning med Azure AD.
1. **[Konfigurera Evidence.com SSO](#configure-evidencecom-sso)** – för att konfigurera inställningarna för enkel inloggning på program sidan.
    1. **[Skapa Evidence.com test User](#create-evidencecom-test-user)** -om du vill ha en motsvarighet till B. Simon i Evidence.com som är länkad till Azure AD-representation av användare.
1. **[Testa SSO](#test-sso)** – för att kontrol lera om konfigurationen fungerar.

## <a name="configure-azure-ad-sso"></a>Konfigurera Azure AD SSO

Följ de här stegen för att aktivera Azure AD SSO i Azure Portal.

1. I [Azure Portal](https://portal.azure.com/)går du till sidan för program integrering i **Evidence.com** , letar upp avsnittet **Hantera** och väljer **enkel inloggning**.
1. På sidan **Välj metod för enkel inloggning** väljer du **SAML**.
1. På sidan **Konfigurera enkel inloggning med SAML** klickar du på ikonen Redigera/penna för **grundläggande SAML-konfiguration** för att redigera inställningarna.

   ![Redigera grundläggande SAML-konfiguration](common/edit-urls.png)

1. Utför följande steg på sidan **Konfigurera enkel inloggning med SAML** :

    a. I textrutan **Inloggnings-URL** anger du en URL enligt följande mönster: `https://<yourtenant>.evidence.com`

    b. I textrutan **Identifierare (entitets-ID)** anger du en URL enligt följande mönster: `https://<yourtenant>.evidence.com`

    c. I textrutan **Svars-URL** skriver du en URL med följande mönster: `https://<your tenant>.evidence.com/?class=UIX&proc=Login`
    
    > [!NOTE]
    > Dessa värden är inte verkliga. Uppdatera de här värdena med faktisk inloggnings-URL, identifierare och svars-URL. Kontakta [kundsupporten för Evidence.com](https://communities.taser.com/support/SupportContactUs?typ=LE) och be om dessa värden. Du kan även se mönstren som visas i avsnittet **Grundläggande SAML-konfiguration** i Azure-portalen.

5. På sidan **Konfigurera enkel inloggning med SAML** går du till avsnittet **SAML-signeringscertifikat**, klickar du på **Ladda ned** för att ladda ned **Certifikat (Base64)** från de angivna alternativen enligt dina behov och sparar det på datorn.

    ![Länk för nedladdning av certifikatet](common/certificatebase64.png)

1. I avsnittet **konfigurera Evidence.com** kopierar du lämpliga URL: er baserat på ditt krav.

    ![Kopiera konfigurations-URL:er](common/copy-configuration-urls.png)

### <a name="create-an-azure-ad-test-user"></a>Skapa en Azure AD-testanvändare

I det här avsnittet ska du skapa en test användare i Azure Portal som kallas B. Simon.

1. I den vänstra rutan i Azure Portal väljer du **Azure Active Directory**, väljer **användare**och väljer sedan **alla användare**.
1. Välj **ny användare** överst på skärmen.
1. I **användar** egenskaperna följer du de här stegen:
   1. I **Namn**-fältet skriver du `B.Simon`.  
   1. I fältet **användar namn** anger du username@companydomain.extension. Till exempel `B.Simon@contoso.com`.
   1. Markera kryssrutan **Visa lösenord** och skriv sedan ned det värde som visas i rutan **Lösenord**.
   1. Klicka på **Skapa**.

### <a name="assign-the-azure-ad-test-user"></a>Tilldela Azure AD-testanvändaren

I det här avsnittet ska du aktivera B. Simon för att använda enkel inloggning med Azure genom att bevilja åtkomst till Evidence.com.

1. I Azure Portal väljer du **företags program**och väljer sedan **alla program**.
1. I programlistan väljer du **Evidence.com**.
1. På sidan Översikt för appen letar du reda på avsnittet **Hantera** och väljer **användare och grupper**.

   ![Länken ”Användare och grupper”](common/users-groups-blade.png)

1. Välj **Lägg till användare**och välj sedan **användare och grupper** i dialog rutan **Lägg till tilldelning** .

    ![Länken Lägg till användare](common/add-assign-user.png)

1. I dialog rutan **användare och grupper** väljer du **B. Simon** från listan användare och klickar sedan på knappen **Välj** längst ned på skärmen.
1. Om du förväntar dig ett roll värde i SAML Assertion, i dialog rutan **Välj roll** , väljer du lämplig roll för användaren i listan och klickar sedan på knappen **Välj** längst ned på skärmen.
1. Klicka på knappen **tilldela** i dialog rutan **Lägg till tilldelning** .

## <a name="configure-evidencecom-sso"></a>Konfigurera Evidence.com SSO

1. Logga in på Evidence.com-klienten som administratör i ett separat webbläsarfönster och gå till fliken **admin**

2. Klicka på **Agency Single Sign On** (Enkel inloggning för Agency)

3. Välj **SAML Based Single Sign On** (SAML-baserad enkel inloggning)

4. Kopiera de värden för **Azure AD-identifierare**, **inloggnings-URL** och **utloggnings-URL** som visas i Azure-portalen till motsvarande fält i Evidence.com.

5. Öppna det nedladdade certifikatet (Base64) i Anteckningar, kopiera dess innehåll till Urklipp och klistra in det i textrutan **Säkerhetscertifikat**. 

6. Spara konfigurationen i Evidence.com.

### <a name="create-evidencecom-test-user"></a>Skapa Evidence.com-testanvändare

För att Azure AD-användare ska kunna logga in måste de etableras för åtkomst i Evidence.com-programmet. Det här avsnittet beskriver hur du skapar Azure AD-användarkonton i Evidence.com

**Så här etablerar du ett användarkonto i Evidence.com:**

1. Logga in på din Evidence.com-företags webbplats som administratör i ett webbläsarfönster.

2. Gå till fliken **Admin**.

3. Klicka på **Lägg till användare**.

4. Klicka på knappen **Lägg till**.

5. **E-postadressen** för den tillagda användaren måste matcha användarnamnet för de användare i Azure AD som du vill ge åtkomst. Om användarnamnet och e-postadressen inte har samma värde i din organisation kan du använda avsnittet **Evidence.com > Attribut > Enkel inloggning** i Azure-portalen för att ändra den nameidentifier (namnidentifierare) som skickas till Evidence.com till att bli e-postadressen.

## <a name="test-sso"></a>Testa SSO 

I det här avsnittet testar du konfigurationen för enkel inloggning Azure AD med hjälp av åtkomstpanelen.

När du klickar på Evidence.com-panelen på åtkomstpanelen bör du automatiskt loggas in i Evidence.com som du har konfigurerat enkel inloggning för. Mer information om åtkomstpanelen finns i [introduktionen till åtkomstpanelen](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Ytterligare resurser

- [Lista över självstudier om hur du integrerar SaaS-appar med Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Vad är programåtkomst och enkel inloggning med Azure Active Directory? ](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [Vad är villkorsstyrd åtkomst i Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

- [Prova Evidence.com med Azure AD](https://aad.portal.azure.com/)

- [Vad är session Control i Microsoft Cloud App Security?](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

- [Så här skyddar du Evidence.com med avancerad synlighet och kontroller](https://docs.microsoft.com/cloud-app-security/proxy-intro-aad)

