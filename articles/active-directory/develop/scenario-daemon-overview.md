---
title: 'Bygg en daemon-app som anropar webb-API: er – Microsoft Identity Platform | Azure'
description: 'Lär dig hur du skapar en daemon-app som anropar webb-API: er'
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 01/31/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: df06c4c55941f4424d6b90d2846af17bf055b2e4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80885471"
---
# <a name="scenario-daemon-application-that-calls-web-apis"></a>Scenario: daemon-program som anropar webb-API: er

Lär dig allt du behöver för att bygga ett daemon-program som anropar webb-API: er.

## <a name="prerequisites"></a>Krav

[!INCLUDE [Pre-requisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="overview"></a>Översikt

Ditt program kan hämta en token för att anropa ett webb-API för sig själv (inte för en användares räkning). Det här scenariot är användbart för daemon-program. Den använder standard beviljast OAuth 2,0 [-klientautentiseringsuppgifter.](v2-oauth2-client-creds-grant-flow.md)

![Daemon-appar](./media/scenario-daemon-app/daemon-app.svg)

Här följer några exempel på användnings fall för daemon-appar:

- Webb program som används för att etablera eller administrera användare eller göra batch-processer i en katalog
- Skriv bords program (t. ex. Windows-tjänster på Windows-eller daemon-processer i Linux) som utför batch-jobb eller en tjänst för operativ system som körs i bakgrunden
- Webb-API: er som behöver manipulera kataloger, inte vissa användare

Det finns ett annat vanligt fall där icke-daemon-program använder klientautentiseringsuppgifter: även om de agerar för användarens räkning måste de ha åtkomst till ett webb-API eller en resurs under sin egen identitet av tekniska skäl. Ett exempel är åtkomst till hemligheter i Azure Key Vault eller en Azure SQL-databas för cache.

Program som hämtar en token för sina egna identiteter:

- Är konfidentiella klient program. Dessa appar, med tanke på att de får åtkomst till resurser oberoende av användarna, måste bevisa sin identitet. De är också i stället känsliga appar. De måste godkännas av Azure Active Directory (Azure AD)-klient organisations administratörer.
- Har registrerat en hemlighet (program lösen ord eller certifikat) med Azure AD. Den här hemligheten skickas under anropet till Azure AD för att hämta en token.

## <a name="specifics"></a>Information

> [!IMPORTANT]
>
> - Användare kan inte interagera med ett daemon-program. Ett daemon-program kräver en egen identitet. Den här typen av program begär en åtkomsttoken genom att använda dess program identitet och presentera dess program-ID, autentiseringsuppgift (lösen ord eller certifikat) och program-ID-URI till Azure AD. Efter en lyckad autentisering tar daemonen emot en åtkomsttoken (och en uppdateringstoken) från slut punkten för Microsoft Identity Platform. Denna token används sedan för att anropa webb-API: et (och uppdateras vid behov).
> - Eftersom användarna inte kan interagera med daemon-program går det inte att använda ett stegvist tillstånd. Alla nödvändiga API-behörigheter måste konfigureras vid program registrering. Koden för programmet begär bara statiskt definierade behörigheter. Det innebär också att daemon-program inte stöder stegvist tillstånd.

För utvecklare har slut punkt till slut punkt i det här scenariot följande aspekter:

- Daemon-program kan bara fungera i Azure AD-klienter. Det skulle inte vara bra att skapa ett daemon-program som försöker manipulera Microsoft-personliga konton. Om du är en affärsutvecklare (LOB) kan du skapa daemon-appen i din klient organisation. Om du är en ISV kanske du vill skapa ett daemon-program för flera innehavare. Varje klient organisations administratör måste ge sitt medgivande.
- Svars-URI behövs inte under [program registreringen](./scenario-daemon-app-registration.md). Du måste dela hemligheter eller certifikat eller signerade intyg med Azure AD. Du måste också begära program behörigheter och bevilja administratörs medgivande för att kunna använda dessa program behörigheter.
- [Program konfigurationen](./scenario-daemon-app-configuration.md) måste ange klientautentiseringsuppgifterna som delas med Azure AD under program registreringen.
- Det [omfång](scenario-daemon-acquire-token.md#scopes-to-request) som används för att hämta en token med ett flöde för klientautentiseringsuppgifter måste vara ett statiskt omfång.

## <a name="next-steps"></a>Nästa steg

> [!div class="nextstepaction"]
> [Daemon App – app-registrering](./scenario-daemon-app-registration.md)
