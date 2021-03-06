---
title: Övervaka Azure Event Grid meddelande leverans
description: I den här artikeln beskrivs hur du använder Azure Portal för att se statusen för leverans av Azure Event Grid meddelanden.
services: event-grid
author: spelluru
manager: timlt
ms.service: event-grid
ms.topic: conceptual
ms.date: 01/23/2020
ms.author: spelluru
ms.openlocfilehash: 7a01ab91fe84aaa1fe55018754eddbf8b8f89643
ms.sourcegitcommit: b396c674aa8f66597fa2dd6d6ed200dd7f409915
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2020
ms.locfileid: "82890857"
---
# <a name="monitor-event-grid-message-delivery"></a>Övervaka Event Grid meddelande leverans 

I den här artikeln beskrivs hur du använder portalen för att se statusen för händelse leveranser.

Event Grid tillhandahåller varaktig leverans. Den levererar varje meddelande minst en gång för varje prenumeration. Händelser skickas till den registrerade webhooken för varje prenumeration direkt. Om en webhook inte bekräftar mottagandet av en händelse inom 60 sekunder från det första leverans försöket, Event Grid förnyar leveransen av händelsen.

Om du vill ha information om händelse leverans och försök kan du [Event Grid meddelande leverans och försöka igen](delivery-and-retry.md).

## <a name="delivery-metrics"></a>Leverans mått

Portalen visar mått för statusen för att leverera händelse meddelanden.

Här följer några av måtten för ämnen:

* **Publiceringen lyckades**: händelsen har skickats till ämnet och bearbetats med ett 2xx-svar.
* **Publiceringen misslyckades**: händelsen skickades till ämnet men avvisades med en felkod.
* **Omatchad**: händelsen har publicerats till ämnet, men inte matchad med en händelse prenumeration. Händelsen släpptes.

Här är några av måtten för prenumerationer:

* **Leveransen lyckades**: händelsen har levererats till prenumerationens slut punkt och ett 2xx-svar togs emot.
* **Leveransen misslyckades**: varje gång tjänsten försöker leverera och händelse hanteraren inte returnerar en lyckad 2xx-kod ökar den **misslyckade leverans** räknaren. Om vi försöker leverera samma händelse flera gånger och Miss lyckas, ökar den **misslyckade leverans** räknaren för varje fel.
* **Utgångna händelser**: händelsen levererades inte och alla nya återförsök har skickats. Händelsen släpptes.
* **Matchade händelser**: händelsen i ämnet matchades av händelse prenumerationen.

    > [!NOTE]
    > En fullständig lista över mått finns i [mått som stöds av Azure Event Grid](metrics.md).

## <a name="event-subscription-status"></a>Status för händelse prenumeration

Om du vill se mått för en händelse prenumeration kan du antingen söka efter prenumerations typ eller prenumerationer för en speciell resurs.

Om du vill söka efter händelse prenumerations typ väljer du **alla tjänster**.

![Välj alla tjänster](./media/monitor-event-delivery/all-services.png)

Sök efter **Event Grid** och välj **Event Grid prenumerationer** från de tillgängliga alternativen.

![Sök efter händelse prenumerationer](./media/monitor-event-delivery/search-and-select.png)

Filtrera efter typ av händelse, prenumeration och plats. Välj **mått** för den prenumeration som du vill visa.

![Filtrera händelse prenumerationer](./media/monitor-event-delivery/filter-events.png)

Visa måtten för händelse ämnet och prenumerationen.

![Visa händelse mått](./media/monitor-event-delivery/subscription-metrics.png)

Om du vill hitta måtten för en speciell resurs väljer du den resursen. Välj sedan **händelser**.

![Välj händelser för en resurs](./media/monitor-event-delivery/select-events.png)

Du ser måtten för prenumerationer för resursen.

## <a name="custom-event-status"></a>Status för anpassad händelse

Om du har publicerat ett anpassat ämne kan du visa måtten för det. Välj resurs grupp för ämnet och välj ämnet.

![Välja anpassat ämne](./media/monitor-event-delivery/select-custom-topic.png)

Visa måtten för avsnittet anpassad händelse.

![Visa händelse mått](./media/monitor-event-delivery/custom-topic-metrics.png)

## <a name="set-alerts"></a>Ange aviseringar

Du kan ställa in aviseringar för ämnes-och domän nivå mått för anpassade ämnen och händelse domäner. I översikts bladet för väljer du **aviseringar** från den vänstra resurs menyn för att visa, hantera och skapa aviserings regler. [Läs mer om Azure Monitor aviseringar](../azure-monitor/platform/alerts-overview.md)

![Visa händelse mått](./media/monitor-event-delivery/select-alerts.png)

## <a name="next-steps"></a>Nästa steg

* Om du vill ha information om händelse leverans och försök kan du [Event Grid meddelande leverans och försöka igen](delivery-and-retry.md).
* En introduktion till Event Grid finns i [Om Event Grid](overview.md).
* Information om hur du snabbt kommer igång med Event Grid finns i [skapa och dirigera anpassade händelser med Azure Event Grid](custom-event-quickstart.md).
