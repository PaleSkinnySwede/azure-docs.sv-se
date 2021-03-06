---
title: Prestanda tips för Azure Cosmos DB Java SDK v4
description: Lär dig mer om klient konfigurations alternativ för att förbättra Azure Cosmos Database-prestanda för Java SDK v4
author: anfeldma-ms
ms.service: cosmos-db
ms.devlang: java
ms.topic: conceptual
ms.date: 05/08/2020
ms.author: anfeldma
ms.openlocfilehash: ce4e4d11ead41ee8cc4a4bd1d85f1fbad2af4b07
ms.sourcegitcommit: 999ccaf74347605e32505cbcfd6121163560a4ae
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2020
ms.locfileid: "82982538"
---
# <a name="performance-tips-for-azure-cosmos-db-java-sdk-v4"></a>Prestanda tips för Azure Cosmos DB Java SDK v4

> [!div class="op_single_selector"]
> * [Java SDK v4](performance-tips-java-sdk-v4-sql.md)
> * [Asynkron Java-SDK v2](performance-tips-async-java.md)
> * [Synkron Java-SDK v2](performance-tips-java.md)
> * [.NET](performance-tips.md)
> 

> [!IMPORTANT]  
> Prestanda tipsen i den här artikeln är endast för Azure Cosmos DB Java SDK v4. Se Azure Cosmos DB Java SDK v4-viktig information, [maven-lagringsplatsen](https://mvnrepository.com/artifact/com.azure/azure-cosmos)och Azure Cosmos DB Java SDK v4 v4 [fel söknings guide](troubleshoot-java-sdk-v4-sql.md) för mer information. Om du använder en äldre version än v4 kan du läsa om hur du uppgraderar till v4 i [Azure Cosmos DB Java SDK v4](migrate-java-v4-sdk.md) -guide.
>

Azure Cosmos DB är en snabb och flexibel distribuerad databas som skalar sömlöst med garanterad svars tid och data flöde. Du behöver inte göra större ändringar i arkitekturen eller skriva komplex kod för att skala databasen med Azure Cosmos DB. Att skala upp och ned är lika enkelt som att göra ett enda API-anrop eller SDK-metod anrop. Men eftersom Azure Cosmos DB nås via nätverks anrop finns det optimeringar på klient sidan som du kan göra för att uppnå högsta prestanda när du använder Azure Cosmos DB Java SDK v4.

Så om du frågar "Hur kan jag förbättra min databas prestanda?" Överväg följande alternativ:

## <a name="networking"></a>Nätverk

* **Anslutnings läge: Använd direkt läge**
<a id="direct-connection"></a>
    
    Hur en klient ansluter till Azure Cosmos DB har viktiga konsekvenser för prestanda, särskilt vad gäller svars tid på klient sidan. *ConnectionMode* är en nyckel konfigurations inställning som är tillgänglig för konfigurering av klientens *ConnectionPolicy*. För Azure Cosmos DB Java SDK V4 är de två tillgängliga *ConnectionMode*s:  
      
    * [Gateway (standard)](/java/api/com.microsoft.azure.cosmosdb.connectionmode)  
    * [Direct](/java/api/com.microsoft.azure.cosmosdb.connectionmode)

    Dessa *ConnectionMode*är i princip den väg som begär Anden tar från klient datorn till partitioner i Azure Cosmos DB backend. Normalt är Direct-läge det bästa alternativet för bästa prestanda – det gör att klienten kan öppna TCP-anslutningar direkt till partitioner i Azure Cosmos DB backend-och sändnings begär Anden *Direct*-ly utan mellanmedium. I Gateway-läge dirigeras begär Anden som görs av klienten till en så kallad "Gateway"-server i Azure Cosmos DB klient del, som i sin tur ger ut dina begär anden till lämpliga partitioner i Azure Cosmos DB backend. Om ditt program körs i ett företags nätverk med strikta brand Väggs begränsningar är Gateway-läget det bästa valet eftersom det använder standard-HTTPS-porten och en enda slut punkt. Prestanda kompromissen är dock att Gateway-läget omfattar ytterligare nätverks hopp (klient till gateway och gateway till partition) varje gång data läses eller skrivs till Azure Cosmos DB. Därför erbjuder Direct-läget bättre prestanda på grund av färre nätverks hopp.

    *ConnectionMode* konfigureras under konstruktion av den Azure Cosmos DB klient instansen med parametern *ConnectionPolicy* :
    
   #### <a name="async"></a>[Async](#tab/api-async)

   ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-connection-policy-async"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    public ConnectionPolicy getConnectionPolicy() {
        ConnectionPolicy policy = new ConnectionPolicy();
        policy.setMaxPoolSize(1000);
        return policy;
    }

    ConnectionPolicy connectionPolicy = new ConnectionPolicy();
    CosmosAsyncClient client = new CosmosClientBuilder()
        .setEndpoint(HOST)
        .setKey(MASTER)
        .setConnectionPolicy(connectionPolicy)
        .buildAsyncClient();
    ```

    #### <a name="sync"></a>[Synkronisera](#tab/api-sync)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-sync-api"></a><a id="java4-connection-policy-sync"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) Sync API

    ```java
    public ConnectionPolicy getConnectionPolicy() {
        ConnectionPolicy policy = new ConnectionPolicy();
        policy.setMaxPoolSize(1000);
        return policy;
    }

    ConnectionPolicy connectionPolicy = new ConnectionPolicy();
    CosmosClient client = new CosmosClientBuilder()
        .setEndpoint(HOST)
        .setKey(MASTER)
        .setConnectionPolicy(connectionPolicy)
        .buildClient();
    ```

    --- 

<a name="collocate-clients"></a>
* **Samordna-klienter i samma Azure-region för prestanda**<a id="same-region"></a>

    När det är möjligt kan du placera alla program som anropar Azure Cosmos DB i samma region som Azure Cosmos-databasen. För en ungefärlig jämförelse kan anrop till Azure Cosmos DB inom samma region slutföras inom 1-2 MS, men svars tiden mellan västra USA och östra kust är >50 ms. Den här fördröjningen kan troligt vis variera från begäran till begäran beroende på den väg som tas av begäran när den skickas från klienten till Azure Data Center-gränser. Den lägsta möjliga fördröjningen uppnås genom att se till att det anropande programmet finns i samma Azure-region som den etablerade Azure Cosmos DB slut punkten. En lista över tillgängliga regioner finns i [Azure-regioner](https://azure.microsoft.com/regions/#services).

    ![Bild av Azure Cosmos DB anslutnings princip](./media/performance-tips/same-region.png)

    En app som samverkar med ett Azure Cosmos DB konto i flera regioner måste konfigurera [önskade platser]() för att se till att begär Anden kommer till en samordnad region.

* **Aktivera accelererat nätverk på den virtuella Azure-datorn för kortare svars tid.**

Vi rekommenderar att du följer anvisningarna för att aktivera accelererat nätverk i [Windows (Klicka för instruktioner)](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell) eller [Linux (Klicka för instruktioner)](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) virtuell Azure-dator för att maximera prestanda.

Från och med accelererade nätverk kan IO-överföring mellan din virtuella Azure-dator och andra Azure-resurser vara onödigt dirigerad genom en värd och en virtuell växel som ligger mellan den virtuella datorn och nätverkskortet. Om värd och virtuell växel infogas i Datapath inte bara ökar svars tiden och darr i kommunikations kanalen, stjälas även CPU-cykler från den virtuella datorn. Med accelererat nätverk är de virtuella dator gränssnitten direkt med NÄTVERKSKORTet utan mellanhänder; all nätverks princip information som hanteras av värden och den virtuella växeln hanteras nu i maskin varan på NÄTVERKSKORTet. värden och den virtuella växeln ignoreras. Vanligt vis kan du förväntar dig kortare latens och högre genom strömning, samt mer *konsekvent* svars tid och minskad CPU-användning när du aktiverar accelererat nätverk.

Begränsningar: accelererade nätverk måste stödjas på VM OS och kan bara aktive ras när den virtuella datorn stoppas och frigörs. Det går inte att distribuera den virtuella datorn med Azure Resource Manager.

Mer information finns i [Windows](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-powershell) -och [Linux](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) -instruktionerna.

## <a name="sdk-usage"></a>SDK-användning
* **Installera den senaste SDK: n**

    Azure Cosmos DB SDK: er har ständigt förbättrats för att ge bästa möjliga prestanda. Se de [Azure Cosmos DB SDK](sql-api-sdk-async-java.md) -sidorna för att fastställa de senaste SDK: er och gransknings förbättringarna.

* **Använd en singleton Azure Cosmos DB-klient för programmets livs längd**

    Varje Azure Cosmos DB klient instans är tråd säker och utför effektiv anslutnings hantering och cachelagring av adresser. För att möjliggöra effektiv anslutnings hantering och bättre prestanda av Azure Cosmos DB-klienten rekommenderar vi att du använder en enda instans av Azure Cosmos DB klienten per AppDomain för programmets livs längd.

   <a id="max-connection"></a>

* **Använd den lägsta konsekvens nivå som krävs för ditt program**

    När du skapar en *CosmosClient*används standard konsekvensen om den inte uttryckligen anges är *session*. Om konsekvensen av *sessionen* inte krävs av din program logik ställer du in *konsekvensen* på *eventuell*. Obs! Vi rekommenderar att du använder minst konsekvens för *sessioner* i program som använder Azure Cosmos DB ändra feed-processorn.

* **Använd asynkront API till maximalt allokerat data flöde**

    Azure Cosmos DB Java SDK v4 paketerar två API: er, synkroniserar och asynkrona. Den asynkrona API: n implementerar i stort sett SDK-funktioner, medan Sync API är en tunn omslutning som gör det möjligt att blockera anrop till asynkron API. Detta är i motsats till den äldre Azure Cosmos DB asynkron Java SDK v2, som var asynkront och till den äldre Azure Cosmos DB Sync Java SDK v2, som endast är synkroniserad och hade en helt separat implementering. 
    
    Valet av API fastställs vid klient initieringen. en *CosmosAsyncClient* stöder asynkron API när en *CosmosClient* har stöd för Sync-API. 
    
    Det asynkrona API: et implementerar icke-blockerande i/o och är det bästa valet om målet är att max för strömning när begär anden till Azure Cosmos DB. 
    
    Att använda Sync API kan vara det rätta valet om du vill ha eller behöver ett API som blockerar svar på varje begäran, eller om synkron åtgärd är det dominerande paradigmet i ditt program. Till exempel kanske du vill att Sync-API: n när du sparar data för att Azure Cosmos DB i ett mikrotjänster-program, men det finns inte tillräckligt med data flöde.  
    
    Det är bara att tänka på att Sync API data flöde degraderas med ökande svars tid för begäran, medan det asynkrona API: et kan fylla den fullständiga bandbredden för maskin varan. 
    
    Geografisk samplacering kan ge dig bättre och mer konsekvent data flöde när du använder Sync API (se [samordna-klienter i samma Azure-region för prestanda](#collocate-clients)) men ändå inte förväntas överskrida asynkront API-genomflöde.

    Vissa användare kan också vara okända med [projekt reaktor](https://projectreactor.io/), det reactive Streams-ramverk som används för att implementera Azure Cosmos DB Java SDK v4 async API. Om detta är ett problem rekommenderar vi att du läser vår introduktions [Guide för reaktorer](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-pattern-guide.md) och tar en titt på den här [introduktionen för](https://tech.io/playgrounds/929/reactive-programming-with-reactor-3/Intro) att få en mer aktuell programmering för att bekanta dig. Om du redan har använt Azure Cosmos dB med ett asynkront gränssnitt och SDK: n som du använde Azure Cosmos DB asynkron Java SDK v2, kan du vara bekant med [reactivex-](http://reactivex.io/)/[RxJava](https://github.com/ReactiveX/RxJava) men vara osäker på vad som har ändrats i Project reaktor. I så fall kan du ta en titt på vår [reaktor vs. RxJava-guide](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples/blob/master/reactor-rxjava-guide.md) för att bli bekant.

    Följande kodfragment visar hur du initierar din Azure Cosmos DB-klient för asynkron API eller synkroniserings-API-åtgärd:

    #### <a name="async"></a>[Async](#tab/api-async)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-async-client"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    CosmosAsyncClient client = new CosmosClientBuilder()
        .setEndpoint(HOSTNAME)
        .setKey(MASTERKEY)
        .setConnectionPolicy(CONNECTIONPOLICY)
        .setConsistencyLevel(CONSISTENCY)
        .buildAsyncClient();
    ```

    #### <a name="sync"></a>[Synkronisera](#tab/api-sync)
 
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-sync-api"></a><a id="java4-sync-client"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) Sync API

    ```java
    CosmosClient client = new CosmosClientBuilder()
        .setEndpoint(HOSTNAME)
        .setKey(MASTERKEY)
        .setConnectionPolicy(CONNECTIONPOLICY)
        .setConsistencyLevel(CONSISTENCY)
        .buildClient();
    ```    

    ---

