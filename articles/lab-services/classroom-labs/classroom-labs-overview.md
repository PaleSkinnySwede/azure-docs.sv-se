---
title: Om klass rums labb i Azure Lab Services | Microsoft Docs
description: Lär dig hur du snabbt skapar en klass rums labb miljö i molnet – konfigurera ett labb med en mall-VM med den program vara som krävs för klassen och gör en kopia av den virtuella datorn som är tillgänglig för varje elev i klassen.
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 11/26/2019
ms.author: spelluru
ms.openlocfilehash: bbbea0c943758c2682f7b166e461807d2914f790
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74561681"
---
# <a name="introduction-to-classroom-labs"></a>Introduktion till klassrumslabb
Azure Lab Services låter dig snabbt konfigurera en ett klassrums-labbmiljö i molnet. En undervisare skapar ett klassrumslabb, etablerar virtuella Windows- eller Linux-datorer, installerar nödvändig programvara och verktyg i klassen och gör dem tillgängliga för studenterna. Studenterna i klassen ansluter till virtuella datorer (VM) i labbet och använder dem för sina projekt, uppgifter, klassrumsövningar. 

Klassrumlabben är hanterade labbtyper som hanteras av Azure. Själva tjänsten hanterar all infrastrukturhantering för en hanterad labbtyp, från att skapa virtuella datorer till att hantera fel och skala infrastrukturen. Du kan ange vilken typ av infrastruktur du behöver och installera alla verktyg och programvara som krävs för klassen. 

## <a name="automatic-management-of-azure-infrastructure-and-scale"></a>Automatisk hantering av Azure-infrastruktur och skala 
Azure Lab Services är en hanterad tjänst. Det innebär att etablering och hantering av ett labbs underliggande infrastruktur hanteras automatiskt av tjänsten. Du behöver bara fokusera på att förbereda rätt labbupplevelse för dina användare. Låt tjänsten hantera resten och lansera labbets virtuella datorer till din publik. Skala ditt labb till hundratals virtuella datorer med ett enda klick.

## <a name="simple-experience-for-your-lab-users"></a>Enkelt att använda för dina labbanvändare 
Användare som bjuds in till ditt labb får omedelbar åtkomst till de resurser som du ger dem i dina labb. De behöver bara logga in för att se den fullständig listan över virtuella datorer som de har åtkomst till i flera labb. De kan klicka på en enda knapp för att ansluta till de virtuella datorerna och börja arbeta. Användarna behöver inte ha Azure-prenumerationer för att använda tjänsten. 

## <a name="cost-optimization-and-tracking"></a>Kostnadsoptimering och spårning  
Håll dig inom budgeten genom att kontrollera exakt hur många timmar dina labbanvändare kan använda de virtuella datorerna. Konfigurera scheman i labbet för att ge användarna åtkomst till de virtuella datorerna endast under angivna tidpunkter eller konfigurera återkommande avstängnings- och starttider. Håll koll på individuella användares användning och ange begränsningar.

## <a name="example-class-types"></a>Exempel på klass typer
Du kan ställa in labb för flera typer av klasser med Azure Lab Services. Se [exempel klass typerna på Azure Lab Services](class-types.md) artikel för några exempel typer av klasser för vilka du kan ställa in labb med Azure Lab Services. 

## <a name="next-steps"></a>Nästa steg
Kom igång med att konfigurera ett labbkonto som krävs för att skapa ett klassrumslabb med Azure Lab Services:

- [Konfigurera ett labbkonto](tutorial-setup-lab-account.md)
