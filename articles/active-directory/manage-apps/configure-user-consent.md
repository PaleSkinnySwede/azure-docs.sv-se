---
title: Konfigurera hur slutanvändare godkänner program med hjälp av Azure AD
description: Lär dig hur du hanterar och när användare kan godkänna program som kommer att ha åtkomst till din organisations data.
services: active-directory
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 10/22/2018
ms.author: mimart
ms.reviewer: arvindh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 42337fe958a881ee263d16c866dda69f13fe09c1
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80519613"
---
# <a name="configure-how-end-users-consent-to-applications"></a>Konfigurera hur slutanvändare godkänner program

Program kan integreras med Microsoft Identity Platform så att användarna kan logga in med sitt arbets-eller skol konto i Azure Active Directory (Azure AD) och få åtkomst till din organisations data för att leverera omfattande data drivna upplevelser. Olika behörigheter tillåter programmet olika åtkomst nivåer till dina användare och din organisations data.

Som standard kan användare godkänna program som har åtkomst till din organisations data, men endast för vissa behörigheter. Som standard kan en användare till exempel godkänna att en app får åtkomst till sin egen post låda eller team-konversationer för ett team som användaren äger, men inte kan godkänna att en app utan obevakad åtkomst kan läsa och skriva till alla SharePoint-webbplatser i din organisation. Samtidigt som användare kan godkänna själva kan användarna enkelt förvärva användbara program som integrerar med Microsoft 365, Azure och andra tjänster, men det kan representera en risk om den inte används och övervakas noggrant.

Microsoft rekommenderar att du inaktiverar framtida användar medgivande åtgärder för att minska din yta och minimera risken. Om användar medgivande är inaktiverat kommer tidigare medgivande bidrag fortfarande att bevaras, men alla framtida medgivande åtgärder måste utföras av en administratör. Administratörs medgivande för hela klienten kan begäras av användare via ett integrerat [administrativt medgivande arbets flöde](configure-admin-consent-workflow.md) eller genom egna support processer. Se [fem steg för att skydda din identitets infrastruktur](../../security/fundamentals/steps-secure-identity.md) för mer information.

## <a name="configure-user-consent-to-applications"></a>Konfigurera användar medgivande till program
### <a name="disable-or-enable-user-consent-from-the-azure-portal"></a>Inaktivera eller aktivera användar medgivande från Azure Portal

Du kan använda Azure Portal för att inaktivera eller aktivera användarnas möjlighet att godkänna program som har åtkomst till din organisations data:

