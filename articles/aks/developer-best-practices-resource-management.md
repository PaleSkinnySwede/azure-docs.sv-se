---
title: Metod tips för resurs hantering
titleSuffix: Azure Kubernetes Service
description: Lär dig metod tips för programutvecklare för resurs hantering i Azure Kubernetes service (AKS)
services: container-service
author: zr-msft
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zarhoads
ms.openlocfilehash: 0052657c947f8a9ff9c9d6aef86ff16d9a22adae
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80803491"
---
# <a name="best-practices-for-application-developers-to-manage-resources-in-azure-kubernetes-service-aks"></a>Metod tips för programutvecklare för att hantera resurser i Azure Kubernetes service (AKS)

När du utvecklar och kör program i Azure Kubernetes service (AKS) finns det några viktiga områden att tänka på. Hur du hanterar dina program distributioner kan påverka slutanvändarens upplevelse av tjänster som du tillhandahåller. För att hjälpa dig att lyckas bör du tänka på några metod tips som du kan följa när du utvecklar och kör program i AKS.

I den här artikeln fokuserar vi på hur du kör ditt kluster och dina arbets belastningar från ett program utvecklings perspektiv. Information om rekommenderade metoder finns i [kluster operatörer metod tips för isolering och resurs hantering i Azure Kubernetes service (AKS)][operator-best-practices-isolation]. I den här artikeln lär du dig:

> [!div class="checklist"]
> * Vad är resurs begär Anden och begränsningar för Pod
> * Sätt att utveckla och distribuera program med dev Spaces och Visual Studio Code
> * Använda `kube-advisor` verktyget för att söka efter problem med distributioner

## <a name="define-pod-resource-requests-and-limits"></a>Definiera resurs begär Anden och begränsningar för Pod

**Vägledning för bästa praxis** – ange Pod-begäranden och begränsningar för alla poddar i dina yaml-manifest. Om AKS-klustret använder *resurs kvoter*kan distributionen avvisas om du inte definierar dessa värden.

Ett primärt sätt att hantera beräknings resurserna i ett AKS-kluster är att använda Pod-begäranden och-gränser. Dessa förfrågningar och gränser gör att Kubernetes Scheduler vet vilka beräknings resurser som en POD ska tilldelas.

* **Pod för processor/minne** definierar en mängd processor och minne som Pod behöver regelbundet.
    * När Kubernetes Scheduler försöker placera en POD på en nod, används Pod-begäranden för att avgöra vilken nod som har tillräckligt med resurser tillgängliga för schemaläggning.
    * Om du inte anger en POD-begäran används den definierade gränsen som standard.
    * Det är mycket viktigt att övervaka programmets prestanda för att justera dessa förfrågningar. Om det inte finns tillräckligt med begär Anden kan programmet få försämrade prestanda på grund av att en nod har schemalagts över. Om begär Anden är överskattat kan ditt program ha fått bättre svårigheter att komma åt schemat.
* **Pod CPU/minnes gränser** är den maximala mängd processor och minne som en POD kan använda. Dessa gränser hjälper dig att definiera vilka poddar som ska stoppas i händelse av instabilitet på noden på grund av otillräckliga resurser. Utan rätt gränser kommer poddar att avlivas tills resurs trycket lyfts upp.
    * Pod-gränser hjälper dig att definiera när en pod har förlorat kontrollen av resursförbrukning. När en gräns överskrids prioriteras Pod för avlivning för att upprätthålla nodens hälsa och minimera påverkan på poddar delning av noden.
    * Om du inte anger en POD begränsas den till det högsta tillgängliga värdet på en specifik nod.
    * Ange inte en POD-gräns som är högre än vad dina noder har stöd för. Varje AKS-nod reserverar en angiven mängd processor och minne för kärn komponenterna i Kubernetes. Ditt program kan försöka förbruka för många resurser på noden för att andra poddar ska kunna köras.
    * Återigen är det viktigt att övervaka programmets prestanda vid olika tidpunkter under dagen eller i veckan. Fastställ när den högsta efter frågan är och justera Pod-gränserna till de resurser som krävs för att uppfylla programmets Max krav.

I dina Pod-specifikationer är det **bästa praxis och mycket viktigt** att definiera dessa förfrågningar och gränser baserat på ovanstående information. Om du inte tar med dessa värden kan Kubernetes Scheduler inte ta hänsyn till de resurser som dina program behöver för att få hjälp med att schemalägga beslut.

Om Scheduler placerar en POD på en nod med otillräckliga resurser kommer program prestanda försämras. Det rekommenderas starkt för kluster administratörer att ställa in *resurs kvoter* i ett namn område som kräver att du anger resurs begär Anden och begränsningar. Mer information finns i [resurs kvoter på AKS-kluster][resource-quotas].

När du definierar en processor förfrågan eller en gräns mäts värdet i CPU-enheter. 
* *1,0* CPU motsvarar en underliggande virtuell CPU-kärna på noden. 
* Samma mått används för GPU: er.
* Du kan definiera bråktal som mäts i millicores. Till exempel är *100 miljoner* *0,1* av en underliggande vCPU-kärna.

I följande grundläggande exempel för en enda NGINX-Pod begär Pod *100 miljoner* CPU-tid och *128Mi* av minne. Resurs gränserna för pod är inställda på *250m* CPU-och *256Mi* -minne:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

