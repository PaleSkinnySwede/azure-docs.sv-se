---
title: Använda aktiviteter med flera instanser för att köra MPI-program
description: Lär dig hur du kör MPI-program (Message Passing Interface) med hjälp av uppgifts typen multiinstance i Azure Batch.
ms.topic: article
ms.date: 03/13/2019
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4502fc9632c2cb05d757459d07bcfe17ae96aea2
ms.sourcegitcommit: 4499035f03e7a8fb40f5cff616eb01753b986278
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/03/2020
ms.locfileid: "82735274"
---
# <a name="use-multi-instance-tasks-to-run-message-passing-interface-mpi-applications-in-batch"></a>Använda aktiviteter med flera instanser för att köra MPI-program (Message Passing Interface) i batch

Med aktiviteter med flera instanser kan du köra en Azure Batch aktivitet på flera Compute-noder samtidigt. De här uppgifterna möjliggör data bearbetnings scenarier med höga prestanda som MPI-program (Message Passing Interface) i batch. I den här artikeln får du lära dig hur du kör aktiviteter med flera instanser med [batch .net][api_net] -biblioteket.

> [!NOTE]
> Medan exemplen i den här artikeln fokuserar på batch .NET-, MS-MPI-och Windows Compute-noder, är de aktiviteter för flera instanser som beskrivs här tillämpliga på andra plattformar och tekniker (till exempel python och Intel MPI på Linux-noder).
>
>

## <a name="multi-instance-task-overview"></a>Översikt över aktiviteter med flera instanser
I batch körs varje aktivitet vanligt vis på en enskild Compute-nod – du skickar flera aktiviteter till ett jobb, och batch-tjänsten schemalägger varje aktivitet för körning på en nod. Genom att konfigurera en aktivitets inställningar för **flera instanser**kan du dock i stället skapa en primär aktivitet och flera under aktiviteter som sedan körs på flera noder.

![Översikt över aktiviteter med flera instanser][1]

När du skickar en aktivitet med inställningar för flera instanser till ett jobb utför batch flera steg som är unika för aktiviteter med flera instanser:

1. Batch-tjänsten skapar en **primär** och flera **under aktiviteter** baserat på inställningarna för flera instanser. Det totala antalet aktiviteter (primära plus alla under aktiviteter) matchar antalet **instanser** (Compute-noder) som du anger i inställningarna för flera instanser.
2. Batch anger en av datornoderna som **huvud server**och schemalägger den primära aktiviteten så att den körs på huvud servern. Den schemalägger de under aktiviteter som ska köras på resten av de Compute-noder som allokeras till aktiviteten för flera instanser, en under aktivitet per nod.
3. De primära och alla under aktiviteterna hämtar alla **delade resursfiler** som du anger i inställningarna för flera instanser.
4. När de delade resursfiler har hämtats, kör de primära och under aktiviteterna **koordinations kommandot** som du anger i inställningarna för flera instanser. Koordinations kommandot används vanligt vis för att förbereda noder för körning av uppgiften. Detta kan innefatta att starta bakgrunds tjänster (t. `smpd.exe`ex. Microsoft- [MPI][msmpi_msdn]) och kontrol lera att noderna är redo att bearbeta meddelanden mellan noder.
5. Den primära aktiviteten kör **program kommandot** på huvud-noden *när* koordinations kommandot har slutförts av den primära aktiviteten och alla under aktiviteter. Program kommandot är kommando raden för aktiviteten för flera instanser och körs bara av den primära aktiviteten. I en [MS-MPI][msmpi_msdn]-baserad lösning är det här du kör ditt MPI-aktiverade program med hjälp `mpiexec.exe`av.

> [!NOTE]
> Även om den är funktionellt åtskild, är "Multi-instance Task" inte en unik typ av aktivitet som [StartTask][net_starttask] eller [aktivitets typerna jobpreparationtask][net_jobprep]. Aktiviteten för flera instanser är helt enkelt en standard-batch-uppgift ([CloudTask][net_task] i batch .net) vars inställningar för flera instanser har kon figurer ATS. I den här artikeln hänvisar vi till detta som **aktiviteten för flera instanser**.
>
>

