---
title: 'Anpassade roller: migrering online SQL Server till SQL-hanterad instans'
titleSuffix: Azure Database Migration Service
description: Lär dig att använda de anpassade rollerna för SQL Server till Azure SQL Database hanterade instanser online-migreringar.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: article
ms.date: 10/25/2019
ms.openlocfilehash: e9a1024ca3ab68841474ab051c029042df4915b5
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "78254947"
---
# <a name="custom-roles-for-sql-server-to-sql-database-managed-instance-online-migrations"></a>Anpassade roller för SQL Server till SQL Database hanterade instanser online-migreringar

Azure Database Migration Service använder ett APP-ID för att interagera med Azure-tjänster. APP-ID: t kräver antingen rollen deltagare på prenumerations nivån (som många företags säkerhets avdelningar inte tillåter) eller skapar anpassade roller som ger de behörigheter som krävs för Azure Database migrations service. Eftersom det finns en gräns på 2 000 anpassade roller i Azure Active Directory kanske du vill kombinera alla behörigheter som krävs specifikt av APP-ID: t i en eller två anpassade roller och sedan ge appen ID den anpassade rollen för specifika objekt eller resurs grupper (jämfört med prenumerations nivån). Om antalet anpassade roller inte är ett problem kan du dela upp de anpassade rollerna efter resurs typ för att skapa tre anpassade roller totalt enligt beskrivningen nedan.

I avsnittet AssignableScopes i JSON-strängen för roll definition kan du styra var behörigheterna ska visas i användar gränssnittet **Lägg till roll tilldelning** i portalen. Du vill förmodligen definiera rollen i resurs gruppen eller till och med resurs nivå för att undvika att användar gränssnittet blir rörigt med extra roller. Observera att detta inte utför den faktiska roll tilldelningen.

## <a name="minimum-number-of-roles"></a>Minsta antal roller

Vi rekommenderar för närvarande att du skapar minst två anpassade roller för APP-ID: t, en på resurs nivå och den andra på prenumerations nivån.

> [!NOTE]
> Det senaste anpassade roll kravet kan komma att tas bort, eftersom nya SQL Database Hanterad instans kod distribueras till Azure.

**Anpassad roll för app-ID**. Den här rollen krävs för att Azure Database Migration Service migrering på *resurs* -eller *resurs grupps* nivå (mer information om app-ID finns i artikeln [använda portalen för att skapa ett Azure AD-program och tjänstens huvud namn som kan komma åt resurser](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)).

```json
{
  "Name": "DMS Role - App ID",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write",
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read",
        "Microsoft.DataMigration/locations/*",
        "Microsoft.DataMigration/services/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/ResourceGroups/<StorageAccount_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<ManagedInstance_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<DMS_rg_name>",
  ]
}
```

**Anpassad roll för app-ID – prenumeration**. Den här rollen krävs för att Azure Database Migration Service migrering på *prenumerations* nivå.

```json
{
  "Name": "DMS Role - App ID - Sub",
  "IsCustom": true,
  "Description": "DMS App ID access at subscription level to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>"
  ]
}
```

JSON ovan måste lagras i tre textfiler och du kan använda antingen AzureRM, AZ PowerShell-cmdletar eller Azure CLI för att skapa roller med hjälp av antingen **New-AzureRmRoleDefinition (AzureRM)** eller **New-AzRoleDefinition (AZ)**.

Mer information finns i artikeln [anpassade roller för Azure-resurser](https://docs.microsoft.com/azure/role-based-access-control/custom-roles).

När du har skapat dessa anpassade roller måste du lägga till roll tilldelningar till användare och APP-ID: n till lämpliga resurser eller resurs grupper:

* Rollen "DMS-roll-app-ID" måste beviljas till det APP-ID som ska användas för migreringen, och även på lagrings kontot Azure Database Migration Service instans och SQL Database hanterade instans resurs nivåer.
* Rollen "DMS-roll-app-ID-sub" måste beviljas till APP-ID på prenumerations nivån (det går inte att bevilja resursen eller resurs gruppen). Det här kravet är tillfälligt tills en kod uppdatering har distribuerats.

## <a name="expanded-number-of-roles"></a>Expanderat antal roller

Om antalet anpassade roller i Azure Active Directory inte är ett problem, rekommenderar vi att du skapar totalt tre roller. Du behöver fortfarande ha rollen "DMS-roll-app-ID – sub", men rollen "DMS-roll-app-ID" ovan är uppdelad efter resurs typ i två olika roller.

**Anpassad roll för APP-ID för SQL Database Hanterad instans**

```json
{
  "Name": "DMS Role - App ID - SQL MI",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<ManagedInstance_rg_name>"
  ]
}
```

**Anpassad roll för APP-ID för lagring**

```json
{
  "Name": "DMS Role - App ID - Storage",
  "IsCustom": true,
  "Description": "DMS App ID storage access to complete MI migrations",
  "Actions": [
"Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<StorageAccount_rg_name>"
  ]
}
```

## <a name="role-assignment"></a>Rolltilldelning

Om du vill tilldela en roll till användare/APP-ID öppnar du Azure Portal och utför följande steg:

1. Navigera till resurs gruppen eller resursen (förutom den roll som behöver beviljas för prenumerationen), går du till **Access Control**och bläddrar sedan för att hitta de anpassade roller som du nyss skapade.

2. Välj lämplig roll, Välj APP-ID och spara ändringarna.

  Dina APP-ID: n visas nu i listan på fliken **roll tilldelningar** .

## <a name="next-steps"></a>Nästa steg

* Läs igenom migrations vägledningen för ditt scenario i Microsoft [Database migration guide](https://datamigration.microsoft.com/).
