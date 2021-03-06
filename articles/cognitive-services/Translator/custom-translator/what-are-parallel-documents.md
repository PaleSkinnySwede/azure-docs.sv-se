---
title: Vad är parallella dokument? – Anpassad översättare
titleSuffix: Azure Cognitive Services
description: Parallella dokument är par med dokument där det ena är översättningen av det andra. Ett dokument i paret innehåller meningar i käll språket och det andra dokumentet innehåller de meningarna översatta till mål språket.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 02/21/2019
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: d7c38a44e3111a319e4146b3c9b71a22b0d31bfd
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "72675472"
---
# <a name="what-are-parallel-documents"></a>Vad är parallella dokument?

Parallella dokument är par med dokument där det ena är översättningen av det andra. Ett dokument i paret innehåller meningar i käll språket och det andra dokumentet innehåller de meningarna översatta till mål språket.
Det spelar ingen roll vilket språk som har marker ATS som "källa" och vilket språk som marker ATS som "mål" – ett parallellt dokument kan användas för att träna ett översättnings system i båda riktningarna.

## <a name="requirements"></a>Krav

Du behöver minst 10 000 unika justerade parallella meningar för att träna ett system. Den här begränsningen är ett säkerhets nät för att se till att dina parallella meningar innehåller tillräckligt med unik vokabulär för att kunna träna en översättnings modell. Bästa praxis är att kontinuerligt lägga till mer Parallel-innehåll och omträna för att förbättra kvaliteten på ditt översättnings system. Se [menings justering](https://docs.microsoft.com/azure/cognitive-services/translator/custom-translator/sentence-alignment).

Microsoft kräver att dokument som överförs till den anpassade översättaren inte bryter mot en tredje parts upphovs rätt eller immateriell egendom. Mer information finns i [användnings villkoren](https://azure.microsoft.com/support/legal/cognitive-services-terms/).
Att ladda upp ett dokument med hjälp av portalen ändrar inte ägande rätten till den immateriella egendomen i själva dokumentet.

## <a name="use-of-parallel-documents"></a>Användning av parallella dokument

Parallella dokument används av systemet:

1.  För att lära dig hur ord, fraser och meningar ofta mappas mellan de två språken.

2.  För att lära dig att bearbeta lämplig kontext beroende på de omgivande fraserna. Ett ord kan inte alltid översättas till exakt samma ord på det andra språket.

Vi rekommenderar att du ser till att det finns en 1:1 mening mellan käll-och mål språk versionerna av dokumenten.

Om projektet är en domän (kategori) bör dokumenten vara konsekventa i terminologin i den kategorin. Kvaliteten på det resulterande översättnings systemet beror på antalet meningar i dokument uppsättningen och meningens kvalitet. De fler exempel som dokumenten innehåller med olika användnings områden för ett ord som är speciellt för din kategori, vilket är det bästa jobbet som systemet kan utföra under översättningen.

Överförda dokument är privata för varje arbets yta och kan användas i så många projekt eller utbildningar som du vill. Meningar som extraheras från dina dokument lagras separat i lagrings platsen som vanliga Unicode-textfiler och är tillgängliga för borttagning. Använd inte den anpassade översättaren som en dokument lagrings plats. du kommer inte att kunna ladda ned de dokument du laddade upp i det format du laddade upp dem.



## <a name="next-steps"></a>Nästa steg

- Lär dig hur du använder en [ord lista](what-is-dictionary.md) i en anpassad översättare.
