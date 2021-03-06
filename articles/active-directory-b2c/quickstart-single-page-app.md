---
title: 'Snabb start: Konfigurera inloggning för en enskild sida-app (SPA)'
titleSuffix: Azure AD B2C
description: I den här snabb starten kör du ett exempel på en Enkels Ides applikation som använder Azure Active Directory B2C för att tillhandahålla konto inloggning.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: quickstart
ms.date: 04/04/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0a754873aeafe8d4e7b48d49647469874ff40f7e
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80875901"
---
# <a name="quickstart-set-up-sign-in-for-a-single-page-app-using-azure-active-directory-b2c"></a>Snabbstart – Konfigurera inloggning för en ensidesapp med Azure Active Directory B2C

Azure Active Directory B2C (Azure AD B2C) tillhandahåller moln identitets hantering för att hålla ditt program, din verksamhet och dina kunder skyddade. Med Azure AD B2C kan program autentisera med konton på sociala medier och företagskonton med öppna protokoll. I den här snabbstarten använder du ett enkelsidigt program för ASP.NET till att logga in med en social identitetsprovider och anropa en Azure AD B2C-skyddad webb-API.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Krav

- [Visual Studio-koden](https://code.visualstudio.com/)
- [Node.js](https://nodejs.org/en/download/)
- Socialt konto från Facebook, Google eller Microsoft
- Kod exempel från GitHub: [Active-Directory-B2C-JavaScript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp)

    Du kan [Hämta zip-arkivet](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) eller klona lagrings platsen:

    ```console
    git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
    ```

## <a name="run-the-application"></a>Köra appen

1. Starta servern genom att köra följande kommandon från Node.js-kommandotolken:

    ```console
    cd active-directory-b2c-javascript-msal-singlepageapp
    npm install && npm update
    npm start
    ```

    Servern som startades av *Server. js* visar den port den lyssnar på:

    ```console
    Listening on port 6420...
    ```

1. Bläddra till programmets URL. Till exempel `http://localhost:6420`.

    ![Exempel program för en enda sida som visas i webbläsare](./media/quickstart-single-page-app/sample-app-spa.png)

## <a name="sign-in-using-your-account"></a>Logga in på ditt konto

1. Välj **Logga** in för att starta användar resan.
1. Azure AD B2C visar en inloggnings sida för ett fiktivt företag som kallas Fabrikam för exempel webb programmet. Om du vill registrera dig med en social identitetsprovider väljer du knappen för den identitetsprovider som du vill använda.

    ![Sidan logga in eller registrera dig med identitets leverantörs knappar](./media/quickstart-single-page-app/sign-in-or-sign-up-spa.png)

    Du autentiserar (loggar in) med dina autentiseringsuppgifter för ditt sociala konto och godkänner att programmet läser information från ditt sociala konto. När du beviljar åtkomst kan programmet hämta profilinformation från det sociala kontot, till exempel ditt namn och din ort.

1. Avsluta inloggningsprocessen för identitetsprovidern.

## <a name="access-a-protected-api-resource"></a>Få åtkomst till en skyddad API-resurs

Välj **anropa API** om du vill att ditt visnings namn ska returneras från webb-API: et som ett JSON-objekt.

![Exempel program i webbläsare som visar svaret på webb-API](./media/quickstart-single-page-app/call-api-spa.png)

Det enkelsidiga exempelprogrammet innehåller en åtkomsttoken i begäran till den skyddade webb-API-resursen.

## <a name="next-steps"></a>Nästa steg

I den här snabb starten använde du ett exempel på en Enkels Ides applikation för att:

- Logga in med en social identitetsprovider
- Skapa ett Azure AD B2C användar konto (skapas automatiskt vid inloggning)
- Anropa ett webb-API som skyddas av Azure AD B2C

Kom igång med att skapa en egen Azure AD B2C-klientorganisation.

> [!div class="nextstepaction"]
> [Skapa en Azure Active Directory B2C-klientorganisation i Azure-portalen](tutorial-create-tenant.md)
