---
title: 'Azure frontend-API: er för autentisering'
description: Förklarar hur du använder C#-klient delens API för autentisering
author: florianborn71
ms.author: flborn
ms.date: 02/12/2010
ms.topic: how-to
ms.openlocfilehash: 04296a3dab61fdb569126abc1bc1f975d69e226d
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80681355"
---
# <a name="use-the-azure-frontend-apis-for-authentication"></a>Använda Azure Frontend-API:er för autentisering

I det här avsnittet beskriver vi hur du använder C# API för autentisering.

## <a name="azurefrontendaccountinfo"></a>AzureFrontendAccountInfo

AzureFrontendAccountInfo används för att ställa in Autentiseringsinformationen för en ```AzureFrontend``` instans i SDK.

De viktiga fälten är:

```cs

    public class AzureFrontendAccountInfo
    {
        // Something akin to "<region>.mixedreality.azure.com"
        public string AccountDomain;

        // Can use one of:
        // 1) ID and Key.
        // 2) AuthenticationToken.
        // 3) AccessToken.
        public string AccountId = Guid.Empty.ToString();
        public string AccountKey = string.Empty;
        public string AuthenticationToken = string.Empty;
        public string AccessToken = string.Empty;
    }

```

Använd en [region nära dig](../reference/regions.md)för _region_ delen i domänen.

