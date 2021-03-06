---
title: Hantera DHCP
description: Den här artikeln beskriver hur du hanterar DHCP i Azure VMWare-lösning (AVS)
ms.topic: conceptual
ms.date: 05/04/2020
ms.openlocfilehash: ccf28c94e1991681c238f51847fe228313abe29e
ms.sourcegitcommit: d9cd51c3a7ac46f256db575c1dfe1303b6460d04
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/04/2020
ms.locfileid: "82740452"
---
# <a name="how-to-manage-dhcp-in-azure-vmware-solution-avs-preview"></a>Hantera DHCP i Azure VMWare-lösningen (AVS) Preview

NSX – T ger möjlighet att konfigurera DHCP för ditt privata moln. Om du planerar att använda NSX-T som värd för DHCP-servern, se [skapa DHCP-server](#create-dhcp-server). Annars, om du har en extern DHCP-server från tredje part i nätverket och du vill vidarebefordra begär anden till DHCP-servern, se [skapa DHCP Relay service](#create-dhcp-relay-service).

## <a name="create-dhcp-server"></a>Skapa DHCP-server

Använd följande steg för att konfigurera en DHCP-server på NSX-T.

Från NSX Manager går du till fliken **nätverk** och väljer **DHCP** under **IP-hantering**. Välj knappen **Lägg till Server** . Ange sedan Server namnet och serverns IP-adress. När du är färdig väljer du **Spara**.

:::image type="content" source="./media/manage-dhcp/dhcp-server-settings.png" alt-text="Lägg till DHCP-server" border="true":::

### <a name="connect-dhcp-server-to-the-tier-1-gateway"></a>Anslut DHCP-server till nivån-1 Gateway.

Välj Gateway för **nivå 1**, Välj gatewayen och välj **Redigera**

:::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway.png" alt-text="Välj den gateway som ska användas" border="true":::

Lägg till ett undernät genom att välja **ingen IP-allokerade uppsättning**

:::image type="content" source="./media/manage-dhcp/add-subnet.png" alt-text="Lägg till ett undernät" border="true":::

På nästa skärm väljer du den **lokala DHCP-servern** i list rutan **typ** . För **DHCP-server**väljer du **standard-DHCP** och sedan **Spara**.

:::image type="content" source="./media/manage-dhcp/set-ip-address-management.png" alt-text="Välj alternativ för DHCP-server" border="true":::

I fönstret **nivå-1-Gateway** väljer du **Spara**. På nästa skärm ser du **sparade ändringar**och väljer **Stäng redigering** för att slutföra.

### <a name="add-a-network-segment"></a>Lägg till ett nätverks segment

När du har skapat din DHCP-server måste du lägga till nätverks segment i den.

I NSX-T väljer du fliken **nätverk** och väljer **segment** under **anslutning**. Välj **Lägg till segment**. Namnge segmentet och anslutningen till nivån-1 Gateway. Välj sedan **Ange undernät** för att konfigurera ett nytt undernät. 

:::image type="content" source="./media/manage-dhcp/add-segment.png" alt-text="Lägg till ett nytt nätverks segment" border="true":::

I fönstret **Ange undernät** väljer du **Lägg till undernät**. Ange IP-adressen för gatewayen och DHCP-intervallet och välj **Lägg till** och sedan **tillämpa**

:::image type="content" source="./media/manage-dhcp/add-subnet-segment.png" alt-text="Lägg till nätverks segment" border="true":::

När du är klar väljer du **Spara** för att slutföra tillägget av ett nätverks segment.

:::image type="content" source="./media/manage-dhcp/segments-complete.png" alt-text="slut för ande segment" border="true":::

## <a name="create-dhcp-relay-service"></a>Skapa DHCP Relay service

I fönstret NXT-T väljer du fliken **nätverk** och under **IP-hantering**väljer du **DHCP**. Välj **Lägg till Server**. Välj DHCP-relä för **Server typ** och ange Server namn och IP-adress för relä servern. Välj **Spara** för att spara ändringarna.

:::image type="content" source="./media/manage-dhcp/create-dhcp-relay.png" alt-text="Skapa DHCP-relä Server" border="true":::

Välj **nivå 1 Gateway** under **anslutning**. Välj den lodräta ellipsen på nivå 1-gatewayen och välj **Redigera**.

:::image type="content" source="./media/manage-dhcp/edit-tier-1-gateway-relay.png" alt-text="redigera nivå 1-Gateway" border="true":::

Välj **ingen IP-allokerad uppsättning** för att definiera IP-adressallokering.

:::image type="content" source="./media/manage-dhcp/edit-ip-address-allocation.png" alt-text="Redigera IP-adressallokering" border="true":::

I dialog rutan väljer du **DHCP Relay server**för **typ**. I list rutan **DHCP-relä** väljer du DHCP-relä servern. När du är färdig väljer du **Spara**

:::image type="content" source="./media/manage-dhcp/set-ip-address-management-relay.png" alt-text="Ange hantering av IP-adresser" border="true":::

Ange IP-intervallets IP-adress i segment:

> [!NOTE]
> Den här konfigurationen krävs för att realisera DHCP-relä i DHCP-klientcertifikatet. 

Under **anslutning**väljer du **segment**. Markera de lodräta ellipserna och välj **Redigera**. Om du i stället vill lägga till ett nytt segment kan du välja **Lägg till segment** för att skapa ett nytt segment.

:::image type="content" source="./media/manage-dhcp/edit-segments.png" alt-text="Redigera ett nätverks under nät" border="true":::

Lägg till information om segmentet. Välj värdet under **undernät** eller **Ange undernät** för att lägga till eller ändra under nätet.

:::image type="content" source="./media/manage-dhcp/network-segments.png" alt-text="nätverks segment" border="true":::

Markera de lodräta ellipserna och välj **Redigera**. Om du behöver skapa ett nytt undernät väljer du **Lägg till undernät** för att skapa en gateway och konfigurera ett DHCP-intervall. Ange intervallet för IP-adresspoolen och välj **Verkställ**och välj sedan **Spara**

:::image type="content" source="./media/manage-dhcp/edit-subnet.png" alt-text="redigera undernät" border="true":::

Nu är en DHCP-adresspool tilldelad till segmentet.

:::image type="content" source="./media/manage-dhcp/assigned-to-segment.png" alt-text="DHCP-serverpoolen tilldelad till segment" border="true":::
