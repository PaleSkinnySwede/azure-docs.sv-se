---
title: Vad är för hands version av Azure Firewall Manager?
description: Lär dig mer om Azure Firewall Manager-funktioner
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: overview
ms.date: 03/13/2020
ms.author: victorh
ms.openlocfilehash: 149782f627d586e927c828506a7d4f1b5437b987
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "79366282"
---
# <a name="what-is-azure-firewall-manager-preview"></a>Vad är för hands version av Azure Firewall Manager?

[!INCLUDE [Preview](../../includes/firewall-manager-preview-notice.md)]

För hands versionen av Azure Firewall Manager är en säkerhets hanterings tjänst som tillhandahåller centrala säkerhets principer och väg hantering för molnbaserade säkerhetsperimetern. 

Brand Väggs hanteraren kan tillhandahålla säkerhets hantering för två typer av nätverks arkitektur:

- **skyddad virtuell hubb**

   En [virtuell WAN-hubb i Azure](../virtual-wan/virtual-wan-about.md#resources) är en Microsoft-hanterad resurs som gör att du enkelt kan skapa Hubbs-och eker-arkitekturer. När säkerhets-och routningsprinciperna är associerade med en sådan hubb kallas den för en *[säker virtuell hubb](secured-virtual-hub.md)*. 
- **hubb virtuellt nätverk**

   Det här är ett standard Azure Virtual Network som du skapar och hanterar själv. När säkerhets principer är associerade med en sådan hubb kallas det ett *nav virtuellt nätverk*. För närvarande stöds endast Azure Firewall-principen. Du kan använda peer-ekrar för virtuella nätverk som innehåller dina arbets belastnings servrar och-tjänster. Du kan också hantera brand väggar i fristående virtuella nätverk som inte är peer-kopplade till ekrar.

En detaljerad jämförelse mellan *skyddade virtuella hubbar* och *virtuella hubbar i hubben* finns i [Vad är arkitektur alternativ för Azure Firewall Manager?](vhubs-and-vnets.md).

![Firewall-Manager](media/overview/firewallmanagerv5.png)

## <a name="azure-firewall-manager-preview-features"></a>Förhands gransknings funktioner i Azure Firewall Manager

I för hands versionen av Azure Firewall Manager finns följande funktioner:

### <a name="central-azure-firewall-deployment-and-configuration"></a>Central Azure Firewall-distribution och konfiguration

Du kan distribuera och konfigurera flera Azure Firewall-instanser centralt som sträcker sig över olika Azure-regioner och prenumerationer. 

### <a name="hierarchical-policies-global-and-local"></a>Hierarkiska principer (globala och lokala)

Du kan använda Azure Firewall Manager Preview för att centralt hantera Azure Firewall-principer över flera skyddade virtuella nav. Dina centrala IT-team kan skapa globala brand Väggs principer som upprätthåller organisationens bred brand Väggs policy över team. Lokalt skapade brand Väggs principer tillåter en DevOps-självbetjänings modell för bättre flexibilitet.

### <a name="integrated-with-third-party-security-as-a-service-for-advanced-security"></a>Integrerat med säkerhets-som-en-tjänst från tredje part för avancerad säkerhet

Förutom Azure-brandväggen kan du integrera SECaaS-providers (Security as a Service) från tredje part för att ge ytterligare nätverks skydd för ditt VNet-och filial Internet-anslutningar.

Den här funktionen är endast tillgänglig med säkra virtuella nav distributioner.

- Trafik filtrering för VNet till Internet (V2I)

   - Filtrera utgående virtuell nätverks trafik med din önskade säkerhets leverantör från tredje part.
   - Använd avancerad användar medveten Internet skydd för dina moln arbets belastningar som körs på Azure.

- Trafik filtrering för gren till Internet (B2I)

   Dra nytta av din Azure-anslutning och global distribution för att enkelt lägga till tredjeparts filtrering för förgreningar i Internet scenarier.

Mer information om betrodda säkerhetsproviders finns i [Vad är betrodda säkerhets partner för Azure Firewall Manager (för hands version)?](trusted-security-partners.md)

### <a name="centralized-route-management"></a>Centraliserad flödes hantering

Dirigera snabbt trafik till din säkra hubb för filtrering och loggning utan att behöva konfigurera användardefinierade vägar manuellt (UDR) i ekrar virtuella nätverk. 

Den här funktionen är endast tillgänglig med säkra virtuella nav distributioner.

Du kan använda tredjepartsleverantörer för B2I-trafik (Branch to Internet), sida vid sida med Azure-brandväggen för gren till VNet (B2V), VNet till VNet (V2V) och VNet till Internet (V2I). Du kan också använda tredjepartsleverantörer för V2I-trafik filtrering så länge det inte krävs någon Azure-brandvägg för B2V eller V2V. 

## <a name="region-availability"></a>Regional tillgänglighet

Azure Firewall-principer kan användas i olika regioner. Du kan till exempel skapa en princip i USA, västra och använda den i USA, östra. 

## <a name="known-issues"></a>Kända problem

För hands versionen av Azure Firewall Manager har följande kända problem:

|Problem  |Beskrivning  |Åtgärd  |
|---------|---------|---------|
|Begränsningar för filtrering av tredje part.|V2I trafik filtrering med tredjeparts leverantörer stöds inte med Azure Firewall B2V och V2V.|Undersöker|
|Delning av trafik stöds inte för närvarande.|Det finns för närvarande inte stöd för att dela upp Office 365 och Azure offentlig PaaS-trafik. Det innebär att om du väljer en tredjeparts-Provider för V2I eller B2I skickas även all Azures offentliga PaaS och Office 365-trafik via partner tjänsten.|Undersöker delning av trafik på hubben.
|En säker virtuell hubb per region.|Du kan inte ha mer än en säker virtuell hubb per region.|Skapa flera virtuella WAN-näti en region.|
|Grundläggande principer måste finnas i samma region som den lokala principen.|Skapa alla lokala principer i samma region som bas principen. Du kan fortfarande använda en princip som har skapats i en region på ett skyddat nav från en annan region.|Undersöker|
|Kommunikation mellan olika hubbar fungerar inte med skyddad virtuell hubb|Skyddad virtuell hubb till skyddad virtuell nav-kommunikation stöds inte ännu.|Undersöker|
|Alla skyddade virtuella hubbar som delar samma virtuella WAN-nätverk måste finnas i samma resurs grupp.|Det här beteendet är justerat med virtuella WAN-hubbar idag.|Skapa flera virtuella WAN-administratörer för att kunna skapa säkra virtuella hubbar i olika resurs grupper.|
|IP-grupper stöds inte i brand Väggs principer.|IP-grupper finns i en offentlig för hands version och stöds för närvarande bara med traditionella brand Väggs regler.|Korrigering pågår.
|Prenumerationer på moln lösnings leverantörer (CSP) stöds inte.|CSP- [prenumerationer](https://azure.microsoft.com/offers/ms-azr-0145p/) stöds för närvarande inte.|Undersöker

## <a name="next-steps"></a>Nästa steg

- Läs [Översikt över Azure Firewall Manager Preview Deployment](deployment-overview.md)
- Läs mer om [skyddade virtuella nav](secured-virtual-hub.md).