---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 02/10/2020
ms.author: erhopf
ms.openlocfilehash: e0bc7eeff276d44f3cd23e30eb54c9bd1ca94efd
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80659165"
---
1. Starta Eclipse.

1. I fältet **Arbetsyta** i Eclipse-startfönstret anger du namnet på en ny katalog för arbetsytan. Välj sedan **Starta**.

   ![Skärmbild av Eclipse-startfönstret](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-01-create-new-eclipse-workspace.png)

1. Efter en liten stund visas huvudfönstret i Eclipse IDE. Stäng **välkomst** skärmen om en sådan finns.

1. Skapa ett nytt projekt från meny raden för Sol förmörkelse genom att välja **Arkiv** > **nytt** > **projekt**.

1. Dialogrutan **Nytt projekt** visas. Välj **Java-projekt** och välj **Nästa**.

   ![Skärmbild av dialogrutan Nytt projekt, med Java-projekt markerat](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-02-select-wizard.png)

1. Guiden **nytt Java-projekt** startar. I fältet **Projektnamn** anger du **quickstart** (snabbstart) och väljer **JavaSE 1.8** som körningsmiljö. Välj **Slutför**.

   ![Skärmbild av guiden Nytt Java-projekt](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-03-create-java-project.png)

1. Om fönstret **Open Associated Perspective?** (Vill du öppna associerat perspektiv?) visas väljer du **Open Perspective** (Öppna perspektiv).

1. I **Paketutforskaren** högerklickar du på projektet **quickstart** (snabbstart). Välj **Konfigurera** > **konvertera till Maven-projekt** från snabb menyn.

   ![Skärmbild av Paketutforskaren](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-04-convert-to-maven-project.png)

1. Fönstret **Create new POM** (Skapa ny POM) visas. I fältet **grupp-ID** anger du *com. Microsoft. cognitiveservices. Speech. Samples*och i fältet **artefakt-ID** anger du *snabb start*. Välj sedan **Slutför**.

   ![Skärmbild av fönstret Create new POM (Skapa ny POM)](../articles/cognitive-services/Speech-Service/media/sdk/qs-java-jre-05-configure-maven-pom.png)

1. Öppna filen *pom.xml* och redigera den.

   * I slutet av filen, före sluttaggen `</project>`, skapar du ett `repositories`-element med en referens till Maven-databasen för Speech SDK, som du ser här:

     [!code-xml[POM Repositories](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-microphone/pom.xml#repositories)]

   * Lägg också till `dependencies` ett-element med tal SDK-versionen 1.11.0 som ett beroende:

     [!code-xml[POM Dependencies](~/samples-cognitive-services-speech-sdk/quickstart/java/jre/from-microphone/pom.xml#dependencies)]

   * Spara ändringarna.
