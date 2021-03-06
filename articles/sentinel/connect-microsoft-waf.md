---
title: Anslut brand Väggs data för webb program till Azure Sentinel
description: Lär dig hur du ansluter Microsoft webb programs brand Väggs data till Azure Sentinel.
author: yelevin
manager: rkarlin
ms.assetid: bfa2eca4-abdc-49ce-b11a-0ee229770cdd
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.topic: conceptual
ms.date: 09/23/2019
ms.author: yelevin
ms.openlocfilehash: a5cef16694fa2cfae036152d22cfa4473956fc72
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "77588186"
---
# <a name="connect-data-from-microsoft-web-application-firewall"></a>Anslut data från Microsoft webb programs brand vägg



Du kan strömma loggar från Azure Application gatewayens Microsoft Web Application-brandvägg (WAF). Den här WAF skyddar dina program mot vanliga webb sårbarheter som SQL-inmatning och Cross-Site-skript och gör att du kan anpassa regler för att minska antalet falska positiva identifieringar. Följ dessa anvisningar för att strömma dina Microsoft Web Application Firewall-loggar till Azure Sentinel.


## <a name="prerequisites"></a>Krav

- En befintlig Application Gateway-resurs

## <a name="connect-to-microsoft-web-application-firewall"></a>Anslut till Microsoft Web Application-brandvägg

Om du redan har Microsoft webb program brand vägg kontrollerar du att du har en befintlig gateway-resurs.
När din Microsoft Web Application-brandvägg har distribuerats och hämta data kan aviserings data enkelt strömmas till Azure Sentinel.
    
1. I Azure Sentinel-portalen väljer du **data anslutningar**.
1. På sidan data anslutningar väljer du panelen **WAF** .
1. Gå till [Application Gateway resurs](https://ms.portal.azure.com/#blade/HubsExtension/BrowseAllResourcesBlade/resourceType/Microsoft.Network%2FapplicationGateways) och välj din WAF.
    1. Välj **diagnostikinställningar**.
    1. Välj **+ Lägg till diagnostisk inställning** under tabellen.
    1. På sidan **diagnostikinställningar** anger du ett **namn** och väljer **Skicka till Log Analytics**.
    1. Under **Log Analytics arbets yta** väljer du Azure Sentinel-arbetsytan.
    1. Välj de logg typer som du vill analysera. Vi rekommenderar: ApplicationGatewayAccessLog och ApplicationGatewayFirewallLog.
1. Om du vill använda det relevanta schemat i Log Analytics för brand Väggs aviseringar i Microsoft Web Application söker du efter **AzureDiagnostics**.

## <a name="next-steps"></a>Nästa steg
I det här dokumentet har du lärt dig hur du ansluter Microsoft Web Application-brandväggen till Azure Sentinel. Mer information om Azure Sentinel finns i följande artiklar:
- Lär dig hur du [får insyn i dina data och potentiella hot](quickstart-get-visibility.md).
- Kom igång [med att identifiera hot med Azure Sentinel](tutorial-detect-threats-built-in.md).
