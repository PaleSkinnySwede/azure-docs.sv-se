---
title: Integrera Azure Service Bus med Azure Private Link service
description: Lär dig hur du integrerar Azure Service Bus med Azure Private Link service
services: service-bus-messaging
author: spelluru
ms.author: spelluru
ms.date: 03/13/2020
ms.service: service-bus-messaging
ms.topic: article
ms.openlocfilehash: 33e6ce1d5feb50080b00fcbecdeb9e512980eab6
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82141953"
---
# <a name="integrate-azure-service-bus-with-azure-private-link-preview"></a>Integrera Azure Service Bus med en privat Azure-länk (förhands granskning)

Azure Private Link service ger dig åtkomst till Azure-tjänster (till exempel Azure Service Bus, Azure Storage och Azure Cosmos DB) och Azure-värdbaserade kund-/partner tjänster via en **privat slut punkt** i det virtuella nätverket.

En privat slut punkt är ett nätverks gränssnitt som ansluter privat och säkert till en tjänst som drivs av en privat Azure-länk. Den privata slut punkten använder en privat IP-adress från ditt virtuella nätverk, vilket effektivt ansluter tjänsten till ditt VNet. All trafik till tjänsten kan dirigeras via den privata slut punkten, så inga gatewayer, NAT-enheter, ExpressRoute-eller VPN-anslutningar eller offentliga IP-adresser krävs. Trafik mellan ditt virtuella nätverk och tjänsten passerar över Microsofts stamnätverk, vilket eliminerar exponering från det offentliga Internet. Du kan ansluta till en instans av en Azure-resurs, vilket ger dig den högsta nivån av granularitet i åtkomst kontroll.

Mer information finns i [Vad är en privat Azure-länk?](../private-link/private-link-overview.md)

>[!WARNING]
> Implementering av privata slut punkter kan förhindra att andra Azure-tjänster interagerar med Service Bus.
>
> Betrodda Microsoft-tjänster stöds inte när du använder virtuella nätverk.
>
> Vanliga Azure-scenarier som inte fungerar med virtuella nätverk (Observera att listan **inte** är fullständig) –
> - Integrering med Azure Event Grid
> - Azure IoT Hub vägar
> - Azure IoT-Device Explorer
>
> De Microsoft-tjänster som behövs nedan måste finnas i ett virtuellt nätverk
> - Azure App Service
> - Azure Functions

> [!IMPORTANT]
> Den här funktionen stöds med **Premium** -nivån för Azure Service Bus. Mer information om Premium-nivån finns i artikeln [Service Bus Premium-och standard meddelande nivåer](service-bus-premium-messaging.md) .
>
> Den här funktionen är för närvarande en för **hands version**. 


## <a name="add-a-private-endpoint-using-azure-portal"></a>Lägg till en privat slut punkt med Azure Portal

### <a name="prerequisites"></a>Krav

Om du vill integrera ett Service Bus-namnområde med en privat Azure-länk behöver du följande entiteter eller behörigheter:

- Ett Service Bus namn område.
- Ett virtuellt Azure-nätverk.
- Ett undernät i det virtuella nätverket.
- Ägar-eller deltagar behörighet för både Service Bus-namnrymden och det virtuella nätverket.

Din privata slut punkt och det virtuella nätverket måste finnas i samma region. När du väljer en region för den privata slut punkten med hjälp av portalen filtreras automatiskt endast virtuella nätverk i den regionen. Ditt Service Bus namn område kan finnas i en annan region. Och den privata slut punkten använder en privat IP-adress i det virtuella nätverket.

### <a name="steps"></a>steg

Om du redan har ett befintligt namn område kan du skapa en privat slut punkt genom att följa dessa steg:

