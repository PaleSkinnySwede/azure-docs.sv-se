---
title: Benchmarking av ditt program på Azure-disklagring
description: Lär dig mer om processen för att mäta ditt program på Azure.
author: roygara
ms.author: rogarana
ms.date: 01/11/2019
ms.topic: conceptual
ms.service: virtual-machines-linux
ms.subservice: disks
ms.openlocfilehash: 640ec54e9634751d05c2cea90d7c03d02e7a3387
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75720063"
---
# <a name="benchmarking-a-disk"></a>Prestandatest av en disk

Benchmarking är en process för att simulera olika arbets belastningar i ditt program och mäta programmets prestanda för varje arbets belastning. Med hjälp av stegen som beskrivs i [artikeln designa för hög prestanda](premium-storage-performance.md). Genom att köra benchmarking-verktyg på de virtuella datorer som är värdar för programmet kan du bestämma vilka prestanda nivåer som programmet kan uppnå med Premium Storage. I den här artikeln ger vi exempel på att du kan mäta en standard VM-DS14 som tillhandahålls med Azure Premium Storage-diskar.

Vi har använt vanliga benchmarking-verktyg IOMeter och FIO, för Windows respektive Linux. Dessa verktyg skapar flera trådar som simulerar en produktion som arbets belastning och mäter systemets prestanda. Med hjälp av verktygen kan du också konfigurera parametrar som block storlek och ködjup, som du normalt inte kan ändra för ett program. Detta ger dig större flexibilitet för att öka den maximala prestandan på en hög skala VM som tillhandahålls med Premium diskar för olika typer av program arbets belastningar. Om du vill veta mer om varje benchmarking-verktyg går du till [IOMeter](http://www.iometer.org/) och [FIO](http://freecode.com/projects/fio).

Om du vill följa exemplen nedan skapar du en standard virtuell DS14-dator och lägger till 11 Premium Storage diskar på den virtuella datorn. För de 11 diskarna konfigurerar du 10 diskar med cachelagring av värden som "ingen" och stripe dem i en volym som heter NoCacheWrites. Konfigurera cachelagring av värd som "ReadOnly" på den återstående disken och skapa en volym som heter CacheReads med den här disken. Med den här installationen kan du se den maximala Läs-och skriv prestandan från en vanlig virtuell DS14-dator. Detaljerade anvisningar om hur du skapar en virtuell DS14-dator med Premium-diskar finns i [utforma för hög prestanda](premium-storage-performance.md).

[!INCLUDE [virtual-machines-disks-benchmarking](../../../includes/virtual-machines-managed-disks-benchmarking.md)]

## <a name="next-steps"></a>Nästa steg

Fortsätt till vår artikel om hur du [utformar för hög prestanda](premium-storage-performance.md).

I den artikeln skapar du en check lista som liknar ditt befintliga program för prototypen. Med hjälp av benchmarking-verktyg kan du simulera arbets belastningarna och mäta prestanda för prototyp programmet. Genom att göra det kan du bestämma vilket disk erbjudande som kan matcha eller överskrida dina program prestanda krav. Sedan kan du implementera samma rikt linjer för ditt produktions program.