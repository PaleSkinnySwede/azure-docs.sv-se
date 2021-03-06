---
title: Design för språk – QnA Maker
description: QnA Maker resursen och alla kunskaps baser i resursen stöder ett enda språk. Det enda språket är nödvändigt för att ge bästa svars resultat för en fråga.
ms.topic: conceptual
ms.date: 01/27/2020
ms.openlocfilehash: 5cb1dcd35649debbafd2e234606ad4c9d6906ea6
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "76843428"
---
# <a name="design-knowledge-base-for-content-language"></a>Utforma kunskaps bas för innehålls språk

QnA Maker resursen och alla kunskaps baser i resursen, stöder ett enda språk. Det enda språket är nödvändigt för att ge bästa svars resultat för en fråga.

## <a name="single-language-per-resource"></a>Enskilt språk per resurs

QnA Maker överväganden för språk stöd:

* En QnA Maker tjänst, och alla dess kunskaps baser, stöder bara ett språk.
* Språket anges explicit när den första kunskaps basen i tjänsten skapas
* Språket bestäms av de filer och URL: er som läggs till när kunskaps basen skapas
* Det går inte att ändra språket för andra kunskaps banker i tjänsten
* Språket används av Kognitiv sökning tjänsten (Ranging #1) och QnA Maker tjänsten (Ranging #2) för att generera det bästa svaret på en fråga

## <a name="supporting-multiple-languages"></a>Stöd för flera språk

Om du behöver stöd för ett kunskaps bas system, som innehåller flera språk, kan du välja någon av följande metoder:

* Använd [översättnings text tjänsten](../../translator/translator-info-overview.md) för att översätta en fråga till ett språk innan du skickar frågan till din kunskaps bas. På så sätt kan du fokusera på kvaliteten på ett enda språk och kvaliteten på de alternativa frågorna och svaren.
* Skapa en QnA Maker resurs och en kunskaps bas i resursen för varje språk. På så sätt kan du hantera separata alternativa frågor och svars text som är mer nyanserade för varje språk. Detta ger dig mycket mer flexibilitet men kräver en mycket högre underhålls kostnad när frågorna eller Svaren ändras på alla språk.

Granska [språk som stöds](../overview/language-support.md) för QNA Maker.

### <a name="support-each-language-with-a-qna-maker-resource"></a>Stöd för varje språk med en QnA Maker-resurs

* Skapa en QnA Maker resurs för varje språk
* Lägg endast till filer och URL: er för språket
* Använd en namngivnings konvention för resursen för att identifiera språket. Ett exempel är `qna-maker-fr` för alla kunskaps baser om franska dokument

## <a name="next-steps"></a>Nästa steg

Lär dig [begrepp](query-knowledge-base.md) om hur du frågar kunskaps basen efter ett svar.