1. Logga in på [Azure-portalen](https://portal.azure.com). 
2. I Sök fältet skriver du in **Service Bus**.
3. Välj det **namn område** i listan som du vill lägga till en privat slut punkt för.
4. Välj fliken **nätverk** under **Inställningar**.
5. Välj fliken **anslutningar för privat slut punkt (för hands version)** överst på sidan
6. Välj knappen **+ privat slut punkt** överst på sidan.

    ![Knappen Lägg till privat slut punkt](./media/private-link-service/private-link-service-3.png)
7. Följ de här stegen på sidan **grundläggande** : 
    1. Välj den **Azure-prenumeration** där du vill skapa den privata slut punkten. 
    2. Välj **resurs grupp** för den privata slut punkts resursen.
    3. Ange ett **namn** för den privata slut punkten. 
    5. Välj en **region** för den privata slut punkten. Din privata slut punkt måste finnas i samma region som ditt virtuella nätverk, men kan finnas i en annan region än den privata länk resurs som du ansluter till. 
    6. Välj **Nästa: resurs >s** knappen längst ned på sidan.

        ![Sidan skapa privat slut punkt – grunder](./media/private-link-service/create-private-endpoint-basics-page.png)
8. Följ de här stegen på sidan **resurs** :
    1. För anslutnings metod, om du väljer **Anslut till en Azure-resurs i min katalog**, följer du dessa steg:   
        1. Välj den **Azure-prenumeration** där **Service Bus namn området** finns. 
        2. För **resurs typ**väljer du **Microsoft. Service Bus/Namespaces** för **resurs typen**.
        3. För **resurs**väljer du ett Service Bus namn område i list rutan. 
        4. Bekräfta att **mål under resursen** har angetts till **namnrymd**.
        5. Välj **Nästa: konfiguration >s** knappen längst ned på sidan. 
        
            ![Skapa privat slut punkt – resurs sida](./media/private-link-service/create-private-endpoint-resource-page.png)
    2. Om du väljer **Anslut till en Azure-resurs efter resurs-ID eller alias**följer du dessa steg:
        1. Ange **resurs-ID** eller **alias**. Det kan vara det resurs-ID eller alias som någon har delat med dig. Det enklaste sättet att hämta resurs-ID är att navigera till Service Bus namn området i Azure Portal och kopiera del av URI: n som börjar `/subscriptions/`från. Se följande bild för ett exempel. 
        2. För **under resurs för mål**anger du **namnrymd**. Det är den typ av under resurs som din privata slut punkt kan komma åt. 
        3. valfritt Ange ett **meddelande om begäran**. Resurs ägaren ser det här meddelandet vid hantering av privat slut punkts anslutning. 
        4. Välj sedan **Nästa: konfiguration >** knappen längst ned på sidan. 

            ![Skapa privat slut punkt – Anslut med resurs-ID](./media/private-link-service/connect-resource-id.png)
9. På sidan **konfiguration** väljer du det undernät i ett virtuellt nätverk som du vill distribuera den privata slut punkten till. 
    1. Välj ett **virtuellt nätverk**. Endast virtuella nätverk i den valda prenumerationen och platsen visas i list rutan. 
    2. Välj ett **undernät** i det virtuella nätverk som du har valt. 
    3. Välj **Nästa: tagg >** -knappen längst ned på sidan. 

        ![Skapa privat slut punkt – konfigurations sida](./media/private-link-service/create-private-endpoint-configuration-page.png)
10. På sidan **taggar** skapar du alla Taggar (namn och värden) som du vill koppla till den privata slut punkts resursen. Välj sedan **Granska + skapa** längst ned på sidan. 
11. Granska alla inställningar på sidan **Granska och skapa**och välj **skapa** för att skapa den privata slut punkten.
    
    ![Skapa privat slut punkt – sidan Granska och skapa](./media/private-link-service/create-private-endpoint-review-create-page.png)
12. Bekräfta att den privata slut punkten har skapats. Om du är ägare till resursen och har valt **Anslut till en Azure-resurs i mitt katalog** alternativ för **anslutnings metoden**, ska slut punkts anslutningen **automatiskt godkännas**. Om det är i **vänte** läge, se avsnittet [hantera privata slut punkter med Azure Portal](#manage-private-endpoints-using-azure-portal) .

    ![Den privata slut punkten har skapats](./media/private-link-service/private-endpoint-created.png)

## <a name="add-a-private-endpoint-using-powershell"></a>Lägg till en privat slut punkt med PowerShell
I följande exempel visas hur du använder Azure PowerShell för att skapa en privat slut punkts anslutning till ett Service Bus namn område.

Din privata slut punkt och det virtuella nätverket måste finnas i samma region. Ditt Service Bus namn område kan finnas i en annan region. Och den privata slut punkten använder en privat IP-adress i det virtuella nätverket.

```azurepowershell-interactive

$rgName = "<RESOURCE GROUP NAME>"
$vnetlocation = "<VNET LOCATION>"
$vnetName = "<VIRTUAL NETWORK NAME>"
$subnetName = "<SUBNET NAME>"
$namespaceLocation = "<NAMESPACE LOCATION>"
$namespaceName = "<NAMESPACE NAME>"
$peConnectionName = "<PRIVATE ENDPOINT CONNECTION NAME>"

# create resource group
az group create -l $vnetLocation -n $rgName

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName $rgName `
                    -Location $vnetlocation `
                    -Name $vnetName `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name $subnetName `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork

# create premium service bus namespace
$namespaceResource = New-AzResource -Location $namespaceLocation -ResourceName $namespaceName -ResourceGroupName $rgName -Sku @{name = "Premium"; capacity = 1} -Properties @{} -ResourceType "Microsoft.ServiceBus/namespaces" -

# create a private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name $peConnectionName `
                                -PrivateLinkServiceId $namespaceResource.ResourceId `
                                -GroupId "namespace"

# get subnet object that you will use in the next step                                
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName  $rgName -Name $vnetName
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                                | Where-Object  {$_.Name -eq $subnetName}  
   
# now, create private endpoint   
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $rgName  `
                                -Name $vnetName   `
                                -Location $vnetlocation `
                                -Subnet  $subnet   `
                                -PrivateLinkServiceConnection $privateEndpointConnection

(Get-AzResource -ResourceId $namespaceResource.ResourceId -ExpandProperties).Properties


```


## <a name="manage-private-endpoints-using-azure-portal"></a>Hantera privata slut punkter med hjälp av Azure Portal

När du skapar en privat slut punkt måste anslutningen godkännas. Om den resurs som du skapar en privat slut punkt för finns i din katalog kan du godkänna anslutningsbegäran förutsatt att du har tillräcklig behörighet. Om du ansluter till en Azure-resurs i en annan katalog måste du vänta tills ägaren av resursen har godkänt din anslutningsbegäran.

Det finns fyra etablerings tillstånd:

| Tjänst åtgärd | Status för privat slut punkt för tjänst förbrukare | Beskrivning |
|--|--|--|
| Inga | Väntar | Anslutningen skapas manuellt och väntar på godkännande från ägaren till den privata länk resursen. |
| Godkänn | Godkända | Anslutningen godkändes automatiskt eller manuellt och är redo att användas. |
| Avvisa | Avvisad | Anslutningen avvisades av ägaren till den privata länk resursen. |
| Ta bort | Frånkopplad | Anslutningen togs bort av ägaren till den privata länk resursen, den privata slut punkten blir informativ och bör tas bort för rensning. |
 
###  <a name="approve-reject-or-remove-a-private-endpoint-connection"></a>Godkänn, avvisa eller ta bort en privat slut punkts anslutning

1. Logga in på Azure-portalen.
1. I Sök fältet skriver du in **Service Bus**.
1. Välj det **namn område** som du vill hantera.
1. Välj fliken **nätverk** .
5. Gå till lämpligt avsnitt nedan, baserat på den åtgärd du vill: Godkänn, avvisa eller ta bort. 

### <a name="approve-a-private-endpoint-connection"></a>Godkänna en privat slut punkts anslutning

1. Om det finns några anslutningar som väntar, visas en anslutning som anges i **väntan** på etablerings status. 
2. Välj den **privata slut punkt** som du vill godkänna
3. Välj knappen **Godkänn** .

    ![Godkänn privat slut punkt](./media/private-link-service/private-endpoint-approve.png)
4. På sidan **Godkänn anslutning** anger du en valfri **kommentar**och väljer **Ja**. Om du väljer **Nej**händer ingenting. 

    ![Sidan Godkänn anslutning](./media/private-link-service/approve-connection-page.png)
5. Du bör se status för anslutningen i listan ändrad till **godkänd**. 

    ![Anslutnings status-godkänd](./media/private-link-service/connection-status-approved.png)

### <a name="reject-a-private-endpoint-connection"></a>Avvisa en privat slut punkts anslutning

1. Om det finns anslutningar för privata slut punkter som du vill avvisa, oavsett om det är en väntande begäran eller en befintlig anslutning som har godkänts tidigare, väljer du slut punkts anslutningen och klickar på knappen **avvisa** .

    ![Knappen avvisa](./media/private-link-service/private-endpoint-reject.png)
2. På sidan **avvisa anslutning** anger du en valfri kommentar och väljer **Ja**. Om du väljer **Nej**händer ingenting. 

    ![Sidan avvisa anslutning](./media/private-link-service/reject-connection-page.png)
3. Du bör se status för anslutningen i listan ändrad **avvisad**. 

    ![Avvisad slut punkt](./media/private-link-service/endpoint-rejected.png)


### <a name="remove-a-private-endpoint-connection"></a>Ta bort en privat slut punkts anslutning

1. Om du vill ta bort en privat slut punkts anslutning markerar du den i listan och väljer **ta bort** i verktygsfältet. 

    ![Knappen Ta bort](./media/private-link-service/remove-endpoint.png)
2. På sidan **ta bort anslutning** väljer du **Ja** för att bekräfta borttagningen av den privata slut punkten. Om du väljer **Nej**händer ingenting. 

    ![Sidan ta bort anslutning](./media/private-link-service/delete-connection-page.png)
3. Du bör se statusen ändrad till **frånkopplad**. Sedan visas slut punkten kvar från listan. 

## <a name="validate-that-the-private-link-connection-works"></a>Kontrol lera att anslutningen till den privata länken fungerar

Du bör kontrol lera att resurserna i samma undernät i den privata slut punkts resursen ansluter till Service Bus namn området över en privat IP-adress och att de har rätt integrering av privata DNS-zoner.

Börja med att skapa en virtuell dator genom att följa stegen i [skapa en virtuell Windows-dator i Azure Portal](../virtual-machines/windows/quick-create-portal.md)

På fliken **nätverk** :

1. Ange **virtuellt nätverk** och **undernät**. Du kan skapa ett nytt virtuellt nätverk eller välja ett befintligt. Om du väljer en befintlig, se till att regionen stämmer.
1. Ange en **offentlig IP-** resurs.
1. Välj **ingen**för nätverkskortets **nätverks säkerhets grupp**.
1. För **belastnings utjämning**väljer du **Nej**.

Öppna kommando raden och kör följande kommando:

```console
nslookup <your-service-bus-namespace-name>.servicebus.windows.net
```

Om du kör kommandot ns lookup för att matcha IP-adressen för ett Service Bus namn område över en offentlig slut punkt visas ett resultat som ser ut så här:

```console
c:\ >nslookup <your-service-bus-namespace-name>.servicebus.windows.net

Non-authoritative answer:
Name:    
Address:  (public IP address)
Aliases:  <your-service-bus-namespace-name>.servicebus.windows.net
```

Om du kör kommandot ns lookup för att matcha IP-adressen för ett Service Bus namn område över en privat slut punkt visas ett resultat som ser ut så här:

```console
c:\ >nslookup your_service-bus-namespace-name.servicebus.windows.net

Non-authoritative answer:
Name:    
Address:  10.1.0.5 (private IP address)
Aliases:  <your-service-bus-namespace-name>.servicebus.windows.net
```

## <a name="limitations-and-design-considerations"></a>Begränsningar och design överväganden

**Priser**: information om priser finns i [priser för privata Azure-länkar](https://azure.microsoft.com/pricing/details/private-link/).

**Begränsningar**: den privata slut punkten för Azure Service Bus finns i en offentlig för hands version. Den här funktionen är tillgänglig i alla offentliga Azure-regioner.

**Maximalt antal privata slut punkter per Service Bus namnrymd**: 120.

Mer information finns i [Azure Private Link service: begränsningar](../private-link/private-link-service-overview.md#limitations)

## <a name="next-steps"></a>Nästa steg

- Läs mer om [Azures privata länk](../private-link/private-link-service-overview.md)
- Läs mer om [Azure Service Bus](service-bus-messaging-overview.md)