## <a name="requirements-for-multi-instance-tasks"></a>Krav för aktiviteter med flera instanser
Aktiviteter med flera instanser kräver en pool med **kommunikation mellan noder**och med **inaktive rad körning av aktivitet inaktive rad**. Om du vill inaktivera körning av samtidiga aktiviteter ställer du in egenskapen [CloudPool. MaxTasksPerComputeNode](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.cloudpool) på 1.

> [!NOTE]
> Batch [begränsar](batch-quota-limit.md#pool-size-limits) storleken på en pool som har kommunikation mellan noder aktiverat.


Det här kodfragmentet visar hur du skapar en pool för aktiviteter med flera instanser med hjälp av batch .NET-biblioteket.

```csharp
CloudPool myCloudPool =
    myBatchClient.PoolOperations.CreatePool(
        poolId: "MultiInstanceSamplePool",
        targetDedicatedComputeNodes: 3
        virtualMachineSize: "standard_d1_v2",
        cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "5"));

// Multi-instance tasks require inter-node communication, and those nodes
// must run only one task at a time.
myCloudPool.InterComputeNodeCommunicationEnabled = true;
myCloudPool.MaxTasksPerComputeNode = 1;
```

> [!NOTE]
> Om du försöker köra en multi-instance-aktivitet i en pool med kommunikation mellan noder, eller med ett *maxTasksPerNode* -värde som är större än 1, så schemaläggs inte aktiviteten, utan tids gräns i "aktivt" läge. 


### <a name="use-a-starttask-to-install-mpi"></a>Använd en StartTask för att installera MPI
Om du vill köra MPI-program med en multi-instance-aktivitet måste du först installera en MPI-implementering (MS-MPI eller Intel MPI, till exempel) på datornoderna i poolen. Detta är en lämplig tidpunkt för att använda en [StartTask][net_starttask]som körs när en nod ansluter till en pool eller startas om. Det här kodfragmentet skapar en StartTask som anger installations paketet MS-MPI som en [resurs fil][net_resourcefile]. Start aktivitetens kommando rad körs när resurs filen har laddats ned till noden. I det här fallet utför kommando raden en obevakad installation av MS-MPI.

```csharp
// Create a StartTask for the pool which we use for installing MS-MPI on
// the nodes as they join the pool (or when they are restarted).
StartTask startTask = new StartTask
{
    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
    UserIdentity = new UserIdentity(new AutoUserSpecification(elevationLevel: ElevationLevel.Admin)),
    WaitForSuccess = true
};
myCloudPool.StartTask = startTask;

// Commit the fully configured pool to the Batch service to actually create
// the pool and its compute nodes.
await myCloudPool.CommitAsync();
```

### <a name="remote-direct-memory-access-rdma"></a>Direkt åtkomst till fjärrminne (RDMA)
När du väljer en [RDMA-kompatibel storlek](../virtual-machines/windows/sizes-hpc.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) , till exempel A9 för Compute-noderna i batch-poolen, kan ditt MPI-program dra nytta av Azures höga prestanda för direkt åtkomst till fjärrminne (RDMA) med låg latens.

Sök efter de storlekar som anges som "RDMA-kompatibel" i följande artiklar:

* **CloudServiceConfiguration** pooler

  * [Storlekar för Cloud Services](../cloud-services/cloud-services-sizes-specs.md) (endast Windows)
* **VirtualMachineConfiguration** pooler

  * [Storlekar för virtuella datorer i Azure](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux)
  * [Storlekar för virtuella datorer i Azure](../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows)

> [!NOTE]
> Om du vill dra nytta av RDMA på [Linux-datornoder](batch-linux-nodes.md)måste du använda **Intel-MPI** på noderna. 
>

## <a name="create-a-multi-instance-task-with-batch-net"></a>Skapa en aktivitet med flera instanser med batch .NET
Nu när vi har täckt kraven för poolen och MPI-paketet, ska vi skapa aktiviteten för flera instanser. I det här kodfragmentet skapar vi en standard- [CloudTask][net_task]och konfigurerar sedan dess [MultiInstanceSettings][net_multiinstance_prop] -egenskap. Som tidigare nämnts är aktiviteten för flera instanser ingen distinkt typ av aktivitet, men en standard-batch-aktivitet som kon figurer ATS med inställningar för flera instanser.

```csharp
// Create the multi-instance task. Its command line is the "application command"
// and will be executed *only* by the primary, and only after the primary and
// subtasks execute the CoordinationCommandLine.
CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");

// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
// the primary and all subtasks.
myMultiInstanceTask.MultiInstanceSettings =
    new MultiInstanceSettings(numberOfNodes) {
    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
    CommonResourceFiles = new List<ResourceFile> {
    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
                     "MyMPIApplication.exe")
    }
};

// Submit the task to the job. Batch will take care of splitting it into subtasks and
// scheduling them for execution on the nodes.
await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);
```

## <a name="primary-task-and-subtasks"></a>Primär aktivitet och under aktiviteter
När du skapar inställningar för flera instanser för en aktivitet anger du antalet datornoder som ska utföra aktiviteten. När du skickar uppgiften till ett jobb skapar batch-tjänsten en **primär** aktivitet och tillräckligt många **under aktiviteter** som tillsammans matchar antalet noder som du har angett.

De här uppgifterna tilldelas ett heltals-ID i intervallet 0 till *numberOfInstances* -1. Uppgiften med ID 0 är den primära aktiviteten och alla andra ID: n är under aktiviteter. Om du till exempel skapar följande inställningar för flera instanser för en aktivitet skulle den primära aktiviteten ha ID: t 0, och under aktiviteterna har ID 1 till 9.

```csharp
int numberOfNodes = 10;
myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);
```

### <a name="master-node"></a>Huvud nod
När du skickar en aktivitet med flera instanser, anger batch-tjänsten en av datornoderna som "Master"-noden och schemalägger den primära aktiviteten så att den körs på huvudnoden. Under aktiviteterna är schemalagda att köras på resten av noderna som allokerats till aktiviteten för flera instanser.

## <a name="coordination-command"></a>Koordinations kommando
**Koordinations kommandot** utförs av både den primära aktiviteten och under aktiviteterna.

Anrops kommandot är blockerande – batch kör inte program kommandot förrän koordinations kommandot har returnerats för alla under aktiviteter. Koordinerings kommandot bör därför starta alla nödvändiga bakgrunds tjänster, kontrol lera att de är klara att använda och sedan avsluta. Detta koordinerings kommando för en lösning som använder MS-MPI version 7 startar till exempel tjänsten SMPD på noden och avslutas sedan:

```
cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d
```

Observera användningen av `start` i detta koordinations kommando. Detta är obligatoriskt eftersom `smpd.exe` programmet inte returnerar omedelbart efter körning. Utan att använda [Start][cmd_start] kommandot returneras inte det här koordinations kommandot, och därför blockeras program kommandot från att köras.

## <a name="application-command"></a>Program kommando
När den primära aktiviteten och alla under aktiviteter har slutfört körningen av koordinations kommandot, utförs *endast*körnings aktivitetens kommando rad av den primära aktiviteten. Vi anropar detta **program kommando** för att skilja den från koordinations kommandot.

För MS-MPI-program använder du kommandot Application för att köra ditt MPI-aktiverade program `mpiexec.exe`med. Här är till exempel ett program kommando för en lösning som använder MS-MPI version 7:

```
cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe
```

> [!NOTE]
> Eftersom MS-MPI `mpiexec.exe` använder `CCP_NODES` variabeln som standard (se [miljövariabler](#environment-variables)), undantas exempel kommando raden ovan.
>
>

## <a name="environment-variables"></a>Miljövariabler
Batch skapar flera [miljövariabler][msdn_env_var] som är speciella för aktiviteter med flera instanser på de Compute-noder som tilldelas en aktivitet med flera instanser. Dina samordnings-och program kommando rader kan referera till dessa miljövariabler, som de skript och program som de kör.

Följande miljövariabler skapas av batch-tjänsten för användning av aktiviteter med flera instanser:

* `CCP_NODES`
* `AZ_BATCH_NODE_LIST`
* `AZ_BATCH_HOST_LIST`
* `AZ_BATCH_MASTER_NODE`
* `AZ_BATCH_TASK_SHARED_DIR`
* `AZ_BATCH_IS_CURRENT_NODE_MASTER`

Fullständig information om dessa och de andra miljövariablerna för batch Compute-noden, inklusive innehåll och synlighet, finns i [Compute Node-miljövariabler][msdn_env_var].

> [!TIP]
> Kod exemplet för batch Linux MPI innehåller ett exempel på hur flera av de här miljövariablerna kan användas.

## <a name="resource-files"></a>Resursfiler
Det finns två uppsättningar resursfiler att överväga för aktiviteter med flera instanser: **delade resursfiler** som *alla* aktiviteter hämtar (både primära och under aktiviteter) och **resursfiler** som anges för aktiviteten för flera instanser, och som *bara hämtas av den primära* aktiviteten.

Du kan ange en eller flera **gemensamma resursfiler** i inställningarna för flera instanser för en aktivitet. Dessa delade resursfiler laddas ned från [Azure Storage](../storage/common/storage-introduction.md) till varje nods **delade katalog** med den primära aktiviteten och alla under aktiviteter. Du kan komma åt den delade katalogen aktivitet från program-och samordnings kommando `AZ_BATCH_TASK_SHARED_DIR` rader med hjälp av miljövariabeln. `AZ_BATCH_TASK_SHARED_DIR` Sökvägen är identisk på varje nod som tilldelas till aktiviteten för flera instanser, och därför kan du dela ett enda koordinerings kommando mellan den primära och alla under aktiviteter. Batch har inte "dela" katalogen i en fjärråtkomst, men du kan använda den som en monterings-eller delnings punkt som tidigare nämnts i tipset på miljövariabler.

Resursfiler som du anger för aktiviteter med flera instanser hämtas som standard till aktivitetens arbets katalog `AZ_BATCH_TASK_WORKING_DIR`. Som det nämnts, i motsats till vanliga resursfiler, är det bara resursfiler som anges för aktiviteten för flera instanser som har angetts för den primära aktiviteten.

> [!IMPORTANT]
> Använd alltid miljövariablerna `AZ_BATCH_TASK_SHARED_DIR` och `AZ_BATCH_TASK_WORKING_DIR` för att referera till dessa kataloger i kommando raderna. Försök inte att skapa Sök vägarna manuellt.
>
>

## <a name="task-lifetime"></a>Aktivitetens livs längd
Den primära aktivitetens livs längd styr livs längden för hela aktiviteten för flera instanser. När den primära utgången avslutas avslutas alla under aktiviteter. Slut koden för den primära aktiviteten är avslutnings koden för aktiviteten och används därför för att avgöra om uppgiften lyckades eller Miss lyckas för nya försök.

Om någon av under aktiviteterna Miss lyckas avslutas med en returkod som inte är noll, till exempel hela aktiviteten för flera instanser Miss lyckas. Aktiviteten för flera instanser avbryts sedan och försöker igen, upp till gränsen för återförsök.

När du tar bort en aktivitet med flera instanser raderas även den primära och alla under aktiviteter av batch-tjänsten. Alla under aktivitets kataloger och deras filer tas bort från Compute-noderna, precis som för en standard uppgift.

[TaskConstraints][net_taskconstraints] för en aktivitet med flera instanser, till exempel egenskaperna [MaxTaskRetryCount][net_taskconstraint_maxretry], [MaxWallClockTime][net_taskconstraint_maxwallclock]och [RetentionTime][net_taskconstraint_retention] , kan användas som de är för en standard uppgift och tillämpas på den primära och alla under aktiviteter. Men om du ändrar egenskapen [RetentionTime][net_taskconstraint_retention] efter att ha lagt till aktiviteten för flera instanser i jobbet, tillämpas den här ändringen endast på den primära aktiviteten. Alla under aktiviteter fortsätter att använda den ursprungliga [RetentionTime][net_taskconstraint_retention].

En Compute Nodes senaste uppgifts lista visar ID för en under aktivitet om den senaste aktiviteten ingick i en aktivitet med flera instanser.

## <a name="obtain-information-about-subtasks"></a>Hämta information om under aktiviteter
Om du vill få information om under aktiviteter med hjälp av batch .NET-biblioteket anropar du metoden [CloudTask. ListSubtasks][net_task_listsubtasks] . Den här metoden returnerar information om alla under aktiviteter och information om den Compute-nod som utförde aktiviteterna. Med hjälp av den här informationen kan du fastställa varje under aktivitets rot Katalog, pool-ID, aktuellt tillstånd, slut kod med mera. Du kan använda den här informationen i kombination med metoden [PoolOperations. GetNodeFile][poolops_getnodefile] för att hämta under aktivitetens filer. Observera att den här metoden inte returnerar information om den primära aktiviteten (ID 0).

> [!NOTE]
> Om inget annat anges gäller batch .NET-metoder som fungerar på [CloudTask][net_task] för flera instanser *enbart* för den primära aktiviteten. När du till exempel anropar metoden [CloudTask. ListNodeFiles][net_task_listnodefiles] i en grupp med flera instanser returneras bara den primära aktivitetens filer.
>
>

Följande kodfragment visar hur du hämtar information om under aktiviteter, samt innehållet i begär ande filen från de noder där de kördes.

```csharp
// Obtain the job and the multi-instance task from the Batch service
CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");

// Now obtain the list of subtasks for the task
IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();

// Asynchronously iterate over the subtasks and print their stdout and stderr
// output if the subtask has completed
await subtasks.ForEachAsync(async (subtask) =>
{
    Console.WriteLine("subtask: {0}", subtask.Id);
    Console.WriteLine("exit code: {0}", subtask.ExitCode);

    if (subtask.State == SubtaskState.Completed)
    {
        ComputeNode node =
            await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
                                                                 subtask.ComputeNodeInformation.ComputeNodeId);

        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardOutFileName);
        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\\" + Constants.StandardErrorFileName);
        stdOut = await stdOutFile.ReadAsStringAsync();
        stdErr = await stdErrFile.ReadAsStringAsync();

        Console.WriteLine("node: {0}:", node.Id);
        Console.WriteLine("stdout.txt: {0}", stdOut);
        Console.WriteLine("stderr.txt: {0}", stdErr);
    }
    else
    {
        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
    }
});
```

## <a name="code-sample"></a>Kodexempel
[MultiInstanceTasks][github_mpi] kod exemplet på GitHub visar hur du använder en aktivitet med flera instanser för att köra ett [MS-MPI][msmpi_msdn] -program i batch Compute-noder. Följ stegen i [förberedelser](#preparation) och [körning](#execution) för att köra exemplet.

### <a name="preparation"></a>Förberedelse
1. Följ de två första stegen i [hur du kompilerar och kör ett enkelt MS-MPI-program][msmpi_howto]. Detta uppfyller kraven för följande steg.
2. Bygg en *version* av [MPIHelloWorld][helloworld_proj] Sample MPI-programmet. Detta är det program som ska köras på datornoderna av aktiviteten för flera instanser.
3. Skapa en zip-fil `MPIHelloWorld.exe` som innehåller (som du skapade steg 2 `MSMpiSetup.exe` ) och (som du laddade ned steg 1). Du överför zip-filen som ett programpaket i nästa steg.
4. Använd [Azure Portal][portal] för att skapa ett batch- [program](batch-application-packages.md) med namnet "MPIHelloWorld" och ange den zip-fil som du skapade i föregående steg som version "1,0" av programpaketet. Mer information finns i [Ladda upp och hantera program](batch-application-packages.md#upload-and-manage-applications) .

> [!TIP]
> Bygg en *slut* version av `MPIHelloWorld.exe` så att du inte behöver ta med några ytterligare beroenden (till exempel `msvcp140d.dll` eller `vcruntime140d.dll`) i programpaketet.
>
>

### <a name="execution"></a>Körnings-
1. Ladda ned [Azure-Batch-exemplen][github_samples_zip] från GitHub.
2. Öppna MultiInstanceTasks- **lösningen** i Visual Studio 2019. `MultiInstanceTasks.sln` Lösnings filen finns i:

    `azure-batch-samples\CSharp\ArticleProjects\MultiInstanceTasks\`
3. Ange autentiseringsuppgifterna för batch-och lagrings `AccountSettings.settings` kontot i i **Microsoft. Azure. batch. Samples. common** Project.
4. **Skapa och kör** MultiInstanceTasks-lösningen för att köra exempel programmet MPI på Compute-noder i en batch-pool.
5. *Valfritt*: Använd [Azure Portal][portal] eller [batch Explorer][batch_labs] för att undersöka exempel-pool, jobb och aktivitet ("MultiInstanceSamplePool", "MultiInstanceSampleJob", "MultiInstanceSampleTask") innan du tar bort resurserna.

> [!TIP]
> Du kan ladda ned [Visual Studio Community][visual_studio] kostnads fritt om du inte har Visual Studio.
>
>

Utdata från `MultiInstanceTasks.exe` liknar följande:

```
Creating pool [MultiInstanceSamplePool]...
Creating job [MultiInstanceSampleJob]...
Adding task [MultiInstanceSampleTask] to job [MultiInstanceSampleJob]...
Awaiting task completion, timeout in 00:30:00...

Main task [MultiInstanceSampleTask] is in state [Completed] and ran on compute node [tvm-1219235766_1-20161017t162002z]:
---- stdout.txt ----
Rank 2 received string "Hello world" from Rank 0
Rank 1 received string "Hello world" from Rank 0

---- stderr.txt ----

Main task completed, waiting 00:00:10 for subtasks to complete...

---- Subtask information ----
subtask: 1
        exit code: 0
        node: tvm-1219235766_3-20161017t162002z
        stdout.txt:
        stderr.txt:
subtask: 2
        exit code: 0
        node: tvm-1219235766_2-20161017t162002z
        stdout.txt:
        stderr.txt:

Delete job? [yes] no: yes
Delete pool? [yes] no: yes

Sample complete, hit ENTER to exit...
```

## <a name="next-steps"></a>Nästa steg
* Microsoft HPC-& Azure Batch teamets blogg diskuterar [MPI support för Linux på Azure Batch][blog_mpi_linux]och innehåller information om att använda [openskum][openfoam] med batch. Du hittar python-kod exempel för [exempel på ett exempel på ett exempel på GitHub][github_mpi].
* Lär dig hur du [skapar pooler för Linux Compute Nodes](batch-linux-nodes.md) för användning i dina Azure Batch MPI-lösningar.

[helloworld_proj]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks/MPIHelloWorld

[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: https://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_labs]: https://azure.github.io/BatchExplorer/
[blog_mpi_linux]: https://blogs.technet.microsoft.com/windowshpc/2016/07/20/introducing-mpi-support-for-linux-on-azure-batch/
[cmd_start]: https://technet.microsoft.com/library/cc770297.aspx
[coord_cmd_example]: https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/article_samples/mpi/data/coordination-cmd
[github_mpi]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[msdn_env_var]: https://msdn.microsoft.com/library/azure/mt743623.aspx
[msmpi_msdn]: https://msdn.microsoft.com/library/bb524831.aspx
[msmpi_sdk]: https://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: https://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx
[openfoam]: http://www.openfoam.com/
[visual_studio]: https://www.visualstudio.com/vs/community/

[net_jobprep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[portal]: https://portal.azure.com
[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "Översikt över flera instanser"
