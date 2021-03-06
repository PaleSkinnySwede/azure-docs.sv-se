---
title: Köra batchförutsägelser med Azure Machine Learning Designer (förhandsversion)
titleSuffix: Azure Machine Learning
description: Lär dig hur du tränar en modell och konfigurerar en pipeline för batch förutsägelse med hjälp av designern. Distribuera pipelinen som en parametriserad webb tjänst, som kan utlösas från alla HTTP-bibliotek.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: peterlu
author: peterclu
ms.date: 02/24/2020
ms.custom: Ignite2019
ms.openlocfilehash: 01d69bffcf2c17abceba8ba2e0893360bead8b12
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79477229"
---
# <a name="run-batch-predictions-using-azure-machine-learning-designer-preview"></a>Köra batchförutsägelser med Azure Machine Learning Designer (förhandsversion)
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

I den här artikeln får du lära dig hur du använder designern för att skapa en batch förutsägelse-pipeline. Med batch förutsägelse kan du kontinuerligt räkna ut stora data uppsättningar på begäran med hjälp av en webb tjänst som kan utlösas från alla HTTP-bibliotek.

I den här instruktionen får du lära dig att utföra följande uppgifter:

> [!div class="checklist"]
> * Skapa och publicera en pipeline för batch-härledning
> * Använda en pipeline-slutpunkt
> * Hantera slut punkts versioner

Information om hur du konfigurerar batch-Bedömningstjänster med hjälp av SDK finns i den medföljande [instruktionen](how-to-run-batch-predictions.md).

## <a name="prerequisites"></a>Krav

Den här instruktionen förutsätter att du redan har en utbildnings pipeline. För en guidad introduktion till designern, slutför du [en del av själv studie kursen för designern](tutorial-designer-automobile-price-train-score.md). 

## <a name="create-a-batch-inference-pipeline"></a>Skapa en pipeline för batch-härledning

Din utbildnings pipeline måste köras minst en gång för att kunna skapa en inferencing-pipeline.

1. Gå till **Designer** -fliken i din arbets yta.

1. Välj den utbildnings pipeline som tågen den modell du vill använda för att göra förutsägelse.

1. **Skicka** pipelinen.

    ![Skicka pipelinen](./media/how-to-run-batch-predictions-designer/run-training-pipeline.png)

Nu när utbildnings pipelinen har körts kan du skapa en pipeline för batch-härledning.

1. Bredvid **Skicka**väljer du den nya List rutan **skapa härlednings pipeline**.

1. Välj **pipeline för batch-härledning**.

    ![Skapa en pipeline för batch-härledning](./media/how-to-run-batch-predictions-designer/create-batch-inference.png)
    
Resultatet är en standard pipeline för batch-härledning. 

### <a name="add-a-pipeline-parameter"></a>Lägg till en pipeline-parameter

Om du vill skapa förutsägelser för nya data kan du antingen manuellt ansluta en annan data uppsättning i pipeline-utkastet eller skapa en parameter för din data uppsättning. Med parametrar kan du ändra beteendet för batch inferencing-processen vid körning.

I det här avsnittet skapar du en data mängds parameter för att ange en annan data uppsättning att göra förutsägelser på.

1. Välj data uppsättnings modulen.

1. Ett fönster visas till höger om arbets ytan. Längst ned i fönstret väljer du **Ange som pipeline-parameter**.
   
    Ange ett namn för parametern eller acceptera standardvärdet.

## <a name="publish-your-batch-inferencing-pipeline"></a>Publicera din batch inferencing-pipeline

Nu är du redo att distribuera inferencing-pipeline. Detta kommer att distribuera pipelinen och göra den tillgänglig för andra att använda.

1. Välj sedan knappen **Publicera**.

1. I dialog rutan som visas expanderar du List rutan för **PipelineEndpoint**och väljer **ny PipelineEndpoint**.

1. Ange ett slut punkts namn och en valfri beskrivning.

    Längst ned i dialog rutan kan du se den parameter som du konfigurerade med ett standardvärde för det data uppsättnings-ID som användes under träningen.

1. Välj **Publicera**.

![Publicera en pipeline](./media/how-to-run-batch-predictions-designer/publish-inference-pipeline.png)


## <a name="consume-an-endpoint"></a>Använda en slut punkt

Nu har du en publicerad pipeline med en data uppsättnings parameter. Pipelinen använder den tränade modell som skapats i utbildnings pipelinen för att ge den data uppsättning som du anger som parameter.

### <a name="submit-a-pipeline-run"></a>Skicka en pipeline-körning 

I det här avsnittet ska du konfigurera en manuell pipeline-körning och ändra pipeline-parametern för att visa nya data. 

1. När distributionen är klar går du till avsnittet **slut punkter** .

1. Välj **pipeline-slutpunkter**.

1. Välj namnet på den slut punkt som du har skapat.

![Slut punkts länk](./media/how-to-run-batch-predictions-designer/manage-endpoints.png)

1. Välj **publicerade pipeliner**.

    Den här skärmen visar alla publicerade pipelines som publicerats under den här slut punkten.

1. Välj den pipeline som du har publicerat.

    På sidan pipeline-information visas en detaljerad körnings historik och information om anslutnings strängen för din pipeline. 
    
1. Välj **Skicka** för att skapa en manuell körning av pipelinen.

    ![Pipeline-information](./media/how-to-run-batch-predictions-designer/submit-manual-run.png)
    
1. Ändra parametern till att använda en annan data uppsättning.
    
1. Välj **Skicka** för att köra pipelinen.

### <a name="use-the-rest-endpoint"></a>Använd REST-slutpunkten

Du hittar information om hur du använder pipelines slut punkter och publicerade pipelines i avsnittet **slut punkter** .

Du kan hitta REST-slutpunkten för en pipeline-slutpunkt på panelen kör översikt. Genom att anropa slut punkten förbrukar du den publicerade standard pipelinen.

Du kan också använda en publicerad pipeline på sidan **publicerade pipeliner** . Välj en publicerad pipeline och hitta resten av slut punkten för den. 

![Information om REST-slutpunkt](./media/how-to-run-batch-predictions-designer/rest-endpoint-details.png)

Om du vill göra ett REST-anrop behöver du ett OAuth 2,0-värde för Authentication-typ. I följande [själv studie kurs](tutorial-pipeline-batch-scoring-classification.md#publish-and-run-from-a-rest-endpoint) finns mer information om hur du konfigurerar autentisering till din arbets yta och gör ett parameter rest-anrop.

## <a name="versioning-endpoints"></a>Versions slut punkter

Designern tilldelar en version till varje efterföljande pipeline som du publicerar till en slut punkt. Du kan ange den pipeline-version som du vill köra som en parameter i REST-anropet. Om du inte anger ett versions nummer använder designern standard pipelinen.

När du publicerar en pipeline kan du välja att göra den till den nya standard pipelinen för den slut punkten.

![Ange standard pipeline](./media/how-to-run-batch-predictions-designer/set-default-pipeline.png)

Du kan också ange en ny standard-pipeline på fliken **publicerade pipelines** i slut punkten.

![Ange standard pipeline](./media/how-to-run-batch-predictions-designer/set-new-default-pipeline.png)

## <a name="next-steps"></a>Nästa steg

Följ [själv studie kursen](tutorial-designer-automobile-price-train-score.md) om du vill träna och distribuera en Regressions modell.
''