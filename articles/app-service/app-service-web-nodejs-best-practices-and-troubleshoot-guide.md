---
title: Metod tips och fel sökning för Node. js
description: Lär dig metod tips och fel söknings steg för Node. js-program som körs i Azure App Service.
author: msangapu-msft
ms.assetid: 387ea217-7910-4468-8987-9a1022a99bef
ms.devlang: nodejs
ms.topic: article
ms.date: 11/09/2017
ms.author: msangapu
ms.custom: seodec18
ms.openlocfilehash: 682884d11b298a97e27056af3c10802dfd410e4c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75430565"
---
# <a name="best-practices-and-troubleshooting-guide-for-node-applications-on-azure-app-service-windows"></a>Metod tips och fel söknings guide för Node-program i Azure App Service Windows

I den här artikeln lär du dig metod tips och fel söknings steg för [Node-program](app-service-web-get-started-nodejs.md) som körs på Azure App Service (med [iisnode](https://github.com/azure/iisnode)).

> [!WARNING]
> Var försiktig när du använder fel söknings steg på produktions platsen. Rekommendation är att felsöka din app på en icke-produktions-installation, till exempel mellanlagringsplatsen och när problemet är löst, byta mellanlagringsplatsen med produktions platsen.
>

## <a name="iisnode-configuration"></a>IISNODE-konfiguration

Den här [schema filen](https://github.com/Azure/iisnode/blob/master/src/config/iisnode_schema_x64.xml) visar alla inställningar som du kan konfigurera för iisnode. Några av de inställningar som är användbara för ditt program:

### <a name="nodeprocesscountperapplication"></a>nodeProcessCountPerApplication

Den här inställningen styr antalet Node-processer som startas per IIS-program. Standardvärdet är 1. Du kan starta så många Node. Exes som din VM vCPU Count genom att ändra värdet till 0. Det rekommenderade värdet är 0 för de flesta program så att du kan använda alla virtuella processorer på din dator. Node. exe är en enkel trådad, så att en Node. exe använder maximalt 1 vCPU. Om du vill få högsta möjliga prestanda från Node-programmet vill du använda alla virtuella processorer.

### <a name="nodeprocesscommandline"></a>nodeProcessCommandLine

Den här inställningen styr sökvägen till Node. exe. Du kan ange det här värdet så att det pekar på Node. exe-versionen.

### <a name="maxconcurrentrequestsperprocess"></a>maxConcurrentRequestsPerProcess

Den här inställningen styr det maximala antalet samtidiga förfrågningar som skickats av iisnode till varje Node. exe. På Azure App Service är standardvärdet oändligt. Du kan konfigurera värdet beroende på hur många förfrågningar appen tar emot och hur snabbt programmet bearbetar varje begäran.

### <a name="maxnamedpipeconnectionretry"></a>maxNamedPipeConnectionRetry

Den här inställningen styr det maximala antalet gånger som iisnode försöker upprätta anslutningen på den namngivna pipe för att skicka begär anden till Node. exe. Den här inställningen i kombination med namedPipeConnectionRetryDelay avgör den totala tids gränsen för varje begäran i iisnode. Standardvärdet är 200 på Azure App Service. Total timeout i sekunder = (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay)/1000

### <a name="namedpipeconnectionretrydelay"></a>namedPipeConnectionRetryDelay

Den här inställningen styr hur lång tid (i MS) iisnode väntar mellan varje nytt försök att skicka begäran till Node. exe över namngiven pipe. Standardvärdet är 250 MS.
Total timeout i sekunder = (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay)/1000

Som standard är den totala tids gränsen i iisnode på Azure App Service 200 \* 250 ms = 50 sekunder.

### <a name="logdirectory"></a>logDirectory

Den här inställningen styr katalogen där iisnode loggar STDOUT/stderr. Standardvärdet är iisnode, som är i förhållande till den huvudsakliga skript katalogen (katalogen där main server. js finns)

### <a name="debuggerextensiondll"></a>debuggerExtensionDll

Den här inställningen styr vilken version av iisnode som används vid fel sökning av Node-programmet. Iisnode-Inspector-0.7.3. dll och iisnode-Inspector. dll är för närvarande de enda två giltiga värdena för den här inställningen. Standardvärdet är iisnode-Inspector-0.7.3. dll. Iisnode-Inspector--0.7.3. dll-versionen använder Node-kontrollant-0.7.3 och använder Web Sockets. Aktivera webb-socketar i Azure webapp och Använd den här versionen. Se <https://ranjithblogs.azurewebsites.net/?p=98> för mer information om hur du konfigurerar iisnode till att använda den nya nodens kontrollant.

### <a name="flushresponse"></a>flushResponse

Standard beteendet för IIS är att den buffrar svars data upp till 4 MB innan de töms, eller till slutet av svaret, beroende på vilket som kommer först. iisnode erbjuder en konfigurations inställning för att åsidosätta det här beteendet: om du vill tömma ett fragment för entiteten svars enhet så snart iisnode tar emot det från Node iisnode/@flushResponse . exe måste du ange attributet i Web. config till ' true ':

```xml
<configuration>
    <system.webServer>
        <!-- ... -->
        <iisnode flushResponse="true" />
    </system.webServer>
</configuration>
```

Om du aktiverar tömningen av varje fragment i bröd texten för svars enheten ökar prestanda genom strömningen i systemet med ~ 5% (från och med v-0.1.13). Det bästa sättet att omfånget för den här inställningen endast till slut punkter som kräver direkt uppspelning av `<location>` svar (till exempel med hjälp av elementet i Web. config)

Förutom detta måste du också ange responseBufferLimit för iisnode-hanteraren till 0 för strömmande program.

```xml
<handlers>
    <add name="iisnode" path="app.js" verb="\*" modules="iisnode" responseBufferLimit="0"/>
</handlers>
```

### <a name="watchedfiles"></a>watchedFiles

En semikolonavgränsad lista över filer som är bevakade för ändringar. Alla ändringar i en fil gör att programmet återanvänds. Varje post består av ett valfritt katalog namn och ett fil namn som krävs, som är relativ till den katalog där huvud program start punkten finns. Jokertecken tillåts endast i fil namns delen. Standardvärdet är `*.js;iisnode.yml`

### <a name="recyclesignalenabled"></a>recycleSignalEnabled

Standardvärdet är false. Om aktive rad kan Node-programmet ansluta till en namngiven pipe (\_miljövariabeln\_IISNODE Control pipe) och skicka ett meddelande om "åter användning". Detta gör att W3wp återanvänds på ett smidigt sätt.

### <a name="idlepageouttimeperiod"></a>idlePageOutTimePeriod

Standardvärdet är 0, vilket innebär att den här funktionen är inaktive rad. När värdet är större än 0, kommer iisnode att visa alla sina underordnade processer varje "idlePageOutTimePeriod" i millisekunder. Se [dokumentationen](/windows/desktop/api/psapi/nf-psapi-emptyworkingset) för att förstå vad en sida betyder. Den här inställningen är användbar för program som använder en hög mängd minne och vill kunna använda minne på disk ibland för att frigöra RAM-minne.

> [!WARNING]
> Var försiktig när du aktiverar följande konfigurations inställningar för produktions program. Rekommendationen är att inte aktivera dem i direktsända produktions program.
>

### <a name="debugheaderenabled"></a>debugHeaderEnabled

Standardvärdet är false. Om värdet är true lägger iisnode till ett HTTP-svarshuvuden `iisnode-debug` till varje http-svar som det `iisnode-debug` skickar huvud-värdet är en URL. Enskilda delar av diagnostikinformation kan fås genom att titta på URL-fragment, men en visualisering är tillgänglig genom att öppna URL: en i en webbläsare.

### <a name="loggingenabled"></a>loggingEnabled

Den här inställningen styr loggningen av STDOUT och stderr från iisnode. Iisnode fångar STDOUT/stderr från noden processer som startas och skriver till den katalog som anges i inställningen "logDirectory". När detta är aktiverat skriver ditt program loggar till fil systemet och beroende på hur mycket loggning som utförs av programmet kan det uppstå prestanda konsekvenser.

### <a name="deverrorsenabled"></a>devErrorsEnabled

Standardvärdet är false. När värdet är true, visar iisnode HTTP-statuskod och Win32-felkoden i webbläsaren. Win32-koden är till hjälp vid fel sökning av vissa typer av problem.

### <a name="debuggingenabled-do-not-enable-on-live-production-site"></a>debuggingEnabled (Aktivera inte på Live producting site)

Den här inställningen styr fel söknings funktionen. Iisnode är integrerat med Node-kontrollanten. Genom att aktivera den här inställningen aktiverar du fel sökning av Node-programmet. När du aktiverar den här inställningen skapar iisnode-filer för Node-kontrollant i "debuggerVirtualDir"-katalogen på den första fel söknings förfrågan till ditt Node-program. Du kan läsa in Node-kontrollen genom att skicka en begäran `http://yoursite/server.js/debug`till. Du kan kontrol lera fel söknings-URL-segmentet med inställningen "debuggerPathSegment". Som standard är debuggerPathSegment = ' debug '. Du kan ställa `debuggerPathSegment` in till ett GUID, till exempel så att det blir svårare att upptäcka andra.

Läs [fel sökning Node. js-program i Windows](https://tomasz.janczuk.org/2011/11/debug-nodejs-applications-on-windows.html) för mer information om fel sökning.

## <a name="scenarios-and-recommendationstroubleshooting"></a>Scenarier och rekommendationer/fel sökning

### <a name="my-node-application-is-making-excessive-outbound-calls"></a>Mitt Node-program gör överdriven utgående samtal

Många program vill göra utgående anslutningar som en del av deras vanliga åtgärd. Till exempel, när en förfrågan kommer i, vill du kontakta en REST API någon annan stans och hämta information för att bearbeta begäran. Du vill använda en Keep Alive-agent när du gör http-eller https-anrop. Du kan använda agentkeepalive-modulen som din Keep Alive-agent när du gör dessa utgående anrop.

Agentkeepalive-modulen säkerställer att socketar återanvänds på den virtuella Azure webapp-datorn. Om du skapar en ny socket på varje utgående begäran läggs resurser till i ditt program. Om du har program åter användnings platser för utgående begär Anden ser du till att programmet inte överskrider maxSockets som allokeras per virtuell dator. Rekommendationen Azure App Service är att ange värdet för agentKeepAlive maxSockets till totalt (4 instanser av Node. exe \* 40 maxSockets/instance) 160 Sockets per virtuell dator.

Exempel på [agentKeepALive](https://www.npmjs.com/package/agentkeepalive) -konfiguration:

```nodejs
let keepaliveAgent = new Agent({
    maxSockets: 40,
    maxFreeSockets: 10,
    timeout: 60000,
    keepAliveTimeout: 300000
});
```

> [!IMPORTANT]
> I det här exemplet förutsätts att du har 4 Node. exe som körs på den virtuella datorn. Om du har ett annat antal Node. exe som körs på den virtuella datorn måste du ändra maxSockets-inställningen enligt detta.
>

#### <a name="my-node-application-is-consuming-too-much-cpu"></a>Mitt Node-program förbrukar för mycket processor

Du kan få rekommendationer från Azure App Service på portalen om hög CPU-förbrukning. Du kan också konfigurera Övervakare för att se vissa [mått](web-sites-monitor.md). När du kontrollerar CPU-användningen på [instrument panelen i Azure-portalen](../azure-monitor/app/web-monitor-performance.md)kontrollerar du Max värdena för CPU så att du inte saknar högsta värden.
Om du tror att ditt program förbrukar för mycket CPU och du inte kan förklara varför kan du profilera ditt Node-program för att ta reda på det.

#### <a name="profiling-your-node-application-on-azure-app-service-with-v8-profiler"></a>Profilering av Node-programmet på Azure App Service med V8-profiler

Anta till exempel att du har en Hello World-app som du vill profilera enligt följande:

```nodejs
const http = require('http');
function WriteConsoleLog() {
    for(let i=0;i<99999;++i) {
        console.log('hello world');
    }
}

function HandleRequest() {
    WriteConsoleLog();
}

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    HandleRequest();
    res.end('Hello world!');
}).listen(process.env.PORT);
```

Gå till fel söknings konsolens webbplats`https://yoursite.scm.azurewebsites.net/DebugConsole`

Gå till platsen/wwwroot-katalogen. Du ser en kommando tolk som visas i följande exempel:

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_install_v8.png)

Kör kommandot `npm install v8-profiler`.

Med det här kommandot installerar du V8-profiler under katalogen\_Node-moduler och alla dess beroenden.
Redigera nu Server. js för att profilera ditt program.

```nodejs
const http = require('http');
const profiler = require('v8-profiler');
const fs = require('fs');

function WriteConsoleLog() {
    for(let i=0;i<99999;++i) {
        console.log('hello world');
    }
}

function HandleRequest() {
    profiler.startProfiling('HandleRequest');
    WriteConsoleLog();
    fs.writeFileSync('profile.cpuprofile', JSON.stringify(profiler.stopProfiling('HandleRequest')));
}

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    HandleRequest();
    res.end('Hello world!');
}).listen(process.env.PORT);
```

Föregående kod profilerar WriteConsoleLog-funktionen och skriver sedan profil resultatet till filen Profile. cpuprofile under platsen wwwroot. Skicka en begäran till ditt program. Du ser en ' Profile. cpuprofile '-fil som skapats under platsens wwwroot.

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/scm_profile.cpuprofile.png)

Ladda ned den här filen och öppna den med Chrome F12-verktyg. Tryck på F12 på Chrome och välj sedan fliken **profiler** . Välj knappen **Läs in** . Välj din profil. cpuprofile-fil som du laddade ned. Klicka på den profil som du nyss läst in.

![](./media/app-service-web-nodejs-best-practices-and-troubleshoot-guide/chrome_tools_view.png)

Du kan se att 95% av tiden har använts av funktionen WriteConsoleLog. Utdata visar också de exakta rad nummer och källfiler som orsakade problemet.

### <a name="my-node-application-is-consuming-too-much-memory"></a>Mitt Node-program förbrukar för mycket minne

Om ditt program förbrukar för mycket minne visas ett meddelande från Azure App Service på portalen om hög minnes användning. Du kan ställa in Övervakare för att se vissa [mått](web-sites-monitor.md). När du kontrollerar minnes användningen på [instrument panelen i Azure-portalen](../azure-monitor/app/web-monitor-performance.md)måste du kontrol lera högsta tillåtna värden för minne så att du inte saknar högsta tillåtna värden.

#### <a name="leak-detection-and-heap-diff-for-nodejs"></a>Läcka avkänning och heap-diff för Node. js

Du kan använda [Node-memwatch](https://github.com/lloyd/node-memwatch) som hjälp för att identifiera minnes läckor.
Du kan installera `memwatch` precis som V8-profiler och redigera koden för att avbilda och diff-heaps för att identifiera minnes läckor i ditt program.

### <a name="my-nodeexes-are-getting-killed-randomly"></a>Min Node. exe avslutas slumpmässigt

Det finns några skäl till varför Node. exe stängs av slumpmässigt:

1. Ditt program utlöser undantag för undantag – check d:\\filen\\\\Logging-errors.\\txt för start logg filen för information om det undantag som har utlösts. Den här filen innehåller stack spårning som hjälper dig att felsöka och åtgärda ditt program.
2. Ditt program förbrukar för mycket minne, vilket påverkar andra processer från att komma igång. Om det totala virtuella dator minnet är nära 100% kan Node. exe avslutas av process hanteraren. Process Manager omsorg vissa processer så att andra processer kan göra lite arbete. Åtgärda det här problemet genom att profilera ditt program för minnes läckor. Om programmet kräver stora mängder minne kan du skala upp till en större virtuell dator (vilket ökar det tillgängliga RAM-minne som är tillgängligt för den virtuella datorn).

### <a name="my-node-application-does-not-start"></a>Mitt Node-program startar inte

Om ditt program returnerar 500-fel när det startar kan det finnas några orsaker:

1. Node. exe finns inte på rätt plats. Kontrol lera nodeProcessCommandLine-inställningen.
2. Huvud skript filen finns inte på rätt plats. Kontrol lera Web. config och se till att namnet på huvud skript filen i avsnittet hanterare matchar huvud skript filen.
3. Konfigurationen av Web. config är felaktig – kontrol lera inställningarna namn/värden.
4. Kall start – ditt program tar för lång tid att starta. Om ditt program tar längre än (maxNamedPipeConnectionRetry \* namedPipeConnectionRetryDelay)/1000 sekunder returnerar iisnode ett 500-fel. Öka värdena för de här inställningarna så att de matchar programmets start tid för att förhindra iisnode från timeout och returnera 500-felet.

### <a name="my-node-application-crashed"></a>Mitt Node-program kraschade

Ditt program utlöser undantag med undantags fel `d:\\home\\LogFiles\\Application\\logging-errors.txt` – kontrol lera om det finns information om det undantag som har utlösts. Den här filen har stack spårning för att hjälpa till att diagnostisera och åtgärda ditt program.

### <a name="my-node-application-takes-too-much-time-to-start-cold-start"></a>Det tar för lång tid att starta en Node-app (kall start)

Den vanligaste orsaken till att start tiderna för långa program är ett stort antal filer i Node\_-modulerna. Programmet försöker läsa in de flesta av de här filerna när du startar. Som standard, eftersom filerna lagras på nätverks resursen på Azure App Service, kan det ta tid att läsa in många filer.
Vissa lösningar för att göra den här processen snabbare är:

1. Se till att du har en plan beroende struktur och inga dubbla beroenden genom att använda npm3 för att installera dina moduler.
2. Försök att använda Lazy för att\_läsa in Node-modulerna och inte läsa in alla moduler vid program start. Om du vill använda Lazy load måste anropet till Kräv (' module ') göras när du faktiskt behöver modulen i funktionen innan du börjar köra en kod i en modul.
3. Azure App Service erbjuder en funktion som kallas lokal cache. Med den här funktionen kopieras innehållet från nätverks resursen till den lokala disken på den virtuella datorn. Eftersom filerna är lokala är inläsnings tiden för Node\_-moduler mycket snabbare.

## <a name="iisnode-http-status-and-substatus"></a>IISNODE http-status och under status

`cnodeconstants` [Käll filen](https://github.com/Azure/iisnode/blob/master/src/iisnode/cnodeconstants.h) visar en lista över alla möjliga status-och under status kombinationer iisnode kan returneras på grund av ett fel.

Aktivera FREB för ditt program för att se Win32-felkoden (se till att du bara aktiverar FREB på webbplatser som inte är produktion av prestanda skäl).

| Http-status | Http-understatus | Möjlig orsak? |
| --- | --- | --- |
| 500 |1000 |Det uppstod ett problem med att skicka begäran till IISNODE – kontrol lera om Node. exe startades. Node. exe kunde krascha vid start. Kontrol lera om Web. config-konfigurationen innehåller fel. |
| 500 |1001 |-Win32Error 0x2-appen svarar inte på URL: en. Kontrol lera reglerna för URL-omskrivning eller kontrol lera om rätt vägar har definierats för Express-appen. -Win32Error 0x6d – namngiven pipe är upptagen – Node. exe accepterar inte begär Anden eftersom pipe är upptagen. Kontrol lera hög CPU-användning. -Andra fel – kontrol lera om Node. exe har kraschat. |
| 500 |1002 |Node. exe kraschade – check d:\\Start\\logg filen\\Logging-errors. txt för stack spårning. |
| 500 |1003 |Problem med pipe-konfiguration – den namngivna pipe-konfigurationen är felaktig. |
| 500 |1004-1018 |Det uppstod ett fel vid sändning av begäran eller bearbetning av svaret till/från Node. exe. Kontrol lera om Node. exe har kraschat. Check d:\\Start\\logg filen\\Logging-errors. txt för stack spårning. |
| 503 |1000 |Det finns inte tillräckligt med minne för att allokera fler namngivna pipe-anslutningar. Kontrol lera varför din app förbrukar så mycket minne. Kontrol lera värdet för maxConcurrentRequestsPerProcess-inställning. Om den inte är oändlig och du har många begär Anden ökar du värdet för att förhindra det här felet. |
| 503 |1001 |Det gick inte att skicka begäran till Node. exe eftersom programmet återvinning. När programmet har återvunnits ska begär Anden behandlas normalt. |
| 503 |1002 |Kontrol lera Win32-felkoden för faktisk orsak – det gick inte att skicka begäran till en Node. exe. |
| 503 |1003 |Namngiven pipe är för upptagen – kontrol lera om Node. exe förbrukar överdriven CPU |

NODE. exe har en inställning som `NODE_PENDING_PIPE_INSTANCES`kallas. På Azure App Service är det här värdet inställt på 5000. Det innebär att Node. exe kan godkänna 5000-begäranden i taget på den namngivna pipe. Det här värdet bör vara tillräckligt tillräckligt för de flesta Node-programmen som körs på Azure App Service. Du bör inte se 503,1003 på Azure App Service på grund av det höga värdet för`NODE_PENDING_PIPE_INSTANCES`

## <a name="more-resources"></a>Fler resurser

Följ dessa länkar om du vill veta mer om Node. js-program på Azure App Service.

* [Kom igång med Node.js-webbappar i Azure App Service](app-service-web-get-started-nodejs.md)
* [Felsöka en Node.js-webbapp i Azure Apptjänst](https://blogs.msdn.microsoft.com/azureossds/2018/08/03/debugging-node-js-apps-on-azure-app-services/)
* [Använda Node.js-moduler med Azure-program](../nodejs-use-node-modules-azure-apps.md)
* [Azure App Service Web Apps: Node.js](https://blogs.msdn.microsoft.com/silverlining/2012/06/14/windows-azure-websites-node-js/)
* [Node.js Developer Center](../nodejs-use-node-modules-azure-apps.md)
* [Utforska superhemliga Kudu-felsökningskonsolen](https://azure.microsoft.com/documentation/videos/super-secret-kudu-debug-console-for-azure-web-sites/)
