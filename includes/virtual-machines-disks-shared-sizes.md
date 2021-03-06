---
title: ta med fil
description: ta med fil
services: virtual-machines
author: roygara
ms.service: virtual-machines
ms.topic: include
ms.date: 04/06/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 0b185d545e129c941d5df2e8ce86ee684174b666
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81008372"
---
För närvarande kan endast Ultra disks och Premium-SSD aktivera delade diskar. Olika disk storlekar kan ha olika `maxShares` begränsningar, vilka du inte får överskrida när du `maxShares` anger värdet. För Premium-SSD är disk storlekarna som stöder delning av diskarna p15 och större.

För varje disk kan du definiera ett `maxShares` värde som representerar det högsta antalet noder som samtidigt kan dela disken. Om du till exempel planerar att konfigurera ett redundanskluster med två noder, anger `maxShares=2`du. Det maximala värdet är ett övre gräns värde. Noder kan ansluta till eller lämna klustret (montera eller demontera disken) så länge antalet noder är lägre än det angivna `maxShares` värdet.

> [!NOTE]
> `maxShares` Värdet kan bara anges eller redige ras när disken kopplas från alla noder.

### <a name="premium-ssd-ranges"></a>Premium SSD intervall

I följande tabell visas de tillåtna Max värdena för `maxShares` disk storlekar i Premium:

|Disk storlekar  |maxShares-gräns  |
|---------|---------|
|P15, P20     |2         |
|P30, P40, P50     |5         |
|P60, P70, P80     |10         |

Begränsningarna för IOPS och bandbredd för en disk påverkas inte av `maxShares` värdet. Till exempel är Max IOPS för en p15-disk 1100 om maxShares = 1 eller maxShares > 1.

### <a name="ultra-disk-ranges"></a>Ultra disk-intervall

Minimivärdet `maxShares` är 1, men det maximala `maxShares` värdet är 5. Det finns inga storleks begränsningar på Ultra disks, vilken storlek Ultra disk kan använda valfritt `maxShares`värde för, upp till och inklusive det maximala värdet.