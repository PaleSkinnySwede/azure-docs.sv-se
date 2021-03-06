---
title: Konfigurera en Azure Active Directory-hybridanslutning för hanterade domäner | Microsoft Docs
description: Konfigurera Azure Active Directory-hybridanslutning för hanterade domäner.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 03/06/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: bcd00972c2da0d3d5dafe76a8619e0f0ccaedc19
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79239114"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-join-for-managed-domains"></a>Självstudie: Konfigurera Azure Active Directory Join-hybrid för hanterade domäner

I den här självstudien får du lära dig hur du konfigurerar hybrid-Azure Active Directory (Azure AD)-anslutning för Active Directory domänanslutna enheter. Den här metoden har stöd för en hanterad miljö som innehåller både lokala Active Directory och Azure AD.

Som en användare i din organisation är en enhet en kärn identitet som du vill skydda. Du kan använda en enhets identitet för att skydda dina resurser när du vill och var som helst. Du kan uppnå det här målet genom att hantera enhets identiteter i Azure AD. Använd någon av följande metoder:

- Azure Active Directory-anslutning
- Hybrid Azure Active Directory-anslutning
- Azure Active Directory-registrering

Den här artikeln fokuserar på Hybrid Azure AD-anslutning.

Genom att överföra dina enheter till Azure AD kan du maximera användar produktiviteten genom enkel inloggning (SSO) i molnet och lokala resurser. Du kan skydda åtkomsten till molnet och lokala resurser med [villkorlig åtkomst](../active-directory-conditional-access-azure-portal.md) på samma gång.

Du kan distribuera en hanterad miljö med hjälp av [PHS (Password hash Sync)](../hybrid/whatis-phs.md) eller [direktautentisering (PTA)](../hybrid/how-to-connect-pta.md) med [sömlös enkel inloggning](../hybrid/how-to-connect-sso.md). De här scenarierna kräver inte att du konfigurerar en Federations Server för autentisering.

I den här guiden får du lära dig att:

> [!div class="checklist"]
> * Konfigurera Hybrid Azure Active Directory-anslutning
> * Aktivera äldre Windows-enheter
> * Verifiera anslutna enheter
> * Felsöka

## <a name="prerequisites"></a>Krav

- [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) (1.1.819.0 eller senare)
- Autentiseringsuppgifterna för en global administratör för din Azure AD-klient
- Autentiseringsuppgifter för företags administratör för varje skog

Bekanta dig med de här artiklarna:

- [Vad är en enhetsidentitet?](overview.md)
- [Gör så här: planera din hybrid Azure Active Directory delta-implementering](hybrid-azuread-join-plan.md)
- [Kontrollerad verifiering av Azure AD-anslutningshybrid](hybrid-azuread-join-control.md)

> [!NOTE]
> Azure AD stöder inte smartkort eller certifikat i hanterade domäner.

Kontrol lera att Azure AD Connect har synkroniserat dator objekt för de enheter som du vill ska vara hybrid Azure AD-anslutna till Azure AD. Om datorns objekt tillhör vissa organisationsenheter, konfigurerar du organisationsenheterna för synkronisering i Azure AD Connect. Om du vill veta mer om hur du synkroniserar dator objekt med hjälp av Azure AD Connect, se [organisatorisk enhet – baserad filtrering](../hybrid/how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering).

Från och med version 1.1.819.0 innehåller Azure AD Connect en guide för att konfigurera hybrid Azure AD-anslutning. Guiden fören klar konfigurations processen avsevärt. Guiden konfigurerar tjänst anslutnings punkter (SCP: r) för enhets registrering.

Konfigurations stegen i den här artikeln baseras på hur du använder guiden i Azure AD Connect.

Hybrid Azure AD-anslutning kräver att enheter har åtkomst till följande Microsoft-resurser inifrån din organisations nätverk:  

