---
title: Översikt över distributionen av Azure Firewall Manager
description: Lär dig mer om distributions steg på hög nivå som krävs för för hands versionen av Azure Firewall Manager
author: vhorne
ms.service: firewall-manager
services: firewall-manager
ms.topic: overview
ms.date: 02/18/2020
ms.author: victorh
ms.openlocfilehash: c3a94cea838609f65511a21ee2f64e8782a6adea
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "77443133"
---
# <a name="azure-firewall-manager-preview-deployment-overview"></a>Översikt över distributionen av Azure Firewall Manager

[!INCLUDE [Preview](../../includes/firewall-manager-preview-notice.md)]

Det finns fler än ett sätt att distribuera för hands versionen av Azure Firewall Manager, men följande allmänna process rekommenderas.

## <a name="general-deployment-process"></a>Allmän distributionsprocess

### <a name="hub-virtual-networks"></a>Hubb virtuella nätverk

1.  Skapa en brand Väggs princip

    - Skapa en ny policy
<br>*eller*<br>
    - Härled en grundläggande princip och anpassa en lokal princip
<br>*eller*<br>
    - Importera regler från en befintlig Azure-brandvägg. Se till att ta bort NAT-regler från principer som ska tillämpas på flera brand väggar
1. Skapa din hubb och eker-arkitektur
   - Skapa en hubb Virtual Network med hjälp av virtuella nätverk i Azure Firewall Manager och virtuella peer-ekrar till den med hjälp av peering för virtuella nätverk
<br>*eller*<br>
    - Skapa ett virtuellt nätverk och Lägg till virtuella nätverks anslutningar och peer-ekrar virtuella nätverk med hjälp av peering för virtuella nätverk

3. Välj säkerhets leverantörer och koppla brand Väggs principen. För närvarande är endast Azure-brandväggen en leverantör som stöds.

   - Detta görs när du skapar en hubb Virtual Network
<br>*eller*<br>
    - Konvertera ett befintligt virtuellt nätverk till en hubb Virtual Network. Det är också möjligt att konvertera flera virtuella nätverk.

4. Konfigurera användare definiera vägar för att dirigera trafik till hubben Virtual Network brand väggen.


### <a name="secured-virtual-hubs"></a>Skyddade virtuella hubbar

1. Skapa din hubb och eker-arkitektur

   - Skapa en säker virtuell hubb med Azure Firewall Manager och Lägg till virtuella nätverks anslutningar.<br>*eller*<br>
   - Skapa en virtuell WAN-hubb och Lägg till virtuella nätverks anslutningar.
2. Välj säkerhets leverantörer

   - Klar när du skapade en säker virtuell hubb.<br>*eller*<br>
   - Konvertera en befintlig virtuell WAN-hubb till en säker virtuell hubb.
3. Skapa en brand Väggs princip och koppla den till din hubb

   - Gäller endast om du använder Azure-brandväggen.
   - SECaaS-principer (från tredje part) konfigureras med hjälp av partner hanterings upplevelser.
4. Konfigurera väg inställningar för att dirigera trafik till din skyddade hubb

   - Dirigera snabbt trafik till din säkra hubb för filtrering och loggning utan användardefinierade vägar (UDR) i ekrar virtuella nätverk med hjälp av den säkra väg inställnings sidan för virtuella hubbar.

> [!NOTE]
> - Du kan inte ha mer än en hubb per virtuell WAN per region. Men du kan lägga till flera virtuella WAN-näti regionen för att uppnå detta.
> - Du kan inte ha överlappande IP-utrymmen för hubbar i en vWAN.
> - Hubbens VNet-anslutningar måste finnas i samma region som hubben.

## <a name="next-steps"></a>Nästa steg

- [Självstudie: skydda ditt moln nätverk med för hands versionen av Azure Firewall Manager med hjälp av Azure Portal](secure-cloud-network.md)