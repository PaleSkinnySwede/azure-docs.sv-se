---
title: Vad är en Azure AD-ansluten enhet?
description: Lär dig hur enhets identitets hantering kan hjälpa dig att hantera enheter som har åtkomst till resurser i din miljö.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: conceptual
ms.date: 06/27/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 40f89fbe19b93601f9e0525f0387e402bd175fe4
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "78672668"
---
# <a name="azure-ad-joined-devices"></a>Azure AD-anslutna enheter

Azure AD Join är avsett för organisationer som bara vill vara molnbaserade eller endast i molnet. Alla organisationer kan distribuera Azure AD-anslutna enheter oavsett storlek eller bransch. Azure AD Join fungerar även i en hybrid miljö som möjliggör åtkomst till både molnet och lokala appar och resurser.

|   | Azure AD Join |
| --- | --- |
| **Definition** | Endast ansluten till Azure AD som kräver organisations konto för att logga in på enheten |
| **Primär mål grupp** | Lämpligt för både molnbaserade och hybrid organisationer. |
|   | Gäller för alla användare i en organisation |
| **Äganderätt till enhet** | Organisation |
| **Operativ system** | Alla Windows 10-enheter |
| **Etablering** | Självbetjäning: Windows OOBE eller inställningar |
|   | Massregistrering |
|   | Windows Autopilot |
| **Alternativ för enhets inloggning** | Organisations konton med: |
|   | lösenord |
|   | Windows Hello för företag |
|   | FIDO 2.0-säkerhets nycklar (för hands version) |
| **Enhetshantering** | Hantering av mobila enheter (exempel: Microsoft Intune) |
|   | Samhantering med Microsoft Intune och Microsoft Endpoint Configuration Manager |
| **De viktigaste funktionerna** | Enkel inloggning till både moln resurser och lokala resurser |
|   | Villkorlig åtkomst genom MDM-registrering och MDM-kompatibilitet utvärdering |
|   | Återställning av lösen ord för självbetjäning och Windows Hello-PIN-återställning på Lås skärmen |
|   | Enterprise State Roaming över enheter |

Azure AD-anslutna enheter är inloggade i att använda ett organisations Azure AD-konto. Åtkomst till resurser i organisationen kan ytterligare begränsas baserat på det Azure AD-konto och [principer för villkorlig åtkomst](../conditional-access/overview.md) som tillämpas på enhets identiteten.

Administratörer kan skydda och hantera Azure AD-anslutna enheter med hjälp av MDM-verktyg (Mobile Device Management) som Microsoft Intune eller i samhanterings scenarier med Microsoft Endpoint Configuration Manager. Dessa verktyg ger ett sätt att tvinga fram organisations konfiguration, t. ex. att lagringen ska vara krypterad, lösen ords komplexitet, program varu installationer och program uppdateringar. Administratörer kan göra organisations program tillgängliga för Azure AD-anslutna enheter med hjälp av Configuration Manager för att [Hantera appar från Microsoft Store för företag och utbildning](/configmgr/apps/deploy-use/manage-apps-from-the-windows-store-for-business).

Azure AD Join kan utföras med hjälp av självbetjänings alternativ som OOBE (out of Box Experience), Mass registrering eller [Windows autopilot](/intune/enrollment-autopilot).

Azure AD-anslutna enheter kan fortfarande hantera enkel inloggning till lokala resurser när de befinner sig på organisationens nätverk. Enheter som är Azure AD-anslutna kan fortfarande autentisera till lokala servrar som fil, utskrift och andra program.

## <a name="scenarios"></a>Scenarier

Azure AD Join är främst avsett för organisationer som inte har någon lokal Windows Server Active Directory-infrastruktur, men du kan använda det i scenarier där:

- Du vill gå över till molnbaserad infrastruktur med Azure AD och MDM som Intune.
- Du inte kan använda en lokal domänanslutning, till exempel om du vill hämta mobila enheter som surfplattor och telefoner som kontrolleras.
- Användarna främst behöver åtkomst till Office 365 eller övriga SaaS-appar som är integrerade med Azure AD.
- Du vill hantera en grupp användare i Azure AD istället för Active Directory. Det här scenariot kan gälla, till exempel för säsongs arbetare, entreprenörer eller studenter.
- Du vill erbjuda anslutningsfunktioner för medarbetare på filialkontor med begränsad lokal infrastruktur.

Du kan konfigurera Azure AD-anslutna enheter för Windows 10-enheter.

Målet med Azure AD-anslutna enheter är att förenkla:

- Windows-distributioner av företagsägda enheter
- Åtkomst till företagsappar och -resurser från valfri Windows-enhet
- Molnbaserad hantering av företagsägda enheter
- Användare kan logga in på sina enheter med sina Azure AD-eller Sync-Active Directory arbets-eller skol konton.

![Azure AD-anslutna enheter](./media/concept-azure-ad-join/azure-ad-joined-device.png)

Azure AD Join kan distribueras på något av följande sätt:

- [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot)
- [Massdistribution](/intune/windows-bulk-enroll)
- [Självbetjäning](azuread-joined-devices-frx.md)

## <a name="next-steps"></a>Nästa steg

- [Planera implementeringen av Azure AD-anslutningen](azureadjoin-plan.md)
- [Hantera den lokala gruppen Administratörer på Azure AD-anslutna enheter](assign-local-admin.md)
- [Hantera enhets identiteter med hjälp av Azure Portal](device-management-azure-portal.md)
- [Hantera inaktuella enheter i Azure AD](manage-stale-devices.md)
