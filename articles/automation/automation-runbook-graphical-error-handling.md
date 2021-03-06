---
title: Felhantering i Azure Automation grafiska runbooks
description: Den här artikeln beskriver hur du implementerar felhanteringslogik i Azure Automation grafiska runbooks.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: f1aa605b3e6f32b260ea4a9eee9c056277fcd12d
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79367082"
---
# <a name="error-handling-in-azure-automation-graphical-runbooks"></a>Felhantering i Azure Automation grafiska runbooks

En viktig design princip för din Azure Automation grafiska Runbook är identifieringen av problem som kan uppstå under körningen av runbooken. De här problemen kan omfatta lyckade åtgärder, förväntade feltillstånd och oväntade feltillstånd.

Ofta, om det finns ett icke-avslutande fel som inträffar med en Runbook-aktivitet, hanterar Windows PowerShell aktiviteten genom att bearbeta alla aktiviteter som följer, oavsett fel. Det är troligt att felet genererar ett undantag, men nästa aktivitet kan fortfarande få köras.

Din grafiska Runbook ska innehålla fel hanterings kod för att hantera körnings problem. Om du vill verifiera utdata för en aktivitet eller hantera ett fel kan du använda en PowerShell-kod aktivitet, definiera villkorlig logik för aktivitetens utgående länk eller tillämpa en annan metod.

Grafiska Azure Automation-runbooks har förbättrats och fått en funktion för felhantering. Du kan nu omvandla undantag till icke-avslutande fel och skapa fellänkar mellan aktiviteter. Den förbättrade processen gör det möjligt för din Runbook att fånga fel och hantera realiserade eller oväntade villkor. 

