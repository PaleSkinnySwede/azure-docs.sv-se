---
title: Sök efter fel i pooler och noder
description: Den här artikeln beskriver de bakgrunds åtgärder som kan uppstå, tillsammans med fel för att söka efter och hur du undviker dem när du skapar pooler och noder.
author: mscurrell
ms.author: markscu
ms.date: 08/23/2019
ms.topic: conceptual
ms.openlocfilehash: 5051b9c536ded50e77fb75515c16daba884d5d24
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82115915"
---
# <a name="check-for-pool-and-node-errors"></a>Sök efter fel i pooler och noder

När du skapar och hanterar Azure Batch pooler sker vissa åtgärder omedelbart. Vissa åtgärder är dock asynkrona och körs i bakgrunden, vilket tar flera minuter att slutföra.

Det är enkelt att identifiera problem för åtgärder som sker direkt eftersom eventuella problem returneras direkt av API, CLI eller gränssnitt.

Den här artikeln beskriver bakgrunds åtgärder som kan uppstå för pooler och noder i pooler. Det anger hur du kan identifiera och undvika fel.

## <a name="pool-errors"></a>Allokeringsfel

### <a name="resize-timeout-or-failure"></a>Tids gräns för storlek eller haveriering

När du skapar en ny pool eller ändrar storlek på en befintlig pool anger du mål antalet noder.  Åtgärden skapa eller ändra storlek slutförs omedelbart, men den faktiska allokeringen av nya noder eller borttagning av befintliga noder kan ta flera minuter.  Du anger tids gränsen för storleks ändring i API för att [skapa](https://docs.microsoft.com/rest/api/batchservice/pool/add) eller [ändra storlek](https://docs.microsoft.com/rest/api/batchservice/pool/resize) . Om batch inte kan hämta mål antalet noder under tids gränsen för storleks ändring försätts poolen i ett stabilt tillstånd och rapporter ändrar storlek på fel.

Egenskapen [ResizeError](https://docs.microsoft.com/rest/api/batchservice/pool/get#resizeerror) för den senaste utvärderingen visar en lista över felen som har inträffat.

Vanliga orsaker till fel storleks ändringar är:

- Tids gränsen för storleks ändring är för kort
  - I de flesta fall är standard tids gränsen på 15 minuter tillräckligt lång för att pool-noder ska allokeras eller tas bort.
  - Om du allokerar ett stort antal noder rekommenderar vi att du anger tids gränsen för storleks ändring till 30 minuter. Till exempel när du ändrar storlek på fler än 1 000 noder från en Azure Marketplace-avbildning eller till fler än 300 noder från en anpassad VM-avbildning.
- Otillräcklig kärn kvot
  - Ett batch-konto är begränsat till antalet kärnor som det kan allokera i alla pooler. Batch slutar allokera noder när kvoten har uppnåtts. Du [kan öka](https://docs.microsoft.com/azure/batch/batch-quota-limit) kärn kvoten så att batchen kan allokera fler noder.
- Otillräckliga undernät-IP: er när en [pool finns i ett virtuellt nätverk](https://docs.microsoft.com/azure/batch/batch-virtual-network)
  - Ett undernät för virtuellt nätverk måste ha tillräckligt många otilldelade IP-adresser för att allokera till varje begärd pool-nod. Annars går det inte att skapa noderna.
- Otillräckliga resurser när en [pool finns i ett virtuellt nätverk](https://docs.microsoft.com/azure/batch/batch-virtual-network)
  - Du kan skapa resurser som belastningsutjämnare, offentliga IP-adresser och nätverks säkerhets grupper i samma prenumeration som batch-kontot. Kontrol lera att prenumerations kvoterna räcker för dessa resurser.
- Stora pooler med anpassade VM-avbildningar
  - Det kan ta längre tid att tilldela stora pooler som använder anpassade VM-avbildningar och ändra storlek på tids gränser.  Se [skapa en pool med det delade avbildnings galleriet](batch-sig-images.md) för rekommendationer om gränser och konfiguration.

### <a name="automatic-scaling-failures"></a>Automatiska skalnings problem

Du kan också ange Azure Batch för att automatiskt skala antalet noder i en pool. Du definierar parametrar för den [automatiska skalnings formeln för en pool](https://docs.microsoft.com/azure/batch/batch-automatic-scaling). Batch-tjänsten använder formeln för att regelbundet utvärdera antalet noder i poolen och ange ett nytt mål nummer. Följande typer av problem kan uppstå:

- Utvärderingen av automatisk skalning Miss lyckas.
- Den resulterande ändrings åtgärden Miss lyckas och tids gränsen uppnås.
- Ett problem med den automatiska skalnings formeln leder till felaktiga värden för mål värden. Storleken på antingen Works eller tids gränsen.

Du kan få information om den senaste automatiska skalnings utvärderingen med hjälp av egenskapen [autoScaleRun](https://docs.microsoft.com/rest/api/batchservice/pool/get#autoscalerun) . Den här egenskapen rapporterar utvärderings tiden, värdena och resultatet samt eventuella prestanda fel.

[Loggen för storleks ändring av pooler](https://docs.microsoft.com/azure/batch/batch-pool-resize-complete-event) samlar in information om alla utvärderingar.

### <a name="delete"></a>Ta bort

När du tar bort en pool som innehåller noder, tar den första batchen bort noderna. Sedan tas själva pool-objektet bort. Det kan ta några minuter innan poolens noder tas bort.

Batch anger det [tillstånd för poolen](https://docs.microsoft.com/rest/api/batchservice/pool/get#poolstate) som ska **tas bort** under borttagnings processen. Det anropande programmet kan identifiera om borttagningen tar för lång tid genom att använda egenskaperna **State** och **stateTransitionTime** .

## <a name="pool-compute-node-errors"></a>Fel vid beräkning av pool

Även om batch allokerar noder i en pool kan olika problem orsaka att några av noderna inte är felfria och inte kan köra uppgifter. De här noderna kostar fortfarande att betala, så det är viktigt att upptäcka problem för att undvika att betala för noder som inte kan användas. Förutom vanliga nodfel kan det vara bra att känna till det aktuella [jobb läget](/rest/api/batchservice/job/get#jobstate) för fel sökning.

### <a name="start-task-failures"></a>Starta aktivitets felen

Du kanske vill ange en valfri [Start uppgift](/rest/api/batchservice/pool/add#starttask) för en pool. Som med vilken aktivitet som helst kan du använda en kommando rad och resursfiler för att ladda ned från lagringen. Start aktiviteten körs för varje nod när den har startats. Egenskapen **waitForSuccess** anger om batch väntar tills start aktiviteten har slutförts innan aktiviteterna schemaläggs till en nod.

Vad händer om du har konfigurerat noden för att vänta på att aktiviteten slutförs, men start aktiviteten Miss lyckas? I så fall går det inte att använda noden, men kommer fortfarande att debiteras.

Du kan identifiera start aktivitets problem genom att använda egenskaperna [result](/rest/api/batchservice/computenode/get#taskexecutionresult) och [failureInfo](/rest/api/batchservice/computenode/get#taskfailureinformation) för den översta [startTaskInfo](/rest/api/batchservice/computenode/get#starttaskinformation) Node-egenskapen.

En misslyckad start aktivitet gör också att batch anger nodens [tillstånd](/rest/api/batchservice/computenode/get#computenodestate) till **starttaskfailed** om **waitForSuccess** har angetts till **True**.

Precis som med alla aktiviteter kan det finnas många orsaker till att start aktiviteten Miss Missing.  Du kan felsöka genom att kontrol lera STDOUT, stderr och eventuella ytterligare verksamhetsspecifika loggfiler.

Start aktiviteter måste vara omordnade eftersom det är möjligt att start aktiviteten körs flera gånger på samma nod. Start aktiviteten körs när en nod återställs eller startas om. I sällsynta fall körs en start aktivitet efter att en händelse orsakade en omstart av en nod, där ett av operativ systemen eller de tillfälliga diskarna återställdes när den andra inte var det. Eftersom batch-startuppgifter (t. ex. alla batch-aktiviteter) körs från den tillfälliga disken är detta vanligt vis inget problem, men i vissa fall där start aktiviteten installerar ett program på operativ system disken och behåller andra data på den tillfälliga disken kan detta orsaka problem eftersom det inte är synkroniserat. Skydda ditt program efter behov om du använder båda diskarna.

### <a name="application-package-download-failure"></a>Hämtnings problem för program paket

Du kan ange ett eller flera programpaket för en pool. Batch laddar ned de angivna paketfilerna till varje nod och expanderar filerna när noden har startat, men innan aktiviteter schemaläggs. Det är vanligt att använda en kommando rad för start aktivitet tillsammans med programpaket. Till exempel för att kopiera filer till en annan plats eller köra installations programmet.

Egenskapen Node [errors](/rest/api/batchservice/computenode/get#computenodeerror) rapporterar ett fel vid hämtning och avkomprimering av ett programpaket. Node-tillstånd är inställt på **oanvändbart**.

### <a name="container-download-failure"></a>Hämtnings problem för behållare

Du kan ange en eller flera behållar referenser i en pool. Batch hämtar de angivna behållarna till varje nod. Egenskapen Node [errors](/rest/api/batchservice/computenode/get#computenodeerror) rapporterar ett fel vid hämtning av en behållare och anger att nodens tillstånd är **oanvändbar**.

### <a name="node-in-unusable-state"></a>Noden är oanvändbar

Azure Batch kan ange att [nodens tillstånd](/rest/api/batchservice/computenode/get#computenodestate) inte kan **användas** av många skäl. När Node State är **oanvändbart**kan aktiviteter inte schemaläggas till noden, men de debiteras ändå.

Noder i ett **oanvändbart** tillstånd, men utan [fel](/rest/api/batchservice/computenode/get#computenodeerror) innebär att batchen inte kan kommunicera med den virtuella datorn. I det här fallet försöker batch alltid återställa den virtuella datorn. Batch försöker inte automatiskt att återställa virtuella datorer som inte kunde installera programpaket eller behållare trots att deras tillstånd är **oanvändbart**.

Om batch kan bestämma orsaken rapporterar egenskapen Node [errors](/rest/api/batchservice/computenode/get#computenodeerror) .

Fler exempel på orsaker till **oanvändbara** noder är:

- En anpassad virtuell dator avbildning är ogiltig. Till exempel en bild som inte är korrekt för beredd.

- En virtuell dator flyttas på grund av ett infrastruktur haveri eller en uppgradering på lägre nivå. Batch återställer noden.

- En avbildning av en virtuell dator har distribuerats på maskin vara som inte stöder den. Försök till exempel att köra en CentOS HPC-avbildning på en [Standard_D1_v2](../virtual-machines/dv2-dsv2-series.md) virtuell dator.

- De virtuella datorerna finns i ett [virtuellt Azure-nätverk](batch-virtual-network.md)och trafiken har blockerats för nyckel portar.

- De virtuella datorerna finns i ett virtuellt nätverk, men utgående trafik till Azure Storage blockeras.

- De virtuella datorerna finns i ett virtuellt nätverk med en kund-DNS-konfiguration och DNS-servern kan inte lösa Azure Storage.

### <a name="node-agent-log-files"></a>Loggfiler för Node agent

Batch agent-processen som körs på varje pool-nod kan ge loggfiler som kan vara till hjälp om du behöver kontakta supporten om ett problem med en adresspool. Loggfiler för en nod kan överföras via Azure Portal, Batch Explorer eller ett [API](/rest/api/batchservice/computenode/uploadbatchservicelogs). Det är praktiskt att ladda upp och spara loggfilerna. Efteråt kan du ta bort noden eller poolen för att spara kostnaden för de noder som körs.

### <a name="node-disk-full"></a>Nod disken är full

Den tillfälliga enheten för en pool med virtuella datorer används av batch för projektfiler, filer och delade filer.

- Programpaket-filer
- Resurs fil för aktivitet
- Programspecifika filer som laddats ned till en av batch-mapparna
- STDOUT-och stderr-filer för varje aktivitets program körning
- Programspecifika utdatafiler

En del av de här filerna skrivs bara en gång när pooler har skapats, till exempel paket för programpaket eller resurs fil för att starta aktivitet. Även om de här filerna är för stora kan de fylla den temporära enheten även om det bara skrivs en gång när noden skapas.

Andra filer skrivs ut för varje aktivitet som körs på en nod, till exempel STDOUT och stderr. Om ett stort antal aktiviteter körs på samma nod och/eller om originalfilerna är för stora kan de fylla den temporära enheten.

Storleken på den tillfälliga enheten beror på storleken på den virtuella datorn. Ett övervägande när du väljer en VM-storlek är att säkerställa att den tillfälliga enheten har tillräckligt med utrymme.

- I Azure Portal när du lägger till en pool, kan den fullständiga listan med VM-storlekar visas och det finns en resurs disk storleks kolumn.
- Artiklarna som beskriver alla VM-storlekar har tabeller med kolumnen Temp Storage. till exempel [Compute-optimerade VM-storlekar](/azure/virtual-machines/windows/sizes-compute)

För filer som skrivs ut av varje aktivitet kan en kvarhållningsperiod anges för varje aktivitet som avgör hur länge originalfilerna sparas innan de rensas automatiskt. Retentions tiden kan minskas för att minska lagrings kraven.


Om den temporära disken tar slut på utrymme (eller ligger mycket nära att det tar slut på utrymme) flyttas noden till [oanvändbart](/rest/api/batchservice/computenode/get#computenodestate) läge och ett nodfel rapporteras om att disken är full.

### <a name="what-to-do-when-a-disk-is-full"></a>Vad du gör när en disk är full

Ta reda på varför disken är full: om du inte är säker på vad som tar upp utrymme på noden, rekommenderar vi att du fjärransluter till noden och undersöker manuellt var utrymmet har försvunnit. Du kan också använda [API: erna för batch-lista](https://docs.microsoft.com/rest/api/batchservice/file/listfromcomputenode) för att undersöka filer i grupphanterade mappar (till exempel Uppgiftsutdata). Observera att detta API endast listar filer i de grupper som hanteras av gruppen och om dina aktiviteter har skapat filer någon annan stans kommer de inte att visas.

Kontrol lera att alla data som du behöver har hämtats från noden eller laddats upp till ett varaktigt lager. All minskning av diskens fullständiga problem innebär att ta bort data för att frigöra utrymme.

### <a name="recovering-the-node"></a>Återställer noden

1. Om poolen är en [C. loudServiceConfiguration](https://docs.microsoft.com/rest/api/batchservice/pool/add#cloudserviceconfiguration) -pool kan du återställa noden via [batch-avbildningen-API](https://docs.microsoft.com/rest/api/batchservice/computenode/reimage). Då rensas hela disken. Reimage stöds inte för närvarande för [VirtualMachineConfiguration](https://docs.microsoft.com/rest/api/batchservice/pool/add#virtualmachineconfiguration) -pooler.

2. Om poolen är en [VirtualMachineConfiguration](https://docs.microsoft.com/rest/api/batchservice/pool/add#virtualmachineconfiguration)kan du ta bort noden från poolen med hjälp av API: [et Remove Nodes](https://docs.microsoft.com/rest/api/batchservice/pool/removenodes). Sedan kan du växa poolen igen för att ersätta den felaktiga noden med en ny.

3.  Ta bort gamla slutförda jobb eller gamla slutförda uppgifter vars uppgifts data fortfarande finns på noderna. En ledtråd för vilka jobb/uppgifter-data finns på noderna du kan titta på i [RecentTasks-samlingen](https://docs.microsoft.com/rest/api/batchservice/computenode/get#taskinformation) på noden eller på [filer på noden](https://docs.microsoft.com//rest/api/batchservice/file/listfromcomputenode). Om du tar bort jobbet raderas alla aktiviteter i jobbet och om du tar bort uppgifterna i jobbet utlöses data i aktivitets katalogerna på noden, vilket frigör utrymme. När du har frigjort tillräckligt med utrymme kan du starta om noden och den ska flyttas från "oanvändbar"-tillstånd och till "inaktive rad" igen.

## <a name="next-steps"></a>Nästa steg

Kontrol lera att du har angett att ditt program ska implementera omfattande fel kontroller, särskilt för asynkrona åtgärder. Det kan vara viktigt att snabbt identifiera och diagnostisera problem.