1. Logga in på [Azure Portal](https://portal.azure.com) som [Global administratör](../users-groups-roles/directory-assign-admin-roles.md#global-administrator--company-administrator).
2. Välj **Azure Active Directory**, sedan **företags program**och sedan **användar inställningar**.
3. Aktivera eller inaktivera användar medgivande med kontrollen märkta **användare kan godkänna appar som har åtkomst till företags information för deras räkning**.
4. Valfritt Konfigurera [arbets flödet för administratörs godkännande av begäran](configure-admin-consent-workflow.md) för att se till att användare som inte har tillåtelse att godkänna en app kan begära godkännande.

> [!TIP]
> Om du vill tillåta att användare begär en administratörs granskning av ett program som användaren inte har tillåtelse att godkänna till (till exempel eftersom användar medgivande har inaktiverats eller om programmet begär behörigheter som användaren inte har tillåtelse att bevilja) bör du överväga att [Konfigurera arbets flödet för administratörs medgivande](configure-admin-consent-workflow.md).

### <a name="disable-or-enable-user-consent-using-powershell"></a>Inaktivera eller aktivera användar medgivande med PowerShell

Du kan använda Azure AD PowerShell v1-modulen ([MSOnline](https://docs.microsoft.com/powershell/module/msonline/?view=azureadps-1.0)) för att aktivera eller inaktivera användarnas möjlighet att godkänna program som har åtkomst till organisationens data.

1. Logga in på din organisation genom att köra den här cmdleten:

    ```powershell
    Connect-MsolService
    ```

2. Kontrol lera om användar medgivande har Aktiver ATS genom att köra denna cmdlet:

    ```powershell
    Get-MsolCompanyInformation | Format-List UsersPermissionToUserConsentToAppEnabled
    ```

3. Aktivera eller inaktivera användar medgivande. Om du till exempel vill inaktivera användar medgivande kör du denna cmdlet:

    ```powershell
    Set-MsolCompanySettings -UsersPermissionToUserConsentToAppEnabled $false
    ```

## <a name="configure-group-owner-consent-to-apps-accessing-group-data"></a>Konfigurera grupp ägare medgivande till appar som har åtkomst till grupp data

> [!IMPORTANT]
> Följande information gäller för en kommande funktion som gör det möjligt för grupp ägare att ge program åtkomst till sina gruppers data. När den här funktionen släpps, aktive ras den som standard. Även om den här funktionen ännu inte har släppts mycket, kan du använda dessa instruktioner för att inaktivera funktionen i förväg.

Grupp ägare kan auktorisera program (till exempel program som publicerats av tredjepartsleverantörer) för att komma åt din organisations data som är associerade med en grupp. Till exempel kan en grupp ägare (som är ägare av Office 365-gruppen för teamet) tillåta att en app läser alla team meddelanden i teamet eller visar den grundläggande profilen för en grupps medlemmar.

> [!NOTE]
> Oberoende av den här inställningen tillåts en grupp ägare alltid att lägga till andra användare eller appar direkt som grupp ägare.

### <a name="configure-group-owner-consent-using-powershell"></a>Konfigurera grupp ägarens medgivande med PowerShell

Du kan använda Azure AD PowerShell Preview-modulen ([AzureADPreview](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview)) för att aktivera eller inaktivera grupp ägarens möjlighet att godkänna program som har åtkomst till organisationens data för de grupper som de äger.

1. Se till att du använder modulen [AzureADPreview](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview) (det här steget är viktigt om du har installerat både [AzureAD](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0) -modulen och [AzureADPreview](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview) -modulen).

    ```powershell
    Remove-Module AzureAD
    Import-Module AzureADPreview
    ```

2. Anslut till Azure AD PowerShell.

   ```powershell
   Connect-AzureAD
   ```

3. Hämta det aktuella värdet för katalog inställningarna för *medgivande princip inställningarna* i din klient organisation. Detta kräver att du kontrollerar om katalog inställningarna för den här funktionen har skapats, och om inte använder värdena från motsvarande katalog inställnings mall.

    ```powershell
    $consentSettingsTemplateId = "dffd5d46-495d-40a9-8e21-954ff55e198a" # Consent Policy Settings
    $settings = Get-AzureADDirectorySetting -All $true | Where-Object { $_.TemplateId -eq $consentSettingsTemplateId }

    if (-not $settings) {
        $template = Get-AzureADDirectorySettingTemplate -Id $consentSettingsTemplateId
        $settings = $template.CreateDirectorySetting()
    }

    $enabledValue = $settings.Values | ? { $_.Name -eq "EnableGroupSpecificConsent" }
    $limitedToValue = $settings.Values | ? { $_.Name -eq "ConstrainGroupSpecificConsentToMembersOfGroupId" }
    ```

4. Förstå inställnings värden. Det finns två inställnings värden som definierar vilka användare som ska kunna tillåta en app åtkomst till deras grupp data:

    | Inställning       | Typ         | Beskrivning  |
    | ------------- | ------------ | ------------ |
    | _EnableGroupSpecificConsent_   | Boolesk |  Flagga som anger om grupp ägare tillåts att bevilja gruppdefinierade behörigheter. |
    | _ConstrainGroupSpecificConsentToMembersOfGroupId_ | GUID | Om _EnableGroupSpecificConsent_ är inställt på "true" och det här värdet har angetts till en grupps objekt-ID, kommer medlemmar i gruppen som identifieras att kunna bevilja gruppspecifika behörigheter till de grupper som de äger. |

5. Uppdatera inställnings värden för önskad konfiguration:

    ```powershell
    # Disable group-specific consent entirely
    $enabledValue.Value = "False"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for all users
    $enabledValue.Value = "True"
    $limitedToValue.Value = ""
    ```

    ```powershell
    # Enable group-specific consent for users in a given group
    $enabledValue.Value = "True"
    $limitedToValue.Value = "{group-object-id}"
    ```

6. Spara inställningar.

    ```powershell
    if ($settings.Id) {
        # Update an existing directory settings
        Set-AzureADDirectorySetting -Id $settings.Id -DirectorySetting $settings
    } else {
        # Create a new directory settings to override the default setting 
        New-AzureADDirectorySetting -DirectorySetting $settings
    }
    ```

## <a name="configure-risk-based-step-up-consent"></a>Konfigurera riskfylldt steg-för-steg-godkännande

Riskfylldt steg-för-steg-godkännande bidrar till att minska användar exponeringen för skadliga appar som gör [illegala medgivande begär Anden](https://docs.microsoft.com/microsoft-365/security/office-365-security/detect-and-remediate-illicit-consent-grants). Om Microsoft identifierar en riskfylld begäran om godkännande av slutanvändare kräver begäran ett "steg-för-steg" för administratörs medgivande i stället. Den här funktionen är aktive rad som standard, men den kommer bara att resultera i en funktion som ändras när slutanvändaren har Aktiver ATS.

När en riskfylld begäran identifieras visar medgivande frågan ett meddelande som anger att administratörs godkännande krävs. Om [arbets flödet för administratörs medgivande](configure-admin-consent-workflow.md) är aktiverat kan användaren skicka begäran till en administratör för ytterligare granskning direkt från medgivande frågan. Om den inte är aktive rad visas följande meddelande:

* **AADSTS90094:** &lt;clientAppDisplayName&gt; behöver behörighet för att få åtkomst till resurser i din organisation som bara en administratör kan bevilja. Be en administratör att bevilja behörighet till den här appen innan du använder den.

I det här fallet loggas även en gransknings händelse med en kategori av typen "ApplicationManagement", aktivitets typen "medgivande till program" och status orsaken "riskfylld program upptäcktes".

> [!IMPORTANT]
> Administratörer bör [utvärdera alla medgivande begär Anden](manage-consent-requests.md#evaluating-a-request-for-tenant-wide-admin-consent) noggrant innan de godkänns, särskilt när Microsoft har identifierat risk.

### <a name="disable-or-re-enable-risk-based-step-up-consent-using-powershell"></a>Inaktivera eller återaktivera riskfylldt Step-based-godkännande med PowerShell

Du kan använda Azure AD PowerShell Preview-modulen ([AzureADPreview](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0-preview)) för att inaktivera steget-upp till administratörs medgivande som krävs i de fall där Microsoft identifierar risker eller aktiverar det igen om det tidigare har inaktiverats.

Detta kan göras med hjälp av samma steg som ovan för att [Konfigurera grupp ägarens medgivande med hjälp av PowerShell](#configure-group-owner-consent-using-powershell), men som ersätter ett annat inställnings värde. Det finns tre skillnader i stegen: 

1. Förstå inställnings värden för riskfylldt steg-för-steg-godkännande:

    | Inställning       | Typ         | Beskrivning  |
    | ------------- | ------------ | ------------ |
    | _BlockUserConsentForRiskyApps_   | Boolesk |  Flagga som anger om användar medgivande ska blockeras när en riskfylld begäran identifieras. |

2. Ersätt följande värde i steg 3:

    ```powershell
    $riskBasedConsentEnabledValue = $settings.Values | ? { $_.Name -eq "BlockUserConsentForRiskyApps" }
    ```
3. Ersätt något av följande i steg 5:

    ```powershell
    # Disable risk-based step-up consent entirely
    $riskBasedConsentEnabledValue.Value = "False"
    ```

    ```powershell
    # Re-enable risk-based step-up consent, if disabled previously
    $riskBasedConsentEnabledValue.Value = "True"
    ```

## <a name="next-steps"></a>Nästa steg

[Konfigurera arbets flödet för administratörs medgivande](configure-admin-consent-workflow.md)

[Lär dig hur du hanterar medgivande till program och att utvärdera medgivande begär Anden](manage-consent-requests.md)

[Bevilja ett program administratörs medgivande för hela klienten](grant-admin-consent.md)

[Behörigheter och medgivande i Microsoft Identity Platform](../develop/active-directory-v2-scopes.md)

[Azure AD på StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)
