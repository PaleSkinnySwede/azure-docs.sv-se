---
title: Översikt över fel sökning av Windows Virtual Desktop – Azure
description: En översikt för fel sökning av problem när du konfigurerar en Windows-miljö för virtuella Skriv bords klienter.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: troubleshooting
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 196c4f692b425d2fc6592888ba89f2fa6dafd3de
ms.sourcegitcommit: 50ef5c2798da04cf746181fbfa3253fca366feaa
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82612511"
---
# <a name="troubleshooting-overview-feedback-and-support"></a>Översikt, feedback och support för felsökning

>[!IMPORTANT]
>Det här innehållet gäller för våren 2020-uppdateringen med Azure Resource Manager virtuella Windows Desktop-objekt. Om du använder den virtuella Windows-datorn med version 2019 utan Azure Resource Manager objekt, se [den här artikeln](./virtual-desktop-fall-2019/troubleshoot-set-up-overview-2019.md).
>
> Den virtuella Windows-skrivbordets våren 2020-uppdateringen är för närvarande en offentlig för hands version. Den här för hands versionen tillhandahålls utan service nivå avtal och vi rekommenderar inte att du använder den för produktions arbets belastningar. Vissa funktioner kanske inte stöds eller kan vara begränsade. 
> Mer information finns i [Kompletterande villkor för användning av Microsoft Azure-förhandsversioner](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Den här artikeln innehåller en översikt över de problem som du kan stöta på när du konfigurerar en Windows-klient för virtuella skriv bord och ger lösningar på problemen.

## <a name="report-issues-during-public-preview"></a>Rapportera problem under en offentlig för hands version

Om du vill rapportera problem eller föreslå funktioner under en offentlig för hands version av våren 2020-versionen går du till [Windows Virtual Desktop-Tech-communityn](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop). Du kan använda Tech-communityn för att diskutera bästa praxis eller föreslå och rösta på nya funktioner. När du rapporterar ett problem som rör den offentliga för hands versionen ska du se till att märka problem typen som **våren 2020 Update (för hands version)**.

När du gör ett inlägg som ber om hjälp eller föreslår en ny funktion ska du se till att du beskriver ditt ämne i så mycket information som möjligt. Detaljerad information kan hjälpa andra användare att besvara din fråga eller förstå den funktion som du föreslår en röst för.

## <a name="escalation-tracks"></a>Eskalerade spår

Innan du gör något annat måste du kontrol lera [sidan Azure-status](https://status.azure.com/status) och [Azure Service Health](https://azure.microsoft.com/features/service-health/) för att kontrol lera att Azure-tjänsten körs på rätt sätt.

Använd följande tabell för att identifiera och lösa problem som kan uppstå när du konfigurerar en klient miljö med hjälp av fjärr skrivbords klienten. När din klient organisation har konfigurerat kan du använda vår nya [diagnostik-tjänst](diagnostics-role-service.md) för att identifiera problem med vanliga scenarier.

| **Problem**                                                            | **Föreslagen lösning**  |
|----------------------------------------------------------------------|-------------------------------------------------|
| Åtkomst till Marketplace-mallar i Azure Portal       | Windows Virtual Desktop-mallar för Azure Marketplace är tillgängliga på ett kostnads fritt.|
| Session Host pool Azure-Virtual Network (VNET) och Express Route-inställningar               | [Öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/)och välj lämplig tjänst (under kategorin nätverk). |
| Skapa virtuell dator för Session Host pool (VM) när Azure Resource Manager mallar som medföljer Windows Virtual Desktop används inte | [Öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/)och välj sedan **virtuell dator som kör Windows** för tjänsten. <br> <br> Om du har problem med Azure Resource Manager mallar som medföljer Windows Virtual Desktop kan du läsa avsnittet Skapa Windows-klient för virtuella skriv bord i en [miljö och skapa en värd pool](troubleshoot-set-up-issues.md). |
| Hantera Windows virtuell skrivbord värd miljö från Azure Portal    | [Öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/). <br> <br> För hanterings problem när du använder Fjärrskrivbordstjänster/Windows Virtual Desktop PowerShell, se [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md) eller [öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/), Välj **Windows Virtual Desktop** för tjänsten, Välj **konfiguration och hantering** för problem typen och välj sedan **problem med att konfigurera klient med PowerShell** som underordnat problem. |
| Hantera konfiguration av virtuella Windows-skrivbordet knutna till värdar och program grupper (app-grupper)      | Se [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md)eller [öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/), Välj **Windows Virtual Desktop** för tjänsten och välj sedan lämplig problem typ.|
| Distribuera och hantera FSLogix profil behållare | Se [fel söknings guide för FSLogix-produkter](/fslogix/fslogix-trouble-shooting-ht/) och om det inte löser [problemet öppnar du en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/), väljer **Windows Virtual Desktop** för tjänsten, väljer **FSLogix** för problem typen och väljer sedan motsvarande typ av problem. |
| Fjärr skrivbords klienter fungerar inte på Start                                                 | Se [Felsöka fjärr skrivbords klienten](troubleshoot-client.md) och om det inte löser problemet [öppnar du en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/), väljer **Windows Virtual Desktop** för tjänsten och väljer sedan **fjärr skrivbords klienter** för problem typen.  <br> <br> Om det är ett nätverks problem måste användarna kontakta sin nätverks administratör. |
| Ansluten men ingen feed                                                                 | Felsök med hjälp av [användaren ansluter men ingenting visas (ingen feed)](troubleshoot-service-connection.md#user-connects-but-nothing-is-displayed-no-feed) i [Windows Virtual Desktop Service-anslutningar](troubleshoot-service-connection.md). <br> <br> Om dina användare har tilldelats en app-grupp öppnar du [en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/), väljer **Windows Virtual Desktop** för tjänsten och väljer sedan **fjärr skrivbords klienter** för problem typen. |
| Problem med identifiering av feed på grund av nätverket                                            | Användarna måste kontakta sin nätverks administratör. |
| Ansluta klienter                                                                    | Se [Windows Virtual Desktop Service-anslutningar](troubleshoot-service-connection.md) och om det inte löser problemet kan du läsa [konfiguration av virtuell dator i Session Host](troubleshoot-vm-configuration.md). |
| Svars tider för fjärrprogram eller skriv bord                                      | Om problemen är knutna till en specifik applikation eller produkt kontaktar du det team som ansvarar för produkten. |
| Licensierings meddelanden eller fel                                                          | Om problemen är knutna till en specifik applikation eller produkt kontaktar du det team som ansvarar för produkten. |
| Problem med autentiseringsmetoder från tredje part | Kontrol lera att din tredjeparts leverantör stöder scenarier för virtuella Windows-datorer och se om det finns några kända problem. |
| Problem med att använda Log Analytics för virtuella Windows-datorer | [Öppna en support förfrågan för Azure](https://azure.microsoft.com/support/create-ticket/)om du har problem med det diagnostiska schemat.<br><br>För frågor, visualisering eller andra problem i Log Analytics väljer du lämplig problem typ under Log Analytics. |

## <a name="next-steps"></a>Nästa steg

- Information om hur du felsöker problem när du skapar en adresspool i en Windows Virtual Desktop-miljö finns i avsnittet om att [skapa en miljö och en värddator](troubleshoot-set-up-issues.md).
- Information om hur du felsöker problem när du konfigurerar en virtuell dator (VM) i Windows Virtual Desktop finns i [konfiguration av Session Host-dator](troubleshoot-vm-configuration.md).
- Information om hur du felsöker problem med klient anslutningar för virtuella Windows-datorer finns i [Windows Virtual Desktop Service Connections](troubleshoot-service-connection.md).
- Information om hur du felsöker problem med fjärr skrivbords klienter finns i [Felsöka fjärr skrivbords klienten](troubleshoot-client.md)
- Information om hur du felsöker problem när du använder PowerShell med Windows Virtual Desktop finns i [Windows Virtual Desktop PowerShell](troubleshoot-powershell.md).
- Mer information om tjänsten finns i [Windows Virtual Desktop-miljö](environment-setup.md).
- Information om hur du går igenom en fel söknings kurs finns i [Självstudier: Felsöka distributioner av Resource Manager-mallar](../azure-resource-manager/templates/template-tutorial-troubleshoot.md).
- Mer information om gransknings åtgärder finns i [gransknings åtgärder med Resource Manager](../azure-resource-manager/management/view-activity-logs.md).
- Information om åtgärder för att fastställa fel under distributionen finns i [Visa distributions åtgärder](../azure-resource-manager/templates/deployment-history.md).
