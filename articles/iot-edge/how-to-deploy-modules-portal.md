---
title: Distribuera moduler från Azure Portal-Azure IoT Edge
description: Använd din IoT Hub i Azure Portal för att skicka en IoT Edge-modul från din IoT Hub till din IoT Edge-enhet enligt konfigurationen i ett distributions manifest.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 12/30/2019
ms.topic: conceptual
ms.reviewer: menchi
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 754c106db42f3f0695ad023e736993bee82e9757
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82133926"
---
# <a name="deploy-azure-iot-edge-modules-from-the-azure-portal"></a>Distribuera Azure IoT Edge moduler från Azure Portal

När du skapar IoT Edge-moduler med din affärs logik vill du distribuera dem till dina enheter så att de körs på gränsen. Om du har flera moduler som arbetar tillsammans för att samla in och bearbeta data kan du distribuera dem samtidigt och deklarera de routningsregler som ansluter dem.

Den här artikeln visar hur Azure Portal vägleder dig genom att skapa ett distributions manifest och distribuera distributionen till en IoT Edge enhet. Information om hur du skapar en distribution som riktar sig till flera enheter baserat på deras delade taggar finns i [distribuera och övervaka IoT Edge moduler i stor skala](how-to-deploy-at-scale.md).

## <a name="prerequisites"></a>Krav

