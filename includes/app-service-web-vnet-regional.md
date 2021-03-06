---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 04/15/2020
ms.author: ccompy
ms.openlocfilehash: f7208307df51ecefb76f9adaedea59b327cdc19e
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81604888"
---
Genom att använda regional VNet-integrering kan din app komma åt:

* Resurser i ett VNet i samma region som din app.
* Resurser i virtuella nätverk som peer-kopplas till det virtuella nätverk som din app är integrerat med.
* Säkra tjänster för tjänst slut punkt.
* Resurser över Azure ExpressRoute-anslutningar.
* Resurser i det virtuella nätverk som du är integrerad med.
* Resurser mellan peer-anslutningar, som innehåller Azure ExpressRoute-anslutningar.
* Privata slut punkter 

När du använder VNet-integrering med virtuella nätverk i samma region kan du använda följande funktioner i Azure-nätverk:

* **Nätverks säkerhets grupper (NSG: er)**: du kan blockera utgående trafik med en NSG som placeras i ditt integrations undernät. Reglerna för inkommande trafik gäller inte eftersom du inte kan använda VNet-integrering för att ge inkommande åtkomst till din app.
* **Routningstabeller (UDR)**: du kan placera en routningstabell i integrations under nätet för att skicka utgående trafik där du vill.

Som standard dirigerar din app endast RFC1918 trafik till ditt VNet. Om du vill dirigera all utgående trafik till ditt VNet använder du appens inställning WEBSITE_VNET_ROUTE_ALL till din app. Konfigurera appens inställning:

1. Gå till **konfigurations** gränssnittet på din app-Portal. Välj **ny program inställning**.
1. Ange **WEBSITE_VNET_ROUTE_ALL** i rutan **namn** och skriv **1** i rutan **värde** .

   ![Ange program inställning][4]

1. Välj **OK**.
1. Välj **Spara**.

Om du dirigerar all utgående trafik till ditt VNet, omfattas den av NSG: er och UDR som tillämpas på ditt integrations undernät. När du dirigerar all utgående trafik till ditt VNet, är dina utgående adresser fortfarande de utgående adresser som visas i dina app-egenskaper, om du inte tillhandahåller vägar för att skicka trafiken någon annan stans.

Det finns vissa begränsningar med att använda VNet-integrering med virtuella nätverk i samma region:

* Du kan inte komma åt resurser över global peering anslutningar.
* Funktionen är endast tillgänglig från nyare Azure App Service skalnings enheter som stöder PremiumV2 App Service-planer.
* Integrations under nätet kan bara användas av en App Service plan.
* Funktionen kan inte användas av isolerade plan-appar som finns i en App Service-miljön.
* Funktionen kräver ett oanvänt undernät som är a/27 med 32 adresser eller större i ett Azure Resource Manager VNet.
* Appen och VNet måste finnas i samma region.
* Du kan inte ta bort ett VNet med en integrerad app. Ta bort integrationen innan du tar bort det virtuella nätverket.
* Du kan bara integrera med virtuella nätverk i samma prenumeration som appen.
* Du kan bara ha en regional VNet-integration per App Service plan. Flera appar i samma App Service plan kan använda samma VNet.
* Du kan inte ändra prenumerationen på en app eller en plan när det finns en app som använder regional VNet-integrering.
* Din app kan inte matcha adresser i Azure DNS Private Zones utan konfigurations ändringar

En adress används för varje plan instans. Om du skalar din app till fem instanser används fem adresser. Eftersom det inte går att ändra under näts storleken efter tilldelningen, måste du använda ett undernät som är tillräckligt stort för att anpassa vilken skala appen kan komma åt. En/26 med 64-adresser är den rekommenderade storleken. A/26 med 64 adresser uppfyller en Premium-plan med 30 instanser. När du skalar upp eller ned en plan behöver du två gånger så många adresser under en kort tids period.

Om du vill att dina appar i en annan plan ska kunna komma åt ett VNet som redan är anslutet till av appar i ett annat abonnemang väljer du ett annat undernät än det som används av den befintliga VNet-integreringen.

Funktionen är i för hands version för Linux. Linux-formen för funktionen stöder bara anrop till RFC 1918-adresser (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16).

### <a name="web-or-function-app-for-containers"></a>Webb eller Funktionsapp för behållare

