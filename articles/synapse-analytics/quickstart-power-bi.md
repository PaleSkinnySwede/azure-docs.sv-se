---
title: Snabb start – länka en Power BI arbets yta till en Synapse-arbetsyta
description: Länka en Power BI arbets yta till en Azure Synapse Analytics-arbetsyta genom att följa stegen i den här hand boken.
services: synapse-analytics
author: jocaplan
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: ''
ms.date: 04/29/2020
ms.author: jocaplan
ms.reviewer: jrasnick, carlrab
ms.openlocfilehash: 83e9e802d2150ee7f43f01e9f6d6827ef3acfcdd
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2020
ms.locfileid: "83204756"
---
# <a name="quickstart-linking-a-power-bi-workspace-to-a-synapse-analytics-workspace-preview"></a>Snabb start: länka en Power BI arbets yta till en Synapse Analytics-arbetsyta (för hands version)

I den här snabb starten får du lära dig hur du ansluter en Power BI arbets yta till en Synapse Analytics-arbetsyta för att skapa nya Power BI-rapporter och data uppsättningar från Synapse Studio (för hands version).

Om du inte har en Azure-prenumeration kan du [skapa ett kostnads fritt konto innan du börjar](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Krav

- [Skapa en Azure Synapse-arbetsyta och tillhör ande lagrings konto](quickstart-create-workspace.md)
- [En Power BI Professional-eller Premium-arbetsyta](https://docs.microsoft.com/power-bi/service-create-the-new-workspaces)

## <a name="link-power-bi-workspace-to-your-synapse-workspace"></a>Länka Power BI arbets yta till din Synapse-arbetsyta

1. Starta från Synapse Studio och klicka på **Hantera**.

    ![Synapse Studio Klicka på hantera.](media/quickstart-link-powerbi/synapse-studio-click-manage.png)

2. Under **externa anslutningar**klickar du på **länkade tjänster**.

    ![Länkade tjänster är markerade.](media/quickstart-link-powerbi/manage-click-linked-services.png)

3. Klicka på **+ Ny**.

    ![+ Nya länkade tjänster är markerade.](media/quickstart-link-powerbi/new-highlighted.png)

4. Klicka på **Power BI** och klicka på **Fortsätt**.

    ![Visar Power BI länkad tjänst.](media/quickstart-link-powerbi/powerbi-linked-service.png)

5. Ange ett namn för den länkade tjänsten och välj en arbets yta i list rutan.

    ![Visar Power BI länkad tjänst konfiguration.](media/quickstart-link-powerbi/workspace-link-dialog.png)

6. Klicka på **Skapa**.

## <a name="view-power-bi-workspace-in-synapse-studio"></a>Visa Power BI arbets yta i Synapse Studio

När dina arbets ytor är länkade kan du bläddra i Power BI data uppsättningar, redigera/skapa nya Power BI rapporter från Synapse Studio.

1. Klicka på **utveckla**.

    ![Synapse Studio Klicka på utveckla.](media/quickstart-link-powerbi/synapse-studio-click-develop.png)

2. Expandera Power BI och den arbets yta som du vill använda.

    ![Expandera Power BI och arbets ytan.](media/quickstart-link-powerbi/develop-expand-powerbi.png)

Nya rapporter kan skapas genom att klicka **+** längst upp på fliken **utveckla** . du kan redigera befintliga rapporter genom att klicka på rapportens namn. Eventuella sparade ändringar skrivs tillbaka till Power BI arbets ytan.

![Visa och redigera Power BI rapporten.](media/quickstart-link-powerbi/powerbi-report.png)


## <a name="next-steps"></a>Nästa steg

Sedan kan du [skapa Power BIs rapport för filer som lagras på Azure Storage](sql/tutorial-connect-power-bi-desktop.md)