- `https://enterpriseregistration.windows.net`
- `https://login.microsoftonline.com`
- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com`(Om du använder eller planerar att använda sömlös enkel inloggning)

Om din organisation kräver åtkomst till Internet via en utgående proxy rekommenderar vi att du [implementerar Web Proxy Auto-Discovery (WPAD)](https://docs.microsoft.com/previous-versions/tn-archive/cc995261(v%3dtechnet.10)) för att aktivera Windows 10-datorer för enhets registrering med Azure AD. Information om hur du löser problem med att konfigurera och hantera WPAD finns i [Felsöka automatisk identifiering](/previous-versions/tn-archive/cc302643(v=technet.10)).

Om du inte använder WPAD kan du konfigurera proxyinställningar på datorn som börjar med Windows 10 1709. Mer information finns i [Inställningar för WinHTTP-proxy som distribueras av GPO](https://blogs.technet.microsoft.com/netgeeks/2018/06/19/winhttp-proxy-settings-deployed-by-gpo/).

> [!NOTE]
> Om du konfigurerar proxyinställningar på datorn med hjälp av WinHTTP-inställningar kommer alla datorer som inte kan ansluta till den konfigurerade proxyn inte att kunna ansluta till Internet.

Om din organisation kräver åtkomst till Internet via en autentiserad utgående proxy kontrollerar du att dina Windows 10-datorer kan autentiseras mot den utgående proxyn. Eftersom Windows 10-datorer kör enhets registrering med hjälp av dator kontext konfigurerar du utgående proxy-autentisering med hjälp av dator kontext. Kontrollera konfigurationskraven med leverantören av den utgående proxyn.

Kontrol lera att enheten har åtkomst till ovanstående Microsoft-resurser under system kontot med hjälp av anslutnings skriptet för [test av enhets registrering](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0) .

## <a name="configure-hybrid-azure-ad-join"></a>Konfigurera Hybrid Azure Active Directory-anslutning

Konfigurera en hybrid Azure AD-anslutning med hjälp av Azure AD Connect:

1. Starta Azure AD Connect och välj sedan **Konfigurera**.

   ![Välkommen](./media/hybrid-azuread-join-managed-domains/welcome-azure-ad-connect.png)

1. Välj **Konfigurera enhets alternativ**i **Ytterligare aktiviteter**och välj sedan **Nästa**.

   ![Ytterligare uppgifter](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-additional-tasks.png)

1. I **Översikt**väljer du **Nästa**.

   ![Översikt](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-overview.png)

1. I **Anslut till Azure AD**anger du autentiseringsuppgifterna för en global administratör för din Azure AD-klient.  

   ![Anslut till Azure AD](./media/hybrid-azuread-join-managed-domains/connect-to-azure-ad-username-password.png)

1. I **enhets alternativ**väljer du **Konfigurera hybrid Azure AD-anslutning**och väljer sedan **Nästa**.

   ![Enhetsalternativ](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-options.png)

1. I **SCP-konfigurationen**, för varje skog där du vill Azure AD Connect konfigurera SCP, slutför du följande steg och väljer sedan **Nästa**.

   1. Välj en **skog**.
   1. Välj en **autentiseringstjänst**.
   1. Välj **Lägg till** för att ange autentiseringsuppgifter för företags administratören.

   ![SCP](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-scp-configuration.png)

1. I **enhetens operativ system**väljer du de operativ system som enheterna i din Active Directory miljö använder och väljer sedan **Nästa**.

   ![Enhetsoperativsystem](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-device-operating-systems.png)

1. I **redo att konfigurera**väljer du **Konfigurera**.

   ![Klart att konfigurera](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-ready-to-configure.png)

1. I **konfigurationen har slutförts**väljer du **Avsluta**.

   ![Konfigurationen är klar](./media/hybrid-azuread-join-managed-domains/azure-ad-connect-configuration-complete.png)

## <a name="enable-windows-down-level-devices"></a>Aktivera äldre Windows-enheter

Om några av dina domänanslutna enheter är Windows-enheter med äldre versioner måste du:

- Konfigurera inställningarna för det lokala intranätet för enhetsregistrering
- Konfigurera sömlös SSO
- Installera Microsoft Workplace Join för Windows-datorer med äldre versioner

> [!NOTE]
> Support för Windows 7 upphörde 14 januari 2020. Mer information finns i [Support för Windows 7 avslutad](https://support.microsoft.com/help/4057281/windows-7-support-ended-on-january-14-2020).

### <a name="configure-the-local-intranet-settings-for-device-registration"></a>Konfigurera inställningarna för det lokala intranätet för enhetsregistrering

För att slutföra hybrid Azure AD-anslutning av dina Windows-enheter med äldre versioner och för att undvika certifikat meddelanden när enheter autentiserar till Azure AD kan du skicka en princip till dina domänanslutna enheter för att lägga till följande URL: er till zonen Lokalt intranät i Internet Explorer:

- `https://device.login.microsoftonline.com`
- `https://autologon.microsoftazuread-sso.com`

