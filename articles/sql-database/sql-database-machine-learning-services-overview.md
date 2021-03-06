---
title: Machine Learning Services med R (för hands version)
description: Den här artikeln beskriver Azure SQL Database Machine Learning Services (med R) och förklarar hur det fungerar.
services: sql-database
ms.service: sql-database
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: dphansen
ms.author: davidph
ms.reviewer: carlrab
manager: cgronlun
ms.date: 11/20/2019
ROBOTS: NOINDEX
ms.openlocfilehash: 46ca4661d06b52c861431a680a69297575ac99b0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81461420"
---
# <a name="azure-sql-database-machine-learning-services-with-r-preview"></a>Azure SQL Database Machine Learning Services med R (för hands version)

Machine Learning Services är en funktion i Azure SQL Database som används för att köra in-Database R-skript. Funktionen innehåller Microsoft R-paket för förutsägelse analys och maskin inlärning med höga prestanda. Relations data kan användas i R-skript genom lagrade procedurer, T-SQL-skript som innehåller R-instruktioner eller R-kod som innehåller T-SQL.

[!INCLUDE[ml-preview-note](../../includes/sql-database-ml-preview-note.md)]

## <a name="what-you-can-do-with-r"></a>Vad du kan göra med R

Använd kraften hos R-språket för att leverera avancerad analys och maskininlärning i databasen. Den här funktionen ger beräkning och bearbetning där data finns, vilket eliminerar behovet av att hämta data över nätverket. Du kan också utnyttja kraften hos Enterprise R-paket för att leverera avancerad analys i stor skala.

Machine Learning Services innehåller en grundläggande distribution av R med överlägg med R-företagspaket från Microsoft. Microsofts R-funktioner och algoritmer är utformade för både stor skala och nytta. De levererar förutsägelseanalys, statistisk modellering, datavisualiseringar och ledande algoritmer för maskininlärning.

### <a name="r-packages"></a>R-paket

De flesta vanliga R-paket med öppen källkod är förinstallerade i Machine Learning Services. Följande R-paket från Microsoft ingår också:

| R-paket | Beskrivning|
|-|-|
| [Microsoft R Open](https://mran.microsoft.com/rro) | Microsoft R Open är den förbättrade distributionen av R från Microsoft. Det är en komplett plattform med öppen källkod för statistisk analys och data vetenskap. Den är baserad på och 100% kompatibel med R och innehåller ytterligare funktioner för bättre prestanda och reproducerbarhet. |
| [RevoScaleR](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-revoscaler) | RevoScaleR är det primära biblioteket för skalbar R. funktioner i det här biblioteket är bland de vanligaste. Data transformationer och manipulering, statistisk sammanfattning, visualisering och många former av modellering och analyser finns i dessa bibliotek. Dessutom distribuerar funktioner i dessa bibliotek automatiskt arbets belastningar över tillgängliga kärnor för parallell bearbetning, med möjlighet att arbeta med segment med data som koordineras och hanteras av beräknings motorn. |
| [MicrosoftML (R)](https://docs.microsoft.com/sql/advanced-analytics/r/ref-r-microsoftml) | MicrosoftML lägger till Machine Learning-algoritmer för att skapa anpassade modeller för text analys, bild analys och sentiment analys. |

Förutom de förinstallerade paketen kan du [installera ytterligare paket](sql-database-machine-learning-services-add-r-packages.md).

<a name="signup"></a>

## <a name="sign-up-for-the-preview-closed"></a>Registrera dig för för hands versionen (stängd)

> [!IMPORTANT]
> Registrera dig för Azure SQL Database Machine Learning Services (förhands granskning) är för närvarande **stängd**.

## <a name="next-steps"></a>Nästa steg

- Se [viktiga skillnader från SQL Server Machine Learning Services](sql-database-machine-learning-services-differences.md).
- Information om hur du använder R för att fråga Azure SQL Database Machine Learning Services (förhands granskning) finns i [snabb starts guiden](sql-database-connect-query-r.md).
- För att komma igång med vissa enkla R-skript, se [skapa och köra enkla r-skript i Azure SQL Database Machine Learning Services (för hands version)](sql-database-quickstart-r-create-script.md).
