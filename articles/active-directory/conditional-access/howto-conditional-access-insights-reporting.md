---
title: Arbets bok för villkorlig åtkomst och rapportering – Azure Active Directory
description: Använda Azure ADs arbets bok för villkorlig åtkomst och rapportering för att felsöka principer
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: article
ms.date: 05/01/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: d4113328a5de02c36b7285c837bd5314d11e526b
ms.sourcegitcommit: 366e95d58d5311ca4b62e6d0b2b47549e06a0d6d
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/01/2020
ms.locfileid: "82690421"
---
# <a name="conditional-access-insights-and-reporting"></a>Villkorlig åtkomst insikter och rapportering

Med arbets boken för villkorlig åtkomst i insikter och rapporter kan du förstå hur principer för villkorlig åtkomst fungerar i organisationen över tid. Under inloggningen kan en eller flera principer för villkorlig åtkomst tillämpas, bevilja åtkomst om vissa beviljande kontroller är uppfyllda eller nekar åtkomst i övrigt. Eftersom flera principer för villkorlig åtkomst kan utvärderas under varje inloggning kan du använda arbets boken insikter och rapportering för att undersöka effekten av en enskild princip eller en delmängd av alla principer.  

## <a name="prerequisites"></a>Krav

Om du vill aktivera arbets boken insikter och rapportering måste klienten ha en Log Analytics arbets yta för att behålla inloggnings loggar data. Användare måste ha Azure AD Premium P1-eller P2-licenser för att kunna använda villkorlig åtkomst.

Följande roller kan komma åt insikter och rapportering:  

- Administratör för villkorlig åtkomst 
- Säkerhetsläsare 
- Säkerhetsadministratör 
- Global läsare 
- Global administratör 

Användare behöver också en av följande Log Analytics arbets ytans roller:  

- Läsare 
- Övervaknings läsare 
- Log Analytics läsare 
- Deltagare  
- Ägare 

### <a name="stream-sign-in-logs-from-azure-ad-to-azure-monitor-logs"></a>Strömma inloggnings loggar från Azure AD till Azure Monitor loggar 

Om du inte har integrerat Azure AD-loggar med Azure Monitor loggar måste du utföra följande steg innan arbets boken kommer att läsas in:  

1. [Skapa en Log Analytics arbets yta i Azure Monitor](../../azure-monitor/learn/quick-create-workspace.md).
1. [Integrera Azure AD-loggar med Azure Monitor loggar](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md).

## <a name="how-it-works"></a>Så här fungerar det 

För att komma åt arbets boken insikter och rapportering:  

1. Logga in på **Azure-portalen**.
1. Bläddra till **Azure Active Directory** > **säkerhet** > för**villkorlig åtkomst** > **och rapportering**.

### <a name="get-started-select-parameters"></a>Kom igång: Välj parametrar 

På instrument panelen insikter och rapporter kan du se effekten av en eller flera principer för villkorlig åtkomst under en angiven period. Börja med att ställa in var och en av parametrarna överst i arbets boken. 

![Instrument panelen för villkorlig åtkomst insikter och rapportering i Azure Portal](./media/howto-conditional-access-insights-reporting/conditional-access-insights-and-reporting-dashboard.png)

**Princip för villkorlig åtkomst**: Välj en eller flera principer för villkorlig åtkomst för att visa deras kombinerade påverkan. Principer är indelade i två grupper: aktiverade och endast för rapport-principer. Som standard är alla aktiverade principer markerade. De här aktiverade principerna är de principer som för närvarande används i din klient organisation.  

**Tidsintervall**: Välj ett tidsintervall från 4 timmar till så långt tillbaka som 90 dagar. Om du väljer ett tidsintervall ytterligare tillbaka än när du integrerar Azure AD-loggarna med Azure Monitor, visas bara inloggningar efter tidpunkten för integreringen.  

**Användare**: instrument panelen visar som standard effekten av de valda principerna för alla användare. Om du vill filtrera efter en enskild användare skriver du namnet på användaren i textfältet. Om du vill filtrera efter alla användare skriver du "alla användare" i textfältet eller låter parametern vara tom. 

**App**: som standard visar instrument panelen effekten av de valda principerna för alla appar. Om du vill filtrera efter en enskild app skriver du namnet på appen i textfältet. Om du vill filtrera efter alla appar skriver du "alla appar" i textfältet eller låter parametern vara tom. 

**Datavy**: Välj om du vill att instrument panelen ska visa resultat avseende antalet användare eller antal inloggningar. En enskild användare kan ha hundratals inloggnings program till många appar med många olika resultat under ett visst tidsintervall. Om du väljer datavyn som användare, kan en användare inkluderas i både lyckade och misslyckade räkningar (till exempel om det finns 10 användare kan 8 av dem ha haft ett resultat av framgång under de senaste 30 dagarna och 9 av dem kunde ha haft ett resultat av ett haveri under de senaste 30 dagarna).

