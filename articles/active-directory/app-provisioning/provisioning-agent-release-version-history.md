---
title: 'Azure AD Connect etablerings agent: versions historik | Microsoft Docs'
description: Den här artikeln innehåller en lista över alla versioner av Azure AD Connect etablerings agenten och beskriver nya funktioner och problem som har åtgärd ATS
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.topic: reference
ms.workload: identity
ms.date: 02/26/2020
ms.subservice: app-provisioning
ms.author: chmutali
ms.reviewer: celested
ms.openlocfilehash: 9a756d276b194aa8d1acb0297a7b4909a8082a84
ms.sourcegitcommit: 3abadafcff7f28a83a3462b7630ee3d1e3189a0e
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82593258"
---
# <a name="azure-ad-connect-provisioning-agent-version-release-history"></a>Azure AD Connect etablerings agent: versions historik

Den här artikeln innehåller versioner och funktioner i Azure Active Directory Connect etablerings agent som har släppts. Azure AD-teamet uppdaterar regelbundet etablerings agenten med nya funktioner och funktioner. Etablerings agenten uppdateras automatiskt när en ny version släpps. 

Microsoft tillhandahåller direkt support för den senaste agent versionen och en version.

## <a name="11960"></a>1.1.96.0

### <a name="release-status"></a>Versionsstatus

4 december 2019: släpps för hämtning

### <a name="new-features-and-improvements"></a>Nya funktioner och förbättringar

* Innehåller stöd för [Azure AD Connect moln etablering](../cloud-provisioning/what-is-cloud-provisioning.md) för att synkronisera användare, kontakt och grupp data från lokala Active Directory till Azure AD


## <a name="11670"></a>1.1.67.0

### <a name="release-status"></a>Versionsstatus

9 september 2019: lanseras för automatisk uppdatering

### <a name="new-features-and-improvements"></a>Nya funktioner och förbättringar

* Möjlighet att konfigurera ytterligare spårning och loggning för fel sökning av etablerings agent problem
* Möjlighet att bara hämta de Azure AD-attribut som konfigureras i mappningen för att förbättra synkroniseringens prestanda

### <a name="fixed-issues"></a>Åtgärdade problem

* Ett fel har åtgärd ATS där agenten övergick i ett tillstånd som inte svarar om det uppstod problem med anslutnings fel i Azure AD
* En bugg har åtgärd ATS som orsakade problem när binära data lästes från Azure Active Directory
* Ett fel har åtgärd ATS där agenten inte kunde förnya förtroende med moln hybrid identitets tjänsten

## <a name="11300"></a>1.1.30.0

### <a name="release-status"></a>Versionsstatus

23 januari 2019: släpps för hämtning

### <a name="new-features-and-improvements"></a>Nya funktioner och förbättringar

* Förbättringar för etablerings agenten och anslutnings arkitekturen ger bättre prestanda, stabilitet och tillförlitlighet 
* Förenklad konfiguration av konfigurations agenten med hjälp av UI-driven installations guide 
* Stöd har lagts till för automatiska agent uppdateringar

