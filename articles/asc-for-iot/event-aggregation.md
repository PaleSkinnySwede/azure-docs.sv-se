---
title: Händelsesammansättning
description: Lär dig mer om Azure Security Center för agg regering av IoT-händelser.
services: asc-for-iot
ms.service: asc-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.assetid: d40dac0b-abd0-4ff5-82eb-0f511ee13725
ms.subservice: asc-for-iot
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/26/2019
ms.author: mlottner
ms.openlocfilehash: f72ef8cc5161bd6f885249e7d39344a57fa2368e
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81311422"
---
# <a name="azure-security-center-for-iot-event-aggregation"></a>Azure Security Center för insamling av IoT-händelser

Azure Security Center för IoT-säkerhetsagenter samlar in data-och system händelser från den lokala enheten och skickar dessa data till Azure-molnet för bearbetning och analys. Säkerhets agenten samlar in många typer av enhets händelser, inklusive nya processer och nya anslutnings händelser. Både nya processer och nya anslutnings händelser kan på ett legitimt sätt ske ofta på en enhet inom en sekund, och även om det är viktigt för robust och omfattande säkerhet, tvingas det att skicka säkerhets agenter för meddelanden som snabbt når eller överskrider din IoT Hub kvot och dina kostnads gränser. Dessa händelser innehåller dock mycket värdefull säkerhets information som är viktig för att skydda enheten.

För att minska den ytterligare kvoten och kostnaderna samtidigt som dina enheter skyddas kan Azure Security Center för IoT-agenter samla dessa typer av händelser.

Händelse agg regering är **aktiverat** som standard och även om det inte rekommenderas, **kan stängas av manuellt när som** helst.

Aggregator är för närvarande tillgängligt för följande typer av händelser:

* ProcessCreate
* ConnectionCreate
* ProcessTerminate (endast Windows)

## <a name="how-does-event-aggregation-work"></a>Hur fungerar händelse agg regering?

När händelse agg regering är kvar **på**Azure Security Center för IoT-agenter aggregera händelser för intervall perioden eller tids perioden.
När intervall perioden har passerat skickar agenten de aggregerade händelserna till Azure-molnet för ytterligare analys.
De aggregerade händelserna lagras i minnet tills de skickas till Azure-molnet.

När agenten samlar in en identisk händelse till en som redan finns i minnet, ökar agenten antalet träffar för den här händelsen. När den sammanställnings tids perioden passerar skickar agenten antalet träffar för varje typ av händelse som har inträffat. Händelse agg regering är bara agg regeringen av antalet träffar för varje insamlad typ av händelse.

Händelser anses vara identiska endast när följande villkor uppfylls:

* ProcessCreate-händelser – när **kommandorad**, **körbara filer**, **användar namn**och **UserID** är identiska
* ConnectionCreate-händelser – när **kommando raden**, **userId**, **Direction**, **Local Address**, **Remote Address**, * * Protocol och **målport** är identiska
* ProcessTerminate-händelser – när den **körbara filen** och **avslutnings statusen** är identiska

### <a name="working-with-aggregated-events"></a>Arbeta med sammanställda händelser

Under agg regering ignoreras händelse egenskaper som inte aggregeras och visas i Log Analytics med värdet 0.

* ProcessCreate-händelser – **ProcessID**och **parentProcessId** har angetts till 0
* ConnectionCreate-händelser – **ProcessID**och **käll porten** har angetts till 0

## <a name="event-aggregation-based-alerts"></a>Händelse agg regering baserade aviseringar

Efter analysen skapar Azure Security Center för IoT säkerhets aviseringar för misstänkta sammanställda händelser. Aviseringar som skapas från sammanställda händelser visas bara en gång för varje sammanställd händelse.

Samlingens start tid, slut tid och antal träffar för varje händelse loggas i fältet Event **ExtraDetails** i Log Analytics som används vid undersökningar.

Varje sammanställd händelse representerar en 24-timmarsperiod med insamlade aviseringar. Med hjälp av menyn händelse alternativ längst upp till vänster i varje händelse kan du **stänga** av varje enskild sammanställd händelse.

## <a name="event-aggregation-twin-configuration"></a>Dubbel konfiguration av händelse sammansättning

Ändra konfigurationen av Azure Security Center för IoT Event-aggregering i [agent konfigurations objekt](how-to-agent-configuration.md) för modulens dubbla identitet för **azureiotsecurity** -modulen.

| Konfigurations namn | Möjliga värden | Information | Anmärkningar |
|:-----------|:---------------|:--------|:--------|
| aggregationEnabledProcessCreate | boolean | Aktivera/inaktivera händelse agg regering för process-skapa händelser |
| aggregationIntervalProcessCreate | ISO8601 TimeSpan-sträng | Samlings intervall för process skapande av händelser |
| aggregationEnabledConnectionCreate | boolean| Aktivera/inaktivera händelse agg regering för skapande av anslutnings händelser |
| aggregationIntervalConnectionCreate | ISO8601 TimeSpan-sträng | Samlings intervall för skapande av anslutnings händelser |
| aggregationEnabledProcessTerminate | boolean | Aktivera/inaktivera händelse agg regering för process-avsluta händelser | Endast Windows|
| aggregationIntervalProcessTerminate | ISO8601 TimeSpan-sträng | Samlings intervall för process-avsluta händelser | Endast Windows|
|

## <a name="default-configurations-settings"></a>Standardinställningar för konfigurering

| Konfigurations namn | Standardvärden |
|:-----------|:---------------|
| aggregationEnabledProcessCreate | true |
| aggregationIntervalProcessCreate | PT1H|
| aggregationEnabledConnectionCreate | true |
| aggregationIntervalConnectionCreate | PT1H|
| aggregationEnabledProcessTerminate | true |
| aggregationIntervalProcessTerminate | PT1H|
|

## <a name="next-steps"></a>Nästa steg

I den här artikeln har du lärt dig om Azure Security Center för agg regering av IoT Security Agent och de tillgängliga alternativen för händelse konfiguration.

Använd följande artiklar om du vill fortsätta att komma igång med Azure Security Center för IoT-distribution:

- Förstå [autentiseringsmetoder för säkerhets agenter](concept-security-agent-authentication-methods.md)
- Välj och distribuera en [säkerhets agent](how-to-deploy-agent.md)
- Granska Azure Security Center för krav för IoT [-tjänsten](service-prerequisites.md)
- Lär dig hur du [aktiverar Azure Security Center för IoT-tjänsten i din IoT Hub](quickstart-onboard-iot-hub.md)
- Läs mer om tjänsten i [vanliga frågor och svar om Azure Security Center for IoT](resources-frequently-asked-questions.md)
