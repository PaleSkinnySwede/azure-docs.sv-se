---
title: Rikt linjer för tabell design för Azure Storage | Microsoft Docs
description: Utforma Azure Table service för att stödja Läs åtgärder effektivt.
services: storage
author: SnehaGunda
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.author: sngun
ms.subservice: tables
ms.openlocfilehash: d056d29469ad9a60fceeee307aca3c0e1319283c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/27/2020
ms.locfileid: "61269855"
---
# <a name="guidelines-for-table-design"></a>Riktlinjer för tabelldesign

Att designa tabeller för användning med Azure Storage Table-tjänsten skiljer sig mycket från design överväganden för en Relations databas. I den här artikeln beskrivs rikt linjerna för att designa din Table service-lösning för att vara mer effektiv och skriv effektiv.

## <a name="design-your-table-service-solution-to-be-read-efficient"></a>Utforma Table service-lösningen så att den blir Läs effektiv

* ***Design för frågor i Läs-tung-program.*** När du utformar dina tabeller bör du tänka på frågorna (särskilt svars tiden) som du ska köra innan du funderar på hur du kommer att uppdatera dina entiteter. Detta leder vanligt vis till en effektiv och genomförd lösning.  
* ***Ange både PartitionKey och RowKey i dina frågor.*** *Punkt frågor* som dessa är de mest effektiva tabell tjänst frågorna.  
* ***Överväg att lagra kopior av entiteter.*** Table Storage är billigt så att du kan lagra samma entitet flera gånger (med olika nycklar) för att möjliggöra mer effektiva frågor.  
* ***Överväg att avnormalisera dina data.*** Table Storage är billigt så att du kan avnormalisera dina data. Lagra till exempel sammanfattnings enheter så att frågor för sammanställd data bara behöver ha åtkomst till en enda entitet.  
* ***Använd sammansatta nyckel värden.*** De enda nycklar du har är **PartitionKey** och **RowKey**. Använd t. ex. värden för sammansatta nycklar för att aktivera alternativa, åtkomst vägar till entiteter.  
* ***Använd frågans projektion.*** Du kan minska mängden data som överförs via nätverket med hjälp av frågor som bara väljer de fält som du behöver.  

## <a name="design-your-table-service-solution-to-be-write-efficient"></a>Utforma Table service-lösningen så att den blir Skriv effektiv  

* ***Skapa inte varma partitioner.*** Välj nycklar som gör att du kan sprida dina begär anden över flera partitioner vid en viss tidpunkt.  
* ***Undvik toppar i trafik.*** Utjämna trafiken under en rimlig tids period och Undvik toppar i trafiken.
* ***Skapa inte nödvändigt vis en separat tabell för varje typ av entitet.*** När du kräver atomiska transaktioner över entitetstyper kan du lagra dessa typer av flera entiteter i samma partition i samma tabell.
* ***Ta hänsyn till det maximala data flöde som du måste uppnå.*** Du måste vara medveten om skalbarhets målen för Table service och se till att designen inte kommer att överskrida dem.  

När du läser den här guiden visas exempel som har alla dessa principer i praktiken. 

## <a name="next-steps"></a>Nästa steg

- [Mönster för tabelldesign](table-storage-design-patterns.md)
- [Utforma för frågor](table-storage-design-for-query.md)
- [Kryptera tabell data](table-storage-design-encrypt-data.md)
- [Utforma för dataändring](table-storage-design-for-modification.md)
