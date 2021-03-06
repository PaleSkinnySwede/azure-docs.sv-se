---
title: Självstudie – Lägg till en exempel modell – Azure Analysis Services | Microsoft Docs
description: I den här självstudiekursen lär du dig hur du lägger till en exempelmodell i Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: tutorial
ms.date: 10/30/2019
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: a72236843c13e139f33a669b54f108e91679c8c6
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "74326584"
---
# <a name="tutorial-add-a-sample-model-from-the-portal"></a>Självstudiekurs: Lägga till en exempelmodell från portalen

I den här självstudiekursen får du lägga till en exempeldatabas för Adventure Works tabellmodell på servern. Exempelmodellen är en fullständig version av exempelmodellen Adventure Works Internet Sales (1200). En exempelmodell är praktisk för att testa modellhantering, anslutning med verktygs- och klientprogram, samt för att köra frågor till datamodellen. I självstudiekursen används [Azure Portal](https://portal.azure.com) och [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS) för att: 

> [!div class="checklist"]
> * Lägga till en färdig exempelmodell för tabelldata till en server 
> * Ansluta till modellen med SSMS

Om du inte har en Azure-prenumeration kan du [skapa ett kostnads fritt konto](https://azure.microsoft.com/free/) innan du börjar.

## <a name="before-you-begin"></a>Innan du börjar

För att slutföra den här kursen behöver du:

- En Analysis Services-server i Azure. Läs mer i [Skapa en server – portalen](analysis-services-create-server.md).
- Administratörsbehörighet för servern
- [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="sign-in-to-the-azure-portal"></a>Logga in på Azure Portal

Logga in på [portalen](https://portal.azure.com/).

## <a name="add-a-sample-model"></a>Lägg till en exempelmodell

1. I **översikten över servern** klickar du på **Ny modell**.

    ![Skapa en exempelmodell](./media/analysis-services-create-sample-model/aas-create-sample-new-model.png)

2. I **ny modell** > **väljer du en data källa**, verifierar **exempel data** är markerat och klickar sedan på **Lägg till**.

    ![Välj exempeldata](./media/analysis-services-create-sample-model/aas-create-sample-data.png)

3. Kontrollera i **översikten** att `adventureworks` exempelmodellen har lagts till.

    ![Välj exempeldata](./media/analysis-services-create-sample-model/aas-create-sample-verify.png)


## <a name="clean-up-resources"></a>Rensa resurser

Din exempelmodell använder resurser i cacheminnet. Om du inte använder exempelmodellen för testning bör du ta bort den från din server.

I de här anvisningarna beskrivs det hur du tar bort en modell från en server med hjälp av SSMS.

1. I SSMS > **Object Explorer** klickar du på **Anslut** > **Analysis Services**.

2. I **Anslut till Server** klistrar du in servernamnet. Sedan, i **Autentisering**, väljer du **Active Directory – Universal with MFA support** (Active Directory – Universell med stöd för MFA), anger ditt användarnamn och klickar på **Anslut**.

    ![Logga in](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-signin.png)

3. I **Object Explorer** högerklickar du på exempeldatabasen `adventureworks` och klickar på **Ta bort**.

    ![Ta bort en exempeldatabas](./media/analysis-services-create-sample-model/aas-create-sample-cleanup-delete.png)

## <a name="next-steps"></a>Nästa steg 

I den här självstudiekursen har du lärt dig hur du lägger till en grundläggande exempelmodell på din server. Nu när du har en modelldatabas kan du ansluta till den från SQL Server Management Studio och lägga till användarroller. Fortsätt med nästa självstudiekurs om du vill veta mer.

> [!div class="nextstepaction"]
> [Självstudiekurs: Konfigurera roller för serveradministratör och användare](tutorials/analysis-services-tutorial-roles.md)


