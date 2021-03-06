---
title: Nätverks isolering & sekretess
titleSuffix: Azure Machine Learning
description: Använd en isolerad Azure-Virtual Network med Azure Machine Learning för att skydda experiment/utbildning samt jobb för att begränsa/göra poäng.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 05/11/2020
ms.custom: contperfq4
ms.openlocfilehash: 5099cc2ce2228bcdbf49d3484e488e7373883ec0
ms.sourcegitcommit: a8ee9717531050115916dfe427f84bd531a92341
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/12/2020
ms.locfileid: "83119045"
---
# <a name="secure-your-machine-learning-lifecycles-with-private-virtual-networks"></a>Skydda dina Machine Learning-livscykler med privata virtuella nätverk
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

I den här artikeln får du lära dig hur du isolerar experimentering/utbildnings jobb och jobb för jobb härledning/-Poäng i Azure Machine Learning i ett Azure-Virtual Network (VNet). Du lär dig också om några *avancerade säkerhets inställningar*, information som inte behövs för grundläggande eller experiment användnings fall.

> [!WARNING]
> Om det underliggande lagrings utrymmet finns i ett virtuellt nätverk, kan användarna inte använda Azure Machine Learning Studio-webbmiljön, inklusive:
> - Dra-n-släpp designer
> - Användar gränssnitt för automatisk maskin inlärning
> - GRÄNSSNITT för data etiketter
> - Användar gränssnitt för data uppsättningar
> - Notebooks
> 
> Om du försöker igen får du ett meddelande som liknar följande fel:`__Error: Unable to profile this dataset. This might be because your data is stored behind a virtual network or your data does not support profile.__`

## <a name="what-is-a-vnet"></a>Vad är ett VNET?

Ett **virtuellt nätverk** fungerar som en säkerhets gränser som isolerar dina Azure-resurser från det offentliga Internet. Du kan också ansluta ett virtuellt Azure-nätverk till ditt lokala nätverk. Genom att ansluta till nätverk kan du på ett säkert sätt träna dina modeller och komma åt dina distribuerade modeller för att få en mer härledning.

Azure Machine Learning förlitar sig på andra Azure-tjänster för beräknings resurser, även kallade [beräknings mål](concept-compute-target.md), för att träna och distribuera modeller. Målen kan skapas i ett virtuellt nätverk. Du kan till exempel använda Azure Machine Learning Compute för att träna en modell och sedan distribuera modellen till Azure Kubernetes service (AKS). 


## <a name="prerequisites"></a>Krav

+ En Azure Machine Learning- [arbetsyta](how-to-manage-workspace.md).

+ Allmänt fungerande kunskap om både [Azure Virtual Network-tjänsten](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview) och [IP-nätverk](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm).

+ Ett befintligt virtuellt nätverk och undernät som ska användas med dina beräknings resurser.

## <a name="private-endpoints"></a>Privata slut punkter

Du kan också [Aktivera Azure Private-länken](how-to-configure-private-link.md) för att ansluta till din arbets yta med en privat slut punkt. Den privata slut punkten är en uppsättning privata IP-adresser i det virtuella nätverket. [Lär dig hur du konfigurerar den här privata slut punkten.](how-to-configure-private-link.md)



> [!TIP]
> Du kan kombinera virtuella nätverk och privata länkar tillsammans för att skydda kommunikationen mellan arbets ytan och andra Azure-resurser. Vissa kombinationer kräver dock en Enterprise Edition-arbetsyta. Använd följande tabell för att ta reda på vilka scenarier som kräver Enterprise Edition:
>
> | Scenario | Enterprise</br>Edition | Basic</br>Edition |
> | ----- |:-----:|:-----:| 
> | Inget virtuellt nätverk eller en privat länk | ✔ | ✔ |
> | Arbets yta utan privat länk. Andra resurser (utom Azure Container Registry) i ett virtuellt nätverk | ✔ | ✔ |
> | Arbets yta utan privat länk. Andra resurser med privat länk | ✔ | |
> | Arbets yta med privat länk. Andra resurser (utom Azure Container Registry) i ett virtuellt nätverk | ✔ | ✔ |
> | Arbets yta och andra resurser med privat länk | ✔ | |
> | Arbets yta med privat länk. Andra resurser utan privat länk eller virtuellt nätverk | ✔ | ✔ |
> | Azure Container Registry i ett virtuellt nätverk | ✔ | |
> | Kundhanterade nycklar för arbets ytan | ✔ | |
> 

> [!WARNING]
> För hands versionen av Azure Machine Learning Compute instances stöds inte i en arbets yta där privat länk är aktive rad.
> 
> Azure Machine Learning stöder inte användning av en Azure Kubernetes-tjänst som har privat länk aktive rad. I stället kan du använda Azure Kubernetes-tjänsten i ett virtuellt nätverk. Mer information finns i [skydda Azure ml-experimentering och härlednings jobb i en Azure-Virtual Network](how-to-enable-virtual-network.md).


<a id="amlcompute"></a>

## <a name="compute-clusters--instances"></a><a name="compute-instance"></a>Beräknings kluster & instanser

