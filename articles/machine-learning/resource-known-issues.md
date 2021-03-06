---
title: Kända problem & fel sökning
titleSuffix: Azure Machine Learning
description: Hämta en lista över kända problem, lösningar och fel sökning för Azure Machine Learning.
services: machine-learning
author: j-martens
ms.author: jmartens
ms.reviewer: mldocs
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 03/31/2020
ms.openlocfilehash: 93015da810f163a48529704e69e1747ac1aec401
ms.sourcegitcommit: b396c674aa8f66597fa2dd6d6ed200dd7f409915
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2020
ms.locfileid: "82889399"
---
# <a name="known-issues-and-troubleshooting-azure-machine-learning"></a>Kända problem och fel söknings Azure Machine Learning

Den här artikeln hjälper dig att hitta och korrigera fel eller fel som kan uppstå när du använder Azure Machine Learning.

## <a name="diagnostic-logs"></a>Diagnostikloggar

Ibland kan det vara bra om du kan ange diagnostikinformation när du ber om hjälp. Så här visar du några loggar: 
1. Besök [Azure Machine Learning Studio](https://ml.azure.com). 
1. Välj **experiment** på vänster sida 
1. Välj ett experiment.
1. Välj en körning.
1. Välj **utdata + loggar**överst.

> [!NOTE]
> Azure Machine Learning loggar information från en rad olika källor under utbildningen, till exempel AutoML eller Docker-behållaren som kör övnings jobbet. Många av dessa loggar dokumenteras inte. Om du stöter på problem och kontaktar Microsoft-supporten kan det hända att de kan använda dessa loggar under fel sökning.


## <a name="resource-quotas"></a>Resurskvoter

Lär dig mer om [resurs kvoter](how-to-manage-quotas.md) som du kan stöta på när du arbetar med Azure Machine Learning.

## <a name="installation-and-import"></a>Installation och import
                           
* **Pip-installation: beroenden är inte garanterat konsekventa med en enskild rad installation**: 

   Detta är en känd begränsning i PIP, eftersom den inte har en fungerande beroende lösare när du installerar som en enskild rad. Det första unika beroendet är det enda som det ser ut. 

   I följande kod `azure-ml-datadrift` och `azureml-train-automl` båda installeras med en enda rad pip-installation. 
     ```
       pip install azure-ml-datadrift, azureml-train-automl
     ```
   I det här exemplet måste vi säga `azure-ml-datadrift` att version > 1,0 och `azureml-train-automl` kräver version < 1,2. Om den senaste versionen av `azure-ml-datadrift` är 1,3 uppgraderas båda paketen till 1,3, oavsett `azureml-train-automl` paket kravet för en äldre version. 

   För att se till att rätt versioner är installerade för dina paket kan du installera med flera rader som i följande kod. Ordningen är inte ett problem här eftersom pip degraderas explicit som en del av nästa rad anrop. Och därmed tillämpas rätt versions beroenden.
    
     ```
        pip install azure-ml-datadrift
        pip install azureml-train-automl 
     ```
     
* **Förklarings paketet är inte guarateed som ska installeras när du installerar azureml-träna-automl-client:** 
   
   När du kör en fjärran sluten automl med modell förklaring aktive rad visas ett fel meddelande om att "" installera azureml-deklarning-Model-paketet för modell förklaringar ". Detta är ett känt problem och som en lösning följer du ett av stegen nedan:
  
  1. Installera azureml – förklara-Model lokalt.
   ```
      pip install azureml-explain-model
   ```
  2. Inaktivera funktionen för välklare ring helt genom att skicka model_explainability = falskt i automl-konfigurationen.
   ```
      automl_config = AutoMLConfig(task = 'classification',
                             path = '.',
                             debug_log = 'automated_ml_errors.log',
                             compute_target = compute_target,
                             run_configuration = aml_run_config,
                             featurization = 'auto',
                             model_explainability=False,
                             training_data = prepped_data,
                             label_column_name = 'Survived',
                             **automl_settings)
    ``` 
    
* **Panda-fel: visas vanligt vis under AutoML experiment:**
   
   När du manuellt konfigurerar din Environmnet med hjälp av pip kommer du att märka felkoder (särskilt från Pandas) på grund av att paket versioner som inte stöds installeras. För att förhindra sådana fel måste du [Installera AUTOML SDK med hjälp av automl_setup. cmd](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/README.md):
   
    1. Öppna en Anaconda-prompt och klona GitHub-lagringsplatsen för en uppsättning exempel antecknings böcker.

    ```bash
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```
    
    2. CD till mappen "How-to-use-azureml/automatisk-Machine-Learning där exempel antecknings böckerna extraherades och kör sedan:
    
    ```bash
    automl_setup
    ```
  
* **Fel meddelande: det går inte att avinstallera ' PyYAML '**

    Azure Machine Learning SDK för python: PyYAML är ett `distutils` installerat projekt. Därför kan vi inte korrekt avgöra vilka filer som tillhör den om det finns en delvis avinstallation. Om du vill fortsätta att installera SDK och ignorera det här felet använder du:
    
    ```Python
    pip install --upgrade azureml-sdk[notebooks,automl] --ignore-installed PyYAML
    ```

* **Databricks-problem vid installation av paket**

    Azure Machine Learning SDK-installationen Miss lyckas på Azure Databricks när fler paket är installerade. Vissa paket, till exempel `psutil`, kan orsaka konflikter. Undvik installations fel genom att installera paket genom att frysa biblioteks versionen. Det här problemet är relaterat till Databricks och inte till Azure Machine Learning SDK. Du kan också uppleva det här problemet med andra bibliotek. Exempel:
    
    ```python
    psutil cryptography==1.5 pyopenssl==16.0.0 ipython==2.2.0
    ```

    Du kan också använda init-skript om du behåller problem med att installera mot python-bibliotek. Den här metoden stöds inte officiellt. Mer information finns i [kluster omfång init-skript](https://docs.azuredatabricks.net/user-guide/clusters/init-scripts.html#cluster-scoped-init-scripts).

* **Import fel i Databricks: det går inte att importera namnet timedelta från Pandas. _libs. tslibs**: om du ser det här felet när du använder automatisk maskin inlärning, kör du två följande rader i din bärbara dator:
    ```
    %sh rm -rf /databricks/python/lib/python3.7/site-packages/pandas-0.23.4.dist-info /databricks/python/lib/python3.7/site-packages/pandas
    %sh /databricks/python/bin/pip install pandas==0.23.4
    ```

* **Import fel i Databricks: det finns ingen modul med namnet ' Pandas. Core. index '**: om du ser det här felet när du använder automatisk maskin inlärning:

    1. Kör det här kommandot för att installera två paket i Azure Databricks klustret:
    
       ```bash
       scikit-learn==0.19.1
       pandas==0.22.0
       ```
    
    1. Koppla från och återanslut sedan klustret till din bärbara dator.
    
    Om de här stegen inte löser problemet kan du försöka med att starta om klustret.

* **Databricks FailToSendFeather**: om du ser ett `FailToSendFeather` fel när du läser data på Azure Databricks kluster kan du läsa följande lösningar:
    
    * Uppgradera `azureml-sdk[automl]` paketet till den senaste versionen.
    * Lägg `azureml-dataprep` till version 1.1.8 eller senare.
    * Lägg `pyarrow` till version 0,11 eller senare.
    
## <a name="create-and-manage-workspaces"></a>Skapa och hantera arbets ytor

> [!WARNING]
> Det finns inte stöd för att flytta Azure Machine Learning arbets ytan till en annan prenumeration eller flytta den ägande prenumerationen till en ny klient. Detta kan orsaka fel.

* **Azure Portal**: om du går direkt för att visa din arbets yta från en resurs länk från SDK eller portalen kan du inte visa **översikts** sidan för normal med prenumerations information i tillägget. Du kommer inte heller att kunna byta till en annan arbets yta. Om du behöver visa en annan arbets yta går du direkt till [Azure Machine Learning Studio](https://ml.azure.com) och söker efter arbets ytans namn.

## <a name="set-up-your-environment"></a>Konfigurera din miljö

* **Problem med att skapa AmlCompute**: det är en sällsynt risk att vissa användare som har skapat sin Azure Machine Learning-arbetsyta från Azure Portal innan ga-versionen kanske inte kan skapa AmlCompute på arbets ytan. Du kan antingen utlösa en supportbegäran mot tjänsten eller skapa en ny arbets yta via portalen eller SDK för att häva blockeringen direkt.

## <a name="work-with-data"></a>Arbeta med data

### <a name="overloaded-azurefile-storage"></a>Överlagrad AzureFile-lagring

Använd följande lösningar om du `Unable to upload project files to working directory in AzureFile because the storage is overloaded`får ett fel meddelande.

Om du använder fil resurs för andra arbets belastningar, till exempel data överföring, är rekommendationen att använda blobbar så att fil resursen är kostnads fri att användas för att skicka körningar. Du kan också dela upp arbets belastningen mellan två olika arbets ytor.

### <a name="passing-data-as-input"></a>Skicka data som indata

*  **TypeError: FileNotFound: det finns ingen sådan fil eller katalog**: det här felet uppstår om fil Sök vägen som du anger inte är den plats där filen finns. Du måste kontrol lera att det sätt som du refererar till filen är konsekvent med var du monterade data uppsättningen på beräknings målet. För att säkerställa ett deterministiskt tillstånd rekommenderar vi att du använder den abstrakta sökvägen när du monterar en data uppsättning till ett beräknings mål. I följande kod monterar du till exempel data uppsättningen under roten i fil systemet för beräknings målet `/tmp`. 
    
    ```python
    # Note the leading / in '/tmp/dataset'
    script_params = {
        '--data-folder': dset.as_named_input('dogscats_train').as_mount('/tmp/dataset'),
    } 
    ```

    Om du inte tar med det inledande snedstrecket "/" måste du ange prefix för arbets katalogen t. ex. `/mnt/batch/.../tmp/dataset` på beräknings målet för att ange var du vill att data uppsättningen ska monteras.

### <a name="data-labeling-projects"></a>Data märknings projekt

|Problem  |Lösning  |
|---------|---------|
|Det går bara att använda data uppsättningar som skapats på BLOB-datalager     |  Detta är en känd begränsning i den aktuella versionen.       |
|När projektet har skapats visar projektet "initiera" under en längre tid     | Uppdatera sidan manuellt. Initieringen bör fortsätta ungefär 20 Datapoints per sekund. Bristen på Autouppdatering är ett känt problem.         |
|När du visar bilder visas nyligen märkta bilder inte     |   Om du vill läsa in alla märkta bilder väljer du den **första** knappen. Den **första** knappen tar dig tillbaka till början av listan, men läser in alla etiketterade data.      |
|Tryck på ESC-tangenten när etiketter för objekt identifiering skapar en etikett för noll storlek i det övre vänstra hörnet. Det går inte att skicka etiketter i det här läget.     |   Ta bort etiketten genom att klicka på kryss rutan bredvid det.  |

## <a name="azure-machine-learning-designer"></a>Azure Machine Learning designer

Kända problem:

* **Förberedelse tid för lång beräkning**: det kan vara några minuter eller ännu längre första gången du ansluter till eller skapar ett beräknings mål. 

## <a name="train-models"></a>Inlärningsmodeller

* **ModuleErrors (ingen modul med namnet)**: om du kör i ModuleErrors när du skickar experiment i Azure ml, innebär det att utbildnings skriptet förväntar sig att ett paket ska installeras men inte läggs till. När du har angett paket namnet kommer Azure ML att installera paketet i den miljö som används för att köra din utbildning. 

    Om du använder [uppskattningar](concept-azure-machine-learning-architecture.md#estimators) för att skicka experiment kan du ange ett paket namn via `pip_packages` eller `conda_packages` parameter i uppskattningen baserat på från vilken källa du vill installera paketet. Du kan också ange en YML-fil med alla dina beroenden med `conda_dependencies_file`eller lista alla dina pip-krav i en `pip_requirements_file` txt-fil med hjälp av parametern. Om du har ett eget Azure ML-miljö objekt som du vill åsidosätta standard avbildningen som används av uppskattaren kan du ange den miljön via `environment` -parametern för den uppskattade konstruktorn.

    Azure ML tillhandahåller också branschspecifika uppskattningar för Tensorflow, PyTorch, Kedjorer och SKLearn. Genom att använda dessa uppskattningar ser du till att de viktigaste Ramverks beroendena är installerade för din räkning i miljön som används för utbildning. Du kan välja att ange extra beroenden enligt beskrivningen ovan. 
 
    Azure ML-underhållna Docker-avbildningar och deras innehåll kan visas i [azureml-behållare](https://github.com/Azure/AzureML-Containers).
    De Ramverks-/regionsspecifika beroendena visas i respektive Framework-dokumentation – [kedjar](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py#remarks), [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py#remarks), [TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py#remarks), [SKLearn](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.sklearn.sklearn?view=azure-ml-py#remarks).

    > [!Note]
    > Om du tror att ett visst paket är tillräckligt vanligt för att läggas till i Azure ML-underhållna bilder och miljöer kan du generera ett GitHub-problem i [azureml-behållare](https://github.com/Azure/AzureML-Containers). 
 
* **NameError (namn har inte definierats), AttributeError (objektet har inget attribut)**: det här undantaget bör komma från dina utbildnings skript. Du kan titta på loggfilerna från Azure Portal för att få mer information om det angivna namnet inte är definierat eller ett attributvärde. Från SDK kan du använda `run.get_details()` för att titta på fel meddelandet. Detta visar även alla loggfiler som genererats för din körning. Se till att ta en titt på ditt utbildnings skript och åtgärda felet innan du skickar om körningen. 

* **Horovod har stängts av**: i de flesta fall, om du stöter på "AbortedError: Horovod har stängts av" det här undantaget innebär det ett underliggande undantag i en av de processer som orsakade att Horovod stängdes. Varje rang i MPI-jobbet hämtar den egna dedikerade logg filen i Azure ML. De här loggarna `70_driver_logs`kallas. I händelse av distribuerad utbildning suffixs logg namnen med `_rank` för att göra det enklare att skilja loggarna åt. Om du vill hitta det exakta fel som orsakade Horovod stänger du igenom alla loggfiler och letar efter `Traceback` i slutet av driver_log-filerna. Med en av de här filerna får du det faktiska underliggande undantaget. 

* **Körning eller experimentering**: experiment kan arkiveras med hjälp av metoden [experiment. Archive](https://docs.microsoft.com/python/api/azureml-core/azureml.core.experiment(class)?view=azure-ml-py#archive--) eller från fliken experiment i Azure Machine Learning Studio-klienten via "arkivera experiment"-knappen. Den här åtgärden döljer experimentet från List frågor och vyer, men tar inte bort den.

    Permanent borttagning av enskilda experiment eller körningar stöds inte för närvarande. Mer information om hur du tar bort arbets ytans till gångar finns i [Exportera eller ta bort data för Machine Learning service-arbetsytan](how-to-export-delete-data.md).

* **Mått dokumentet är för stort**: Azure Machine Learning har interna gränser för mått objekts storlek som kan loggas samtidigt från en utbildnings körning. Om ett "Metric-dokument är för stort"-fel vid loggning av ett List värdes mått kan du försöka dela listan i mindre segment, till exempel:

    ```python
    run.log_list("my metric name", my_metric[:N])
    run.log_list("my metric name", my_metric[N:])
    ```

    Internt sammanfogar Azure ML blocken med samma mått namn i en sammanhängande lista.

## <a name="automated-machine-learning"></a>Automatiserad maskininlärning

* **Styrkorts flöde**: Automatisk maskin inlärning har för närvarande inte stöd för styrkorts flödes version 1,13. Om den här versionen installeras kommer paket beroenden att sluta fungera. Vi arbetar för att åtgärda det här problemet i en framtida version.

* **Experiment diagram**: binära klassificerings diagram (precision-återkallande, Roc, kurva osv.) som visas i automatiserade ml experiment-iterationer återges inte korrekt i användar gränssnittet sedan 4/12. Diagram observationer visar för närvarande inversa resultat, där modeller med bättre prestanda visas med lägre resultat. En lösning är under undersökning.

* **Databricks avbryter en automatiserad maskin inlärnings körning**: när du använder automatiserade funktioner för maskin inlärning på Azure Databricks, för att avbryta en körning och starta en ny experiment körning, startar du om Azure Databricks klustret.

* **Databricks >10 iterationer för automatisk maskin inlärning**: i automatiserade inställningar för maskin inlärning, om du har fler än 10 iterationer `show_output` , `False` anger du när du skickar in körningen.

* **Databricks-widget för Azure Machine Learning SDK och automatisk maskin inlärning**: widgeten Azure Machine Learning SDK stöds inte i en Databricks Notebook eftersom antecknings böckerna inte kan parsa HTML-widgetar. Du kan visa widgeten i portalen genom att använda den här python-koden i din Azure Databricks Notebook-cell:

    ```
    displayHTML("<a href={} target='_blank'>Azure Portal: {}</a>".format(local_run.get_portal_url(), local_run.id))
    ```

## <a name="deploy--serve-models"></a>Distribuera och hantera modeller

Utför följande åtgärder för följande fel:

|Fel  | Lösning  |
|---------|---------|
|Bild skapande problem vid distribution av webb tjänst     |  Lägg till "pynacl = = 1.2.1" som ett pip-beroende till Conda-filen för avbildnings konfiguration       |
|`['DaskOnBatch:context_managers.DaskOnBatch', 'setup.py']' died with <Signals.SIGKILL: 9>`     |   Ändra SKU: n för virtuella datorer som används i distributionen till en som har mer minne. |
|FPGA-problem     |  Du kommer inte att kunna distribuera modeller på FPGAs förrän du har begärt och godkänts för FPGA-kvoten. Om du vill begära åtkomst fyller du i formuläret kvot förfrågan:https://aka.ms/aml-real-time-ai       |

### <a name="updating-azure-machine-learning-components-in-aks-cluster"></a>Uppdatera Azure Machine Learning-komponenter i AKS-kluster

Uppdateringar av Azure Machine Learning-komponenter som är installerade i ett Azure Kubernetes service-kluster måste appliceras manuellt. 

Du kan tillämpa dessa uppdateringar genom att koppla från klustret från arbets ytan Azure Machine Learning och sedan återansluta klustret till arbets ytan. Om TLS är aktiverat i klustret måste du ange TLS/SSL-certifikatet och den privata nyckeln när du återansluter klustret. 

```python
compute_target = ComputeTarget(workspace=ws, name=clusterWorkspaceName)
compute_target.detach()
compute_target.wait_for_completion(show_output=True)

attach_config = AksCompute.attach_configuration(resource_group=resourceGroup, cluster_name=kubernetesClusterName)

## If SSL is enabled.
attach_config.enable_ssl(
    ssl_cert_pem_file="cert.pem",
    ssl_key_pem_file="key.pem",
    ssl_cname=sslCname)

attach_config.validate_configuration()

compute_target = ComputeTarget.attach(workspace=ws, name=args.clusterWorkspaceName, attach_configuration=attach_config)
compute_target.wait_for_completion(show_output=True)
```

Om du inte längre har TLS/SSL-certifikatet och den privata nyckeln, eller om du använder ett certifikat som har genererats av Azure Machine Learning, kan du hämta filerna innan du kopplar från klustret genom att ansluta till `kubectl` klustret med hjälp av och `azuremlfessl`Hämta hemligheten.

```bash
kubectl get secret/azuremlfessl -o yaml
```

>[!Note]
>Kubernetes lagrar hemligheterna i Base-64-kodat format. Du behöver Base-64-avkoda- `cert.pem` och `key.pem` -komponenterna i hemligheterna innan du ger dem. `attach_config.enable_ssl` 

### <a name="webservices-in-azure-kubernetes-service-failures"></a>Webservice i Azure Kubernetes service-problem

Många WebService-fel i Azure Kubernetes-tjänsten kan felsökas genom att ansluta till klustret med `kubectl`hjälp av. Du kan hämta ett `kubeconfig.json` Azure Kubernetes service-kluster genom att köra

```azurecli-interactive
az aks get-credentials -g <rg> -n <aks cluster name>
```

## <a name="authentication-errors"></a>Autentiseringsfel

Om du utför en hanterings åtgärd på ett beräknings mål från ett Fjärrjobb får du ett av följande fel:

```json
{"code":"Unauthorized","statusCode":401,"message":"Unauthorized","details":[{"code":"InvalidOrExpiredToken","message":"The request token was either invalid or expired. Please try again with a valid token."}]}
```

```json
{"error":{"code":"AuthenticationFailed","message":"Authentication failed."}}
```

Till exempel visas ett fel meddelande om du försöker skapa eller ansluta ett beräknings mål från en ML-pipeline som skickas för fjärrkörning.
