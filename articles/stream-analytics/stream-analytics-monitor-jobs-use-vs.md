---
title: Övervaka och hantera Azure Stream Analytics med Visual Studio
description: Den här artikeln beskriver hur du använder Visual Studio för att övervaka och hantera Azure Stream Analytics-jobb.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: e43a8d157baaf7e02ab7f8db4c777009d2a0abe9
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75431692"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-visual-studio"></a>Övervaka och hantera Stream Analytics-jobb med Visual Studio

Den här artikeln visar hur du övervakar Stream Analytics jobb i Visual Studio. Azure Stream Analytics Tools för Visual Studio ger en övervaknings miljö som liknar Azure Portal utan att du behöver lämna IDE-versionen. Du kan börja övervaka ett jobb så snart du **skickar till Azure** från **skriptet. asaql**, eller så kan du övervaka befintliga jobb oavsett hur de har skapats. 

## <a name="job-summary"></a>Jobb Sammanfattning

**Jobb sammanfattningen** och **jobb måtten** ger en snabb ögonblicks bild av ditt jobb. Du kan snabbt se ett jobbs status-och händelse information.]

<img src="./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-job-summary-metrics.png" alt="Stream Analytics job summary and job metrics" width="300px"/> 


## <a name="job-metrics"></a>Jobb mått

Du kan komprimera **jobb sammanfattningen** och klicka på fliken **jobb mått** om du vill visa en graf med viktiga mått. Kontrol lera och avmarkera mått typer för att lägga till och ta bort dem från grafen.

![Stream Analytics mått i Visual Studio](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-metrics.png)


## <a name="error-monitoring"></a>Fel övervakning

Du kan också övervaka fel genom att klicka på fliken **fel** .

![Stream Analytics fel i Visual Studio](./media/stream-analytics-monitor-jobs-use-vs/stream-analytics-vs-errors.png)


## <a name="get-support"></a>Få support
Om du behöver ytterligare hjälp kan du prova vårt [Azure Stream Analytics-forum](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics). 

## <a name="next-steps"></a>Nästa steg
* [Introduktion till Azure Stream Analytics](stream-analytics-introduction.md)
* [Skapa ett Azure Stream Analytics jobb med Visual Studio](stream-analytics-quick-create-vs.md)
* [Installera Azure Stream Analytics verktyg för Visual Studio](stream-analytics-tools-for-visual-studio-install.md) 


