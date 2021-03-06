---
title: GetCurrentDateTime i Azure Cosmos DB frågespråk
description: Lär dig mer om SQL system Function GetCurrentDateTime i Azure Cosmos DB.
author: ginamr
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/13/2019
ms.author: girobins
ms.custom: query-reference
ms.openlocfilehash: d50b08ab85c7e299c465c3eb6f34e867d6634006
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "78303910"
---
# <a name="getcurrentdatetime-azure-cosmos-db"></a>GetCurrentDateTime (Azure Cosmos DB)
 Returnerar den aktuella UTC-tiden (Coordinated Universal Time) som en ISO 8601-sträng.
  
## <a name="syntax"></a>Syntax
  
```sql
GetCurrentDateTime ()
```
  
## <a name="return-types"></a>Retur typer
  
  Returnerar aktuellt UTC-datum och klock slags ISO 8601-sträng värde `YYYY-MM-DDThh:mm:ss.fffffffZ` i formatet där:
  
  |||
  |-|-|
  |MMMM|fyrsiffrigt år|
  |MM|tvåsiffrig månad (01 = januari osv.)|
  |FORMATET|fyrsiffrigt dag i månad (01 till 31)|
  |T|indikerare för start av tids element|
  |hh|Dubbels siffrig timme (00 – 23)|
  |mm|två-siffriga minuter (00 till 59)|
  |ss|tvåsiffriga sekunder (00 till 59)|
  |.fffffff|sju siffrors decimal sekunder|
  |Z|UTC-beteckning (Coordinated Universal Time)||
  
  Mer information om ISO 8601-formatet finns i [ISO_8601](https://en.wikipedia.org/wiki/ISO_8601)

## <a name="remarks"></a>Anmärkningar

  GetCurrentDateTime () är en icke-deterministisk funktion. 
  
  Resultatet som returneras är UTC.

  Precision är 7 siffror, med en noggrannhet på 100 nanosekunder.

## <a name="examples"></a>Exempel
  
  I följande exempel visas hur du hämtar den aktuella UTC-datum tiden med den inbyggda funktionen GetCurrentDateTime ().
  
```sql
SELECT GetCurrentDateTime() AS currentUtcDateTime
```  
  
 Här är ett exempel på en resultat uppsättning.
  
```json
[{
  "currentUtcDateTime": "2019-05-03T20:36:17.1234567Z"
}]  
```  

## <a name="next-steps"></a>Nästa steg

- [Datum-och tids funktioner Azure Cosmos DB](sql-query-date-time-functions.md)
- [System funktioner Azure Cosmos DB](sql-query-system-functions.md)
- [Introduktion till Azure Cosmos DB](introduction.md)
