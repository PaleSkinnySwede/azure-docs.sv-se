---
title: 'Självstudie: Azure Active Directory integrering med Convercent | Microsoft Docs'
description: Lär dig hur du konfigurerar enkel inloggning mellan Azure Active Directory och Convercent.
services: active-directory
documentationCenter: na
author: jeevansd
manager: mtillman
ms.reviewer: barbkess
ms.assetid: f9c9d290-0e13-490b-b559-0be772d6a690
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 01/25/2019
ms.author: jeedes
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4e9c96db69ed3bc49081607ab6e2c3ad6077c3c6
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "73159340"
---
# <a name="tutorial-azure-active-directory-integration-with-convercent"></a>Självstudie: Azure Active Directory integrering med Convercent

I den här självstudien lär du dig att integrera Convercent med Azure Active Directory (AD Azure).
Integreringen av Convercent med Azure AD medför följande fördelar:

* Du kan i Azure AD styra vem som har åtkomst till Convercent.
* Du kan göra så att dina användare automatiskt loggas in på Convercent (enkel inloggning) med sina Azure AD-konton.
* Du kan hantera dina konton på en central plats – Azure-portalen.

Om du vill ha mer information om SaaS-appintegrering med Azure AD läser du avsnittet om [programåtkomst och enkel inloggning med Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis).
Om du inte har en Azure-prenumeration kan du [skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) innan du börjar.

## <a name="prerequisites"></a>Krav

För att konfigurera Azure AD-integrering med Convercent behöver du följande:

* En Azure AD-prenumeration. Om du inte har någon Azure AD-miljö kan du hämta en månads utvärderingsversion [här](https://azure.microsoft.com/pricing/free-trial/)
* Convercent-prenumeration med enkel inloggning aktiverat

## <a name="scenario-description"></a>Scenariobeskrivning

I den här självstudien konfigurerar och testar du enkel inloggning med Azure AD i en testmiljö.

* Convercent har stöd för **SP**- och **IDP**-initierad enkel inloggning

## <a name="adding-convercent-from-the-gallery"></a>Lägga till Convercent från galleriet

För att konfigurera integreringen av Convercent i Azure AD måste du lägga till Convercent från galleriet till din lista över hanterade SaaS-appar.

**Utför följande steg för att lägga till Convercent från galleriet:**

1. I **[Azure-portalen](https://portal.azure.com)** går du till den vänstra navigeringspanelen och klickar på **Azure Active Directory**-ikonen.

    ![Azure Active Directory-knappen](common/select-azuread.png)

2. Gå till **Företagsprogram** och välj alternativet **Alla program**.

    ![Bladet Företagsprogram](common/enterprise-applications.png)

3. Lägg till ett nytt program genom att klicka på knappen **Nytt program** högst upp i dialogrutan.

    ![Knappen Nytt program](common/add-new-app.png)

4. I sökrutan skriver du **Convercent**, väljer **Convercent** i resultatpanelen och klickar på knappen **Lägg till** för att lägga till programmet.

     ![Convercent i resultatlistan](common/search-new-app.png)

## <a name="configure-and-test-azure-ad-single-sign-on"></a>Konfigurera och testa enkel inloggning med Azure AD

I det här avsnittet konfigurerar och testar du enkel inloggning i Azure AD med Convercent baserat på en testanvändare med namnet **Britta Simon**. För att enkel inloggning ska fungera måste en länkrelation mellan en Azure AD-användare och den relaterade användaren i Convercent upprättas.

För att konfigurera och testa enkel inloggning med Azure AD med Convercent behöver du utföra följande byggstenar:

1. **[Konfigurera enkel inloggning med Azure AD](#configure-azure-ad-single-sign-on)** – så att användarna kan använda den här funktionen.
2. **[Konfigurera enkel inloggning för Convercent](#configure-convercent-single-sign-on)** – för att konfigurera inställningarna för enkel inloggning på programsidan.
3. **[Skapa en Azure AD-testanvändare](#create-an-azure-ad-test-user)** – för att testa enkel inloggning med Azure AD med Britta Simon.
4. **[Tilldela Azure AD-testanvändaren](#assign-the-azure-ad-test-user)** – så att Britta Simon kan använda enkel inloggning med Azure AD.
5. **[Skapa Convercent-testanvändare](#create-convercent-test-user)** – för att skapa en motsvarighet till Britta Simon i Convercent som är länkad till Azure AD-representationen av användaren.
6. **[Testa enkel inloggning](#test-single-sign-on)** – för att verifiera om konfigurationen fungerar.

### <a name="configure-azure-ad-single-sign-on"></a>Konfigurera enkel inloggning med Azure AD

I det här avsnittet aktiverar du enkel inloggning med Azure AD i Azure-portalen.

Utför följande steg för att konfigurera enkel inloggning i Azure AD med Convercent:

1. I [Azure-portalen](https://portal.azure.com/) går du till sidan för **Convercent**-programintegrering och väljer **Enkel inloggning**.

    ![Konfigurera länk för enkel inloggning](common/select-sso.png)

2. I dialogrutan **Välj en metod för enkel inloggning** väljer du läget **SAML/WS-Fed** för att aktivera enkel inloggning.

    ![Välja läge för enkel inloggning](common/select-saml-option.png)

3. På sidan **Konfigurera enkel inloggning med SAML** klickar du på **redigeringsikonen** för att öppna dialogrutan **Grundläggande SAML-konfiguration**.

    ![Redigera grundläggande SAML-konfiguration](common/edit-urls.png)

4. Om du vill konfigurera appen i **IDP**-initierat läge gör du följande i avsnittet **Grundläggande SAML-konfiguration**:

    ![Information om enkel inloggning med Convercent-domän och -URL:er](common/both-identifier.png)

    I textrutan **Identifierare** skriver du en URL med följande mönster:  `https://<instancename>.convercent.com/`

5. Klicka på **Ange ytterligare URL:er** och utför följande steg om du vill konfigurera programmet i **SP**-initierat läge:

    ![Information om enkel inloggning med Convercent-domän och -URL:er](common/both-advanced-urls.png)

    a. I text rutan **inloggnings-URL** skriver du en URL med följande mönster:`https://<instancename>.convercent.com/`

    b. Skriv en URL med följande mönster i textrutan **Vidarebefordransstatus**: `https://<instancename>.convercent.com/`

    > [!NOTE]
    > Dessa värden är inte verkliga. Uppdatera de här värdena med faktisk identifierare, inloggnings-URL och vidarebefordransstatus. Kontakta [kundsupporten för Convercent](http://support.convercent.com/) om be om dessa värden. Du kan även se mönstren som visas i avsnittet **Grundläggande SAML-konfiguration** i Azure-portalen.

6. På sidan **Set up Single Sign-On with SAML** (Konfigurera enkel inloggning med SAML) går du till avsnittet **SAML Signing Certificate** (SAML-signeringscertifikat), klickar på **Ladda ned** för att ladda ned **Federation Metadata-XML** från de angivna alternativen enligt dina behov och spara den på datorn.

    ![Länk för nedladdning av certifikatet](common/metadataxml.png)

7. I avsnittet **Konfigurera Convercent** kopierar du lämpliga URL:er enligt dina behov.

    ![Kopiera konfigurations-URL:er](common/copy-configuration-urls.png)

    a. Inloggnings-URL

    b. Azure AD-identifierare

    c. Utloggnings-URL

### <a name="configure-convercent-single-sign-on"></a>Konfigurera enkel inloggning för Convercent

För att konfigurera enkel inloggning på **Convercent**-sidan behöver du skicka nedladdad **federationsmetadata-XML** och lämpliga kopierade URL:er från Azure-portalen till [supportteamet för Convercent](http://support.convercent.com/). De anger inställningen så att SAML SSO-anslutningen ställs in korrekt på båda sidorna.

### <a name="create-an-azure-ad-test-user"></a>Skapa en Azure AD-testanvändare 

Målet med det här avsnittet är att skapa en testanvändare i Azure-portalen med namnet Britta Simon.

1. Gå till den vänstra rutan i Azure-portalen och välj **Azure Active Directory**, välj **Users** och sedan **Alla användare**.

    ![Länkarna ”Användare och grupper” och ”Alla grupper”](common/users.png)

2. Välj **ny användare** överst på skärmen.

    ![Knappen Ny användare](common/new-user.png)

3. Genomför följande steg i Användaregenskaper.

    ![Dialogrutan Användare](common/user-properties.png)

    a. I fältet **Namn** anger du **BrittaSimon**.
  
    b. I fältet **användar namn** skriver du **brittasimon\@yourcompanydomain. extension**  
    Till exempel, BrittaSimon@contoso.com

    c. Markera kryssrutan **Visa lösenord** och skriv sedan ned det värde som visas i rutan Lösenord.

    d. Klicka på **Skapa**.

### <a name="assign-the-azure-ad-test-user"></a>Tilldela Azure AD-testanvändaren

I det här avsnittet gör du det möjligt för Britta Simon att använda enkel inloggning med Azure genom att ge åtkomst till Convercent.

1. I Azure-portalen väljer du **Företagsprogram**, **Alla program** och sedan **Convercent**.

    ![Bladet Företagsprogram](common/enterprise-applications.png)

2. I programlistan väljer du **Convercent**.

    ![Länken för Convercent i programlistan](common/all-applications.png)

3. På menyn till vänster väljer du **Användare och grupper**.

    ![Länken ”Användare och grupper”](common/users-groups-blade.png)

4. Klicka på knappen **Lägg till användare** och välj sedan **Användare och grupper** i dialogrutan **Lägg till tilldelning**.

    ![Fönstret Lägg till tilldelning](common/add-assign-user.png)

5. I dialogrutan **Användare och grupper** väljer du **Britta Simon** i listan med användare och klickar på knappen **Välj** längst ned på skärmen.

6. Om du förväntar dig ett roll värde i SAML-kontrollen väljer du lämplig roll för användaren i listan i dialog rutan **Välj roll** och klickar sedan på knappen **Välj** längst ned på skärmen.

7. I dialogrutan **Lägg till tilldelning** klickar du på knappen **Tilldela**.

### <a name="create-convercent-test-user"></a>Skapa Convercent-testanvändare

I det här avsnittet skapar du en användare med namnet Britta Simon i Convercent. Kontakta [supportteamet för Convercent](http://support.convercent.com/) och lägg till användarna i Convercent-plattformen. Användare måste skapas och aktiveras innan du använder enkel inloggning.

### <a name="test-single-sign-on"></a>Testa enkel inloggning 

I det här avsnittet testar du konfigurationen för enkel inloggning Azure AD med hjälp av åtkomstpanelen.

När du klickar på Convercent-panelen i åtkomstpanelen bör du automatiskt loggas in på Convercent som du har konfigurerat enkel inloggning för. Mer information om åtkomstpanelen finns i [introduktionen till åtkomstpanelen](https://docs.microsoft.com/azure/active-directory/active-directory-saas-access-panel-introduction).

## <a name="additional-resources"></a>Ytterligare resurser

- [Lista över självstudier om hur du integrerar SaaS-appar med Azure Active Directory](https://docs.microsoft.com/azure/active-directory/active-directory-saas-tutorial-list)

- [Vad är program åtkomst och enkel inloggning med Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)

- [Vad är villkorlig åtkomst i Azure Active Directory?](https://docs.microsoft.com/azure/active-directory/conditional-access/overview)

