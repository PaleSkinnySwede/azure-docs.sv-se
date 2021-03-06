---
title: Använd Azure Portal för att tilldela en RBAC-roll för data åtkomst
titleSuffix: Azure Storage
description: Lär dig hur du använder Azure Portal för att tilldela behörigheter till ett Azure Active Directory säkerhets objekt med rollbaserad åtkomst kontroll (RBAC). Azure Storage stöder inbyggda och anpassade RBAC-roller för autentisering via Azure AD.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 03/31/2020
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: d224bd9e9e7b1f8fc9eb45d85e78811d8642fc78
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80519568"
---
# <a name="use-the-azure-portal-to-assign-an-rbac-role-for-access-to-blob-and-queue-data"></a>Använd Azure Portal för att tilldela en RBAC-roll för åtkomst till blob-och Queue-data

Azure Active Directory (Azure AD) tillåter åtkomst rättigheter till skyddade resurser via [rollbaserad åtkomst kontroll (RBAC)](../../role-based-access-control/overview.md). Azure Storage definierar en uppsättning inbyggda RBAC-roller som omfattar vanliga uppsättningar av behörigheter som används för att få åtkomst till BLOB-eller Queue data.

När en RBAC-roll tilldelas till ett säkerhets objekt i Azure AD ger Azure åtkomst till dessa resurser för säkerhets objekt. Åtkomst kan begränsas till prenumerations nivån, resurs gruppen, lagrings kontot eller en enskild behållare eller kö. Ett säkerhets objekt i Azure AD kan vara en användare, en grupp, ett huvud namn för program tjänsten eller en [hanterad identitet för Azure-resurser](../../active-directory/managed-identities-azure-resources/overview.md).

Den här artikeln beskriver hur du använder Azure Portal för att tilldela RBAC-roller. Azure Portal ger ett enkelt gränssnitt för att tilldela RBAC-roller och hantera åtkomst till dina lagrings resurser. Du kan också tilldela RBAC-roller för blob-och Queue-resurser med hjälp av Azures kommando rads verktyg eller API: er för Azure Storage hantering. Mer information om RBAC-roller för lagrings resurser finns i [autentisera åtkomst till Azure-blobbar och köer med hjälp av Azure Active Directory](storage-auth-aad.md).

## <a name="rbac-roles-for-blobs-and-queues"></a>RBAC-roller för blobbar och köer

[!INCLUDE [storage-auth-rbac-roles-include](../../../includes/storage-auth-rbac-roles-include.md)]

## <a name="determine-resource-scope"></a>Bestäm resurs omfång

[!INCLUDE [storage-auth-resource-scope-include](../../../includes/storage-auth-resource-scope-include.md)]

## <a name="assign-rbac-roles-using-the-azure-portal"></a>Tilldela RBAC-roller med hjälp av Azure Portal

När du har bestämt lämplig omfattning för en roll tilldelning navigerar du till resursen i Azure Portal. Visa inställningarna för **Access Control (IAM)** för resursen och följ de här anvisningarna för att hantera roll tilldelningar:

1. Tilldela lämplig Azure Storage RBAC-roll för att bevilja åtkomst till ett säkerhets objekt i Azure AD.

