---
title: Lägga till eller ta bort noder i ett fristående Service Fabric-kluster
description: Lär dig hur du lägger till eller tar bort noder i ett Azure Service Fabric-kluster på en fysisk eller virtuell dator som kör Windows Server, som kan vara lokalt eller i ett moln.
author: dkkapur
ms.topic: conceptual
ms.date: 11/02/2017
ms.author: dekapur
ms.openlocfilehash: f9bee35ee8e82070b4cf601139b471562ba5e10b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75934210"
---
# <a name="add-or-remove-nodes-to-a-standalone-service-fabric-cluster-running-on-windows-server"></a>Lägga till eller ta bort noder i ett fristående Service Fabric-kluster som körs på Windows Server
När du har [skapat ditt fristående Service Fabric-kluster på Windows Server-datorer](service-fabric-cluster-creation-for-windows-server.md), kan dina (företaget) behov ändras och du måste lägga till eller ta bort noder i klustret. Den här artikeln innehåller detaljerade anvisningar för att åstadkomma detta. Observera att funktionen för att lägga till/ta bort noder inte stöds i lokala utvecklings kluster.

## <a name="add-nodes-to-your-cluster"></a>Lägg till noder i klustret

1. Förbered den virtuella datorn/datorn som du vill lägga till i klustret genom att följa stegen som beskrivs i [planera och förbereda Service Fabric kluster distributionen](service-fabric-cluster-standalone-deployment-preparation.md).

2. Identifiera vilken fel domän och vilken uppgraderings domän du ska lägga till den här virtuella datorn eller datorn i.

   Om du använder certifikat för att skydda klustret förväntas certifikaten installeras i de lokala certifikat arkiven som förberedelse för noden att ansluta till klustret. Den analoga funktionen kan användas när du använder andra typer av säkerhet.

3. Fjärr skrivbord (RDP) i den virtuella datorn/datorn som du vill lägga till i klustret.

4. Kopiera eller [Hämta det fristående paketet för Service Fabric för Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690) till den virtuella datorn/datorn och packa upp paketet.

5. Kör PowerShell med utökade privilegier och gå till platsen för det zippade paketet.

6. Kör skriptet *Addnode. ps1* med parametrarna som beskriver den nya noden som ska läggas till. I följande exempel lägger du till en ny nod med namnet VM5, med typen NodeType0 och IP-182.17.34.52 i UD1 och fd:/DC1/R0. `ExistingClusterConnectionEndPoint`är en anslutnings slut punkt för en nod som redan finns i det befintliga klustret, vilket kan vara IP-adressen för *en nod i* klustret. 

   Osäker (prototyping):

   ```
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -AcceptEULA
   ```

   Säker (certifikatbaserad):

   ```  
   $CertThumbprint= "***********************"
    
   .\AddNode.ps1 -NodeName VM5 -NodeType NodeType0 -NodeIPAddressorFQDN 182.17.34.52 -ExistingClientConnectionEndpoint 182.17.34.50:19000 -UpgradeDomain UD1 -FaultDomain fd:/dc1/r0 -X509Credential -ServerCertThumbprint $CertThumbprint  -AcceptEULA

   ```

   När skriptet har körts klart kan du kontrol lera om den nya noden har lagts till genom att köra cmdleten [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) .

7. För att säkerställa konsekvens över olika noder i klustret måste du starta en konfigurations uppgradering. Kör [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) för att hämta den senaste konfigurations filen och Lägg till den nyligen tillagda noden i avsnittet "noder". Vi rekommenderar också att alltid ha den senaste kluster konfigurationen tillgänglig om du behöver distribuera om ett kluster som har samma konfiguration.

   ```
    {
        "nodeName": "vm5",
        "iPAddress": "182.17.34.52",
        "nodeTypeRef": "NodeType0",
        "faultDomain": "fd:/dc1/r0",
        "upgradeDomain": "UD1"
    }
   ```

8. Starta uppgraderingen genom att köra [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) .

   ```
   Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
   ```

   Du kan övervaka förloppet för uppgraderingen på Service Fabric Explorer. Du kan också köra [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps).