## <a name="impact-summary"></a>Effekt Sammanfattning 

När du har angett parametrarna läses effekt sammanfattningen in. Sammanfattningen visar hur många användare eller inloggningar under tidsintervallet som resulterade i "lyckad", "Misslyckad", "användar åtgärd krävs" eller "används inte" när de valda principerna utvärderades.  

![Effekt Sammanfattning i arbets boken för villkorlig åtkomst](./media/howto-conditional-access-insights-reporting/workbook-impact-summary.png)

**Totalt**: antalet användare eller inloggningar under den tids period då minst en av de valda principerna utvärderades.

**Lyckades**: antalet användare eller inloggningar under den tids period då det kombinerade resultatet av de valda principerna var "lyckades" eller "endast rapporter: lyckades".

**Misslyckades**: antalet användare eller inloggningar under den tids period då resultatet av minst en av de valda principerna var "misslyckades" eller "endast rapporter: misslyckades".

**Användar åtgärd krävs**: antalet användare eller inloggningar under den tids period då det kombinerade resultatet av de valda principerna var "endast rapporter: användar åtgärd krävs". Användar åtgärd krävs när en interaktiv beviljande kontroll, till exempel Multi-Factor Authentication, krävs av en princip för villkorlig åtkomst för endast rapporter. Eftersom interaktiva beviljade kontroller inte tillämpas av endast rapport principer, går det inte att fastställa lyckade eller misslyckade åtgärder.  

**Används inte**: antalet användare eller inloggningar under den tids period då ingen av de valda principerna tillämpades.

### <a name="understanding-the-impact"></a>Förstå effekten 

![Arbets boks uppdelning per villkor och status](./media/howto-conditional-access-insights-reporting/workbook-breakdown-condition-and-status.png)

Visa nedbrytningen av användare eller inloggningar för varje villkor. Du kan filtrera inloggnings tilläggen för ett visst resultat (till exempel lyckat eller misslyckat) genom att välja på sammanfattnings panelerna överst i arbets boken. Du kan se fördelningen av inloggningar för var och en av de villkorliga åtkomst villkoren: enhets tillstånd, enhets plattform, klient program, plats, program och inloggnings risk.  

## <a name="sign-in-details"></a>Inloggnings information 

![Inloggnings information för arbets bok](./media/howto-conditional-access-insights-reporting/workbook-sign-in-details.png)

Du kan också undersöka inloggnings tilläggen för en speciell användare genom att söka efter inloggningar längst ned i instrument panelen. Frågan till vänster visar de vanligaste användarna. Om du väljer en användare filtreras frågan till höger.  

## <a name="troubleshooting"></a>Felsökning

### <a name="why-is-the-workbook-taking-a-long-time-to-load"></a>Varför tar det lång tid att läsa in arbets boken?  

Beroende på vilket tidsintervall som valts och storleken på din klient, kan arbets boken utvärdera ett ovanligt stort antal inloggnings händelser. För stora klienter kan det hända att volymen av inloggningar överskrider Log Analyticss kapacitet. Försök att förkorta tidsintervallet till 4 timmar för att se om arbets boken läses in.  

### <a name="after-loading-for-a-few-minutes-why-is-the-workbook-returning-zero-results"></a>När du har läst in några minuter, varför returnerar arbets boken noll resultat? 

Om volymerna för inloggningar överskrider fråge kapaciteten för Log Analytics kommer arbets boken returnera noll resultat. Försök att förkorta tidsintervallet till 4 timmar för att se om arbets boken läses in.  

### <a name="can-i-save-my-parameter-selections"></a>Kan jag spara mina parameter val?  

Du kan spara dina parameter val längst upp i arbets boken genom att gå till **Azure Active Directory** > **arbets böcker** > **villkorlig åtkomst insikter och rapportering**. Här hittar du arbets boks mal len där du kan redigera arbets boken och spara en kopia på din arbets yta, inklusive parameter valen, i **Mina rapporter** eller **delade rapporter**. 

### <a name="can-i-edit-and-customize-the-workbook-with-additional-queries"></a>Kan jag redigera och anpassa arbets boken med fler frågor? 

Du kan redigera och anpassa arbets boken genom att gå till **Azure Active Directory** > **arbets böcker** > **villkorlig åtkomst insikter och rapportering**. Här hittar du arbets boks mal len där du kan redigera arbets boken och spara en kopia på din arbets yta, inklusive parameter valen, i **Mina rapporter** eller **delade rapporter**. Om du vill börja redigera frågorna klickar du på **Redigera** överst i arbets boken.  
 
## <a name="next-steps"></a>Nästa steg

[Endast rapport läge för villkorlig åtkomst](concept-conditional-access-report-only.md)
