---
title: Analysera aktivitets loggar med Azure Monitor loggar | Microsoft Docs
description: Lär dig hur du analyserar Azure Active Directory aktivitets loggar med Azure Monitor loggar
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4535ae65-8591-41ba-9a7d-b7f00c574426
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2d6212692465270182db541889bed5f03a08a345
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74008273"
---
# <a name="analyze-azure-ad-activity-logs-with-azure-monitor-logs"></a>Analysera Azure AD-aktivitets loggar med Azure Monitor loggar

När du har [integrerat Azure AD-aktivitets loggar med Azure Monitor loggar](howto-integrate-activity-logs-with-log-analytics.md)kan du använda kraften i Azure Monitor loggar för att få insikter om din miljö. Du kan också installera [Log Analytics-vyer för Azure AD-aktivitets loggar](howto-install-use-log-analytics-views.md) för att få åtkomst till färdiga rapporter som rör gransknings-och inloggnings händelser i din miljö.

I den här artikeln får du lära dig hur du analyserar Azure AD-aktivitets loggarna på din Log Analytics-arbetsyta. 

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Krav 

För att följa med måste du:

* En Log Analytics arbets yta i din Azure-prenumeration. Lär dig hur du [skapar en arbets yta för Log Analytics](https://docs.microsoft.com/azure/log-analytics/log-analytics-quick-create-workspace).
* Slutför först stegen för att [dirigera Azure AD-aktivitets loggarna till din Log Analytics-arbetsyta](howto-integrate-activity-logs-with-log-analytics.md).
*  [Åtkomst](https://docs.microsoft.com/azure/azure-monitor/platform/manage-access#manage-access-using-workspace-permissions) till Log Analytics-arbetsytan
* Följande roller i Azure Active Directory (om du får åtkomst till Log Analytics via Azure Active Directory Portal)
    - Säkerhetsadministratör
    - Säkerhetsläsare
    - Rapport läsare
    - Global administratör
    
## <a name="navigate-to-the-log-analytics-workspace"></a>Navigera till arbets ytan Log Analytics

1. Logga in på [Azure-portalen](https://portal.azure.com). 

2. Välj **Azure Active Directory**och välj sedan **loggar** i avsnittet **övervakning** för att öppna arbets ytan Log Analytics. Arbets ytan öppnas med en standard fråga.

    ![Standard fråga](./media/howto-analyze-activity-logs-log-analytics/defaultquery.png)


## <a name="view-the-schema-for-azure-ad-activity-logs"></a>Visa schemat för Azure AD-aktivitets loggar

Loggarna skickas till tabellerna **AuditLogs** och **SigninLogs** i arbets ytan. Så här visar du schemat för dessa tabeller:

1. Från vyn standard fråga i föregående avsnitt väljer du **schema** och expanderar arbets ytan. 

2. Expandera avsnittet **logg hantering** och expandera sedan antingen **AuditLogs** eller **SignInLogs** för att Visa logg schema.
    ![Logga loggar](./media/howto-analyze-activity-logs-log-analytics/auditlogschema.png) ![för gransknings loggar](./media/howto-analyze-activity-logs-log-analytics/signinlogschema.png)

## <a name="query-the-azure-ad-activity-logs"></a>Fråga Azure AD-aktivitets loggarna

Nu när du har loggarna på arbets ytan kan du nu köra frågor mot dem. Om du till exempel vill hämta de vanligaste programmen som används under den senaste veckan ersätter du standard frågan med följande och väljer **Kör**

```
SigninLogs 
| where CreatedDateTime >= ago(7d)
| summarize signInCount = count() by AppDisplayName 
| sort by signInCount desc 
```

Använd följande fråga för att hämta de viktigaste gransknings händelserna under den senaste veckan:

```
AuditLogs 
| where TimeGenerated >= ago(7d)
| summarize auditCount = count() by OperationName 
| sort by auditCount desc 
```
## <a name="alert-on-azure-ad-activity-log-data"></a>Avisering om aktivitets logg data i Azure AD

Du kan också ställa in aviseringar på din fråga. Om du till exempel vill konfigurera en avisering när fler än 10 program har använts under den senaste veckan:

1. Från arbets ytan väljer du **Ange avisering** för att öppna sidan **Skapa regel** .

    ![Ange avisering](./media/howto-analyze-activity-logs-log-analytics/setalert.png)

2. Välj standard **varnings villkor** som skapats i aviseringen och uppdatera **tröskelvärdet** i standard måttet till 10.

    ![Aviserings villkor](./media/howto-analyze-activity-logs-log-analytics/alertcriteria.png)

3. Ange ett namn och en beskrivning av aviseringen och välj allvarlighets grad. I vårt exempel kunde vi ställa in det till **information**.

4. Välj den **Åtgärds grupp** som ska aviseras när signalen inträffar. Du kan välja att meddela ditt team via e-post eller SMS, eller så kan du automatisera åtgärden med Webhooks, Azure Functions eller Logic Apps. Läs mer om hur [du skapar och hanterar aviserings grupper i Azure Portal](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups).

5. När du har konfigurerat aviseringen väljer du **skapa avisering** för att aktivera den. 

## <a name="install-and-use-pre-built-views-for-azure-ad-activity-logs"></a>Installera och använda förskapade vyer för Azure AD-aktivitets loggar

Du kan också hämta de förinställda vyerna i Log Analytics för Azure AD-aktivitets loggar. Vyerna innehåller flera rapporter relaterade till vanliga scenarier som rör gransknings-och inloggnings händelser. Du kan också varna för någon av de data som anges i rapporterna med hjälp av stegen som beskrivs i föregående avsnitt.

* **Azure AD-kontots etablerings händelser**: i den här vyn visas rapporter om etablerings aktiviteter, till exempel antalet nya användare som har skapats och etablerings felen, antalet användare som har uppdaterats och uppdaterings felen och antalet användare som har upprättat och motsvarande problem.    
* **Inloggnings händelser**: i den här vyn visas de mest relevanta rapporterna om övervakning av inloggnings aktivitet, till exempel inloggningar av program, användare, enhet, samt en sammanfattningsvy som spårar antalet inloggningar över tid.
* **Användare som utför medgivande**: i den här vyn visas rapporter om användar medgivande, t. ex. medgivande som beviljats av användare, inloggningar av användare som har beviljat medgivande samt inloggningar efter program för alla medgivandebaserade program. 

Lär dig att [installera och använda logganalysvyer för Azure AD-aktivitetsloggar](howto-install-use-log-analytics-views.md). 


## <a name="next-steps"></a>Nästa steg

* [Kom igång med frågor i Azure Monitor loggar](https://docs.microsoft.com/azure/log-analytics/query-language/get-started-queries)
* [Skapa och hantera aviserings grupper i Azure Portal](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-action-groups)
* [Installera och Använd Log Analytics-vyerna för Azure Active Directory](howto-install-use-log-analytics-views.md)
