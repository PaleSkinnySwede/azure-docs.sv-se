---
title: Aviseringar från Azure Monitor for VMs
description: Beskriver hur du skapar aviserings regler från prestanda data som samlas in av Azure Monitor for VMs.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/23/2020
ms.openlocfilehash: 987537d8497b3d8f2728941334d8328320ec6997
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80289607"
---
# <a name="how-to-create-alerts-from-azure-monitor-for-vms"></a>Så här skapar du aviseringar från Azure Monitor for VMs
[Aviseringar i Azure Monitor](../platform/alerts-overview.md) proaktivt meddela dig om intressanta data och mönster i dina övervaknings data. Azure Monitor for VMs innehåller inte förkonfigurerade aviserings regler, men du kan skapa egna baserat på de data som samlas in. Den här artikeln innehåller vägledning om hur du skapar aviserings regler, inklusive en uppsättning exempel frågor.


## <a name="alert-rule-types"></a>Typer av varnings regler
Azure Monitor har [olika typer av varnings regler](../platform/alerts-overview.md#what-you-can-alert-on) baserat på de data som används för att skapa aviseringen. Alla data som samlas in av Azure Monitor for VMs lagras i Azure Monitor loggar som stöder [logg aviseringar](../platform/alerts-log.md). Du kan för närvarande inte använda [mått aviseringar](../platform/alerts-log.md) med prestanda data som samlats in från Azure Monitor for VMS eftersom data inte samlas in i Azure Monitor mått. Om du vill samla in data för mått aviseringar installerar du [tillägget Diagnostics](../platform/diagnostics-extension-overview.md) för virtuella Windows-datorer eller [teleympkvistar-agenten](../platform/collect-custom-metrics-linux-telegraf.md) för virtuella Linux-datorer för att samla in prestanda data i mått.

Det finns två typer av logg aviseringar i Azure Monitor:

- [Antal resultat varningar](../platform/alerts-unified-log.md#number-of-results-alert-rules) skapar en enskild avisering när en fråga returnerar minst ett angivet antal poster. Dessa är idealiska för icke-numeriska data, t. ex. Windows-och Syslog-händelser som samlas in av [Log Analytics-agenten](../platform/log-analytics-agent.md) eller för att analysera prestanda trender på flera datorer.
- [Mått mätnings aviseringar](../platform/alerts-unified-log.md#metric-measurement-alert-rules) skapar en separat avisering för varje post i en fråga som har ett värde som överskrider ett tröskelvärde som definierats i aviserings regeln. Dessa varnings regler är idealiska för prestanda data som samlas in av Azure Monitor for VMs eftersom de kan skapa enskilda aviseringar för varje dator.


## <a name="alert-rule-walkthrough"></a>Genom gång av varnings regel
Det här avsnittet beskriver hur du skapar en mått mätnings aviserings regel med hjälp av prestanda data från Azure Monitor for VMs. Du kan använda den här grundläggande processen med en mängd olika logg frågor för att få aviseringar om olika prestanda räknare.

Börja med att skapa en ny aviserings regel genom att följa stegen i [skapa, Visa och hantera logg aviseringar med hjälp av Azure Monitor](../platform/alerts-log.md). För **resursen**väljer du Log Analytics arbets ytan som Azure Monitor virtuella datorer ska använda i din prenumeration. Eftersom mål resursen för logg aviserings regler alltid är en Log Analytics arbets yta måste logg frågan innehålla alla filter för specifika virtuella datorer eller skalnings uppsättningar för virtuella datorer. 

Använd någon av frågorna i [avsnittet nedan](#sample-alert-queries) som **Sök fråga**för **villkoret** för varnings regeln. Frågan måste returnera en numerisk egenskap med namnet *AggregatedValue*. Den bör sammanfatta data efter dator så att du kan skapa en separat avisering för varje virtuell dator som överskrider tröskelvärdet.

Välj **mått mått** i **aviserings logiken**och ange sedan ett **tröskelvärde**. I **Utlös avisering baserat på**anger du hur många gånger tröskelvärdet måste överskridas innan en avisering skapas. Till exempel är du förmodligen inte bekymrad om processorn överskrider ett tröskelvärde en gång och sedan återgår till normal, men du är försiktig om den fortsätter att överskrida tröskelvärdet för flera efterföljande mätningar.

Avsnittet **utvärdera baserat på** definierar hur ofta frågan ska köras och tids perioden för frågan. I exemplet nedan kommer frågan att köras var 15: e minut och utvärdera prestanda värden som samlats in under de senaste 15 minuterna.


![Varnings regel för mått mått](media/vminsights-alerts/metric-measurement-alert.png)

## <a name="sample-alert-queries"></a>Exempel på aviserings frågor
Följande frågor kan användas med en mått mätnings aviserings regel med hjälp av prestanda data som samlas in av Azure Monitor for VMs. Varje sammanfattar data efter dator så att en avisering skapas för varje dator med ett värde som överstiger tröskelvärdet.

### <a name="cpu-utilization"></a>CPU-användning

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "Processor" and Name == "UtilizationPercentage" 
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-mb"></a>Tillgängligt minne i MB

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Memory" and Name == "AvailableMB"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="available-memory-in-percentage"></a>Tillgängligt minne i procent

```kusto
InsightsMetrics 
| where Origin == "vm.azm.ms" 
| where Namespace == "Memory" and Name == "AvailableMB" 
| extend TotalMemory = toreal(todynamic(Tags)["vm.azm.ms/memorySizeMB"])
| extend AvailableMemoryPercentage = (toreal(Val) / TotalMemory) * 100.0 
| summarize AggregatedValue = avg(AvailableMemoryPercentage) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---all-disks-on-each-computer"></a>Använd logisk disk – alla diskar på varje dator

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="logical-disk-used---individual-disks"></a>Använd logisk disk – enskilda diskar

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "LogicalDisk" and Name == "FreeSpacePercentage"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, Disk
```

### <a name="logical-disk-iops"></a>IOPS för logisk disk

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "TransfersPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) ), Computer, _ResourceId, Disk
```

### <a name="logical-disk-data-rate"></a>Data hastighet för logisk disk

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms" 
| where Namespace == "LogicalDisk" and Name == "BytesPerSecond"
| extend Disk=tostring(todynamic(Tags)["vm.azm.ms/mountId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m) , Computer, _ResourceId, Disk
```

### <a name="network-interfaces-bytes-received---all-interfaces"></a>Mottagna byte för nätverks gränssnitt-alla gränssnitt

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId 
```

### <a name="network-interfaces-bytes-received---individual-interfaces"></a>Mottagna byte för nätverks gränssnitt – enskilda gränssnitt

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "ReadBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="network-interfaces-bytes-sent---all-interfaces"></a>Skickade byte för nätverks gränssnitt-alla gränssnitt

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId
```

### <a name="network-interfaces-bytes-sent---individual-interfaces"></a>Nätverks gränssnitt byte skickade – enskilda gränssnitt

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where Namespace == "Network" and Name == "WriteBytesPerSecond"
| extend NetworkInterface=tostring(todynamic(Tags)["vm.azm.ms/networkDeviceId"])
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), Computer, _ResourceId, NetworkInterface
```

### <a name="virtual-machine-scale-set"></a>Skaluppsättning för virtuella datorer
Ändra med ditt prenumerations-ID, resurs grupp och namn på den virtuella datorns skalnings uppsättning.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/my-vm-scaleset" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="specific-virtual-machine"></a>Speciell virtuell dator
Ändra med ditt prenumerations-ID, resurs grupp och namn på virtuell dator.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId =~ "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/my-vm" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m)
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-subscription"></a>PROCESSOR belastning för alla beräknings resurser i en prenumeration
Ändra med ditt prenumerations-ID.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" and (_ResourceId contains "/providers/Microsoft.Compute/virtualMachines/" or _ResourceId contains "/providers/Microsoft.Compute/virtualMachineScaleSets/")
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId
```

### <a name="cpu-utilization-for-all-compute-resources-in-a-resource-group"></a>PROCESSOR belastning för alla beräknings resurser i en resurs grupp
Ändra med ditt prenumerations-ID och din resurs grupp.

```kusto
InsightsMetrics
| where Origin == "vm.azm.ms"
| where _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachines/"
or _ResourceId startswith "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/my-resource-group/providers/Microsoft.Compute/virtualMachineScaleSets/" 
| where Namespace == "Processor" and Name == "UtilizationPercentage"
| summarize AggregatedValue = avg(Val) by bin(TimeGenerated, 15m), _ResourceId

```

## <a name="next-steps"></a>Nästa steg

- Lär dig mer om [aviseringar i Azure Monitor](../platform/alerts-overview.md).
- Läs mer om [logg frågor som använder data från Azure Monitor for VMS](vminsights-log-search.md).