Konto informationen kan hämtas från portalen enligt beskrivningen i stycket [Hämta konto information](create-an-account.md#retrieve-the-account-information) .

## <a name="azure-frontend"></a>Azure-frontend

Relevanta klasser är ```AzureFrontend``` och ```AzureSession```. ```AzureFrontend```används för konto hanterings-och konto nivå funktioner, bland annat: till gångs konvertering och åter givnings skapande av session. ```AzureSession```används för funktioner på sidnivå och innehåller: uppdatering av sessionen, frågor, förnyande och inaktive ring.

Alla öppna/skapade ```AzureSession``` behåller en referens till den klient som skapade den. För att stänga av datorn måste alla sessioner frigöras innan klient delen frigörs.

Om du avallokerar en session stoppas inte den virtuella `AzureSession.StopAsync` datorn på Azure, måste uttryckligen anropas.

När en session har skapats och dess status har marker ATS som klar, kan den ansluta till körnings miljön för fjärrrendering med `AzureSession.ConnectToRuntime`.

### <a name="threading"></a>Threading

Alla asynkrona anrop för AzureSession och AzureFrontend har slutförts i en bakgrunds tråd, inte huvud program tråden.

### <a name="conversion-apis"></a>API: er för konvertering

Mer information om konverterings tjänsten finns i [modell konverterings REST API](conversion/conversion-rest-api.md).

#### <a name="start-asset-conversion"></a>Starta till gångs konvertering

``` cs
private StartConversionAsync _pendingAsync = null;

void StartAssetConversion(AzureFrontend frontend, string modelName, string modelUrl, string assetContainerUrl)
{
    _pendingAsync = frontend.StartConversionAsync(
        new AssetConversionParams(modelName, modelUrl, assetContainerUrl));
    _pendingAsync.Completed +=
        (StartConversionAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                Console.WriteLine("Failed to start asset conversion!");
            }
        };

        _pendingAsync = null;
}
```

#### <a name="get-conversion-status"></a>Hämta konverterings status

``` cs
private ConversionStatusAsync _pendingAsync = null
void GetConversionStatus(AzureFrontend frontend, string assetId)
{
    _pendingAsync = frontend.GetConversionStatusAsync(assetId);
    _pendingAsync.Completed +=
        (ConversionStatusAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                Console.WriteLine("Failed to get status of asset conversion!");
            }

            _pendingAsync = null;
        };
}
```

### <a name="rendering-apis"></a>API för rendering

Mer information om sessionshantering finns i [REST API för hantering av sessioner](session-rest-api.md) .

En rendering-session kan antingen skapas dynamiskt på tjänsten eller också kan ett redan befintligt sessions-ID öppnas i ett AzureSession-objekt.

#### <a name="create-rendering-session"></a>Skapa åter givnings session

``` cs
private CreateSessionAsync _pendingAsync = null;
void CreateRenderingSession(AzureFrontend frontend, RenderingSessionVmSize vmSize, ARRTimeSpan maxLease)
{
    _pendingAsync = frontend.CreateNewRenderingSessionAsync(
        new RenderingSessionCreationParams(vmSize, maxLease));

    _pendingAsync.Completed +=
        (CreateSessionAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                Console.WriteLine("Failed to create session!");
            }
            _pendingAsync = null;
        };
}
```

#### <a name="open-an-existing-rendering-session"></a>Öppna en befintlig åter givnings session

Att öppna en befintlig session är ett synkront anrop.

``` cs
void CreateRenderingSession(AzureFrontend frontend, string sessionId)
{
    AzureSession session = frontend.OpenRenderingSession(sessionId);
    // Query session status, etc.
}
```

#### <a name="get-current-rendering-sessions"></a>Hämta aktuella åter givnings sessioner

``` cs
private SessionPropertiesArrayAsync _pendingAsync = null;
void GetCurrentRenderingSessions(AzureFrontend frontend)
{
    _pendingAsync = frontend.GetCurrentRenderingSessionsAsync();
    _pendingAsync.Completed +=
        (SessionPropertiesArrayAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                Console.WriteLine("Failed to get current rendering sessions!");
            }
            _pendingAsync = null;
        };
}
```

### <a name="session-apis"></a>Sessioners-API: er

#### <a name="get-rendering-session-properties"></a>Hämta egenskaper för åter givnings session

``` cs
private SessionPropertiesAsync _pendingAsync = null;
void GetRenderingSessionProperties(AzureSession session)
{
    _pendingAsync = session.GetPropertiesAsync();
    _pendingAsync.Completed +=
        (SessionPropertiesAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                //use res.Result
            }
            else
            {
                Console.WriteLine("Failed to get properties of session!");
            }
            _pendingAsync = null;
        };
}
```

#### <a name="update-rendering-session"></a>Uppdatera åter givnings session

``` cs
private SessionAsync _pendingAsync;
void UpdateRenderingSession(AzureSession session, ARRTimeSpan updatedLease)
{
    _pendingAsync = session.RenewAsync(
        new RenderingSessionUpdateParams(updatedLease));
    _pendingAsync.Completed +=
        (SessionAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                Console.WriteLine("Rendering session renewed succeeded!");
            }
            else
            {
                Console.WriteLine("Failed to renew rendering session!");
            }
            _pendingAsync = null;
        };
}
```

#### <a name="stop-rendering-session"></a>Stoppa åter givning av session

``` cs
private SessionAsync _pendingAsync;
void StopRenderingSession(AzureSession session)
{
    _pendingAsync = session.StopAsync();
    _pendingAsync.Completed +=
        (SessionAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                Console.WriteLine("Rendering session stopped successfully!");
            }
            else
            {
                Console.WriteLine("Failed to stop rendering session!");
            }
            _pendingAsync = null;
        };
}
```

#### <a name="connect-to-arr-inspector"></a>Anslut till ARR-kontrollant

``` cs
private ArrInspectorAsync _pendingAsync = null;
void ConnectToArrInspector(AzureSession session, string hostname)
{
    _pendingAsync = session.ConnectToArrInspectorAsync(hostname);
    _pendingAsync.Completed +=
        (ArrInspectorAsync res) =>
        {
            if (res.IsRanToCompletion)
            {
                // Launch the html file with default browser
                string htmlPath = res.Result;
#if WINDOWS_UWP
                UnityEngine.WSA.Application.InvokeOnUIThread(async () =>
                {
                    var file = await Windows.Storage.StorageFile.GetFileFromPathAsync(htmlPath);
                    await Windows.System.Launcher.LaunchFileAsync(file);
                }, true);
#else
                InvokeOnAppThreadAsync(() =>
                {
                    System.Diagnostics.Process.Start("file:///" + htmlPath);
                });
#endif
            }
            else
            {
                Console.WriteLine("Failed to connect to ARR inspector!");
            }
        };
}
```

## <a name="next-steps"></a>Nästa steg

* [Skapa ett konto](create-an-account.md)
* [PowerShell-exempelskript](../samples/powershell-example-scripts.md)
