---
title: Distribuera och konfigurera Azure-brandväggen med Azure CLI
description: I den här artikeln får du lära dig hur du distribuerar och konfigurerar Azure-brandväggen med hjälp av Azure CLI.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 08/29/2019
ms.author: victorh
ms.topic: article
ms.openlocfilehash: e97783d1a32916cad151f1d0858a8190d0005fd0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "73831970"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-cli"></a>Distribuera och konfigurera Azure-brandväggen med Azure CLI

En viktig del av en övergripande säkerhetsplan för nätverket är att kontrollera utgående nätverksåtkomst. Du kanske till exempel vill begränsa åtkomsten till webbplatser. Eller så kanske du vill begränsa de utgående IP-adresserna och portarna som kan nås.

Med Azure Firewall kan du kontrollera åtkomsten till utgående nätverk från ett Azure-undernät. Med Azure Firewall kan du konfigurera:

* Programreglerna som definierar fullständigt kvalificerade domännamn (FQDN) kan nås från ett undernät. FQDN kan även [innehålla SQL-instanser](sql-fqdn-filtering.md).
* Nätverksregler som definierar källadress, protokoll, målport och måladress.

Nätverkstrafiken måste följa konfigurerade brandväggsregler när du vidarebefordrar den till brandväggen som standardgateway för undernätet.

I den här artikeln skapar du ett förenklat enda VNet med tre undernät för enkel distribution. För produktions distributioner rekommenderas en [nav-och ekrars modell](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) . Brand väggen finns i ett eget VNet. Arbets belastnings servrarna finns i peer-virtuella nätverk i samma region med ett eller flera undernät.

* **AzureFirewallSubnet** – brandväggen ligger i det här undernätet.
* **Workload-SN** – arbetsbelastningsservern ligger i det här undernätet. Det här undernätets nätverkstrafik går genom brandväggen.
* **Jump-SN** – ”hopp”-servern ligger i det här undernätet. Hoppservern har en offentlig IP-adress som du kan ansluta till via Fjärrskrivbord. Därifrån kan du sedan ansluta till arbetsbelastningsservern (via ett annat Fjärrskrivbord).

![Självstudie om nätverksinfrastruktur](media/tutorial-firewall-rules-portal/Tutorial_network.png)

I den här artikeln kan du se hur du:

> [!div class="checklist"]
> * konfigurera en testnätverksmiljö
> * distribuera en brandvägg
> * Skapa en standardväg
> * Konfigurera en program regel för att tillåta åtkomst till www.google.com
> * Konfigurera en nätverksregel för att tillåta åtkomst till externa DNS-servrar
> * testa brandväggen.

Om du vill kan du slutföra den här proceduren med hjälp av [Azure Portal](tutorial-firewall-deploy-portal.md) eller [Azure PowerShell](deploy-ps.md).

Om du inte har en Azure-prenumeration kan du skapa ett [kostnadsfritt](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) konto innan du börjar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Krav

### <a name="azure-cli"></a>Azure CLI

Om du väljer att installera och använda CLI lokalt kör du Azure CLI version 2.0.4 eller senare. Du hittar versionen genom att köra **AZ--version**. Information om att installera eller uppgradera finns i [Installera Azure CLI]( /cli/azure/install-azure-cli).

Installera Azure Firewall-tillägget:

```azurecli-interactive
az extension add -n azure-firewall
```


## <a name="set-up-the-network"></a>Konfigurera nätverket

Skapa först en resursgrupp som ska innehålla de resurser som behövs till att distribuera brandväggen. Sedan skapa ett virtuellt nätverk, undernät och testservrar.

### <a name="create-a-resource-group"></a>Skapa en resursgrupp

Resurs gruppen innehåller alla resurser för distributionen.

```azurecli-interactive
az group create --name Test-FW-RG --location eastus
```

### <a name="create-a-vnet"></a>Skapa ett virtuellt nätverk

Det här virtuella nätverket har tre undernät.