Om du är värd för din app på Linux med de inbyggda avbildningarna fungerar regional VNet-integrering utan ytterligare ändringar. Om du använder webb-eller Funktionsapp för behållare måste du ändra Docker-avbildningen för att använda VNet-integrering. I Docker-avbildningen använder du PORT miljö variabeln som den huvudsakliga webb serverns lyssnings port, i stället för att använda ett hårdkodad port nummer. PORT miljö variabeln anges automatiskt av plattformen vid behållarens start tid. Om du använder SSH måste SSH-daemon konfigureras för att lyssna på det port nummer som anges av SSH_PORT-miljövariabeln när du använder regional VNet-integrering. Det finns inget stöd för gateway-nödvändig VNet-integrering på Linux.

### <a name="service-endpoints"></a>Tjänstslutpunkter

Regional VNet-integrering gör att du kan använda tjänst slut punkter. Om du vill använda tjänstens slut punkter med din app använder du regional VNet-integrering för att ansluta till ett valt VNet och konfigurerar sedan tjänstens slut punkter med mål tjänsten på under nätet som du använde för integreringen. Om du sedan ville få åtkomst till en tjänst via tjänst slut punkter:

1. Konfigurera regional VNet-integrering med din webbapp
1. Gå till mål tjänsten och konfigurera tjänst slut punkter mot det undernät som används för integrering

### <a name="network-security-groups"></a>Nätverkssäkerhetsgrupper

Du kan använda nätverks säkerhets grupper för att blockera inkommande och utgående trafik till resurser i ett VNet. En app som använder regional VNet-integrering kan använda en [nätverks säkerhets grupp][VNETnsg] för att blockera utgående trafik till resurser i ditt VNet eller Internet. Om du vill blockera trafik till offentliga adresser måste du ha program inställningen WEBSITE_VNET_ROUTE_ALL inställd på 1. Reglerna för inkommande trafik i en NSG gäller inte för din app eftersom VNet-integrering endast påverkar utgående trafik från din app.

Om du vill kontrol lera inkommande trafik till din app använder du funktionen begränsningar för åtkomst. En NSG som tillämpas på ditt integrations under nät gäller oavsett vilka vägar som tillämpas på ditt integrations under nät. Om WEBSITE_VNET_ROUTE_ALL har angetts till 1 och du inte har några vägar som påverkar offentlig adress trafik i ditt integrations undernät, omfattas all utgående trafik fortfarande till NSG: er som har tilldelats ditt integrations under nät. Om WEBSITE_VNET_ROUTE_ALL inte har angetts tillämpas NSG: er endast på RFC1918-trafik.

### <a name="routes"></a>Vägar

Du kan använda routningstabeller för att dirigera utgående trafik från din app till var du vill. Som standard påverkar routningstabeller bara din RFC1918-destinations trafik. Om du anger WEBSITE_VNET_ROUTE_ALL till 1 påverkas alla utgående samtal. Vägar som anges i ditt integrations undernät påverkar inte svar på inkommande app-begäranden. Vanliga mål kan omfatta brand Väggs enheter eller gatewayer.

Om du vill dirigera all utgående trafik lokalt kan du använda en routningstabell för att skicka all utgående trafik till din ExpressRoute-Gateway. Om du dirigerar trafik till en gateway måste du ställa in vägar i det externa nätverket för att skicka svar tillbaka.

Border Gateway Protocol (BGP) vägar påverkar också din app-trafik. Om du har BGP-vägar från något som liknar en ExpressRoute-gateway kommer din app utgående trafik att påverkas. BGP-vägar påverkar som standard bara din RFC1918-destinations trafik. Om WEBSITE_VNET_ROUTE_ALL är inställt på 1 kan all utgående trafik påverkas av BGP-vägar.

### <a name="azure-dns-private-zones"></a>Azure DNS Private Zones 

När din app har integrerats med ditt VNet, använder den samma DNS-server som ditt VNet har kon figurer ATS med. Som standard fungerar inte appen med Azure DNS Private Zones. Om du vill arbeta med Azure DNS Private Zones måste du lägga till följande appinställningar:

1. WEBSITE_DNS_SERVER med värdet 168.63.129.16 
1. WEBSITE_VNET_ROUTE_ALL med värdet 1

De här inställningarna kommer att skicka alla utgående samtal från din app till ditt VNet, förutom att aktivera din app för att använda Azure DNS privata zoner.

### <a name="private-endpoints"></a>Privata slut punkter

Om du vill göra anrop till [privata slut punkter][privateendpoints]måste du antingen integrera med Azure DNS Private Zones eller hantera den privata slut punkten på den DNS-server som används av din app. 

<!--Image references-->
[4]: ../includes/media/web-sites-integrate-with-vnet/vnetint-appsetting.png

<!--Links-->
[VNETnsg]: https://docs.microsoft.com/azure/virtual-network/security-overview/
[privateendpoints]: https://docs.microsoft.com/azure/app-service/networking/private-endpoint
