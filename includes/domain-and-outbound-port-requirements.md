---
title: ta med fil
description: ta med fil
services: data-factory
author: nabhishek
ms.service: data-factory
ms.topic: include
ms.date: 10/09/2019
ms.author: abnarain
ms.openlocfilehash: 6af6795fefb41f1d8f2b56e0aa1fb367fc18cee2
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "74559309"
---
| Domännamn                  | Utgående portar | Beskrivning                              |
| ----------------------------- | -------------- | ---------------------------------------- |
| `*.servicebus.windows.net`    | 443            | Krävs av integration runtime med egen värd för att ansluta till data flytt tjänster i Azure Data Factory. |
| `*.frontend.clouddatahub.net` | 443            | Krävs av integration runtime med egen värd för att ansluta till tjänsten Data Factory. |
| `download.microsoft.com`    | 443            | Krävs av den egna värdbaserade integrerings körningen för att ladda ned uppdateringarna. Om du har inaktiverat automatisk uppdatering kan du hoppa över konfigurationen av den här domänen. |
| `*.core.windows.net`          | 443            | Används av integration runtime med egen värd för att ansluta till Azure Storage-kontot när du använder funktionen för [mellanlagrad kopiering](https://docs.microsoft.com/azure/data-factory/copy-activity-performance#staged-copy) . |
| `*.database.windows.net`      | 1433           | Krävs endast när du kopierar från eller till Azure SQL Database eller Azure SQL Data Warehouse och valfritt annat. Använd funktionen för mellanlagrad kopiering för att kopiera data till SQL Database eller SQL Data Warehouse utan att öppna port 1433. |
| `*.azuredatalakestore.net`<br>`login.microsoftonline.com/<tenant>/oauth2/token`    | 443            | Krävs endast när du kopierar från eller till Azure Data Lake Store och valfritt annat. |
