---
title: Aktivera loggning av webb tjänst
titleSuffix: ML Studio (classic) - Azure
description: 'Lär dig hur du aktiverar loggning för Machine Learning Studio (klassiska) webb tjänster. Loggning ger ytterligare information som hjälper dig att felsöka API: erna.'
services: machine-learning
author: likebupt
ms.author: keli19
editor: cgronlun
ms.assetid: c54d41e1-0300-46ef-bbfc-d6f7dca85086
ms.service: machine-learning
ms.subservice: studio
ms.topic: conceptual
ms.date: 06/15/2017
ms.openlocfilehash: dc4a2f83f59ab94d1bff8a005b37124f2822c72b
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82209425"
---
# <a name="enable-logging-for-azure-machine-learning-studio-classic-web-services"></a>Aktivera loggning för webb tjänster för Azure Machine Learning Studio (klassisk)

[!INCLUDE [Notebook deprecation notice](../../../includes/aml-studio-notebook-notice.md)]

Det här dokumentet innehåller information om loggnings funktionerna i Machine Learning Studio (klassiska) webb tjänster. Loggning ger ytterligare information, utöver bara ett fel nummer och ett meddelande, som kan hjälpa dig att felsöka anrop till API: erna Machine Learning Studio (klassisk).  

## <a name="how-to-enable-logging-for-a-web-service"></a>Så här aktiverar du loggning för en webb tjänst

Du aktiverar loggning från den [Azure Machine Learning Studio (klassiska) webb tjänst](https://services.azureml.net) portalen. 

1. Logga in på den Azure Machine Learning Studio (klassiska) webb tjänst portalen på [https://services.azureml.net](https://services.azureml.net). För en klassisk webb tjänst kan du också gå till portalen genom att klicka på **ny webb tjänst upplevelse** på sidan Machine Learning Studio (klassisk) webb tjänster i Studio (klassisk).

   ![Länk till nya webb tjänst upplevelser](./media/web-services-logging/new-web-services-experience-link.png)

2. På den översta meny raden klickar du på **webb tjänster** för en ny webb tjänst eller på **klassiska webb tjänster** för en klassisk webb tjänst.

   ![Välj nya eller klassiska webb tjänster](./media/web-services-logging/select-web-service.png)

3. För en ny webb tjänst klickar du på namnet på webb tjänsten. För en klassisk webb tjänst klickar du på namnet på webb tjänsten och klickar sedan på lämplig slut punkt på nästa sida.

4. Klicka på **Konfigurera**på den översta meny raden.

5. Ange alternativet **Aktivera loggning** till *fel* (Logga endast fel) eller *alla* (för fullständig loggning).

   ![Välj loggnings nivå](./media/web-services-logging/enable-logging.png)

6. Klicka på **Spara**.

7. För klassiska webb tjänster skapar du behållaren för **ml-diagnostik** .

   Alla webb tjänst loggar sparas i en BLOB-behållare med namnet **ml-Diagnostics** i det lagrings konto som är associerat med webb tjänsten. För nya webb tjänster skapas den här behållaren första gången du ansluter till webb tjänsten. För klassiska webb tjänster måste du skapa behållaren om den inte redan finns. 

   1. I [Azure Portal](https://portal.azure.com)går du till det lagrings konto som är associerat med webb tjänsten.

   2. Under **Blob Service** klickar du på **Containers**.

   3. Om behållarens **ml-diagnostik** inte finns klickar du på **+ container**, ger behållaren namnet "ml-diagnostik" och väljer **åtkomst typen** som "BLOB". Klicka på **OK**.

      ![Skapa en ny behållare för att lagra dina diagnostikloggar](./media/web-services-logging/create-ml-diagnostics-container.png)

> [!TIP]
>
> För en klassisk webb tjänst har instrument panelen för webb tjänster i Machine Learning Studio (klassisk) också en växel för att aktivera loggning. Men eftersom loggning nu hanteras via webb tjänst portalen måste du aktivera loggning via portalen enligt beskrivningen i den här artikeln. Om du redan har aktiverat loggning i Studio (klassisk) inaktiverar du loggning på webb tjänst portalen och aktiverar det igen.


## <a name="the-effects-of-enabling-logging"></a>Effekterna av att aktivera loggning
När loggning är aktiverat loggas diagnostiken och felen från webb tjänst slut punkten i BLOB-behållaren för **ml-diagnostik** i det Azure Storage konto som är kopplat till användarens arbets yta. Den här behållaren innehåller all diagnostikinformation för alla webb tjänst slut punkter för alla arbets ytor som är associerade med det här lagrings kontot.

Loggarna kan visas med hjälp av flera av de verktyg som finns tillgängliga för att utforska ett Azure Storage-konto. Det enklaste sättet kan vara att navigera till lagrings kontot i Azure Portal, klicka på **behållare**och sedan klicka på container **ml-diagnostik**.  

## <a name="log-blob-detail-information"></a>Information om log BLOB
Varje BLOB i behållaren innehåller diagnostikinformation för exakt en av följande åtgärder:

* En körning av batch-körnings metoden  
* En körning av metoden för begäran/svar  
* Initiering av en behållare för begär ande svar

Namnet på varje BLOB har prefixet i följande format: 


`{Workspace Id}-{Web service Id}-{Endpoint Id}/{Log type}`


Var _logg typ_ är ett av följande värden:  

* batch  
* Poäng/förfrågningar  
* Poäng/init  