Du måste också aktivera **Tillåt uppdateringar av statusfältet via skript** i användarens lokala intranät zon.

### <a name="configure-seamless-sso"></a>Konfigurera sömlös SSO

För att slutföra hybrid Azure AD-anslutning av dina Windows-enheter med äldre versioner i en hanterad domän som använder [hash-synkronisering av lösen ord](../hybrid/whatis-phs.md) eller [direktautentisering](../hybrid/how-to-connect-pta.md) som din Azure AD Cloud-autentiseringsmetod, måste du också [Konfigurera sömlös SSO](../hybrid/how-to-connect-sso-quick-start.md#step-2-enable-the-feature).

### <a name="install-microsoft-workplace-join-for-windows-down-level-computers"></a>Installera Microsoft Workplace Join för Windows-datorer med äldre versioner

För att registrera Windows-enheter på låg nivå måste organisationerna installera [Microsoft Workplace Join för datorer som inte är Windows 10-datorer](https://www.microsoft.com/download/details.aspx?id=53554). Microsoft Workplace Join för datorer som inte är Windows 10-datorer finns i Microsoft Download Center.

Du kan distribuera paketet med hjälp av ett program distributions system som [Microsoft Endpoint Configuration Manager](/configmgr/). Paketet stöder vanliga obevakade installations alternativ med `quiet` parametern. Den aktuella versionen av Configuration Manager erbjuder förmåner jämfört med tidigare versioner, t. ex. möjligheten att spåra slutförda registreringar.

Installations programmet skapar en schemalagd aktivitet på det system som körs i användar kontexten. Aktiviteten utlöses när användaren loggar in i Windows. Uppgiften ansluter tyst till enheten med Azure AD med hjälp av användarautentiseringsuppgifterna när den har autentiserats med Azure AD.

## <a name="verify-the-registration"></a>Verifiera registreringen

Verifiera enhetens registrerings tillstånd i din Azure-klient med hjälp av **[Get-MsolDevice](/powershell/msonline/v1/get-msoldevice)**. Denna cmdlet finns i [Azure Active Directory PowerShell-modulen](/powershell/azure/install-msonlinev1?view=azureadps-2.0).

När du använder cmdleten **Get-MSolDevice** för att kontrol lera tjänst informationen:

- Det måste finnas ett objekt med det **enhets-ID** som matchar ID: t för Windows-klienten.
- Värdet för **DeviceTrustType** **är domänanslutna**. Den här inställningen motsvarar **hybrid Azure AD-anslutna** tillstånd på sidan **enheter** i Azure AD-portalen.
- För enheter som används i villkorlig åtkomst är värdet för **Enabled** **True** och **DeviceTrustLevel** **hanteras**.

Så här kontrollerar du tjänstinformationen:

1. Öppna Windows PowerShell som administratör.
1. Ange `Connect-MsolService` för att ansluta till din Azure-klient.  
1. Ange `get-msoldevice -deviceId <deviceId>`.
1. Kontrollera att **Aktiverad** är inställd på **SANT**.

## <a name="troubleshoot-your-implementation"></a>Felsöka din implementering

Om du får problem med att slutföra hybrid Azure AD Join för domänanslutna Windows-enheter, se:

- [Felsöka hybrid Azure Active Directory anslutna enheter](troubleshoot-hybrid-join-windows-current.md)
- [Felsöka hybrid Azure Active Directory anslutna enheter med äldre versioner](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Nästa steg

Gå vidare till nästa artikel om du vill lära dig hur du hanterar enhets identiteter med hjälp av Azure Portal.
> [!div class="nextstepaction"]
> [Hantera enhetsidentiteter](device-management-azure-portal.md)
