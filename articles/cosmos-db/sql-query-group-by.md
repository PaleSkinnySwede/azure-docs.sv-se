---
title: GROUP BY-sats i Azure Cosmos DB
description: Lär dig mer om GROUP BY-satsen för Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/10/2020
ms.author: tisande
ms.openlocfilehash: 8a3cbbafc066747b62f79934f2cd12301aa1ba17
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81261609"
---
# <a name="group-by-clause-in-azure-cosmos-db"></a>GROUP BY-sats i Azure Cosmos DB

GROUP BY-satsen delar in frågans resultat enligt värdena för en eller flera angivna egenskaper.

> [!NOTE]
> Azure Cosmos DB stöder för närvarande GROUP BY i .NET SDK 3,3 och senare samt Java Script SDK 3,4 och senare.
> Stöd för andra språk-SDK: er är inte tillgängligt för tillfället, men det är planerat.

## <a name="syntax"></a>Syntax

```sql  
<group_by_clause> ::= GROUP BY <scalar_expression_list>

<scalar_expression_list> ::=
          <scalar_expression>
        | <scalar_expression_list>, <scalar_expression>
```  

## <a name="arguments"></a>Argument

- `<scalar_expression_list>`

   Anger de uttryck som ska användas för att dividera frågeresultaten.

- `<scalar_expression>`
  
   Ett skalärt uttryck tillåts förutom skalära under frågor och skalära mängder. Varje skalärt uttryck måste innehålla minst en egenskaps referens. Det finns ingen gräns för antalet enskilda uttryck eller kardinalitet för varje uttryck.

## <a name="remarks"></a>Anmärkningar
  
  När en fråga använder en GROUP BY-sats kan SELECT-satsen bara innehålla del mängden av de egenskaper och system funktioner som ingår i GROUP BY-satsen. Ett undantag är [agg regerings system funktioner](sql-query-aggregates.md)som kan visas i SELECT-satsen utan att ingå i Group by-satsen. Du kan också alltid inkludera litterala värden i SELECT-satsen.

  GROUP BY-satsen måste vara efter SELECT-, FROM-och WHERE-satsen och före OFFSET LIMIT-satsen. Du kan för närvarande inte använda GROUP BY med en ORDER BY-sats men detta är planerat.

  GROUP BY-satsen tillåter inte något av följande:
  
- Egenskaper för alias eller alias för system funktioner (alias tillåts fortfarande i SELECT-satsen)
- Under frågor
- Sammanställda system funktioner (dessa är endast tillåtna i SELECT-satsen)

Frågor med en sammansatt systemfunktion och en under fråga med `GROUP BY` stöds inte. Följande fråga stöds exempelvis inte:

```sql
SELECT COUNT(UniqueLastNames) FROM (SELECT AVG(f.age) FROM f GROUP BY f.lastName) AS UniqueLastNames
```

## <a name="examples"></a>Exempel

I de här exemplen används närings data uppsättningen som är tillgänglig via [Azure Cosmos DB testplats för databasfrågor](https://www.documentdb.com/sql/demo).

Här är ett exempel på en fråga som returnerar det totala antalet objekt i varje foodGroup:

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, f.foodGroup
FROM Food f
GROUP BY f.foodGroup
```

Vissa resultat är (det översta nyckelordet används för att begränsa resultaten):

```json
[{
  "foodGroup": "Fast Foods",
  "foodGroupCount": 371
},
{
  "foodGroup": "Finfish and Shellfish Products",
  "foodGroupCount": 267
},
{
  "foodGroup": "Meals, Entrees, and Side Dishes",
  "foodGroupCount": 113
},
{
  "foodGroup": "Sausages and Luncheon Meats",
  "foodGroupCount": 244
}]
```

Den här frågan har två uttryck som används för att dividera resultat:

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, f.foodGroup, f.version
FROM Food f
GROUP BY f.foodGroup, f.version
```

Vissa resultat:

```json
[{
  "version": 1,
  "foodGroup": "Nut and Seed Products",
  "foodGroupCount": 133
},
{
  "version": 1,
  "foodGroup": "Finfish and Shellfish Products",
  "foodGroupCount": 267
},
{
  "version": 1,
  "foodGroup": "Fast Foods",
  "foodGroupCount": 371
},
{
  "version": 1,
  "foodGroup": "Sausages and Luncheon Meats",
  "foodGroupCount": 244
}]
```

Den här frågan har en systemfunktion i GROUP BY-satsen:

```sql
SELECT TOP 4 COUNT(1) AS foodGroupCount, UPPER(f.foodGroup) AS upperFoodGroup
FROM Food f
GROUP BY UPPER(f.foodGroup)
```

Vissa resultat:

```json
[{
  "foodGroupCount": 371,
  "upperFoodGroup": "FAST FOODS"
},
{
  "foodGroupCount": 267,
  "upperFoodGroup": "FINFISH AND SHELLFISH PRODUCTS"
},
{
  "foodGroupCount": 389,
  "upperFoodGroup": "LEGUMES AND LEGUME PRODUCTS"
},
{
  "foodGroupCount": 113,
  "upperFoodGroup": "MEALS, ENTREES, AND SIDE DISHES"
}]
```

Den här frågan använder både nyckelord och system funktioner i uttryck för objekt egenskaper:

```sql
SELECT COUNT(1) AS foodGroupCount, ARRAY_CONTAINS(f.tags, {name: 'orange'}) AS containsOrangeTag,  f.version BETWEEN 0 AND 2 AS correctVersion
FROM Food f
GROUP BY ARRAY_CONTAINS(f.tags, {name: 'orange'}), f.version BETWEEN 0 AND 2
```

Resultatet är:

```json
[{
  "correctVersion": true,
  "containsOrangeTag": false,
  "foodGroupCount": 8608
},
{
  "correctVersion": true,
  "containsOrangeTag": true,
  "foodGroupCount": 10
}]
```

## <a name="next-steps"></a>Nästa steg

- [Komma igång](sql-query-getting-started.md)
- [SELECT-sats](sql-query-select.md)
- [Mängd funktioner](sql-query-aggregates.md)
