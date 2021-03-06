---
title: Skapa Azure Cosmos-behållare och databaser i autoskalning av allokerat data flöde.
description: Lär dig mer om fördelar, användnings fall och hur du etablerar Azure Cosmos-databaser och behållare i autoskalning av etablerat data flöde.
author: kirillg
ms.author: kirillg
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/28/2020
ms.openlocfilehash: 81a13dcb7955a7d46f485416bf9b7e4e7be4d9ac
ms.sourcegitcommit: e0330ef620103256d39ca1426f09dd5bb39cd075
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/05/2020
ms.locfileid: "82791722"
---
# <a name="create-azure-cosmos-containers-and-databases-with-autoscale-provisioned-throughput"></a>Skapa Azure Cosmos-behållare och databaser med autoskalning av allokerat data flöde

Med Azure Cosmos DB kan du konfigurera dina behållare med standard (manuellt) tillhandahållet data flöde eller autoskalning av allokerat data flöde. I den här artikeln beskrivs fördelarna med och användnings fall för autoskalning.

> [!NOTE]
> Du kan bara [Aktivera autoskalning för nya databaser och behållare](#create-db-container-autoscale) . Den är inte tillgänglig för befintliga behållare och databaser.

Förutom standard etablering av data flöde kan du nu konfigurera Azure Cosmos-behållare med automatiskt skalning av allokerat data flöde. Behållare och databaser som kon figurer ATS i autoskalning av allokerat data flöde kommer **automatiskt att skala det etablerade data flödet baserat på dina program behov utan att påverka tillgänglighet, svars tid, data flöde eller prestanda för arbets belastningen globalt.**

När du konfigurerar behållare och databaser i autoskalning måste du ange det maximala data `Tmax` flödet som inte överskrids. Behållare kan sedan skala sitt genomflöde så att `0.1*Tmax < T < Tmax`. Med andra ord skalar behållare och databaser direkt utifrån arbets belastnings behoven, från så lågt som 10% av det maximala data flöde svärdet som du har konfigurerat till det konfigurerade maximala data flöde svärdet. När du har konfigurerat autoskalning kan du ändra inställningen för`Tmax`maximal data flöde () för en databas eller behållare vid varje tidpunkt. Med alternativet för autoskalning är 400 RU/s minsta data flöde per behållare eller databas inte längre tillämpligt.

För det angivna maximala data flödet i behållaren eller databasen tillåter systemet drift inom den beräknade lagrings gränsen. Om lagrings gränsen överskrids justeras det maximala data flödet automatiskt till ett högre värde. När du använder data flöde för databas nivå med autoskalning beräknas antalet behållare som tillåts i en databas som `0.001*TMax`:. Om du till exempel etablerar 20 000 AutoScale RU/s kan databasen ha 20 behållare.

## <a name="benefits-of-autoscale-provisioned-throughput"></a><a id="autoscale-benefits"></a>Fördelar med autoskalning av allokerat data flöde

Azure Cosmos-behållare som kon figurer ATS med autoskalning har följande fördelar:

* **Enkel:** Behållare med autoskalning tar bort komplexiteten för att hantera etablerade data flöden (ru: er) och kapacitet manuellt för olika behållare.

* **Skalbarhet:** Behållare med autoskalning sömlöst skalar den allokerade data flödes kapaciteten vid behov. Det uppstår inget avbrott i klient anslutningar, program och de påverkar inte några befintliga service avtal.

* **Kostnads effektiv:** När du använder behållare som kon figurer ATS med autoskalning betalar du bara för de resurser som dina arbets belastningar behöver per timme.

* **Hög tillgänglighet:** Behållare med autoskalning använder samma globalt distribuerade, feltoleranta och hög tillgängliga Server del för att säkerställa data hållbarhet och hög tillgänglighet.

## <a name="use-cases-of-autoscale-provisioned-throughput"></a><a id="autoscale-usecases"></a>Användnings fall för autoskalning av allokerat data flöde

Användnings fall för Azure Cosmos-behållare som kon figurer ATS med autoskalning inkluderar:

* **Variabla arbets belastningar:** När du kör ett program med hög användning med en högsta användning på 1 timme till flera timmar varje dag eller flera gånger per år. Exempel är program för personal, budgetering och drift rapportering. För sådana scenarier kan behållare som kon figurer ATS med autoskalning användas, och du behöver inte längre etablera manuellt för antingen hög kapacitet eller genomsnitts kapacitet.

* **Oförutsägbara arbets belastningar:** När du kör arbets belastningar där det finns en databas användning under dagen, men även toppar av aktiviteter som är svåra att förutsäga. Ett exempel är en trafik plats som ser en överspänning av aktivitet när väder prognoser ändras. Behållare som kon figurer ATS med autoskalning justera kapaciteten för att uppfylla behoven för programmets högsta belastning och skala ned när aktivitetens överspänning är över.

* **Nya program:** Om du distribuerar ett nytt program och är osäker på hur mycket allokerat data flöde (t. ex. hur många ru: er) du behöver. Med behållare som kon figurer ATS med automatisk skalning kan du automatiskt skala till kapacitets behoven och kraven för ditt program.

* **Program som används sällan:** Om du har ett program som bara används för några timmar flera gånger per dag eller vecka eller månad, till exempel en program-/webb-/blogg webbplats med låg volym.

* **Utvecklings-och test databaser:** Om du har utvecklare som använder behållare under arbets tid, men inte behöver dem på nätter eller helger. Med behållare som kon figurer ATS med autoskalning skalas de ned till ett minimum när de inte används.

* **Schemalagda produktions arbets belastningar/frågor:** När du har en serie med schemalagda begär Anden/åtgärder/frågor på en enda behållare, och om det finns inaktiva perioder där du vill köra med ett absolut lågt data flöde, kan du nu enkelt göra det. När en schemalagd fråga/begäran skickas till en behållare som kon figurer ATS med automatisk skalning, skalar den automatiskt upp så mycket som behövs och kör åtgärden.

Lösningar på tidigare problem kräver inte bara en enorma del av tiden i implementeringen, men de introducerar också komplexitet i konfigurationen eller din kod och kräver ofta manuella åtgärder för att åtgärda dem. Med autoskalning kan du använda ovanstående scenarier i rutan så att du inte behöver oroa dig för dessa problem längre.

## <a name="comparison--standard-manual-vs-autoscale-provisioned-throughput"></a>Jämförelse – standard (manuell) jämfört med autoskalning av allokerat data flöde

|  | Behållare som kon figurer ATS med standard-allokerat data flöde  | Behållare som kon figurer ATS med autoskalning av allokerat data flöde |
|---------|---------|---------|
| **Etablerat dataflöde** | Manuellt etablerad. | Automatiskt och automatiskt skalas baserat på användnings mönster för arbets belastning. |
| **Hastighets begränsning av begär Anden/åtgärder (429)**  | Kan inträffa om förbrukningen överskrider den etablerade kapaciteten. | Sker inte om det data flöde som förbrukas är inom det maximala data flöde som du väljer med autoskalning.   |
| **Kapacitetsplanering** |  Du måste utföra en första kapacitets planering och tillhandahålla det data flöde du behöver. |    Du behöver inte bekymra dig om kapacitets planering. Systemet tar automatiskt hand om kapacitets planering och kapacitets hantering. |
| **Prissättning** | Manuellt etablerade RU/s per timme. | För enstaka Skriv regions konton betalar du för det data flöde som används per timme med hjälp av den automatiska skalningen RU/s per tim pris. <br/><br/>För konton med flera Skriv regioner finns det ingen extra kostnad för autoskalning. Du betalar för det data flöde som används per timme med samma taxa för flera huvud-RU/s per timme. |
| **Passar bäst för arbets belastnings typer** |  Förutsägbara och stabila arbets belastningar|   Oförutsedda och varierande arbets belastningar  |

## <a name="create-a-database-or-a-container-with-autoscale"></a><a id="create-db-container-autoscale"></a>Skapa en databas eller en behållare med autoskalning

Du kan konfigurera autoskalning för nya databaser eller behållare när du skapar dem via Azure Portal. Använd följande steg för att skapa en ny databas eller behållare, aktivera autoskalning och ange maximalt data flöde (RU/s).

1. Logga in på [Azure Portal](https://portal.azure.com) eller [Azure Cosmos DB Explorer.](https://cosmos.azure.com/)

1. Gå till ditt Azure Cosmos DB-konto och öppna fliken **datautforskaren** .

1. Välj **ny behållare.** Ange ett namn för din databas, behållare och en partitionsnyckel. Under **data flöde**väljer du alternativet **autoskalning** och väljer det maximala data flöde (ru/s) som databasen eller behållaren inte får överskrida när du använder alternativet för autoskalning.

   ![Skapa en behållare och konfigurera autoskalning genom strömning](./media/provision-throughput-autoscale/create-container-autoscale-mode.png)

1. Välj **OK**.

Du kan skapa en delad data flödes databas med autoskalning genom att välja **data flödes alternativet etablera databas** .

## <a name="throughput-and-storage-limits-for-autoscale"></a><a id="autoscale-limits"></a>Data flödes-och lagrings gränser för autoskalning

I följande tabell visas de maximala gränserna för alla och lagrings gränser för olika alternativ i autoskalning:

|Maximal data flödes gräns  |Maximal lagrings gräns  |
|---------|---------|
|4000 RU/s  |   50 GB    |
|20 000 RU/s  |  200 GB  |
|100 000 RU/s    |  1 TB   |
|500 000 RU/s    |  5 TB  |

## <a name="next-steps"></a>Nästa steg

* Läs [vanliga frågor och svar om autoskalning](autoscale-faq.md).
* Läs mer om [logiska partitioner](partition-data.md).
* Lär dig hur du [etablerar data flöde i en Azure Cosmos-behållare](how-to-provision-container-throughput.md).
* Lär dig hur du [etablerar data flöde i en Azure Cosmos-databas](how-to-provision-database-throughput.md).