Om du vill använda en [hanterad Azure Machine Learning **beräknings mål** ](concept-compute-target.md#azure-machine-learning-compute-managed) eller en [Azure Machine Learning beräknings **instans** ](concept-compute-instance.md) i ett virtuellt nätverk måste följande nätverks krav uppfyllas:

> [!div class="checklist"]
> * Det virtuella nätverket måste finnas i samma prenumeration och region som Azure Machine Learning-arbetsytan.
> * Under nätet som anges för beräknings instansen eller klustret måste ha tillräckligt många otilldelade IP-adresser för att rymma antalet virtuella datorer som är riktade. Om under nätet inte har tillräckligt med otilldelade IP-adresser, tilldelas ett beräknings kluster delvis.
> * Kontrol lera om dina säkerhets principer eller lås på det virtuella nätverkets prenumeration eller resurs grupp begränsar behörigheter för hantering av det virtuella nätverket. Om du planerar att skydda det virtuella nätverket genom att begränsa trafiken lämnar du vissa portar öppna för beräknings tjänsten. Mer information finns i avsnittet [nödvändiga portar](#mlcports) .
> * Om du ska lagra flera beräknings instanser eller kluster i ett virtuellt nätverk kan du behöva begära en kvot ökning för en eller flera av dina resurser.
> * Om Azure Storage kontona för arbets ytan också är skyddade i ett virtuellt nätverk måste de finnas i samma virtuella nätverk som Azure Machine Learning beräknings instans eller kluster. 

> [!TIP]
> Machine Learning beräknings instans eller kluster allokerar automatiskt ytterligare nätverks resurser __i resurs gruppen som innehåller det virtuella nätverket__. För varje beräknings instans eller kluster allokerar tjänsten följande resurser:
> 
> * En nätverks säkerhets grupp
> * En offentlig IP-adress
> * En belastningsutjämnare
> 
> I kluster är de här resurserna borttagna (och återskapas) varje gång klustret skalar ned till 0 noder, men för en instans är resurserna kvar på till instansen helt borttagna (stoppa tar inte bort resurserna). 
> Dessa resurser begränsas av prenumerationens [resurskvoter](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits).


### <a name="required-ports"></a><a id="mlcports"></a>Portar som krävs

Machine Learning-beräkning använder för närvarande tjänsten Azure Batch för att etablera virtuella datorer i det angivna virtuella nätverket. Under nätet måste tillåta inkommande kommunikation från batch-tjänsten. Du använder den här kommunikationen för att schemalägga körningar på Machine Learning-beräkning noder och för att kommunicera med Azure Storage och andra resurser. Batch-tjänsten lägger till nätverks säkerhets grupper (NSG: er) på nivån nätverks gränssnitt (NIC) som är anslutna till virtuella datorer. De här NSG:erna konfigurerar automatiskt regler för inkommande och utgående trafik för att tillåta följande trafik:

- Inkommande TCP-trafik på portarna 29876 och 29877 från en __service tag__ i __BatchNodeManagement__.

    ![En regel för inkommande trafik som använder BatchNodeManagement-tjänst tag gen](./media/how-to-enable-virtual-network/batchnodemanagement-service-tag.png)

- Valfritt Inkommande TCP-trafik på port 22 för att tillåta fjärråtkomst. Använd bara den här porten om du vill ansluta med SSH på den offentliga IP-adressen.

- Utgående trafik på vilken port som helst till det virtuella nätverket.

- Utgående trafik på vilken port som helst till Internet.

- För Compute instance inkommande TCP-trafik på port 44224 från en __service tag__ i __AzureMachineLearning__.

Var försiktig om du ändrar eller lägger till regler för inkommande eller utgående trafik i Batch-konfigurerade NSG:er. Om en NSG blockerar kommunikation till datornoderna, anger beräknings tjänsten status för datornoderna till oanvändbar.

Du behöver inte ange NSG: er på under näts nivån, eftersom Azure Batch tjänsten konfigurerar sin egen NSG: er. Men om det angivna under nätet har en associerad NSG: er eller en brand vägg, konfigurerar du de inkommande och utgående säkerhets reglerna enligt ovan.

Regel konfigurationen för NSG i Azure Portal visas i följande avbildningar:

:::image type="content" source="./media/how-to-enable-virtual-network/amlcompute-virtual-network-inbound.png" alt-text="Regler för inkommande NSG för Machine Learning-beräkning" border="true":::



![Utgående NSG-regler för Machine Learning-beräkning](./media/how-to-enable-virtual-network/experimentation-virtual-network-outbound.png)

### <a name="limit-outbound-connectivity-from-the-virtual-network"></a><a id="limiting-outbound-from-vnet"></a>Begränsa utgående anslutning från det virtuella nätverket

Använd följande steg om du inte vill använda de utgående standard reglerna och du vill begränsa den utgående åtkomsten för ditt virtuella nätverk:

- Neka utgående Internet anslutning med NSG-reglerna.

- För en __beräknings instans__ eller ett __beräknings kluster__begränsar du utgående trafik till följande objekt:
   - Azure Storage med hjälp av __tjänst tag gen__ för __Storage. RegionName__. Där `{RegionName}` är namnet på en Azure-region.
   - Azure Container Registry med hjälp av __service tag gen__ för __AzureContainerRegistry. RegionName__. Där `{RegionName}` är namnet på en Azure-region.
   - Azure Machine Learning med hjälp av __service tag gen__ för __AzureMachineLearning__
   - Azure Resource Manager med hjälp av __service tag gen__ för __AzureResourceManager__
   - Azure Active Directory med hjälp av __service tag gen__ för __AzureActiveDirectory__

Regel konfigurationen för NSG i Azure Portal visas i följande bild:

[![Utgående NSG-regler för Machine Learning-beräkning](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png)](./media/how-to-enable-virtual-network/limited-outbound-nsg-exp.png#lightbox)

> [!NOTE]
> Om du planerar att använda standard Docker-avbildningar som tillhandahålls av Microsoft och aktiverar hanterade beroenden, måste du också använda en __service tag__ - __MicrosoftContainerRegistry. Region_Name__ (till exempel MicrosoftContainerRegistry. öster).
>
> Den här konfigurationen krävs när du har kod som liknar följande kodfragment som en del av dina utbildnings skript:
>
> __RunConfig-utbildning__
> ```python
> # create a new runconfig object
> run_config = RunConfiguration()
> 
> # configure Docker 
> run_config.environment.docker.enabled = True
> # For GPU, use DEFAULT_GPU_IMAGE
> run_config.environment.docker.base_image = DEFAULT_CPU_IMAGE 
> run_config.environment.python.user_managed_dependencies = True
> ```
>
> __Uppskattnings utbildning__
> ```python
> est = Estimator(source_directory='.',
>                 script_params=script_params,
>                 compute_target='local',
>                 entry_script='dummy_train.py',
>                 user_managed=True)
> run = exp.submit(est)
> ```

### <a name="user-defined-routes-for-forced-tunneling"></a>Användardefinierade vägar för Tvingad tunnel trafik

Om du använder Tvingad tunnel trafik med Machine Learning-beräkning lägger du till [användardefinierade vägar (UDR)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-udr-overview) i det undernät som innehåller beräknings resursen.

* Upprätta en UDR för varje IP-adress som används av tjänsten Azure Batch i den region där dina resurser finns. Dessa UDR gör att batch-tjänsten kan kommunicera med datornoder för schemaläggning av aktiviteter. Lägg också till IP-adressen för den Azure Machine Learning tjänst där resurserna finns, eftersom detta krävs för åtkomst till beräknings instanser. Använd någon av följande metoder för att hämta en lista över IP-adresser för batch-tjänsten och Azure Machine Learning tjänsten:

    * Hämta [Azure IP-intervall och service märken](https://www.microsoft.com/download/details.aspx?id=56519) och Sök efter `BatchNodeManagement.<region>` och `AzureMachineLearning.<region>` , där `<region>` är din Azure-region.

    * Använd [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) för att hämta informationen. I följande exempel hämtas IP-adress informationen och filtreras bort informationen för regionen USA, östra 2:

        ```azurecli-interactive
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'Batch')] | [?properties.region=='eastus2']"
        az network list-service-tags -l "East US 2" --query "values[?starts_with(id, 'AzureMachineLearning')] | [?properties.region=='eastus2']"
        ```

* Utgående trafik till Azure Storage får inte blockeras av den lokala nätverks enheten. Mer specifikt är webb adresserna i formatet `<account>.table.core.windows.net` , `<account>.queue.core.windows.net` och `<account>.blob.core.windows.net` .

När du lägger till UDR definierar du vägen för varje relaterat batch-IP-adressprefix och anger __nästa hopp typ__ till __Internet__. Följande bild visar ett exempel på den här UDR i Azure Portal:

![Exempel på en UDR för ett adressprefix](./media/how-to-enable-virtual-network/user-defined-route.png)

Mer information finns i [skapa en Azure Batch pool i ett virtuellt nätverk](../batch/batch-virtual-network.md#user-defined-routes-for-forced-tunneling).

### <a name="create-a-compute-cluster-in-a-virtual-network"></a>Skapa ett beräknings kluster i ett virtuellt nätverk

Använd följande steg för att skapa ett Machine Learning-beräkning kluster:

1. Logga in på [Azure Machine Learning Studio](https://ml.azure.com/)och välj sedan din prenumeration och arbets yta.

1. Välj __Beräkna__ till vänster.

1. Välj __utbildnings kluster__ från mitten och välj sedan __+__ .

1. I dialog rutan __nytt utbildnings kluster__ expanderar du avsnittet __Avancerade inställningar__ .

1. Om du vill konfigurera den här beräknings resursen för att använda ett virtuellt nätverk utför du följande åtgärder i avsnittet __Konfigurera virtuellt nätverk__ :

    1. I list rutan __resurs grupp__ väljer du den resurs grupp som innehåller det virtuella nätverket.
    1. I list rutan __virtuellt nätverk__ väljer du det virtuella nätverk som innehåller under nätet.
    1. I list rutan __undernät__ väljer du det undernät som ska användas.

   ![Inställningarna för virtuella nätverk för Machine Learning-beräkning](./media/how-to-enable-virtual-network/amlcompute-virtual-network-screen.png)

Du kan också skapa ett Machine Learning-beräkning-kluster med hjälp av Azure Machine Learning SDK. Följande kod skapar ett nytt Machine Learning-beräkning-kluster i `default` under nätet för ett virtuellt nätverk med namnet `mynetwork` :

```python
from azureml.core.compute import ComputeTarget, AmlCompute
from azureml.core.compute_target import ComputeTargetException

# The Azure virtual network name, subnet, and resource group
vnet_name = 'mynetwork'
subnet_name = 'default'
vnet_resourcegroup_name = 'mygroup'

# Choose a name for your CPU cluster
cpu_cluster_name = "cpucluster"

# Verify that cluster does not exist already
try:
    cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
    print("Found existing cpucluster")
except ComputeTargetException:
    print("Creating new cpucluster")

    # Specify the configuration for the new cluster
    compute_config = AmlCompute.provisioning_configuration(vm_size="STANDARD_D2_V2",
                                                           min_nodes=0,
                                                           max_nodes=4,
                                                           vnet_resourcegroup_name=vnet_resourcegroup_name,
                                                           vnet_name=vnet_name,
                                                           subnet_name=subnet_name)

    # Create the cluster with the specified name and configuration
    cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

    # Wait for the cluster to be completed, show the output log
    cpu_cluster.wait_for_completion(show_output=True)
```

När du har skapat processen tränar du din modell genom att använda klustret i ett experiment. Mer information finns i [Välj och använda ett beräknings mål för utbildning](how-to-set-up-training-targets.md).

## <a name="use-a-storage-account-for-your-workspace"></a>Använd ett lagrings konto för din arbets yta

Använd följande steg för att använda ett Azure Storage-konto för arbets ytan i ett virtuellt nätverk:

1. Skapa en beräknings resurs (till exempel en Machine Learning beräknings instans eller kluster) bakom ett virtuellt nätverk eller Anslut en beräknings resurs till arbets ytan (till exempel ett HDInsight-kluster, en virtuell dator eller ett Azure Kubernetes service-kluster). Beräknings resursen kan vara till för experimentering eller modell distribution.

   Mer information finns i avsnittet [använda en Machine Learning Compute](#amlcompute), [använda en virtuell dator eller HDInsight-kluster](#vmorhdi)och [använda Azure Kubernetes service-](#aksvnet) avsnitten i den här artikeln.

1. I Azure Portal går du till den lagring som är kopplad till din arbets yta.

   [![Lagrings utrymmet som är kopplat till arbets ytan Azure Machine Learning](./media/how-to-enable-virtual-network/workspace-storage.png)](./media/how-to-enable-virtual-network/workspace-storage.png#lightbox)

1. På sidan **Azure Storage** väljer du __brand väggar och virtuella nätverk__.

   ![Avsnittet "brand väggar och virtuella nätverk" på sidan Azure Storage i Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks.png)

1. På sidan __brand väggar och virtuella nätverk__ utför du följande åtgärder:
    - Välj __Valda nätverk__.
    - Under __virtuella nätverk__väljer du länken __Lägg till befintligt virtuellt nätverk__ . Den här åtgärden lägger till det virtuella nätverk där din beräkning finns (se steg 1).

        > [!IMPORTANT]
        > Lagrings kontot måste finnas i samma virtuella nätverk och undernät som de beräknings instanser eller kluster som används för utbildning eller härledning.

    - Markera kryss rutan __Tillåt att betrodda Microsoft-tjänster har åtkomst till det här lagrings kontot__ .

    > [!IMPORTANT]
    > När du arbetar med Azure Machine Learning SDK måste utvecklings miljön kunna ansluta till Azure Storage-kontot. När lagrings kontot finns i ett virtuellt nätverk måste brand väggen tillåta åtkomst från utvecklings miljöns IP-adress.
    >
    > Om du vill aktivera åtkomst till lagrings kontot går du till __brand väggarna och de virtuella nätverken__ för lagrings kontot *från en webbläsare på utvecklings klienten*. Använd sedan kryss rutan __Lägg till din klient-IP-adress__ för att lägga till KLIENTens IP-adress i __adress intervallet__. Du kan också använda fältet __adress intervall__ för att manuellt ange IP-adressen för utvecklings miljön. När IP-adressen för klienten har lagts till kan den komma åt lagrings kontot med hjälp av SDK.

   [![Fönstret "brand väggar och virtuella nätverk" i Azure Portal](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/storage-firewalls-and-virtual-networks-page.png#lightbox)

> [!IMPORTANT]
> Du kan placera både _standard lagrings kontot_ för Azure Machine Learning eller _andra lagrings konton som inte är standard_ i ett virtuellt nätverk.
>
> Standard lagrings kontot tillhandahålls automatiskt när du skapar en arbets yta.
>
> För lagrings konton som inte är standard `storage_account` kan du ange ett anpassat lagrings konto per Azure-resurs-ID i-parametern i [ `Workspace.create()` funktionen](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace(class)?view=azure-ml-py#create-name--auth-none--subscription-id-none--resource-group-none--location-none--create-resource-group-true--sku--basic---friendly-name-none--storage-account-none--key-vault-none--app-insights-none--container-registry-none--cmk-keyvault-none--resource-cmk-uri-none--hbi-workspace-false--default-cpu-compute-target-none--default-gpu-compute-target-none--exist-ok-false--show-output-true-) .


<a id="aksvnet"></a>

## <a name="azure-kubernetes-service"></a>Azure Kubernetes Service

Använd följande steg för att lägga till Azure Kubernetes service (AKS) i ett virtuellt nätverk till din arbets yta:

> [!IMPORTANT]
> Innan du påbörjar följande procedur följer du kraven i avsnittet [Konfigurera avancerade nätverksfunktioner i Azure Kubernetes service (AKS)](https://docs.microsoft.com/azure/aks/configure-azure-cni#prerequisites) How-to och planera IP-adressering för klustret.
>
> AKS-instansen och det virtuella Azure-nätverket måste finnas i samma region. Om du skyddar de Azure Storage-konton som används av arbets ytan i ett virtuellt nätverk måste de finnas i samma virtuella nätverk som AKS-instansen.

> [!WARNING]
> Azure Machine Learning stöder inte användning av en Azure Kubernetes-tjänst som har privat länk aktive rad.

1. Logga in på [Azure Machine Learning Studio](https://ml.azure.com/)och välj sedan din prenumeration och arbets yta.

1. Välj __Beräkna__ till vänster.

1. Välj __härlednings kluster__ från mitten och välj sedan __+__ .

1. I dialog rutan __nytt kluster__ för __överbelastnings__kluster väljer du __Avancerat__ under nätverks konfiguration.

1. Utför följande åtgärder för att konfigurera beräknings resursen för att använda ett virtuellt nätverk:

    1. I list rutan __resurs grupp__ väljer du den resurs grupp som innehåller det virtuella nätverket.
    1. I list rutan __virtuellt nätverk__ väljer du det virtuella nätverk som innehåller under nätet.
    1. I list rutan __undernät__ väljer du under nätet.
    1. I rutan __Kubernetes service-adressintervall__ anger du tjänst adress intervallet för Kubernetes. Det här adress intervallet använder en klass lös CIDR-notation (Inter-Domain routing) för att definiera de IP-adresser som är tillgängliga för klustret. Det får inte överlappa några IP-adressintervall för undernät (till exempel 10.0.0.0/16).
    1. I rutan __IP-adress för KUBERNETES DNS-tjänst__ anger du IP-adressen för DNS-tjänsten Kubernetes. Den här IP-adressen är tilldelad DNS-tjänsten Kubernetes. Det måste ligga inom Kubernetes-tjänstens adress intervall (till exempel 10.0.0.10).
    1. I rutan __Docker Bridge-adress__ anger du Docker-bryggans adress. Den här IP-adressen är tilldelad Docker-bryggan. Det får inte finnas i några IP-adressintervall för undernät eller Kubernetes-tjänstens adress intervall (till exempel 172.17.0.1/16).

   ![Azure Machine Learning: Machine Learning-beräkning inställningar för virtuellt nätverk](./media/how-to-enable-virtual-network/aks-virtual-network-screen.png)

1. Kontrol lera att NSG-gruppen som styr det virtuella nätverket har en inkommande säkerhets regel aktive rad för poäng slut punkten så att den kan anropas utanför det virtuella nätverket.
   > [!IMPORTANT]
   > Behåll de utgående standard reglerna för NSG. Mer information finns i standard säkerhets regler i [säkerhets grupper](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

   [![En inkommande säkerhets regel](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png)](./media/how-to-enable-virtual-network/aks-vnet-inbound-nsg-scoring.png#lightbox)

Du kan också använda Azure Machine Learning SDK för att lägga till Azure Kubernetes-tjänsten i ett virtuellt nätverk. Om du redan har ett AKS-kluster i ett virtuellt nätverk ansluter du det till arbets ytan enligt beskrivningen i [så här distribuerar du till AKS](how-to-deploy-and-where.md). Följande kod skapar en ny AKS-instans i `default` under nätet för ett virtuellt nätverk med namnet `mynetwork` :

```python
from azureml.core.compute import ComputeTarget, AksCompute

# Create the compute configuration and set virtual network information
config = AksCompute.provisioning_configuration(location="eastus2")
config.vnet_resourcegroup_name = "mygroup"
config.vnet_name = "mynetwork"
config.subnet_name = "default"
config.service_cidr = "10.0.0.0/16"
config.dns_service_ip = "10.0.0.10"
config.docker_bridge_cidr = "172.17.0.1/16"

# Create the compute target
aks_target = ComputeTarget.create(workspace=ws,
                                  name="myaks",
                                  provisioning_configuration=config)
```

När processen har skapats kan du köra en härledning eller modell bedömning i ett AKS-kluster bakom ett virtuellt nätverk. Mer information finns i [så här distribuerar du till AKS](how-to-deploy-and-where.md).

### <a name="use-private-ips-with-azure-kubernetes-service"></a>Använda privata IP-adresser med Azure Kubernetes-tjänsten

Som standard tilldelas en offentlig IP-adress till AKS-distributioner. När du använder AKS i ett virtuellt nätverk kan du använda en privat IP-adress i stället. Privata IP-adresser är bara tillgängliga inifrån det virtuella nätverket eller anslutna nätverk.

En privat IP-adress är aktive rad genom att konfigurera AKS för att använda en _intern belastningsutjämnare_. 

> [!IMPORTANT]
> Du kan inte aktivera privat IP när du skapar Azure Kubernetes service-klustret. Den måste vara aktive rad som en uppdatering av ett befintligt kluster.

Följande kodfragment visar hur du **skapar ett nytt AKS-kluster**och sedan uppdaterar det så att det använder en privat IP/intern belastningsutjämnare:

```python
import azureml.core
from azureml.core.compute.aks import AksUpdateConfiguration
from azureml.core.compute import AksCompute, ComputeTarget

# Verify that cluster does not exist already
try:
    aks_target = AksCompute(workspace=ws, name=aks_cluster_name)
    print("Found existing aks cluster")

except:
    print("Creating new aks cluster")

    # Subnet to use for AKS
    subnet_name = "default"
    # Create AKS configuration
    prov_config = AksCompute.provisioning_configuration(location = "eastus2")
    # Set info for existing virtual network to create the cluster in
    prov_config.vnet_resourcegroup_name = "myvnetresourcegroup"
    prov_config.vnet_name = "myvnetname"
    prov_config.service_cidr = "10.0.0.0/16"
    prov_config.dns_service_ip = "10.0.0.10"
    prov_config.subnet_name = subnet_name
    prov_config.docker_bridge_cidr = "172.17.0.1/16"

    # Create compute target
    aks_target = ComputeTarget.create(workspace = ws, name = "myaks", provisioning_configuration = prov_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
    
    # Update AKS configuration to use an internal load balancer
    update_config = AksUpdateConfiguration(None, "InternalLoadBalancer", subnet_name)
    aks_target.update(update_config)
    # Wait for the operation to complete
    aks_target.wait_for_completion(show_output = True)
```

__Azure CLI__

```azurecli-interactive
az rest --method put --uri https://management.azure.com"/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ContainerService/managedClusters/<aks-resource-id>?api-version=2018-11-19 --body @body.json
```

Innehållet i filen som `body.json` kommandot refererar till liknar följande JSON-dokument:

```json
{ 
    "location": "<region>", 
    "properties": { 
        "resourceId": "/subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ContainerService/managedClusters/<aks-resource-id>", 
        "computeType": "AKS", 
        "provisioningState": "Succeeded", 
        "properties": { 
            "loadBalancerType": "InternalLoadBalancer", 
            "agentCount": <agent-count>, 
            "agentVmSize": "vm-size", 
            "clusterFqdn": "<cluster-fqdn>" 
        } 
    } 
} 
```

> [!NOTE]
> För närvarande kan du inte konfigurera belastningsutjämnaren när du utför en __kopplings__ åtgärd i ett befintligt kluster. Du måste först ansluta klustret och sedan utföra en uppdaterings åtgärd för att ändra belastningsutjämnaren.

Mer information om hur du använder den interna belastningsutjämnaren med AKS finns i [använda intern belastningsutjämnare med Azure Kubernetes service](/azure/aks/internal-lb).

## <a name="use-azure-container-instances-aci"></a>Använda Azure Container Instances (ACI)

Azure Container Instances skapas dynamiskt när du distribuerar en modell. Om du vill aktivera Azure Machine Learning att skapa ACI i det virtuella nätverket måste du aktivera __under näts delegering__ för under nätet som används av distributionen.

Använd följande steg för att använda ACI i ett virtuellt nätverk på din arbets yta:

1. Om du vill aktivera under näts delegering i det virtuella nätverket använder du informationen i artikeln [Lägg till eller ta bort en under näts delegering](../virtual-network/manage-subnet-delegation.md) . Du kan aktivera delegering när du skapar ett virtuellt nätverk eller lägga till det i ett befintligt nätverk.

    > [!IMPORTANT]
    > När du aktiverar delegering ska `Microsoft.ContainerInstance/containerGroups` du använda som värde för __tjänsten delegera till tjänst__ .

2. Distribuera modellen med [AciWebservice. deploy_configuration ()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.webservice.aci.aciwebservice?view=azure-ml-py#deploy-configuration-cpu-cores-none--memory-gb-none--tags-none--properties-none--description-none--location-none--auth-enabled-none--ssl-enabled-none--enable-app-insights-none--ssl-cert-pem-file-none--ssl-key-pem-file-none--ssl-cname-none--dns-name-label-none--primary-key-none--secondary-key-none--collect-model-data-none--cmk-vault-base-url-none--cmk-key-name-none--cmk-key-version-none--vnet-name-none--subnet-name-none-) `vnet_name` och Använd `subnet_name` parametrarna och. Ange de här parametrarna som namn på det virtuella nätverket och under nätet där du aktiverade delegering.

## <a name="azure-firewall"></a>Azure Firewall

Information om hur du använder Azure Machine Learning med Azure-brandväggen finns i [använd Azure Machine Learning arbets yta bakom Azure-brandväggen](how-to-access-azureml-behind-firewall.md).

## <a name="azure-container-registry"></a>Azure Container Registry

> [!IMPORTANT]
> Azure Container Registry (ACR) kan placeras i ett virtuellt nätverk, men du måste uppfylla följande krav:
>
> * Din Azure Machine Learning-arbetsyta måste vara Enterprise Edition. Information om hur du uppgraderar finns i [Uppgradera till Enterprise Edition](how-to-manage-workspace.md#upgrade).
> * Din Azure Container Registry måste vara en Premium version. Mer information om hur du uppgraderar finns i [ändra SKU: er](/azure/container-registry/container-registry-skus#changing-skus).
> * Ditt Azure Container Registry måste finnas i samma virtuella nätverk och undernät som lagrings kontot och beräknings målen som används för utbildning eller härledning.
> * Din Azure Machine Learning-arbetsyta måste innehålla ett [Azure Machine Learning Compute-kluster](how-to-set-up-training-targets.md#amlcompute).
>
>     När ACR ligger bakom ett virtuellt nätverk kan Azure Machine Learning inte använda det för att direkt bygga Docker-avbildningar. I stället används beräknings klustret för att bygga avbildningarna.

1. Använd någon av följande metoder för att hitta namnet på Azure Container Registry för din arbets yta:

    __Azure Portal__

    I översikts avsnittet på arbets ytan länkar __registervärdet__ till Azure Container Registry.

    :::image type="content" source="./media/how-to-enable-virtual-network/azure-machine-learning-container-registry.png" alt-text="Azure Container Registry för arbets ytan" border="true":::

    __Azure CLI__

    Om du har [installerat Machine Learning-tillägget för Azure CLI](reference-azure-machine-learning-cli.md)kan du använda `az ml workspace show` kommandot för att visa information om arbets ytan.

    ```azurecli-interactive
    az ml workspace show -w yourworkspacename -g resourcegroupname --query 'containerRegistry'
    ```

    Det här kommandot returnerar ett värde som liknar `"/subscriptions/{GUID}/resourceGroups/{resourcegroupname}/providers/Microsoft.ContainerRegistry/registries/{ACRname}"` . Den sista delen av strängen är namnet på Azure Container Registry för arbets ytan.

1. Du begränsar åtkomsten till ditt virtuella nätverk genom att följa stegen i [Konfigurera nätverks åtkomst för registret](../container-registry/container-registry-vnet.md#configure-network-access-for-registry). När du lägger till det virtuella nätverket väljer du det virtuella nätverket och under nätet för dina Azure Machine Learning-resurser.

1. Använd Azure Machine Learning python SDK för att konfigurera ett beräknings kluster för att bygga Docker-avbildningar. Följande kodfragment visar hur du gör detta:

    ```python
    from azureml.core import Workspace
    # Load workspace from an existing config file
    ws = Workspace.from_config()
    # Update the workspace to use an existing compute cluster
    ws.update(image_build_compute = 'mycomputecluster')
    ```

    > [!IMPORTANT]
    > Ditt lagrings konto, beräknings kluster och Azure Container Registry måste finnas i samma undernät i det virtuella nätverket.
    
    Mer information finns i metod referensen [Update ()](https://docs.microsoft.com/python/api/azureml-core/azureml.core.workspace.workspace?view=azure-ml-py#update-friendly-name-none--description-none--tags-none--image-build-compute-none--enable-data-actions-none-) .

1. Du måste använda följande Azure Resource Manager mall. Med den här mallen kan din arbets yta kommunicera med ACR.

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "keyVaultArmId": {
        "type": "string"
        },
        "workspaceName": {
        "type": "string"
        },
        "containerRegistryArmId": {
        "type": "string"
        },
        "applicationInsightsArmId": {
        "type": "string"
        },
        "storageAccountArmId": {
        "type": "string"
        },
        "location": {
        "type": "string"
        }
    },
    "resources": [
        {
        "type": "Microsoft.MachineLearningServices/workspaces",
        "apiVersion": "2019-11-01",
        "name": "[parameters('workspaceName')]",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "sku": {
            "tier": "enterprise",
            "name": "enterprise"
        },
        "properties": {
            "sharedPrivateLinkResources":
    [{"Name":"Acr","Properties":{"PrivateLinkResourceId":"[concat(parameters('containerRegistryArmId'), '/privateLinkResources/registry')]","GroupId":"registry","RequestMessage":"Approve","Status":"Pending"}}],
            "keyVault": "[parameters('keyVaultArmId')]",
            "containerRegistry": "[parameters('containerRegistryArmId')]",
            "applicationInsights": "[parameters('applicationInsightsArmId')]",
            "storageAccount": "[parameters('storageAccountArmId')]"
        }
        }
    ]
    }
    ```
    
## <a name="azure-data-lake-storage"></a>Azure Data Lake Storage

Azure Data Lake Storage gen 2 är en uppsättning funktioner för stor data analys som bygger på Azure Blob Storage. Den kan användas för att lagra data som används för att träna modeller med Azure Machine Learning. 

Använd följande steg för att använda Data Lake Storage gen 2 inuti det virtuella nätverket på arbets ytan Azure Machine Learning:

1. Skapa ett konto för Azure Data Lake Storage gen 2. Mer information finns i [skapa ett Azure Data Lake Storage Gen2 lagrings konto](../storage/blobs/data-lake-storage-quickstart-create-account.md).

1. Använd steg 2-4 i föregående avsnitt och [Använd ett lagrings konto för din arbets yta för](#use-a-storage-account-for-your-workspace)att lagra kontot i det virtuella nätverket.

Använd följande vägledning när du använder Azure Machine Learning med Data Lake Storage gen 2 i ett virtuellt nätverk:

* Om du använder __SDK för att skapa en data uppsättning__och systemet som kör koden __inte finns i det virtuella nätverket__, använder du `validate=False` parametern. Den här parametern hoppar över verifiering, vilket Miss lyckas om systemet inte finns i samma virtuella nätverk som lagrings kontot. Mer information finns i metoden [from_files ()](https://docs.microsoft.com/python/api/azureml-core/azureml.data.dataset_factory.filedatasetfactory?view=azure-ml-py#from-files-path--validate-true-) .

* När du använder Azure Machine Learning beräknings instans eller beräknings kluster för att träna en modell med data uppsättningen måste den finnas i samma virtuella nätverk som lagrings kontot.

## <a name="key-vault-instance"></a>Key Vault-instans 

Nyckel valvs instansen som är kopplad till arbets ytan används av Azure Machine Learning för att lagra följande autentiseringsuppgifter:
* Den associerade anslutnings strängen för lagrings kontot
* Lösen ord till Azure Container databas instanser
* Anslutnings strängar till data lager

Använd följande steg för att använda Azure Machine Learning experiment funktioner med Azure Key Vault bakom ett virtuellt nätverk:

1. Gå till nyckel valvet som är kopplat till arbets ytan.

   [![Nyckel valvet som är associerat med Azure Machine Learning-arbetsytan](./media/how-to-enable-virtual-network/workspace-key-vault.png)](./media/how-to-enable-virtual-network/workspace-key-vault.png#lightbox)

1. På sidan **Key Vault** i det vänstra fönstret väljer du __brand väggar och virtuella nätverk__.

   ![Avsnittet "brand väggar och virtuella nätverk" i fönstret Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks.png)

1. På sidan __brand väggar och virtuella nätverk__ utför du följande åtgärder:
    - Under __Tillåt åtkomst från__väljer du __valda nätverk__.
    - Under __virtuella nätverk__väljer du __Lägg till befintliga virtuella nätverk__ för att lägga till det virtuella nätverk där din experiment beräkning finns.
    - Under __Tillåt att betrodda Microsoft-tjänster kringgår den här brand väggen väljer du__ __Ja__.

   [![Avsnittet "brand väggar och virtuella nätverk" i fönstret Key Vault](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png)](./media/how-to-enable-virtual-network/key-vault-firewalls-and-virtual-networks-page.png#lightbox)


## <a name="azure-databricks"></a>Azure Databricks

Om du vill använda Azure Databricks i ett virtuellt nätverk med din arbets yta måste följande krav uppfyllas:

> [!div class="checklist"]
> * Det virtuella nätverket måste finnas i samma prenumeration och region som Azure Machine Learning-arbetsytan.
> * Om Azure Storage kontona för arbets ytan också är skyddade i ett virtuellt nätverk måste de finnas i samma virtuella nätverk som Azure Databricks-klustret.
> * Förutom de __databricks-privata__ och __databricks-offentliga__ undernät som används av Azure Databricks, krävs även det __standard__ -undernät som skapats för det virtuella nätverket.

För detaljerad information om hur du använder Azure Databricks med ett virtuellt nätverk, se [distribuera Azure Databricks i Azure-Virtual Network](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-inject.html).

<a id="vmorhdi"></a>

## <a name="virtual-machine-or-hdinsight-cluster"></a>Virtuell dator eller HDInsight-kluster

> [!IMPORTANT]
> Azure Machine Learning stöder bara virtuella datorer som kör Ubuntu.

Använd följande steg om du vill använda en virtuell dator eller ett Azure HDInsight-kluster i ett virtuellt nätverk med din arbets yta:

1. Skapa ett virtuellt dator kluster eller HDInsight-kluster med hjälp av Azure Portal eller Azure CLI och Lägg klustret i ett virtuellt Azure-nätverk. Mer information finns i följande artiklar:
    * [Skapa och hantera virtuella Azure-nätverk för virtuella Linux-datorer](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)

    * [Utöka HDInsight med ett virtuellt Azure-nätverk](https://docs.microsoft.com/azure/hdinsight/hdinsight-extend-hadoop-virtual-network)

1. Om du vill tillåta Azure Machine Learning att kommunicera med SSH-porten på den virtuella datorn eller klustret konfigurerar du en käll post för nätverks säkerhets gruppen. SSH-porten är vanligt vis port 22. Utför följande åtgärder för att tillåta trafik från den här källan:

    * I list rutan __källa__ väljer du __service tag__.

    * I list rutan __käll tjänst tag__ väljer du __AzureMachineLearning__.

    * I list rutan __käll port intervall__ väljer du __*__ .

    * I list rutan __mål__ väljer du __valfri__.

    * I list rutan __mål Port intervall__ väljer du __22__.

    * Under __protokoll__väljer du __valfri__.

    * Under __åtgärd__väljer du __Tillåt__.

   ![Regler för inkommande trafik för att utföra experimentering i ett virtuellt nätverk eller HDInsight-kluster i ett virtuellt nätverk](./media/how-to-enable-virtual-network/experimentation-virtual-network-inbound.png)

    Behåll standard reglerna för utgående trafik för nätverks säkerhets gruppen. Mer information finns i standard säkerhets regler i [säkerhets grupper](https://docs.microsoft.com/azure/virtual-network/security-overview#default-security-rules).

    Om du inte vill använda de utgående standard reglerna och du vill begränsa den utgående åtkomsten för ditt virtuella nätverk kan du läsa avsnittet [begränsa utgående anslutningar från det virtuella nätverket](#limiting-outbound-from-vnet) .

1. Anslut den virtuella datorn eller HDInsight-klustret till din Azure Machine Learning-arbetsyta. Mer information finns i [Konfigurera beräknings mål för modell träning](how-to-set-up-training-targets.md).


## <a name="next-steps"></a>Nästa steg

* [Konfigurera miljöer för utbildning](how-to-set-up-training-targets.md)
* [Konfigurera privata slut punkter](how-to-configure-private-link.md)
* [Välj var du vill distribuera modeller](how-to-deploy-and-where.md)
* [Använd TLS för att skydda en webb tjänst via Azure Machine Learning](how-to-secure-web-service.md)