1. Tilldela rollen Azure Resource Manager [läsare](../../role-based-access-control/built-in-roles.md#reader) till användare som behöver åtkomst till behållare eller köer via Azure Portal med sina autentiseringsuppgifter för Azure AD. 

I följande avsnitt beskrivs var och en av de här stegen i detalj.

> [!NOTE]
> Som ägare till ditt Azure Storage-konto tilldelas du inte automatiskt behörigheter för åtkomst till data. Du måste uttryckligen tilldela dig en RBAC-roll för Azure Storage. Du kan tilldela den på nivån för din prenumeration, resurs grupp, lagrings konto eller en behållare eller kö.
>
> Du kan inte tilldela en roll som är begränsad till en behållare eller kö om ditt lagrings konto har ett hierarkiskt namn område aktiverat.

### <a name="assign-a-built-in-rbac-role"></a>Tilldela en inbyggd RBAC-roll

Innan du tilldelar en roll till ett säkerhets objekt måste du ta hänsyn till omfattningen av de behörigheter som du beviljar. Granska avsnittet [Bestäm resurs omfång](#determine-resource-scope) för att bestämma lämplig omfattning.

Proceduren som visas här tilldelar en behållare en roll som är begränsad till en behållare, men du kan följa samma steg för att tilldela en roll som är begränsad till en kö:

1. I [Azure Portal](https://portal.azure.com)går du till ditt lagrings konto och visar **översikten** för kontot.
1. Under tjänster väljer du **blobbar**.
1. Leta upp den behållare som du vill tilldela en roll till och Visa behållarens inställningar.
1. Välj **åtkomst kontroll (IAM)** om du vill visa inställningar för åtkomst kontroll för behållaren. Välj fliken **roll tilldelningar** om du vill se en lista över roll tilldelningar.

    ![Skärm bild som visar inställningar för åtkomst kontroll för behållare](media/storage-auth-aad-rbac-portal/portal-access-control-for-storage.png)

1. Klicka på knappen **Lägg till roll tilldelning** för att lägga till en ny roll.
1. I fönstret **Lägg till roll tilldelning** väljer du den Azure Storages roll som du vill tilldela. Sök sedan efter det säkerhets objekt som du vill tilldela rollen.

    ![Skärm bild som visar hur du tilldelar en RBAC-roll](media/storage-auth-aad-rbac-portal/add-rbac-role.png)

1. Klicka på **Spara**. Identiteten som du har tilldelat rollen visas i listan under den rollen. Följande bild visar till exempel att användaren lade till nu har Läs behörighet till data i behållaren med namnet *Sample-container*.

    ![Skärm bild som visar en lista över användare som har tilldelats en roll](media/storage-auth-aad-rbac-portal/container-scoped-role.png)

Du kan följa liknande steg för att tilldela en roll som är begränsad till lagrings kontot, resurs gruppen eller prenumerationen.

### <a name="assign-the-reader-role-for-portal-access"></a>Tilldela rollen läsare för åtkomst till portalen

När du tilldelar en inbyggd eller anpassad roll för Azure Storage till ett säkerhets objekt beviljar du behörigheter till säkerhets objekt för att utföra åtgärder på data i ditt lagrings konto. De inbyggda rollerna för **data läsare** ger Läs behörighet för data i en behållare eller kö, medan de inbyggda rollerna för **data deltagare** ger Läs-, skriv-och borttagnings behörighet till en behållare eller kö. Behörigheter tilldelas till den angivna resursen.  
Om du till exempel tilldelar rollen **Storage BLOB data Contributor** till användare Maria på nivån för en behållare med namnet **Sample-container**, beviljas Marys Läs-, skriv-och borttagnings åtkomst till alla blobar i behållaren.

Men om Mary vill visa en BLOB i Azure Portal, ger inte rollen **Storage BLOB data Contributor** av sig tillräckliga behörigheter för att navigera via portalen till blobben för att kunna visa den. Ytterligare Azure AD-behörigheter krävs för att navigera via portalen och Visa de andra resurser som är synliga där.

Om användarna behöver kunna komma åt blobbar i Azure Portal, tilldelar du dem en ytterligare RBAC-roll, rollen [läsare](../../role-based-access-control/built-in-roles.md#reader) , till dessa användare på lagrings kontots nivå eller över. Rollen **läsare** är en Azure Resource Manager roll som tillåter användare att Visa lagrings konto resurser, men inte ändra dem. Den ger inte Läs behörighet till data i Azure Storage, men endast för att hantera resurser för hantering.

Följ dessa steg om du vill tilldela rollen **läsare** så att en användare kan komma åt blobbar från Azure Portal. I det här exemplet är tilldelningen begränsad till lagrings kontot:

1. I [Azure Portal](https://portal.azure.com)navigerar du till ditt lagrings konto.
1. Välj **åtkomst kontroll (IAM)** om du vill visa inställningarna för åtkomst kontroll för lagrings kontot. Välj fliken **roll tilldelningar** om du vill se en lista över roll tilldelningar.
1. I fönstret **Lägg till roll tilldelning** väljer du rollen **läsare** . 
1. I fältet **tilldela åtkomst till väljer du** **Azure AD-användare, grupp eller tjänstens huvud namn**.
1. Sök efter den säkerhets princip som du vill tilldela rollen.
1. Spara roll tilldelningen.

Du behöver bara tilldela rollen **läsare** för användare som behöver åtkomst till blobbar eller köer med hjälp av Azure Portal.

> [!IMPORTANT]
> För hands versionen av Storage Explorer i Azure Portal stöder inte användning av Azure AD-autentiseringsuppgifter för att visa och ändra BLOB-eller Queue data. Storage Explorer i Azure Portal använder alltid konto nycklar för att komma åt data. Om du vill använda Storage Explorer i Azure Portal måste du ha tilldelats en roll som innehåller **Microsoft. Storage/storageAccounts/listnycklar/Action**.

## <a name="next-steps"></a>Nästa steg

- Mer information om RBAC-roller för lagrings resurser finns i [autentisera åtkomst till Azure-blobbar och köer med hjälp av Azure Active Directory](storage-auth-aad.md). 
- Mer information om RBAC finns i [Vad är rollbaserad åtkomst kontroll (RBAC)?](../../role-based-access-control/overview.md).
- Information om hur du tilldelar och hanterar RBAC-roll tilldelningar med Azure PowerShell, Azure CLI eller REST API finns i följande artiklar:
    - [Hantera rollbaserad åtkomst kontroll (RBAC) med Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)
    - [Hantera rollbaserad åtkomst kontroll (RBAC) med Azure CLI](../../role-based-access-control/role-assignments-cli.md)
    - [Hantera rollbaserad åtkomst kontroll (RBAC) med REST API](../../role-based-access-control/role-assignments-rest.md)
- Information om hur du auktoriserar åtkomst till behållare och köer från dina lagrings program finns i [använda Azure AD med Azure Storage-program](storage-auth-aad-app.md).
