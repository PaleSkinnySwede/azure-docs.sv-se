---
title: Avbilda en webb läsar spårning för fel sökning | Microsoft Docs
description: Samla in nätverks information från en webb läsar spårning för att felsöka problem med Azure Portal.
services: azure-portal
keywords: ''
author: mgblythe
ms.author: mblythe
ms.date: 01/09/2020
ms.topic: troubleshooting
ms.service: azure-portal
manager: mtillman
ms.openlocfilehash: 2b506c9d15dafcd23b24207fe15ed0532939209f
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "76310704"
---
# <a name="capture-a-browser-trace-for-troubleshooting"></a>Samla in en webbläsarspårning för felsökning

Om du felsöker ett problem med Azure Portal och du måste kontakta Microsoft Support rekommenderar vi att du först fångar in en webb läsar spårning och ytterligare information. Informationen du samlar in kan ge viktig information om portalen när problemet uppstår. Följ stegen i den här artikeln för utvecklarverktyg i webbläsaren som du använder: Google Chrome eller Microsoft Edge (krom), Microsoft Edge (EdgeHTML) eller Apple Safari.

## <a name="google-chrome-and-microsoft-edge-chromium"></a>Google Chrome och Microsoft Edge (krom)

Google Chrome och Microsoft Edge (krom) är båda baserade på [projektet krom öppen källkod](https://www.chromium.org/Home). Följande steg visar hur du använder utvecklarverktyg, som liknar varandra i de två webbläsarna. Mer information finns i [DevTools för Chrome](https://developers.google.com/web/tools/chrome-devtools) och [Microsoft Edge (krom) utvecklarverktyg](/microsoft-edge/devtools-guide-chromium).

1. Logga in på [Azure-portalen](https://portal.azure.com). Det är viktigt att logga in _innan_ du börjar spåra så att spårningen inte innehåller känslig information som rör din inloggning. 

1. Börja spela in de steg du tar i portalen med hjälp av [steg registrering](https://support.microsoft.com/help/22878/windows-10-record-steps).

1. I portalen navigerar du till steget precis innan problemet uppstår.

1. Tryck på F12 eller ![Välj skärm bild av ikonen](media/capture-browser-trace/chromium-icon-settings.png) > för webb läsar inställningar**fler verktyg** > **utvecklarverktyg**.

1. Som standard sparas endast spårnings information för den sida som för tillfället är inläst. Ange följande alternativ så att webbläsaren behåller all spårnings information, även om din återskapnings kräver mer än en sida:

    1. Välj fliken **nätverk** och välj sedan **bevara logg**.

          ![Skärm bild av "bevara loggen"](media/capture-browser-trace/chromium-network-preserve-log.png)

    1. Välj konsol **Inställningar**på fliken **konsol** och välj sedan **bevara logg**. Välj **konsol inställningar** igen för att stänga fönstret inställningar.

          ![Skärm bild av "bevara loggen"](media/capture-browser-trace/chromium-console-preserve-log.png)

1. Välj fliken **nätverk** och välj sedan **stoppa registrering av nätverks logg** och **Rensa**.

    ![Skärm bild av "stoppa registrering av nätverks logg" och "Rensa"](media/capture-browser-trace/chromium-stop-clear-session.png)

1. Välj **Registrera nätverks logg**och återskapa sedan problemet i portalen.

    ![Skärm bild av "starta profilerings session"](media/capture-browser-trace/chromium-start-session.png)

    Du kommer att se sessionens utdata som liknar följande bild.

    ![Skärm bild av webb läsar spårnings resultat](media/capture-browser-trace/chromium-browser-trace-results.png)

1. När du har återskapat det oväntade Portal beteendet väljer du **stoppa registrering av nätverks logg**och väljer sedan **Exportera** har och spara filen.

    ![Skärm bild av "export har"](media/capture-browser-trace/chromium-network-export-har.png)

1. Stoppa steg registrering och spara filen.

1. Klicka på fliken **konsol** i fönstret verktyg för utvecklare av webbläsare. Högerklicka och välj sedan **Spara som...** och spara konsolens utdata till en textfil.

    ![Skärm bild av konsolens utdata](media/capture-browser-trace/chromium-console-select.png)

1. Paketera filen, konsolens utdata och skärm inspelningen i ett komprimerat format som. zip och dela med Microsoft support.

## <a name="microsoft-edge-edgehtml"></a>Microsoft Edge (EdgeHTML)

Följande steg visar hur du använder utvecklarverktyg i Microsoft Edge (EdgeHTML). Mer information finns i [Microsoft Edge (EdgeHTML) utvecklarverktyg](/microsoft-edge/devtools-guide).

1. Logga in på [Azure-portalen](https://portal.azure.com). Det är viktigt att logga in _innan_ du börjar spåra så att spårningen inte innehåller känslig information som rör din inloggning. 

1. Börja spela in de steg du tar i portalen med hjälp av [steg registrering](https://support.microsoft.com/help/22878/windows-10-record-steps).

1. I portalen navigerar du till steget precis innan problemet uppstår.

1. Tryck på F12 eller ![Välj skärm bild av ikonen](media/capture-browser-trace/edge-icon-settings.png) > för webb läsar inställningar**fler verktyg** > **utvecklarverktyg**.

1. Som standard sparas endast spårnings information för den sida som för tillfället är inläst. Ange följande alternativ så att webbläsaren behåller all spårnings information, även om din återskapnings kräver mer än en sida:

    1. Välj fliken **nätverk** och avmarkera sedan alternativet **Rensa poster vid navigera**.

          ![Skärm bild av "Rensa poster vid navigering"](media/capture-browser-trace/edge-network-clear-entries.png)

    1. Välj fliken **konsol** och välj sedan **bevara logg**.

          ![Skärm bild av "bevara loggen"](media/capture-browser-trace/edge-console-preserve-log.png)

1. Välj fliken **nätverk** och välj sedan **stoppa profilering** och **ta bort session**.

    ![Skärm bild av "stoppa profilering av sessionen" och "Rensa session"](media/capture-browser-trace/edge-stop-clear-session.png)

1. Välj **Starta profilering av sessionen**och återskapa sedan problemet i portalen.

    ![Skärm bild av "starta profilerings session"](media/capture-browser-trace/edge-start-session.png)

    Du kommer att se sessionens utdata som liknar följande bild.

    ![Skärm bild av webb läsar spårnings resultat](media/capture-browser-trace/edge-browser-trace-results.png)

1. När du har genererat om det oväntade Portal beteendet väljer du **stoppa profilering av session**och väljer sedan **Exportera som** har och sparar filen.

    ![Skärm bild av "Exportera som har"](media/capture-browser-trace/edge-network-export-har.png)

1. Stoppa steg registrering och spara filen.

1. Gå tillbaka i fönstret med webbläsarens utvecklarverktyg, Välj fliken **konsol** och expandera fönstret. Placera markören i början av konsolens utdata och dra och välj hela innehållet i utdata. Högerklicka på och välj sedan **Kopiera**, och spara konsolens utdata till en textfil.

    ![Skärm bild av konsolens utdata](media/capture-browser-trace/edge-console-select.png)

1. Paketera filen, konsolens utdata och skärm inspelningen i ett komprimerat format som. zip och dela med Microsoft support.

## <a name="apple-safari"></a>Apple Safari

Följande steg visar hur du använder utvecklarverktyg i Apple Safari. Mer information finns i [Safari utvecklarverktyg Overview](https://support.apple.com/guide/safari-developer/safari-developer-tools-overview-dev073038698/11.0/mac).

1. Aktivera utvecklarverktyg i Apple Safari:

    1. Välj **Safari**och välj sedan **Inställningar**.

        ![Skärm bild av Safari-inställningar](media/capture-browser-trace/safari-preferences.png)

    1. Välj fliken **Avancerat** och välj sedan **Visa utveckla meny i meny raden**.

        ![Skärm bild av avancerade inställningar för Safari](media/capture-browser-trace/safari-show-develop-menu.png)

1. Logga in på [Azure-portalen](https://portal.azure.com). Det är viktigt att logga in _innan_ du börjar spåra så att spårningen inte innehåller känslig information som rör din inloggning. 

1. Börja spela in de steg du tar i portalen. Mer information finns i [så här registrerar du skärmen på din Mac](https://support.apple.com/HT208721).

1. I portalen navigerar du till steget precis innan problemet uppstår.

1. Välj **utveckla**och välj sedan **Visa webb kontroll**.

    ![Skärm bild av "Visa webb kontroll"](media/capture-browser-trace/safari-show-web-inspector.png)

1. Som standard sparas endast spårnings information för den sida som för tillfället är inläst. Ange följande alternativ så att webbläsaren behåller all spårnings information, även om din återskapnings kräver mer än en sida:

    1. Välj fliken **nätverk** och välj sedan **bevara logg**.

          ![Skärm bild av "bevara loggen"](media/capture-browser-trace/safari-network-preserve-log.png)

    1. Välj fliken **konsol** och välj sedan **bevara logg**.

          ![Skärm bild av "bevara loggen"](media/capture-browser-trace/safari-console-preserve-log.png)

1. Välj fliken **nätverk** och välj sedan **Rensa nätverks objekt**.

    ![Skärm bild av "Rensa nätverks objekt"](media/capture-browser-trace/safari-clear-session.png)

1. Återskapa problemet i portalen. Du kommer att se sessionens utdata som liknar följande bild.

    ![Skärm bild av webb läsar spårnings resultat](media/capture-browser-trace/safari-browser-trace-results.png)

1. När du har genererat om det oväntade Portal beteendet väljer du **Exportera** och sparar filen.

    ![Skärm bild av "export"](media/capture-browser-trace/safari-network-export-har.png)

1. Stoppa skärm inspelaren och spara filen.

1. Gå tillbaka i fönstret med webbläsarens utvecklarverktyg, Välj fliken **konsol** och expandera fönstret. Placera markören i början av konsolens utdata och dra och välj hela innehållet i utdata. Använd kommando-C för att kopiera utdata och spara dem i en textfil.

    ![Skärm bild av konsolens utdata](media/capture-browser-trace/safari-console-select.png)

1. Paketera filen, konsolens utdata och skärm inspelningen i ett komprimerat format som. zip och dela med Microsoft support.

## <a name="next-steps"></a>Nästa steg

[Översikt över Azure Portal](azure-portal-overview.md)