>[!NOTE]
>Den här artikeln har uppdaterats till att använda den nya Azure PowerShell Az-modulen. Du kan fortfarande använda modulen AzureRM som kommer att fortsätta att ta emot felkorrigeringar fram till december 2020 eller längre. Mer information om den nya Az-modulen och AzureRM-kompatibilitet finns i [Introduktion till den nya Azure PowerShell Az-modulen](https://docs.microsoft.com/powershell/azure/new-azureps-module-az?view=azps-3.5.0). Installations anvisningar för AZ-modulen på Hybrid Runbook Worker finns i [installera Azure PowerShell-modulen](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.5.0). För ditt Automation-konto kan du uppdatera dina moduler till den senaste versionen med hjälp av [hur du uppdaterar Azure PowerShell moduler i Azure Automation](automation-update-azure-modules.md).

## <a name="powershell-error-types"></a>PowerShell-feltyper

De typer av PowerShell-fel som kan uppstå under körningen av Runbook avslutar fel och icke-avslutande fel.
 
### <a name="terminating-error"></a>Avslutar fel

Ett avslutande fel är ett allvarligt fel under körningen som avbryter ett kommando-eller skript körning helt. Exempel inkluderar obefintliga cmdlets, syntaxfel som förhindrar att en cmdlet körs och andra allvarliga fel.

### <a name="non-terminating-error"></a>Icke-avslutande fel

Ett icke-avslutande fel är ett icke-allvarligt fel som gör att körningen kan fortsätta trots fel tillståndet. Exempel på detta är drift fel, t. ex. att filen inte hittades fel-och behörighets problem.

## <a name="when-to-use-error-handling"></a>När du ska använda felhantering

Använd fel hantering i din Runbook när en kritisk aktivitet genererar ett fel eller ett undantag. Det är viktigt att förhindra nästa aktivitet i runbooken från att bearbeta och hantera felet på lämpligt sätt. Att hantera felet är särskilt viktigt när dina runbooks stöder en affärs-eller tjänst åtgärds process.

För varje aktivitet som kan producera ett fel kan du lägga till en fellänk som pekar på någon annan aktivitet. Mål aktiviteten kan vara av valfri typ, inklusive kod aktivitet, anrop av en cmdlet, anrop av en annan Runbook och så vidare. Mål aktiviteten kan också ha utgående länkar, antingen vanliga eller fel länkar. Länkarna tillåter att runbooken implementerar komplex fel hanterings logik utan att behöva använda en kod aktivitet.

Den rekommenderade metoden är att skapa en dedikerad fel hanterings-Runbook med vanliga funktioner, men den här metoden är inte obligatorisk. Överväg till exempel en Runbook som försöker starta en virtuell dator och installera ett program på den. Om den virtuella datorn inte startar på rätt sätt:

1. Skickar ett meddelande om det här problemet.
2. Startar en annan Runbook som automatiskt etablerar en ny virtuell dator i stället.

En lösning är att ha en fellänk i Runbook som pekar på en aktivitet som hanterar steg ett. Runbooken kan till exempel ansluta `Write-Warning` cmdleten till en aktivitet för steg två, till exempel cmdleten [Start-AzAutomationRunbook](https://docs.microsoft.com/powershell/module/az.automation/start-azautomationrunbook?view=azps-3.5.0) .

Du kan också generalisera detta beteende för användning i många runbooks genom att placera dessa två aktiviteter i en separat Runbook för fel hantering. Innan den ursprungliga runbooken anropar den här fel hanterings runbooken kan den skapa ett anpassat meddelande från dess data och sedan skicka det som en parameter till den fel hanterings-runbooken.

## <a name="how-to-use-error-handling"></a>Hur du använder felhantering

Varje aktivitet i din Runbook har en konfigurations inställning som inaktiverar undantag till icke-avslutande fel. Som standard är denna inställning inaktiverad. Vi rekommenderar att du aktiverar den här inställningen för alla aktiviteter där Runbook hanterar fel. Den här inställningen säkerställer att Runbook hanterar både avslutande och icke-avslutande fel i aktiviteten som icke-avslutande fel, med hjälp av en fel länk.  

När du har aktiverat konfigurations inställningen måste din Runbook skapa en aktivitet som hanterar felet. Om aktiviteten genererar ett fel följs länkarna för utgående fel. De vanliga länkarna följs inte, även om aktiviteten även genererar vanliga utdata.<br><br> ![Fellänksexempel för Automation Runbook](media/automation-runbook-graphical-error-handling/error-link-example.png)

I följande exempel hämtar en Runbook en variabel som innehåller dator namnet på en virtuell dator. Den försöker sedan starta den virtuella datorn med nästa aktivitet.<br><br> ![Exempel på fel hantering av Automation-Runbook](media/automation-runbook-graphical-error-handling/runbook-example-error-handling.png)<br><br>      

`Get-AutomationVariable` Aktiviteten och cmdleten [Start-AzVM](https://docs.microsoft.com/powershell/module/Az.Compute/Start-AzVM?view=azps-3.5.0) har kon figurer ATS för att konvertera undantag till fel. Om det uppstår problem med att hämta variabeln eller att starta den virtuella datorn genererar koden fel.<br><br> ![Fel hanterings aktivitets inställningar](media/automation-runbook-graphical-error-handling/activity-blade-convertexception-option.png)för Automation Runbook.

Fel länkar flödar från dessa aktiviteter till en `error management` enskild kod aktivitet. Den här aktiviteten har kon figurer ATS med ett enkelt PowerShell- `throw` uttryck som använder nyckelordet för att `$Error.Exception.Message` stoppa bearbetningen, tillsammans med för att hämta det meddelande som beskriver det aktuella undantaget.<br><br> ![Fel hanterings kod exempel för Automation Runbook](media/automation-runbook-graphical-error-handling/runbook-example-error-handling-code.png)

## <a name="next-steps"></a>Nästa steg

* Om du vill veta mer om länkar och länktyper i grafiska runbooks kan du läsa [Graphical authoring in Azure Automation](automation-graphical-authoring-intro.md#links-and-workflow) (Grafisk redigering i Azure Automation).

* Om du vill veta mer om Runbook-körning, övervakning av Runbook-jobb och annan teknisk information, se [Runbook-körning i Azure Automation](automation-runbook-execution.md).