* **Justera ConnectionPolicy**

    Som standard görs direkt läge Cosmos DB begär Anden via TCP när du använder Azure Cosmos DB Java SDK v4. Internt använder SDK en särskild arkitektur för direkt läge för att dynamiskt hantera nätverks resurser och få bästa möjliga prestanda.

    I Azure Cosmos DB Java SDK V4 är Direct-läget det bästa valet för att förbättra databasens prestanda med de flesta arbets belastningar. 

    * ***Översikt över direkt läge***

        ![Bild av arkitekturen för direkt läge](./media/performance-tips-async-java/rntbdtransportclient.png)

        Arkitekturen på klient sidan som används i direkt läge möjliggör förutsägbar nätverks användning och multiplex-åtkomst till Azure Cosmos DB repliker. Diagrammet ovan visar hur Direct-läge dirigerar klient begär anden till repliker i Cosmos DB-backend-servern. Arkitekturen för direkt läge allokerar upp till 10 **kanaler** på klient sidan per DB-replik. En kanal är en TCP-anslutning som föregås av en buffert för begäran, som är en djup på 30 begär Anden. Kanaler som tillhör en replik allokeras dynamiskt efter behov av replikens **tjänst slut punkt**. När användaren utfärdar en begäran i direkt läge dirigerar **TransportClient** begäran till rätt tjänst slut punkt utifrån partitionsnyckel. **Begär ande kön** buffrar begär Anden före tjänst slut punkten.

    * ***ConnectionPolicy konfigurations alternativ för direkt läge***

        Dessa konfigurations inställningar styr beteendet för RNTBD-arkitekturen som styr SDK-beteendet för Direct-läge.
        
        I det första steget använder du följande rekommenderade konfigurations inställningar nedan. Dessa *ConnectionPolicy* -alternativ är avancerade konfigurations inställningar som kan påverka SDK-prestanda på oväntade sätt. Vi rekommenderar att användarna undviker att ändra dem om de inte känner sig för att förstå kompromisserna och det är absolut nödvändigt. Kontakta Azure Cosmos DB- [teamet](mailto:CosmosDBPerformanceSupport@service.microsoft.com) om du stöter på problem på det här specifika ämnet.

        Om du använder Azure Cosmos DB som en referens databas (det vill säga databasen används för många punkt läsnings åtgärder och få Skriv åtgärder) kan det vara acceptabelt att ange *idleEndpointTimeout* till 0 (det vill säga ingen tids gräns).


        | Konfigurations alternativ       | Standardvärde    |
        | :------------------:       | :-----:    |
        | bufferPageSize             | 8192       |
        | connectionTimeout          | "PT1M"     |
        | idleChannelTimeout         | "PT0S"     |
        | idleEndpointTimeout        | "PT1M10S"  |
        | maxBufferCapacity          | 8388608    |
        | maxChannelsPerEndpoint     | 10         |
        | maxRequestsPerChannel      | 30         |
        | receiveHangDetectionTime   | "PT1M5S"   |
        | requestExpiryInterval      | "PT5S"     |
        | requestTimeout             | "PT1M"     |
        | requestTimerResolution     | "PT 0,5 S"   |
        | sendHangDetectionTime      | "PT10S"    |
        | shutdownTimeout            | "PT15S"    |

