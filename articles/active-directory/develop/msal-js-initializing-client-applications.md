---
title: Initiera MSAL. js-klient program | Azure
titleSuffix: Microsoft identity platform
description: Lär dig mer om att initiera klient program med hjälp av Microsoft Authentication Library för Java Script (MSAL. js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/12/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: fbd700c787a844fa7538ed198f76ed5c06af2c28
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81010162"
---
# <a name="initialize-client-applications-using-msaljs"></a>Initiera klient program med MSAL. js
I den här artikeln beskrivs hur du initierar Microsoft Authentication Library för Java Script (MSAL. js) med en instans av ett användar agent program. Användar agent programmet är en form av ett offentligt klient program där klient koden körs i en användar agent, till exempel en webbläsare. Dessa klienter lagrar inte hemligheter eftersom webb läsar kontexten är lättillgänglig. Läs [översikten](msal-client-applications.md)om du vill veta mer om klient program typer och program konfigurations alternativ.

## <a name="prerequisites"></a>Krav
Innan du initierar ett program måste du först [registrera det med Azure Portal](scenario-spa-app-registration.md) så att din app kan integreras med Microsoft Identity Platform. Efter registreringen kan du behöva följande information (som du hittar i Azure Portal):

- Klient-ID (en sträng som representerar ett GUID för ditt program)
- Identitets leverantörens URL (kallas instansen) och inloggnings gruppen för ditt program. Dessa två parametrar kallas gemensamt för utfärdaren.
- Klient-ID: t om du skriver ett affärs program enbart för din organisation (även kallat program med en enda klient).
- För Web Apps måste du också ange redirectUri där identitetsprovider ska återgå till ditt program med säkerhetstoken.

## <a name="initializing-applications"></a>Initierar program

Du kan använda MSAL. js enligt följande i ett oformaterat JavaScript/typescript-program. Initiera kontexten för MSAL- `UserAgentApplication` autentisering genom att instansiera med ett konfigurations objekt. Den minsta konfiguration som krävs för att initiera MSAL. js är clientID för ditt program som du bör hämta från program registrerings portalen.

För autentiseringsmetoder med omdirigerings flöden`loginRedirect` ( `acquireTokenRedirect`och), i MSAL. js 1.2. x eller tidigare, måste du explicit registrera ett återanrop för lyckad eller fel via `handleRedirectCallback()` metoden. Detta krävs eftersom omdirigerings flöden inte returnerar löfte som metoder med en popup-upplevelse. Detta blev valfritt i MSAL. js-version 1.3.0.

```javascript
// Configuration object constructed
const config = {
    auth: {
        clientId: "abcd-ef12-gh34-ikkl-ashdjhlhsdg"
    }
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

// (optional when using redirect methods) register redirect call back for Success or Error
myMSALObj.handleRedirectCallback(authCallback);
```

MSAL. js har utformats för att ha en enda instans och konfiguration av `UserAgentApplication` för att representera en enda autentiserings kontext. Flera instanser rekommenderas inte eftersom de orsakar en motstridig cache-poster och beteende i webbläsaren.

## <a name="configuration-options"></a>Konfigurationsalternativ

MSAL. js har ett konfigurations objekt som visas nedan och som ger en gruppering av konfigurerbara alternativ som är tillgängliga `UserAgentApplication`för att skapa en instans av.

```javascript
type storage = "localStorage" | "sessionStorage";

// Protocol Support
export type AuthOptions = {
    clientId: string;
    authority?: string;
    validateAuthority?: boolean;
    redirectUri?: string | (() => string);
    postLogoutRedirectUri?: string | (() => string);
    navigateToLoginRequestUrl?: boolean;
};

// Cache Support
export type CacheOptions = {
    cacheLocation?: CacheLocation;
    storeAuthStateInCookie?: boolean;
};

// Library support
export type SystemOptions = {
    logger?: Logger;
    loadFrameTimeout?: number;
    tokenRenewalOffsetSeconds?: number;
    navigateFrameWait?: number;
};

// Developer App Environment Support
export type FrameworkOptions = {
    isAngular?: boolean;
    unprotectedResources?: Array<string>;
    protectedResourceMap?: Map<string, Array<string>>;
};

// Configuration Object
export type Configuration = {
    auth: AuthOptions,
    cache?: CacheOptions,
    system?: SystemOptions,
    framework?: FrameworkOptions
};
```

Nedan visas den totala uppsättningen konfigurerbara alternativ som stöds för närvarande i konfigurationsobjektet konfiguration:

- **clientID**: krävs. ClientID för ditt program, du bör hämta detta från program registrerings portalen.

- **auktoritet**: valfritt. En URL som anger en katalog som MSAL kan begära token från. Standardvärdet är `https://login.microsoftonline.com/common`:.
    * I&lt;Azure AD är det av formen https://instance&gt;/&lt;&gt;, där &lt;instance&gt; är en identitets leverantörs domän (till exempel `https://login.microsoftonline.com`) och &lt;mål grupp&gt; är en identifierare som representerar inloggnings mål gruppen. Detta kan vara följande värden:
        * `https://login.microsoftonline.com/<tenant>`-Tenant är en domän som är kopplad till klienten, till exempel contoso.onmicrosoft.com eller GUID som representerar `TenantID` egenskapen för katalogen som används för att logga in användare av en speciell organisation.
        * `https://login.microsoftonline.com/common`– Används för att logga in användare med arbets-och skol konton eller ett personligt Microsoft-konto.
        * `https://login.microsoftonline.com/organizations/`– Används för att logga in användare med arbets-och skol konton.
        * `https://login.microsoftonline.com/consumers/`– Används för att logga in användare med endast personlig Microsoft-konto (Live).
    * I Azure AD B2C är det ett formulär `https://<instance>/tfp/<tenant>/<policyName>/`där instansen är den Azure AD B2C domänen, t. ex. {your-Tenant-Name}. b2clogin. com är klienten namnet på den Azure AD B2C klienten, t. ex. {your-Tenant-Name}. onmicrosoft. com, policyName är namnet på B2C-principen som ska tillämpas.


- **validateAuthority**: valfritt.  Verifiera utfärdaren av tokens. Standardvärdet är `true`. För B2C-program, eftersom auktoritets värdet är känt och kan skilja sig åt per princip, så fungerar inte auktoritets verifieringen och måste ställas in på `false`.

- **redirectUri**: valfritt.  Omdirigerings-URI för appen, där autentiseringsbegäranden kan skickas och tas emot av din app. Det måste exakt matcha en av de omdirigerings-URI: er som du registrerade i portalen. Standardvärdet är `window.location.href`.

- **postLogoutRedirectUri**: valfritt.  Omdirigerar användaren till `postLogoutRedirectUri` efter utloggning. Standardvärdet `redirectUri`är.

- **navigateToLoginRequestUrl**: valfritt. Möjlighet att inaktivera standard navigering till start sidan efter inloggningen. Standardvärdet är true. Detta används endast för omdirigerings flöden.

- **cacheLocation**: valfritt.  Anger webb läsar lagring `localStorage` till `sessionStorage`antingen eller. Standardvärdet är `sessionStorage`.

- **storeAuthStateInCookie**: valfritt.  Den här flaggan introducerades i MSAL. js v 0.2.2 som en korrigering för problemet med [autentiserings-loopen](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#1-issues-due-to-security-zones) i Microsoft Internet Explorer och Microsoft Edge. Aktivera flaggan `storeAuthStateInCookie` till true för att dra nytta av den här korrigeringen. När detta är aktiverat kommer MSAL. js att lagra status för auth-begäran som krävs för verifiering av auth-flöden i webbläsarens cookies. Som standard är den här flaggan inställd `false`på.

- **loggning**: valfritt.  Ett logg objekt med en callback-instans som kan tillhandahållas av utvecklaren för att använda och publicera loggar på ett anpassat sätt. Information om hur du skickar logg objekt finns i [Logga med msal. js](msal-logging.md).

- **loadFrameTimeout**: valfritt.  Antalet millisekunder av inaktivitet innan ett svar på förnyelse av en token från Azure AD bör anses vara tids gränsen. Standardvärdet är 6 sekunder.

- **tokenRenewalOffsetSeconds**: valfritt. Antalet millisekunder som anger det fönster som offset krävde för att förnya token innan det går ut. Standardvärdet är 300 millisekunder.

- **navigateFrameWait**: valfritt. Antalet millisekunder som anger vänte tiden innan dolda iframes navigerar till sitt mål. Standardvärdet är 500 millisekunder.

Dessa kan endast användas för att skickas ned från MSAL-biblioteket:
- **unprotectedResources**: valfritt.  Matris med URI: er som är oskyddade resurser. MSAL kommer inte att koppla en token till utgående begär Anden som har dessa URI: er. Standardvärdet är `null`.

- **protectedResourceMap**: valfritt.  Detta är mappning av resurser till omfattningar som används av MSAL för att automatiskt koppla åtkomsttoken i webb-API-anrop. En enda åtkomsttoken hämtas för resursen. Så du kan mappa en viss resurs Sök väg enligt följande: {https://graph.microsoft.com/v1.0/me"", ["User. Read"]} eller App-URL: en för resursen som {"https://graph.microsoft.com/", ["User. Read", "mail. send"]}. Detta krävs för CORS-anrop. Standardvärdet är `null`.