Mer information om resurs mått och tilldelningar finns i [hantera beräknings resurser för behållare][k8s-resource-limits].

## <a name="develop-and-debug-applications-against-an-aks-cluster"></a>Utveckla och felsöka program mot ett AKS-kluster

**Vägledning för bästa praxis** – utvecklings team bör distribuera och felsöka mot ett AKS-kluster med hjälp av dev Spaces. Den här utvecklings modellen ser till att rollbaserade åtkomst kontroller, nätverk eller lagrings behov implementeras innan appen distribueras till produktion.

Med Azure dev Spaces kan du utveckla, felsöka och testa program direkt mot ett AKS-kluster. Utvecklare i en grupp arbetar tillsammans för att bygga och testa under hela programmets livs cykel. Du kan fortsätta att använda befintliga verktyg som Visual Studio eller Visual Studio Code. Ett tillägg installeras för dev Spaces som ger ett alternativ för att köra och felsöka programmet i ett AKS-kluster:

![Felsöka program i ett AKS-kluster med dev Spaces](media/developer-best-practices-resource-management/dev-spaces-debug.png)

Den här integrerade utvecklings-och test processen med dev-utrymmen minskar behovet av lokala test miljöer, till exempel [minikube][minikube]. I stället kan du utveckla och testa mot ett AKS-kluster. Klustret kan skyddas och isoleras enligt föregående avsnitt om användningen av namn områden för att logiskt isolera ett kluster. När dina appar är klara att distribueras till produktion kan du på ett säkert sätt distribuera allt eftersom din utveckling genomfördes mot ett verkligt AKS-kluster.

Azure dev Spaces är avsett att användas med program som körs på Linux-poddar och noder.

## <a name="use-the-visual-studio-code-extension-for-kubernetes"></a>Använd Visual Studio Code-tillägget för Kubernetes

**Vägledning för bästa praxis** – installera och Använd vs Code-tillägget för Kubernetes när du skriver yaml-manifest. Du kan också använda tillägget för integrerad distributions lösning, vilket kan hjälpa program ägare som sällan interagerar med AKS-klustret.

[Visual Studio Code-tillägget för Kubernetes][vscode-kubernetes] hjälper dig att utveckla och distribuera program till AKS. Tillägget innehåller IntelliSense för Kubernetes-resurser och Helm-diagram och mallar. Du kan också bläddra i, distribuera och redigera Kubernetes-resurser från VS Code. Tillägget innehåller också en IntelliSense-kontroll för resurs begär Anden eller gränser som anges i pod-specifikationerna:

![VS Code-tillägg för Kubernetes varning om saknade minnes gränser](media/developer-best-practices-resource-management/vs-code-kubernetes-extension.png)

## <a name="regularly-check-for-application-issues-with-kube-advisor"></a>Sök regelbundet efter program problem med Kube-Advisor

**Vägledning för bästa praxis** – kör regelbundet den senaste versionen `kube-advisor` av verktyget för öppen källkod för att identifiera problem i klustret. Om du använder resurs kvoter i ett befintligt AKS-kluster, `kube-advisor` kör först för att hitta poddar som inte har några resurs begär Anden och gränser definierade.

Verktyget [Kube-Advisor][kube-advisor] är ett associerat AKS-projekt med öppen källkod som söker igenom ett Kubernetes-kluster och rapporterar om problem som hittas. En bra kontroll är att identifiera poddar som inte har resurs begär Anden och begränsningar på plats.

Kube-verktyget kan rapportera om resurs begär Anden och gränser som saknas i PodSpecs för Windows-program och Linux-program, men Kube-Advisor-verktyget måste vara schemalagt för en Linux-pod. Du kan schemalägga en POD så att den körs på en adresspool med ett särskilt operativ system med hjälp av en [Node-selektor][k8s-node-selector] i pod-konfigurationen.

I ett AKS-kluster som är värd för många utvecklings team och program, kan det vara svårt att spåra poddar utan dessa resurs begär Anden och begränsningar. Vi rekommenderar att du regelbundet kör `kube-advisor` dina AKS-kluster.

## <a name="next-steps"></a>Nästa steg

Den här tips artikeln fokuserar på hur du kör ditt kluster och dina arbets belastningar från ett kluster operatörs perspektiv. Information om rekommenderade metoder finns i [kluster operatörer metod tips för isolering och resurs hantering i Azure Kubernetes service (AKS)][operator-best-practices-isolation].

Information om hur du implementerar några av dessa metod tips finns i följande artiklar:

* [Utveckla med Dev Spaces][dev-spaces]
* [Sök efter problem med Kube-Advisor][aks-kubeadvisor]

<!-- EXTERNAL LINKS -->
[k8s-resource-limits]: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
[vscode-kubernetes]: https://github.com/Azure/vscode-kubernetes-tools
[kube-advisor]: https://github.com/Azure/kube-advisor
[minikube]: https://kubernetes.io/docs/setup/minikube/

<!-- INTERNAL LINKS -->
[aks-kubeadvisor]: kube-advisor-tool.md
[dev-spaces]: ../dev-spaces/get-started-netcore.md
[operator-best-practices-isolation]: operator-best-practices-cluster-isolation.md
[resource-quotas]: operator-best-practices-scheduler.md#enforce-resource-quotas
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
