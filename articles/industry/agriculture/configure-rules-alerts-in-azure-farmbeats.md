---
title: Konfigurera regler och hantera aviseringar
description: Beskriver hur du konfigurerar regler och hanterar aviseringar i FarmBeats
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 02373da114b6dc4663cd3ffc9a0780ac6a7a9e23
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75482990"
---
# <a name="configure-rules-and-manage-alerts"></a>Konfigurera regler och hantera aviseringar

Med Azure FarmBeats kan du skapa regler baserade på affärs logiken, förutom de sensor data som flödar från sensorer och enheter som distribueras i Server gruppen. Reglerna utlöser aviseringar i systemet när sensor värden korsar ett tröskelvärde. Genom att visa och analysera aviseringar som skapats efter tröskelvärdena kan du snabbt agera på eventuella problem och få nödvändiga lösningar.

## <a name="create-rule"></a>Skapa regel

1. På Start sidan går du till **regler**.
2. Välj **ny regel**. Fönstret ny regel visas.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-1.png)

3. Ange **regel namn** och **regel Beskrivning** och välj sedan en Server grupp på list menyn **Välj grupp** .
4. Ange gruppens namn för att välja avsnittet Server grupp och **villkor** visas i samma fönster.  

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-condition-1.png)

5. I **villkor**anger du värden för **mått**, **operator** och **värde**.
6. Skriv mått namnet i den nedrullningsbara menyn **mått** .
7. Välj **+ Lägg till villkor** om du vill lägga till fler villkor i regeln.
8. Välj **allvarlighets grad**.
9. I **åtgärd**, växlar du till knappen för **e-** postaktiverad växling för att aktivera e-postaviseringar.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/new-rule-email-1.png)

10. Ange de **e-postadresser** som du vill skicka e-postaviseringen till, tillsammans med **e-** postmeddelandets ämne och **ytterligare anteckningar**.  
11. I **regel status**växlar du till den **aktiverade** växlings knappen för att aktivera eller inaktivera regeln.
    Du kan visa antalet enheter som kommer att påverkas av regeln.
12. Välj **Använd** för att skapa regeln.

## <a name="view-rule"></a>Visa regel

**Server grupp** sidan visar listan över tillgängliga regler. Välj ett **regel namn**. Ett fönster visar följande information som gäller för den valda regeln:
 - Regelnamn
 - Länk till den server grupp som regeln är kopplad till
 - Skapad datum
 - Datum för senaste uppdatering
 - Allvarlighetsgrad
 - Regel status
 - Lista över villkor  
 - Antal enheter som påverkas av regeln

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/view-rule-1.png)

## <a name="edit-rule"></a>Redigera regel

Följ dessa steg om du vill redigera en regel:

1. På sidan start väljer du **regler** på den vänstra navigerings menyn.
   Fönstret regler visas.
2. Välj den regel som du vill redigera.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-action-bar-1.png)

3. Välj **Redigera** i åtgärds fältet, fönstret **Redigera regel** visas.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-one-1.png)

4. Ändra **regel namnet**och **regel beskrivningen** och välj sedan en Server grupp på list menyn **Välj grupp** .
5. Ange server gruppens namn för att välja Server gruppen och **villkoren** visas i samma fönster.  
6. I **villkor**, redigera **mått**, **operator** och **värde**.
7. Skriv mått namnet i den nedrullningsbara menyn **mått** .
8. Välj **+ Lägg till villkor** för att lägga till/redigera villkor i reglerna.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/edit-rule-two-1.png)

9.  Välj **allvarlighets grad**.  
10. I **åtgärd**, växlar du till knappen för **e-** postaktiverad växling för att aktivera e-postaviseringar.
11. Redigera **e-postadresserna** som du vill skicka e-postaviseringen för, tillsammans med **e-** postmeddelandets ämne och **ytterligare anteckningar**.  
12. I **regel status**växlar du till den **aktiverade** växlings knappen för att aktivera eller inaktivera regeln.
Du kan visa antalet enheter som kommer att påverkas av den här regeln.
13. Välj **Använd** för att redigera regeln.

## <a name="change-rule-status"></a>Ändra regel status

Följ dessa steg om du vill ändra status för en regel:

1. På sidan start väljer du **regler** på den vänstra navigerings menyn. Fönstret regler visas.
2. Välj den regel som du vill ändra status för.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/change-status-rule-action-bar-1.png)

3. Välj **ändra status** i åtgärds fältet. Fönstret **ändra status** visas.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/rule-change-status-1.png)

3. Ändra regel statusen med växlings knappen **ändra status** .
   Du kan visa antalet enheter som kommer att påverkas av regeln.
4. Välj **tillämpa** för att ändra regelns status.

## <a name="delete-rule"></a>Ta bort regel

Följ dessa steg om du vill ta bort en regel:

1. På sidan start väljer du **regler** på den vänstra navigerings menyn. Fönstret regler visas.
2. Välj den regel som du vill ta bort.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/delete-rule-action-bar-1.png)

3. Välj **ta bort** från åtgärds fältet.

    ![Taktslag i projekt grupp](./media/configure-rules-and-alerts-in-azure-farmbeats/delete-rule-1.png)

4. Dialog rutan **ta bort regel** visas. Välj **Ta bort**.
