---
title: CreateUiDefinition. JSON-fil för Portal fönstret
description: Beskriver hur du skapar användar gränssnitts definitioner för Azure Portal. Används när du definierar Azure Managed Applications.
author: tfitzmac
ms.topic: conceptual
ms.date: 08/06/2019
ms.author: tomfitz
ms.openlocfilehash: 2956c76f5bec353639b39228b982db21b6932deb
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80294899"
---
# <a name="createuidefinitionjson-for-azure-managed-applications-create-experience"></a>CreateUiDefinition.json för Azure-upplevelsen för att skapa hanterade program

I det här dokumentet beskrivs de grundläggande begreppen i filen **createUiDefinition. JSON** som Azure Portal använder för att definiera användar gränssnittet när du skapar ett hanterat program.

Mallen är följande

```json
{
   "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
   "handler": "Microsoft.Azure.CreateUIDef",
   "version": "0.1.2-preview",
   "parameters": {
      "basics": [ ],
      "steps": [ ],
      "outputs": { },
      "resourceTypes": [ ]
   }
}
```

En CreateUiDefinition innehåller alltid tre egenskaper: 

* protokollhanteraren
* version
* parameters

Hanteraren bör alltid vara `Microsoft.Azure.CreateUIDef`och den senaste versionen som stöds är. `0.1.2-preview`

Schemat för Parameters-egenskapen är beroende av kombinationen av den angivna hanteraren och versionen. För hanterade program är de egenskaper som `basics`stöds `steps`,, `outputs`och. Egenskaperna grundläggande och steg innehåller [elementen](create-uidefinition-elements.md) , t. ex. text rutor och list rutor, som ska visas i Azure Portal. Egenskapen outputs används för att mappa indatavärdena för de angivna elementen till parametrarna i mallen för Azure Resource Manager distribution.

Inklusive `$schema` rekommenderas, men valfritt. Om det anges måste värdet för `version` matcha versionen i `$schema` URI: n.

Du kan använda en JSON-redigerare för att skapa din createUiDefinition och sedan testa den i [sand boxen createUiDefinition](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade) för att förhandsgranska den. Mer information om sandbox finns i [Testa ditt Portal gränssnitt för Azure Managed Applications](test-createuidefinition.md).

## <a name="basics"></a>Grundläggande inställningar

Grunderna är det första steget som skapas när Azure Portal tolkar filen. Förutom att visa de element som anges i `basics`, infogar portalen element för användare för att välja prenumeration, resurs grupp och plats för distributionen. När det är möjligt bör element som frågar om distributionens globala parametrar, t. ex. namnet på ett kluster eller administratörs behörighet, fortsätta i det här steget.

## <a name="steps"></a>Steg

Egenskapen steg kan innehålla noll eller ytterligare ytterligare steg för att visa efter grunderna, som innehåller ett eller flera element. Överväg att lägga till steg per roll eller nivå för det program som distribueras. Du kan till exempel lägga till ett steg för indata för huvudnoder och ett steg för arbetsnoder i ett kluster.

## <a name="outputs"></a>Utdata

Azure Portal använder `outputs` egenskapen för att mappa element från `basics` och `steps` till parametrarna i mallen för Azure Resource Manager distribution. Nycklarna för den här ord listan är namnen på mallparametrar och värdena är egenskaper för de utgående objekten från de refererade elementen.

Om du vill ange resurs namnet för den hanterade appen måste du inkludera `applicationResourceName` ett värde med namnet i egenskapen outputs. Om du inte anger det här värdet tilldelar programmet ett GUID för namnet. Du kan inkludera en text ruta i användar gränssnittet som begär ett namn från användaren.

```json
"outputs": {
    "vmName": "[steps('appSettings').vmName]",
    "trialOrProduction": "[steps('appSettings').trialOrProd]",
    "userName": "[steps('vmCredentials').adminUsername]",
    "pwd": "[steps('vmCredentials').vmPwd.password]",
    "applicationResourceName": "[steps('appSettings').vmName]"
}
```

## <a name="resource-types"></a>Resurstyper

Om du vill filtrera tillgängliga platser till endast de platser som har stöd för de resurs typer som ska distribueras, anger du en matris av resurs typerna. Om du anger mer än en resurs typ returneras bara de platser som stöder alla resurs typer. Den här egenskapen är valfri.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Azure.CreateUIDef",
    "version": "0.1.2-preview",
    "parameters": {
      "resourceTypes": ["Microsoft.Compute/disks"],
      "basics": [
        ...
```  

## <a name="functions"></a>Functions

CreateUiDefinition innehåller [funktioner](create-uidefinition-functions.md) för att arbeta med element indata och utdata, samt funktioner som till exempel villkor. Dessa funktioner liknar funktionerna i både syntax och funktioner i Azure Resource Manager mallar.

## <a name="next-steps"></a>Nästa steg

Filen createUiDefinition. JSON har ett enkelt schema. Det verkliga djupet i den kommer från alla element och funktioner som stöds. Dessa objekt beskrivs mer detaljerat på:

- [Element](create-uidefinition-elements.md)
- [Functions](create-uidefinition-functions.md)

Ett aktuellt JSON-schema för createUiDefinition finns här: `https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json`.

Ett exempel på en användar gränssnitts fil finns i [createUiDefinition. JSON](https://github.com/Azure/azure-managedapp-samples/blob/master/Managed%20Application%20Sample%20Packages/201-managed-app-using-existing-vnet/createUiDefinition.json).