### <a name="add-nodes-to-clusters-configured-with-windows-security-using-gmsa"></a>Lägg till noder i kluster som kon figurer ATS med Windows-säkerhet med gMSA
För kluster som kon figurer ATS med grupphanterat tjänst konto (https://technet.microsoft.com/library/hh831782.aspx)gMSA) (kan du lägga till en ny nod med en konfigurations uppgradering):
1. Kör [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) på någon av de befintliga noderna för att hämta den senaste konfigurations filen och Lägg till information om den nya noden som du vill lägga till i avsnittet "noder". Se till att den nya noden är en del av samma grupphanterade konto. Det här kontot bör vara en administratör på alla datorer.

    ```
        {
            "nodeName": "vm5",
            "iPAddress": "182.17.34.52",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD1"
        }
    ```
2. Starta uppgraderingen genom att köra [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) .

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>
    ```
    Du kan övervaka förloppet för uppgraderingen på Service Fabric Explorer. Du kan också köra [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps)

### <a name="add-node-types-to-your-cluster"></a>Lägga till nodtyper i klustret
För att lägga till en ny nodtyp ändrar du konfigurationen så att den innehåller den nya nodtypen i avsnittet "NodeTypes" under "egenskaper" och påbörjar en konfigurations uppgradering med [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps). När uppgraderingen är klar kan du lägga till nya noder i klustret med den här nodtypen.

## <a name="remove-nodes-from-your-cluster"></a>Ta bort noder från klustret
En nod kan tas bort från ett kluster med hjälp av en konfigurations uppgradering på följande sätt:

1. Kör [Get-ServiceFabricClusterConfiguration](/powershell/module/servicefabric/get-servicefabricclusterconfiguration?view=azureservicefabricps) för att hämta den senaste konfigurations filen och *ta bort* noden från "Nodes"-avsnittet.
Lägg till parametern "NodesToBeRemoved" i avsnittet "FabricSettings". Värdet måste vara en kommaavgränsad lista med nodnamn för noder som måste tas bort.

    ```
         "fabricSettings": [
            {
            "name": "Setup",
            "parameters": [
                {
                "name": "FabricDataRoot",
                "value": "C:\\ProgramData\\SF"
                },
                {
                "name": "FabricLogRoot",
                "value": "C:\\ProgramData\\SF\\Log"
                },
                {
                "name": "NodesToBeRemoved",
                "value": "vm0, vm1"
                }
            ]
            }
        ]
    ```
2. Starta uppgraderingen genom att köra [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps) .

    ```
    Start-ServiceFabricClusterConfigurationUpgrade -ClusterConfigPath <Path to Configuration File>

    ```
    Du kan övervaka förloppet för uppgraderingen på Service Fabric Explorer. Du kan också köra [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps).

> [!NOTE]
> Borttagning av noder kan initiera flera uppgraderingar. Vissa noder är markerade med `IsSeedNode=”true”` tagg och kan identifieras genom att fråga kluster manifestet med hjälp `Get-ServiceFabricClusterManifest`av. Borttagning av sådana noder kan ta längre tid än andra eftersom startnoderna måste flyttas runt i sådana scenarier. Klustret måste ha minst tre primära noder av Node-typ.
> 
> 

### <a name="remove-node-types-from-your-cluster"></a>Ta bort nodtyper från klustret
Innan du tar bort en nodtyp måste du kontrol lera om det finns noder som refererar till nodtypen. Ta bort de här noderna innan du tar bort motsvarande nodtyp. När alla motsvarande noder har tagits bort kan du ta bort NodeType från kluster konfigurationen och påbörja en konfigurations uppgradering med [Start-ServiceFabricClusterConfigurationUpgrade](/powershell/module/servicefabric/start-servicefabricclusterconfigurationupgrade?view=azureservicefabricps).


### <a name="replace-primary-nodes-of-your-cluster"></a>Ersätt primära noder i klustret
Utbyte av primära noder bör utföras en nod efter en annan, i stället för att ta bort och sedan lägga till i batchar.


## <a name="next-steps"></a>Nästa steg
* [Konfigurations inställningar för fristående Windows-kluster](service-fabric-cluster-manifest.md)
* [Skydda ett fristående kluster i Windows med hjälp av X509-certifikat](service-fabric-windows-cluster-x509-security.md)
* [Skapa ett fristående Service Fabric-kluster med virtuella Azure-datorer som kör Windows](service-fabric-cluster-creation-with-windows-azure-vms.md)