* En [IoT-hubb](../iot-hub/iot-hub-create-through-portal.md) i din Azure-prenumeration.
* En [IoT Edge-enhet](how-to-register-device.md#register-in-the-azure-portal) med IoT Edge runtime installerad.

## <a name="configure-a-deployment-manifest"></a>Konfigurera ett distributions manifest

Ett distributions manifest är ett JSON-dokument som beskriver vilka moduler som ska distribueras, hur data flödar mellan moduler och önskade egenskaper för modulen. Mer information om hur distributions manifest fungerar och hur du skapar dem finns i [förstå hur IoT Edge moduler kan användas, konfigureras och återanvändas](module-composition.md).

Azure Portal har en guide som vägleder dig genom att skapa distributions manifestet, i stället för att skapa JSON-dokumentet manuellt. Det finns tre steg: **Lägg till moduler**, **Ange vägar**och **Granska distribution**.

### <a name="select-device-and-add-modules"></a>Välj enhet och Lägg till moduler

1. Logga in på [Azure Portal](https://portal.azure.com) och navigera till din IoT-hubb.
1. Välj **IoT Edge** på menyn i den vänstra rutan.
1. Klicka på mål enhetens ID i listan över enheter.
1. I den övre stapeln väljer du **Ange moduler**.
1. I avsnittet **container Registry inställningar** på sidan anger du autentiseringsuppgifterna för åtkomst till alla privata behållar register som innehåller dina modulblad.
1. I avsnittet **IoT Edge moduler** på sidan väljer du **Lägg till**.
1. Titta på olika typer av moduler på den nedrullningsbara menyn:

   * **IoT Edge modul** – du anger modulens namn och URI för behållar avbildningen. Till exempel är `mcr.microsoft.com/azureiotedge-simulated-temperature-sensor:1.0`avbildnings-URI: n för exempel SimulatedTemperatureSensor-modulen. Om modulens avbildning lagras i ett privat behållar register, lägger du till autentiseringsuppgifterna på den här sidan för att få åtkomst till avbildningen.
   * **Marketplace-modul** – moduler som finns på Azure Marketplace. Vissa Marketplace-moduler kräver ytterligare konfiguration, så granska informationen i modulen i listan med [IoT Edge moduler för Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) .
   * **Azure Stream Analytics modul** – moduler som genereras från en Azure Stream Analytics arbets belastning.

1. När du har lagt till en modul väljer du modulnamnet i listan för att öppna inställningarna för modulen. Fyll i de valfria fälten om det behövs. För ytterligare information om behållar skapande alternativ, starta om princip och önskad status, se [EdgeAgent önskade egenskaper](module-edgeagent-edgehub.md#edgeagent-desired-properties). Mer information om modulen finns i [definiera eller uppdatera önskade egenskaper](module-composition.md#define-or-update-desired-properties).
1. Om det behövs upprepar du steg 5 till 8 för att lägga till ytterligare moduler i distributionen.
1. Välj **Nästa: vägar** för att fortsätta till avsnittet vägar.

### <a name="specify-routes"></a>Ange vägar

På fliken **vägar** definierar du hur meddelanden ska skickas mellan moduler och IoT Hub. Meddelanden skapas med hjälp av namn/värde-par. Som standard kallas en väg **routning** och definieras som **från\* /Messages/till $upstream**, vilket innebär att alla meddelanden som skickas av alla moduler skickas till din IoT-hubb.  

Lägg till eller uppdatera vägarna med information från [deklarera vägar](module-composition.md#declare-routes)och välj sedan **Nästa: granska + skapa** för att fortsätta till nästa steg i guiden.

### <a name="review-deployment"></a>Granska distribution

I avsnittet granska visas JSON-distributions manifestet som skapades utifrån dina val i de föregående två avsnitten. Observera att det finns två moduler som har deklarerats att du inte har lagt till: **$edgeAgent** och **$edgeHub**. Dessa två moduler utgör den [IoT Edge körningen](iot-edge-runtime.md) och måste vara standardvärden i varje distribution.

Granska distributions informationen och välj sedan **skapa**.

## <a name="view-modules-on-your-device"></a>Visa moduler på enheten

När du har distribuerat moduler till din enhet kan du Visa alla på sidan enhets information i din IoT Hub. Den här sidan visar namnet på varje distribuerad modul, samt användbar information, till exempel distributions status och avslutnings kod.

## <a name="deploy-modules-from-azure-marketplace"></a>Distribuera moduler från Azure Marketplace

[Azure Marketplace](https://azuremarketplace.microsoft.com/) är ett online-program och tjänster för tjänster där du kan bläddra igenom en rad olika företags program och-lösningar som är certifierade och optimerade för att köras på Azure, inklusive [IoT Edge moduler](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules).

Du kan distribuera en IoT Edge-modul från Azure Marketplace och från din IoT Hub.

### <a name="deploy-from-azure-marketplace"></a>Distribuera från Azure Marketplace

Utforska de IoT Edge modulerna på Marketplace och när du hittar den du vill använda kan du distribuera den genom att välja **skapa** eller **Hämta den nu**. Fortsätt med distributions guiden steg som kan variera beroende på vilken IoT Edge-modul du har valt:

1. Bekräfta providerns användnings villkor och sekretess policy genom att välja **Fortsätt**. Du kanske först måste ange kontakt information.
1. Välj din prenumeration och den IoT Hub som mål enheten är ansluten till.
1. Välj **distribuera till en enhet**.
1. Ange namnet på enheten eller Välj **hitta enhet** för att bläddra bland de enheter som är registrerade i hubben.
1. Välj **skapa** för att fortsätta med standard processen att konfigurera ett distributions manifest, inklusive att lägga till andra moduler om du vill. Information om den nya modulen, till exempel bild-URI, skapa alternativ och önskade egenskaper är fördefinierade, men kan ändras.

Kontrol lera att modulen har distribuerats i IoT Hub i Azure Portal. Välj din enhet, Välj **Ange moduler** och modulen ska visas i avsnittet **IoT Edge moduler** .

### <a name="deploy-from-azure-iot-hub"></a>Distribuera från Azure IoT Hub

Du kan snabbt distribuera en modul från Azure Marketplace till din enhet i din IoT Hub i Azure Portal.

1. I Azure Portal navigerar du till IoT Hub.
1. I den vänstra rutan under **Automatisk enhets hantering**väljer du **IoT Edge**.
1. Välj den IoT Edge enhet som ska ta emot distributionen.
1. I den övre stapeln väljer du **Ange moduler**.
1. I avsnittet **IoT Edge moduler** klickar du på **Lägg till**och väljer **Marketplace-modul** på den nedrullningsbara menyn.

![Lägg till modul i IoT Hub](./media/how-to-deploy-modules-portal/iothub-add-module.png)

Välj en modul på sidan **IoT Edge modul Marketplace** . Den modul du väljer konfigureras automatiskt för din prenumeration, resurs grupp och enhet. Den visas sedan i listan med IoT Edge moduler. Vissa moduler kan kräva ytterligare konfiguration.

> [!TIP]
> Informationen om IoT Edge moduler från Azure-IoT Hub är begränsad. Du kan först läsa mer om [IoT Edge-modulerna](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1&subcategories=iot-edge-modules) i Azure Marketplace.

Välj **Nästa: vägar** och fortsätt med distributionen enligt beskrivningen i [Ange vägar](#specify-routes) och [Granska distribution](#review-deployment) tidigare i den här artikeln.

## <a name="next-steps"></a>Nästa steg

Lär dig hur du [distribuerar och övervakar IoT Edge moduler i stor skala](how-to-deploy-at-scale.md)