> [!NOTE]
> Storleken på AzureFirewallSubnet-undernätet är/26. Mer information om under näts storleken finns i [vanliga frågor och svar om Azure Firewall](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

```azurecli-interactive
az network vnet create \
  --name Test-FW-VN \
  --resource-group Test-FW-RG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name AzureFirewallSubnet \
  --subnet-prefix 10.0.1.0/26
az network vnet subnet create \
  --name Workload-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.2.0/24
az network vnet subnet create \
  --name Jump-SN \
  --resource-group Test-FW-RG \
  --vnet-name Test-FW-VN   \
  --address-prefix 10.0.3.0/24
```

### <a name="create-virtual-machines"></a>Skapa virtuella datorer

Skapa nu de virtuella hopp- och arbetsbelastningsdatorerna och placera dem i respektive undernät.
När du uppmanas till det anger du ett lösen ord för den virtuella datorn.

Skapa den virtuella datorn för SRV-hopp.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Jump \
    --location eastus \
    --image win2016datacenter \
    --vnet-name Test-FW-VN \
    --subnet Jump-SN \
    --admin-username azureadmin
az vm open-port --port 3389 --resource-group Test-FW-RG --name Srv-Jump
```



Skapa ett nätverkskort för SRV-arbete med vissa DNS-Server-IP-adresser och ingen offentlig IP-adress att testa med.

```azurecli-interactive
az network nic create \
    -g Test-FW-RG \
    -n Srv-Work-NIC \
   --vnet-name Test-FW-VN \
   --subnet Workload-SN \
   --public-ip-address "" \
   --dns-servers 209.244.0.3 209.244.0.4
```

Nu skapar du den virtuella arbets belastnings datorn.
När du uppmanas till det anger du ett lösen ord för den virtuella datorn.

```azurecli-interactive
az vm create \
    --resource-group Test-FW-RG \
    --name Srv-Work \
    --location eastus \
    --image win2016datacenter \
    --nics Srv-Work-NIC \
    --admin-username azureadmin
```

## <a name="deploy-the-firewall"></a>Distribuera brandväggen

Distribuera nu brand väggen till det virtuella nätverket.

```azurecli-interactive
az network firewall create \
    --name Test-FW01 \
    --resource-group Test-FW-RG \
    --location eastus
az network public-ip create \
    --name fw-pip \
    --resource-group Test-FW-RG \
    --location eastus \
    --allocation-method static \
    --sku standard
az network firewall ip-config create \
    --firewall-name Test-FW01 \
    --name FW-config \
    --public-ip-address fw-pip \
    --resource-group Test-FW-RG \
    --vnet-name Test-FW-VN
az network firewall update \
    --name Test-FW01 \
    --resource-group Test-FW-RG 
az network public-ip show \
    --name fw-pip \
    --resource-group Test-FW-RG
fwprivaddr="$(az network firewall ip-config list -g Test-FW-RG -f Test-FW01 --query "[?name=='FW-config'].privateIpAddress" --output tsv)"
```

Skriv ned den privata IP-adressen. Du kommer att använda den senare när du skapar standardvägen.

## <a name="create-a-default-route"></a>Skapa en standardväg

Skapa en tabell med avaktiverad BGP Route-spridning

```azurecli-interactive
az network route-table create \
    --name Firewall-rt-table \
    --resource-group Test-FW-RG \
    --location eastus \
    --disable-bgp-route-propagation true
```

Skapa vägen.

```azurecli-interactive
az network route-table route create \
  --resource-group Test-FW-RG \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $fwprivaddr
```

Koppla routningstabellen till under nätet

```azurecli-interactive
az network vnet subnet update \
    -n Workload-SN \
    -g Test-FW-RG \
    --vnet-name Test-FW-VN \
    --address-prefixes 10.0.2.0/24 \
    --route-table Firewall-rt-table
```

## <a name="configure-an-application-rule"></a>Konfigurera en programregel

Med program regeln tillåts utgående åtkomst till www.google.com.

```azurecli-interactive
az network firewall application-rule create \
   --collection-name App-Coll01 \
   --firewall-name Test-FW01 \
   --name Allow-Google \
   --protocols Http=80 Https=443 \
   --resource-group Test-FW-RG \
   --target-fqdns www.google.com \
   --source-addresses 10.0.2.0/24 \
   --priority 200 \
   --action Allow
```

Azure Firewall innehåller en inbyggd regelsamling för fullständiga domännamn för mål (FQDN) i infrastrukturen som tillåts som standard. Dessa FQDN är specifika för plattformen och kan inte användas för andra ändamål. Mer information finns i [Infrastruktur-FQDN](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Konfigurera en nätverksregel

Nätverks regeln tillåter utgående åtkomst till två IP-adresser i port 53 (DNS).

```azurecli-interactive
az network firewall network-rule create \
   --collection-name Net-Coll01 \
   --destination-addresses 209.244.0.3 209.244.0.4 \
   --destination-ports 53 \
   --firewall-name Test-FW01 \
   --name Allow-DNS \
   --protocols UDP \
   --resource-group Test-FW-RG \
   --priority 200 \
   --source-addresses 10.0.2.0/24 \
   --action Allow
```

## <a name="test-the-firewall"></a>testa brandväggen.

Testa nu brand väggen för att bekräfta att den fungerar som förväntat.

1. Observera den privata IP-adressen för den virtuella **SRV-Work-** datorn:

   ```azurecli-interactive
   az vm list-ip-addresses \
   -g Test-FW-RG \
   -n Srv-Work
   ```

1. Anslut ett fjärr skrivbord till en virtuell dator med **SRV-hopp** och logga in. Därifrån öppnar du en fjärr skrivbords anslutning till den privata IP-adressen för **SRV-arbete** och loggar in.

3. Öppna ett PowerShell-fönster i **SRV-arbete**och kör följande kommandon:

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Båda kommandona ska returnera svar, vilket visar att dina DNS-frågor går igenom brand väggen.

1. Kör följande kommandon:

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   `www.google.com` Begär Anden ska lyckas och `www.microsoft.com` begär Anden ska Miss lyckas. Detta visar att brand Väggs reglerna fungerar som förväntat.

Nu har du verifierat att brand Väggs reglerna fungerar:

* Du kan omvandla DNS-namn med hjälp av den konfigurerade externa DNS-servern.
* Du kan bläddra till en tillåten FQDN, men inte till andra.

## <a name="clean-up-resources"></a>Rensa resurser

Du kan behålla dina brand Väggs resurser för nästa självstudie eller, om du inte längre behöver, ta bort resurs gruppen **test-VB-RG** för att ta bort alla brand Väggs resurser:

```azurecli-interactive
az group delete \
  -n Test-FW-RG
```

## <a name="next-steps"></a>Nästa steg

* [Självstudie: Övervaka Azure Firewall-loggar](./tutorial-diagnostics.md)
