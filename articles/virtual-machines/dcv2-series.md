---
title: DC-serien – Azure Virtual Machines
description: Specifikationer för virtuella datorer i DC-serien.
services: virtual-machines
author: susaxen
ms.service: virtual-machines
ms.topic: article
ms.date: 02/20/2020
ms.author: lahugh
ms.openlocfilehash: 431c1b1211165f43feb7fe1f93c73c2bf141e004
ms.sourcegitcommit: f57297af0ea729ab76081c98da2243d6b1f6fa63
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/06/2020
ms.locfileid: "82871810"
---
# <a name="dcsv2-series"></a>DCsv2-serien


DCsv2-serien kan hjälpa till att skydda konfidentialiteten och integriteten hos dina data och kod när de bearbetas i det offentliga molnet. De här datorerna backas upp av den senaste generationen Intel XEON E-2288G-processor med SGX-teknik. Med Intel Turbo Boost-tekniken kan de här datorerna gå upp till 5,0 GHz. DCsv2 Series-instanser gör det möjligt för kunder att skapa säkra enklaven-baserade program för att skydda sin kod och sina data medan den används.

Exempel på användnings områden är: konfidentiell data delning för multidelar, bedrägeri identifiering, tvättning av pengar, blockchain, konfidentiell användnings analys, informations analys och konfidentiell maskin inlärning.

Premium Storage: stöds *

Premium Storage cachelagring: stöds *

Direktmigrering: stöds inte

Minnes bebetjänings uppdateringar: stöds inte

* Förutom Standard_DC8_v2



| Storlek             | Virtuell processor | Minne: GiB | Temporär lagring (SSD) GiB | Maximalt antal datadiskar | Maximalt genomflöde för cachelagring och temporär lagring: IOPS / Mbit/s (cachestorlek i GiB) | Maximalt icke cachelagrat diskgenomflöde: IOPS / Mbit/s | Högsta antal nätverkskort/förväntad nätverks bandbredd (Mbit/s) | EPC-minne (MiB) |
|------------------|------|-------------|------------------------|----------------|-------------------------------------------------------------------------|-------------------------------------------|----------------------------------------------|---------------------|
| Standard_DC1s_v2 | 1    | 4           | 50                     | 1              | 2000/16 (21)                                                            | 1600/24                                   | 2   | 28                                      |
| Standard_DC2s_v2 | 2    | 8           | 100                    | 2              | 4000/32 (43)                                                            | 3200/48                                   | 2  | 56                                          |
| Standard_DC4s_v2 | 4    | 16          | 200                    | 4              | 8000/64 (86)                                                            | 6400/96                                   | 2  | 112                                          |
| Standard_DC8_v2  | 8   | 32          | 400                    | 8              | 16000/128 (172)                                                         | 12800/192                                 | 2   | 168                                         |

- Virtuella datorer i DCsv2-serien är virtuella [datorer i generation 2](./linux/generation-2.md#creating-a-generation-2-vm) och stöder `Gen2` bara avbildningar.
- För närvarande endast tillgängligt i Storbritannien, södra, Kanada, centrala och östra USA.
- Tidigare generation av konfidentiella beräknings datorer: [DC-serien](sizes-previous-gen.md#preview-dc-series)
- Skapa virtuella DCsv2-datorer med hjälp av [Azure Portal](./linux/quick-create-portal.md) eller [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview)



## <a name="other-sizes"></a>Andra storlekar

- [Generellt syfte](sizes-general.md)
- [Minnesoptimerad](sizes-memory.md)
- [Lagringsoptimerad](sizes-storage.md)
- [GPU-optimerad](sizes-gpu.md)
- [Databehandling med höga prestanda](sizes-hpc.md)
- [Tidigare generationer](sizes-previous-gen.md)

## <a name="next-steps"></a>Nästa steg

Lär dig mer om hur [Azure Compute Units (ACU)](acu.md) kan hjälpa dig att jämföra beräknings prestanda i Azure SKU: er.
