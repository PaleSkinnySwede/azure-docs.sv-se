---
title: Mv2-serien – Azure Virtual Machines
description: Specifikationer för virtuella datorer i Mv2-serien.
services: virtual-machines
author: ayshakeen
ms.service: virtual-machines
ms.topic: article
ms.date: 04/07/2020
ms.author: lahugh
ms.openlocfilehash: 7df8dd439008258ea1b4986054660fb0fb9070ce
ms.sourcegitcommit: 67bddb15f90fb7e845ca739d16ad568cbc368c06
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82204193"
---
# <a name="mv2-series"></a>Mv2-serien

Mv2-serien har högt data flöde, låg latens-plattform som körs på en Hyper-threadd Intel® Xeon® platina Intel Xeon-processor med en alla grund frekvens på 2,5 GHz och en maximal Turbo frekvens på 3,8 GHz. Alla storlekar för virtuella datorer i Mv2-serien kan använda både standard-och Premium-diskar. Mv2-seriens instanser är minnesoptimerade VM-storlekar som ger oöverträffade beräknings prestanda för att stödja stora minnes databaser och arbets belastningar, med ett högt förhållande mellan minne och CPU som är idealiskt för Relations databas servrar, stora cacheminnen och minnes intern analys.

Mv2-seriens funktion Intel® Hyper-Threading-teknik

Premium Storage: stöds

Premium Storage caching: stöds

Direktmigrering: stöds inte

Minnes bebetjänings uppdateringar: stöds inte

Skrivningsaccelerator: [stöds](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)

|Storlek | Virtuell processor | Minne: GiB | Temporär lagring (SSD) GiB | Maximalt antal datadiskar | Maximalt genomflöde för cachelagring och temporär lagring: IOPS / Mbit/s (cachestorlek i GiB) | Maximalt icke cachelagrat diskgenomflöde: IOPS / Mbit/s | Högsta antal nätverkskort/förväntad nätverks bandbredd (Mbit/s) |
|---|---|---|---|---|---|---|---|
| Standard_M208ms_v2<sup>1</sup> | 208 | 5700 | 4096 | 64 | 80000/800 (7040) | 40000/1000 | 8/16 000 |
| Standard_M208s_v2<sup>1</sup> | 208 | 2850 | 4096 | 64 | 80000/800 (7040) | 40000/1000 | 8/16 000 |
| Standard_M416ms_v2<sup>1</sup> | 416 | 11400 | 8192 | 64 | 250000/1600 (14080) | 80000/2000 | 8 / 32000 |
| Standard_M416s_v2<sup>1</sup> | 416 | 5700 | 8192 | 64 | 250000/1600 (14080) | 80000/2000 | 8 / 32000 |

<sup>1</sup> virtuella datorer i Mv2-serien är endast generation 2 och stöder en delmängd av avbildningar som stöds av generation 2. Nedan visas en fullständig lista över de avbildningar som stöds för Mv2-serien. Om du använder Linux, se [stöd för virtuella datorer i generation 2 på Azure](./linux/generation-2.md) för instruktioner om hur du hittar och väljer en avbildning. Om du använder Windows, se [stöd för virtuella datorer i generation 2 på Azure](./windows/generation-2.md) för instruktioner om hur du hittar och väljer en avbildning. 

- Windows Server 2019 eller senare
- SUSE Linux Enterprise Server 12 SP4 och senare eller SUSE Linux Enterprise Server 15 SP1 och senare
- Red Hat Enterprise Linux 7,6, 7,7, 8,1 eller senare 
- Oracle Enterprise Linux 7,7 eller senare



[!INCLUDE [virtual-machines-common-sizes-table-defs](../../includes/virtual-machines-common-sizes-table-defs.md)]

## <a name="other-sizes"></a>Andra storlekar

- [Generellt syfte](sizes-general.md)
- [Minnesoptimerad](sizes-memory.md)
- [Lagringsoptimerad](sizes-storage.md)
- [GPU-optimerad](sizes-gpu.md)
- [Databehandling med höga prestanda](sizes-hpc.md)
- [Tidigare generationer](sizes-previous-gen.md)

## <a name="next-steps"></a>Nästa steg

Lär dig mer om hur [Azure Compute Units (ACU)](acu.md) kan hjälpa dig att jämföra beräknings prestanda i Azure SKU: er.