* **Justera parallella frågor för partitionerade samlingar**

    Azure Cosmos DB Java SDK v4 stöder parallella frågor som gör att du kan fråga en partitionerad samling parallellt. Mer information finns i [kod exempel](https://github.com/Azure-Samples/azure-cosmos-java-sql-api-samples) för att arbeta med Azure Cosmos DB Java SDK v4. Parallella frågor är utformade för att förbättra svars tid och data flöde för deras serie motsvarighet.

    * ***Justera setMaxDegreeOfParallelism\:***
    
        Parallella frågor fungerar genom att fråga flera partitioner parallellt. Data från en enskild partitionerad samling hämtas dock i serie med avseende på frågan. Använd setMaxDegreeOfParallelism för att ställa in antalet partitioner som har maximal chans att uppnå den mest utförda frågan, förutsatt att alla andra system villkor är desamma. Om du inte känner till antalet partitioner kan du använda setMaxDegreeOfParallelism för att ange ett högt antal, och systemet väljer det lägsta (antal partitioner, indata från användaren) som den högsta graden av parallellitet.

        Det är viktigt att Observera att parallella frågor ger de bästa fördelarna om data är jämnt fördelade över alla partitioner med avseende på frågan. Om den partitionerade samlingen är partitionerad, så att alla eller en majoritet av de data som returneras av en fråga är koncentrerade i några partitioner (en partition i värsta fall), skulle prestandan för frågan bli Flask hals av dessa partitioner.

    * ***Justera setMaxBufferedItemCount\:***
    
        Parallell fråga är utformad för att hämta resultat när den aktuella gruppen med resultat bearbetas av klienten. För hämtning bidrar till den totala tids fördröjnings förbättringen av en fråga. setMaxBufferedItemCount begränsar antalet i förväg hämtade resultat. Om du anger setMaxBufferedItemCount till det förväntade antalet returnerade resultat (eller en högre siffra) kan frågan ta emot maximal nytta av för hämtning.

        För hämtning fungerar på samma sätt oavsett MaxDegreeOfParallelism, och det finns en enda buffert för data från alla partitioner.

* **Skala ut din klient arbets belastning**

    Om du testar på hög data flödes nivå kan klient programmet bli Flask halsen på grund av att datorn capping ut på processor-eller nätverks användning. Om du når den här punkten kan du fortsätta att skicka det Azure Cosmos DB kontot ytterligare genom att skala ut dina klient program över flera servrar.

    En lämplig tumregel är att inte överskrida >50% processor användning på en angiven server, för att hålla svars tiden låg.

   <a id="tune-page-size"></a>

* **Justera sid storleken för frågor/läsa feeds för bättre prestanda**

    När du utför en Mass läsning av dokument med hjälp av funktionen för att läsa feeds (till exempel *readItems*) eller när du utfärdar en SQL-fråga (*queryItems*) returneras resultatet i ett segment om resultat mängden är för stor. Som standard returneras resultaten i segment om 100 objekt eller 1 MB, beroende på vilken gräns som nåtts först.

    Anta att ditt program utfärdar en fråga till Azure Cosmos DB och att programmet kräver en fullständig uppsättning frågeresultat för att kunna slutföra uppgiften. Om du vill minska antalet nätverks fördröjningar som krävs för att hämta alla tillämpliga resultat kan du öka sid storleken genom att justera fältet [x-MS-Max-item-Count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) Request Head. 

    * För frågor med en partition, justera värdet för fältet [x-MS-Max-item-Count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) till-1 (ingen gräns för sid storlek) maximerar svars tiden genom att minimera antalet sidor för svar på frågan: antingen kommer den fullständiga resultat uppsättningen att returneras på en enda sida, eller om frågan tar längre tid än tids gränsen, returneras den fullständiga resultat uppsättningen på det minsta antal sidor som är möjliga. Detta sparar i multipler av tiden för tur och retur-begäran.
    
    * För frågor över olika partitioner ställer du in fältet [x-MS-Max-item-Count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) till-1 och tar bort sid storleks gränsen för att överbelasta SDK: n med ohanterade sid storlekar. I det här fallet rekommenderar vi att du höjer sid storleks gränsen upp till ett stort men begränsat värde, t. ex. 1000, för att minska svars tiden. 
    
    I vissa program kanske du inte behöver en fullständig uppsättning frågeresultat. I de fall där du bara behöver visa några få resultat, till exempel om ditt användar gränssnitt eller ditt program-API bara returnerar 10 resultat i taget, kan du också minska sid storleken till 10 för att minska data flödet som används för läsningar och frågor.

    Du kan också ange det önskade sid storleks argumentet för metoden *byPage* , i stället för att ändra fältet rest-huvud direkt. Tänk på att värdet för [x-MS-Max-item-Count](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) eller det önskade sid storleks argumentet för *byPage* endast anger en övre gräns på sid storlek, inte ett absolut krav. Därför kan du se Azure Cosmos DB retur sidor som är mindre än din önskade sid storlek. 

* **Använd lämplig Scheduler (Undvik att stjäla händelse slingor i IO-nättrådar)**

    De asynkrona funktionerna i Azure Cosmos DB Java SDK baseras på [nett](https://netty.io/) icke-blockerande IO. SDK använder ett fast antal upprepnings trådar i IO-nettning (så många processor kärnor som datorn har) för att köra IO-åtgärder. Flödet som returneras av API: t genererar resultatet av en av de uppdelade IO-händelse slingorna. Det är därför viktigt att inte blockera den delade IO-händelsen av en loop. Att utföra processor intensiva arbeten eller blockera åtgärder i Netstore-tråden i IO-händelseloggen kan orsaka död läge eller avsevärt minska SDK-dataflödet.

    Följande kod kör till exempel ett processor intensivt arbete i-tråden för händelse slingor i IO:
    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-noscheduler"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    Mono<CosmosAsyncItemResponse<CustomPOJO>> createItemPub = asyncContainer.createItem(item);
    createItemPub.subscribe(
        itemResponse -> {
            //this is executed on eventloop IO netty thread.
            //the eventloop thread is shared and is meant to return back quickly.
            //
            // DON'T do this on eventloop IO netty thread.
            veryCpuIntensiveWork();                
        });
    ```

    När resultatet har tagits emot, om du vill göra processor intensivt arbete på resultatet bör du undvika att göra det på händelse loop i/o-kontexten. Du kan i stället tillhandahålla din egen schemaläggare att tillhandahålla en egen tråd för att köra ditt arbete, som du ser nedan.

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-scheduler"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    import reactor.core.scheduler.Schedulers;
    Mono<CosmosAsyncItemResponse<CustomPOJO>> createItemPub = asyncContainer.createItem(item);
    createItemPub
        .subscribeOn(Schedulers.elastic())
        .subscribe(
        itemResponse -> {
            //this is executed on eventloop IO netty thread.
            //the eventloop thread is shared and is meant to return back quickly.
            //
            // DON'T do this on eventloop IO netty thread.
            veryCpuIntensiveWork();                
        });
    ```

    Baserat på typen av arbete bör du använda en lämplig befintlig reaktor Scheduler för ditt arbete. Läs här [``Schedulers``](https://projectreactor.io/docs/core/release/api/reactor/core/scheduler/Schedulers.html).

    Mer information om Azure Cosmos DB Java SDK v4 finns i [Cosmos DB Directory för Azure SDK för Java-monorepo på GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cosmos/azure-cosmos).

* **Optimera loggnings inställningarna i ditt program**

    Av olika orsaker kanske du vill eller behöver lägga till loggning i en tråd som genererar hög data flöde för begäran. Om målet är att helt fylla en behållares etablerade data flöde med begär Anden som genereras av den här tråden, kan loggnings optimeringar förbättra prestanda avsevärt.

    * ***Konfigurera en asynkron loggning***

        Svars tiden för en synkron loggning är nödvändigt vis faktorer i den övergripande svars tids beräkningen för tråden för att skapa förfrågningar. En asynkron loggning, till exempel [log4j2](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Flogging.apache.org%2Flog4j%2Flog4j-2.3%2Fmanual%2Fasync.html&data=02%7C01%7CCosmosDBPerformanceInternal%40service.microsoft.com%7C36fd15dea8384bfe9b6b08d7c0cf2113%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637189868158267433&sdata=%2B9xfJ%2BWE%2F0CyKRPu9AmXkUrT3d3uNA9GdmwvalV3EOg%3D&reserved=0) , rekommenderas för att koppla från loggnings resurser från dina program trådar med höga prestanda.

    * ***Inaktivera loggning av netttjänster***

        Loggning av Netstore-bibliotek är chatt och måste stängas av (under tryckning av inloggnings konfigurationen kanske inte räcker) för att undvika ytterligare processor kostnader. Om du inte är i fel söknings läge inaktiverar du nett loggning helt. Så om du använder Log4J för att ta bort de ytterligare CPU-kostnader som ``org.apache.log4j.Category.callAppenders()`` uppstår vid nettning lägger du till följande rad i kodbasen:

        ```java
        org.apache.log4j.Logger.getLogger("io.netty").setLevel(org.apache.log4j.Level.OFF);
        ```

 * **Resurs gräns för öppna filer i operativ systemet**
 
    Vissa Linux-system (till exempel Red Hat) har en övre gräns för antalet öppna filer och så det totala antalet anslutningar. Kör följande för att visa de aktuella gränserna:

    ```bash
    ulimit -a
    ```

    Antalet öppna filer (nofile) måste vara tillräckligt stort för att det ska finnas tillräckligt med plats för den konfigurerade anslutningspoolen och andra öppna filer av operativ systemet. Den kan ändras för att tillåta en större storlek på anslutningspoolen.

    Öppna filen Limits. conf:

    ```bash
    vim /etc/security/limits.conf
    ```
    
    Lägg till/ändra följande rader:

    ```
    * - nofile 100000
    ```

* **Ange partitionsnyckel vid punkt skrivningar**

    Om du vill förbättra prestandan vid punkt skrivningar anger du objektets partitionsnyckel i API-anropet för punkt skrivning, som du ser nedan:

    #### <a name="async"></a>[Async](#tab/api-async)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-createitem-good-async"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    asyncContainer.createItem(item,new PartitionKey(pk),new CosmosItemRequestOptions()).block();
    ```

    #### <a name="sync"></a>[Synkronisera](#tab/api-sync)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-sync-api"></a><a id="java4-createitem-good-sync"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) Sync API

    ```java
    syncContainer.createItem(item,new PartitionKey(pk),new CosmosItemRequestOptions());
    ```

    ---

    i stället för att bara ange objekt instansen, så som visas nedan:

    #### <a name="async"></a>[Async](#tab/api-async)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-createitem-bad-async"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    asyncContainer.createItem(item).block();
    ```

    #### <a name="sync"></a>[Synkronisera](#tab/api-sync)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-sync-api"></a><a id="java4-createitem-bad-sync"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) Sync API

    ```java
    syncContainer.createItem(item);
    ```

    ---

    Den senare stöds men kommer att lägga till svars tid i ditt program. SDK måste parsa objektet och extrahera partitionsnyckel.

## <a name="indexing-policy"></a>Indexeringspolicy
 
* **Utesluta sökvägar som inte används från indexering för att få snabbare skrivning**

    Med Azure Cosmos DB indexerings principen kan du ange vilka dokument Sök vägar som ska tas med eller undantas från indexering genom att använda indexerings Sök vägar (setIncludedPaths och setExcludedPaths). Användningen av indexerings Sök vägar kan ge bättre skriv prestanda och lägre index lagring för scenarier där fråge mönstren är kända i förväg, eftersom indexerings kostnader direkt korreleras med antalet unika sökvägar som indexeras. Följande kod visar till exempel hur du undantar en hel del av dokumenten (kallas även ett under träd) från indexering med jokertecknet "*".

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos"></a><a id="java4-indexing"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos)
    ```java
    Index numberIndex = Index.Range(DataType.Number);
    indexes.add(numberIndex);
    includedPath.setIndexes(indexes);
    includedPaths.add(includedPath);
    indexingPolicy.setIncludedPaths(includedPaths);        
    containerProperties.setIndexingPolicy(indexingPolicy);
    ``` 

    Mer information finns i [Azure Cosmos DB indexerings principer](indexing-policies.md).

## <a name="throughput"></a>Dataflöde
<a id="measure-rus"></a>

* **Mått och justering för lägre enheter för programbegäran/andra användning**

    Azure Cosmos DB erbjuder en omfattande uppsättning databas åtgärder, inklusive relationella och hierarkiska frågor med UDF: er, lagrade procedurer och utlösare – allt som körs på dokumenten i en databas samling. Den kostnad som hör till var och en av dessa operationer varierar baserat på vilken CPU, vilka IO-resurser och hur mycket minne som krävs för att slutföra operationen. I stället för att tänka på och hantera maskin varu resurser kan du tänka på en enhet för begäran (RU) som ett enda mått för de resurser som krävs för att utföra olika databas åtgärder och betjäna en programbegäran.

    Data flödet har allokerats baserat på antalet enheter för [programbegäran](request-units.md) som angetts för varje behållare. Den begärda enhets förbrukningen utvärderas som en taxa per sekund. Program som överskrider den allokerade enhets hastigheten för deras behållare är begränsade tills hastigheten sjunker under den allokerade nivån för behållaren. Om ditt program kräver en högre data flödes nivå kan du öka data flödet genom att tillhandahålla ytterligare enheter för programbegäran.

    Komplexiteten i en fråga påverkar hur många enheter för programbegäran som används för en åtgärd. Antalet predikat, typen av predikat, antalet UDF: er och storleken på käll data uppsättningen påverkar kostnaden för frågor.

    Om du vill mäta omkostnaderna för en åtgärd (skapa, uppdatera eller ta bort) kan du kontrol lera huvudet [x-MS-Request-avgift](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) för att mäta antalet enheter för programbegäran som används av dessa åtgärder. Du kan också titta på motsvarande RequestCharge-egenskap i ResourceResponse\<T> eller FeedResponse\<T>.

    #### <a name="async"></a>[Async](#tab/api-async)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-async-api"></a><a id="java4-request-charge-async"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) asynkront API

    ```java
    CosmosAsyncItemResponse<CustomPOJO> response = asyncContainer.createItem(item).block();

    response.getRequestCharge();
    ```     

    #### <a name="sync"></a>[Synkronisera](#tab/api-sync)

    ### <a name="java-sdk-v4-maven-comazureazure-cosmos-sync-api"></a><a id="java4-request-charge-sync"></a>Java SDK v4 (maven com. Azure:: Azure-Cosmos) Sync API    

    ```java
    CosmosItemResponse<CustomPOJO> response = syncContainer.createItem(item);

    response.getRequestCharge();
    ```     

    ---

    Begär ande avgiften som returnerades i den här rubriken är en bråkdel av ditt etablerade data flöde. Om du till exempel har 2000 RU/s etablerad, och om föregående fråga returnerar 1000 1 KB-dokument, är kostnaden för åtgärden 1000. I och med den här servern bevarar servern bara två sådana begär Anden innan hastigheten begränsar efterföljande begär Anden. Mer information finns i [enheter för programbegäran](request-units.md) och [Kalkylatorn för begär ande](https://www.documentdb.com/capacityplanner)enheter.

<a id="429"></a>
* **Hastighets begränsning/begär ande frekvens för stor**

    När en klient försöker överskrida det reserverade data flödet för ett konto, finns det ingen prestanda försämring på servern och ingen användning av data flödes kapaciteten utöver den reserverade nivån. Servern kommer att förebyggande syfte avsluta begäran med RequestRateTooLarge (HTTP-statuskod 429) och returnera huvudet [x-MS-retry-efter-MS](/rest/api/cosmos-db/common-cosmosdb-rest-request-headers) som anger hur lång tid i millisekunder som användaren måste vänta innan begäran försöker igen.

        HTTP Status 429,
        Status Line: RequestRateTooLarge
        x-ms-retry-after-ms :100

    SDK: erna fångar alla implicita dessa svar, med den server-angivna återförsöket-efter-rubriken och gör om begäran. Om ditt konto inte kan nås samtidigt av flera klienter kommer nästa försök att lyckas.

    Om du har mer än en klient ackumulerad på ett konsekvent sätt över begär ande frekvensen är standard antalet nya försök som för närvarande är 9 internt av klienten inte tillräckligt. i det här fallet genererar klienten en *CosmosClientException* med status kod 429 i programmet. Standard antalet återförsök kan ändras genom att använda setRetryOptions på ConnectionPolicy-instansen. Som standard returneras *CosmosClientException* med status kod 429 efter en ackumulerad vänte tid på 30 sekunder om begäran fortsätter att köras över begär ande frekvensen. Detta inträffar även om det aktuella antalet återförsök är mindre än max antalet försök, måste det vara standardvärdet 9 eller ett användardefinierat värde.

    Även om det automatiserade återförsöket hjälper till att förbättra återhämtning och användbarhet för de flesta program, kan det komma på strider när du gör prestanda mått, särskilt när du mäter svars tid. Den klientbaserade svars tiden kommer att inaktive ras om experimentet träffar Server begränsningen och gör att klient-SDK: n kan försöka tyst igen. För att undvika fördröjningar vid prestanda experiment kan du mäta den avgift som returneras av varje åtgärd och se till att förfrågningarna fungerar under den reserverade begär ande frekvensen. Mer information finns i [enheter för programbegäran](request-units.md).

* **Design för mindre dokument för högre data flöde**

    Begär ande avgiften (bearbetnings kostnaden för begäran) för en specifik åtgärd är direkt korrelerad med dokumentets storlek. Åtgärder i stora dokument kostar mer än åtgärder för små dokument. Det bästa är att skapa ditt program och dina arbets flöden så att objektets storlek är ~ 1 KB eller liknande ordning eller storlek. För fördröjnings känsliga program bör stora objekt undvikas – multi-MB dokument kommer att sakta ned ditt program.

## <a name="next-steps"></a>Nästa steg

Mer information om hur du utformar programmet för skalning och höga prestanda finns i [partitionering och skalning i Azure Cosmos DB](partition-data.md).
