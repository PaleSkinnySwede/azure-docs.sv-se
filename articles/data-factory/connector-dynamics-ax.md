---
title: Kopiera data från Dynamics AX
description: Lär dig hur du kopierar data från Dynamics AX till mottagar data lager som stöds med hjälp av en kopierings aktivitet i en Azure Data Factory pipeline.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2019
ms.openlocfilehash: 4dd82eea0a80ef81a0f972d1964a62e6c17a80c0
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81417369"
---
# <a name="copy-data-from-dynamics-ax-by-using-azure-data-factory"></a>Kopiera data från Dynamics AX med hjälp av Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Den här artikeln beskriver hur du använder kopierings aktivitet i Azure Data Factory för att kopiera data från Dynamics AX-källan. Artikeln bygger på [kopierings aktivitet i Azure Data Factory](copy-activity-overview.md), som visar en översikt över kopierings aktiviteten.

## <a name="supported-capabilities"></a>Funktioner som stöds

Den här Dynamics AX-anslutningen stöds för följande aktiviteter:

- [Kopierings aktivitet](copy-activity-overview.md) med [matrisen source/Sink som stöds](copy-activity-overview.md)
- [Söknings aktivitet](control-flow-lookup-activity.md)

Du kan kopiera data från Dynamics AX till alla mottagar data lager som stöds. En lista över data lager som kopierings aktiviteten stöder som källor och mottagare finns i [data lager och format som stöds](copy-activity-overview.md#supported-data-stores-and-formats).

Mer specifikt stöder den här Dynamics AX-anslutaren kopiering av data från Dynamics AX med hjälp av **OData-protokollet** med **autentisering av tjänstens huvud namn**.

>[!TIP]
>Du kan också använda den här anslutningen för att kopiera data från **ekonomi och åtgärder i Dynamics 365**. Se Dynamics 365: s [OData-support](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/data-entities/odata) och [autentiseringsmetod](https://docs.microsoft.com/dynamics365/unified-operations/dev-itpro/data-entities/services-home-page#authentication).

## <a name="get-started"></a>Kom igång

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Följande avsnitt innehåller information om egenskaper som du kan använda för att definiera Data Factory entiteter som är speciella för Dynamics AX-anslutning.

## <a name="prerequisites"></a>Krav

Följ dessa steg om du vill använda autentisering av tjänstens huvud namn:

1. Registrera en program enhet i Azure Active Directory (Azure AD) genom [att följa registrera ditt program med en Azure AD-klient](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Anteckna följande värden som du använder för att definiera den länkade tjänsten:

    - Program-ID:t
    - Program nyckel
    - Klientorganisations-ID

2. Gå till Dynamics AX och ge den här tjänstens huvud behörighet rätt behörighet för att komma åt Dynamics AX.

## <a name="linked-service-properties"></a>Egenskaper för länkad tjänst

Följande egenskaper stöds för Dynamics AX-länkad tjänst:

| Egenskap | Beskrivning | Krävs |
|:--- |:--- |:--- |
| typ | Egenskapen **Type** måste anges till **DynamicsAx**. |Ja |
| url | OData-slutpunkten för Dynamics AX (eller Dynamics 365 finans och Operations). |Ja |
| servicePrincipalId | Ange programmets klient-ID. | Ja |
| servicePrincipalKey | Ange programmets nyckel. Markera det här fältet som **SecureString** för att lagra det på ett säkert sätt i Data Factory eller [referera till en hemlighet som lagras i Azure Key Vault](store-credentials-in-key-vault.md). | Ja |
| tenant | Ange den klient information (domän namn eller klient-ID) som programmet finns under. Hämta det genom att hovra musen i det övre högra hörnet av Azure Portal. | Ja |
| aadResourceId | Ange den AAD-resurs som du begär för auktorisering. Om din Dynamics URL till exempel är `https://sampledynamics.sandbox.operations.dynamics.com/data/`, är motsvarande AAD-resurs vanligt vis `https://sampledynamics.sandbox.operations.dynamics.com`. | Ja |
| connectVia | [Integration runtime](concepts-integration-runtime.md) som ska användas för att ansluta till data lagret. Du kan välja Azure Integration Runtime eller en egen värd Integration Runtime (om ditt data lager finns i ett privat nätverk). Om inget värde anges används standard Azure Integration Runtime. |Nej |

**Exempel**

```json
{
    "name": "DynamicsAXLinkedService",
    "properties": {
        "type": "DynamicsAX",
        "typeProperties": {
            "url": "<Dynamics AX instance OData endpoint>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource, e.g. https://sampledynamics.sandbox.operations.dynamics.com>"
        }
    },
    "connectVia": {
        "referenceName": "<name of Integration Runtime>",
        "type": "IntegrationRuntimeReference"
    }
}

```

## <a name="dataset-properties"></a>Egenskaper för datamängd

Det här avsnittet innehåller en lista över egenskaper som stöds av Dynamics AX-datauppsättningen.

En fullständig lista över avsnitt och egenskaper som är tillgängliga för att definiera data uppsättningar finns i [data uppsättningar och länkade tjänster](concepts-datasets-linked-services.md). 

Om du vill kopiera data från Dynamics AX anger du egenskapen **Type** för data uppsättningen till **DynamicsAXResource**. Följande egenskaper stöds:

| Egenskap | Beskrivning | Krävs |
|:--- |:--- |:--- |
| typ | Data uppsättningens **typ** -egenskap måste anges till **DynamicsAXResource**. | Ja |
| path | Sökvägen till Dynamics AX OData-entiteten. | Ja |

**Exempel**

```json
{
    "name": "DynamicsAXResourceDataset",
    "properties": {
        "type": "DynamicsAXResource",
        "typeProperties": {
            "path": "<entity path e.g. dd04tentitySet>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Dynamics AX linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Kopiera aktivitets egenskaper

Det här avsnittet innehåller en lista över egenskaper som Dynamics AX-källan stöder.

En fullständig lista över avsnitt och egenskaper som är tillgängliga för att definiera aktiviteter finns i [pipelines](concepts-pipelines-activities.md). 

### <a name="dynamics-ax-as-source"></a>Dynamics AX som källa

Om du vill kopiera data från Dynamics AX anger du **käll** typen i kopierings aktivitet till **DynamicsAXSource**. Följande egenskaper stöds i avsnittet Kopiera aktivitets **källa** :

| Egenskap | Beskrivning | Krävs |
|:--- |:--- |:--- |
| typ | **Typ** egenskapen för kopierings aktivitets källan måste anges till **DynamicsAXSource**. | Ja |
| DocumentDB | OData-frågealternativ för att filtrera data. Exempel: `"?$select=Name,Description&$top=5"`.<br/><br/>**Obs!** anslutningen kopierar data från den kombinerade URL: en `[URL specified in linked service]/[path specified in dataset][query specified in copy activity source]`:. Mer information finns i [OData URL-komponenter](https://www.odata.org/documentation/odata-version-3-0/url-conventions/). | Nej |

**Exempel**

```json
"activities":[
    {
        "name": "CopyFromDynamicsAX",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics AX input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DynamicsAXSource",
                "query": "$top=10"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```


## <a name="lookup-activity-properties"></a>Egenskaper för Sök aktivitet

Om du vill veta mer om egenskaperna kontrollerar du [söknings aktiviteten](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Nästa steg

En lista över data lager som kopierings aktiviteten stöder som källor och handfat i Azure Data Factory finns i [data lager och format som stöds](copy-activity-overview.md#supported-data-stores-and-formats).
