---
title: Konfigurera Remote Rendering för Unity
description: Så här initierar du Azure Remote rendering i ett Unity-projekt
author: jakrams
ms.author: jakras
ms.date: 02/27/2020
ms.topic: how-to
ms.openlocfilehash: 0415c0e7ee1432521c3cc2026feff5fc2a41d77e
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "80681147"
---
# <a name="set-up-remote-rendering-for-unity"></a>Konfigurera Remote Rendering för Unity

För att aktivera Azure Remote rendering (ARR) i Unity tillhandahåller vi dedikerade metoder som tar hand om vissa delar av Unit-särskilda aspekter.

## <a name="startup-and-shutdown"></a>Start och avstängning

Använd `RemoteManagerUnity`för att initiera fjärrrendering. Den här klassen anropar generiskt `RemoteManager` men implementerar redan uppdelad information åt dig. Enhets systemet använder till exempel ett angivet koordinatsystem. När du `RemoteManagerUnity.Initialize`anropar kommer rätt konvention att konfigureras. Anropet kräver också att du tillhandahåller den Unity-kamera som ska användas för att visa det fjärranslutna innehållet.

```cs
// initialize Azure Remote Rendering for use in Unity:
// it needs to know which camera is used for rendering the scene
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);
```

Anropa `RemoteManagerStatic.ShutdownRemoteRendering()`för att stänga av fjärrrendering.

När en `AzureSession` har skapats och valts som den primära åter givnings sessionen måste den registreras med `RemoteManagerUnity`:

```cs
RemoteManagerUnity.CurrentSession = ...
```

### <a name="full-example-code"></a>Fullständig exempel kod

Koden nedan visar alla steg som krävs för att initiera Azure-fjärrrendering i Unity:

```cs
// initialize Remote Rendering
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);

// create a frontend
AzureFrontendAccountInfo accountInfo = new AzureFrontendAccountInfo();
// ... fill out accountInfo ...
AzureFrontend frontend = new AzureFrontend(accountInfo);

// start a session
AzureSession session = await frontend.CreateNewRenderingSessionAsync(new RenderingSessionCreationParams(RenderingSessionVmSize.Standard, 0, 30)).AsTask();

// let RemoteManagerUnity know about the session we want to use
RemoteManagerUnity.CurrentSession = session;

session.ConnectToRuntime(new ConnectToRuntimeParams());

/// When connected, load and modify content

RemoteManagerStatic.ShutdownRemoteRendering();
```

## <a name="convenience-functions"></a>Bekvämlighets funktioner

### <a name="session-state-events"></a>Händelser i sessionstillstånd

`RemoteManagerUnity.OnSessionUpdate`genererar händelser när dess sessionstillstånd ändras, se kod dokumentationen för mer information.

### <a name="arrserviceunity"></a>ARRServiceUnity

`ARRServiceUnity`är en valfri komponent för att effektivisera installationen och sessionen. Den innehåller alternativ för att stoppa sessionen automatiskt när programmet avslutas eller uppspelnings läget avslutas i redigeraren, samt automatiskt förnya sessionens lån vid behov. Den cachelagrar data, till exempel sessionens egenskaper (se `LastProperties` dess variabel) och exponerar händelser för ändringar i sessionstillstånd och sessions fel.

Det får inte finnas mer än en instans `ARRServiceUnity` av i taget. Den är avsedd för att komma igång snabbare genom att implementera några vanliga funktioner. För ett större program kan det vara bättre att göra dessa saker själv, även om.

Ett exempel på hur du konfigurerar och använder `ARRServiceUnity` finns i [Självstudier: ställa in ett unition-projekt från grunden](../../tutorials/unity/project-setup.md).

## <a name="next-steps"></a>Nästa steg

* [Installera Remote Rendering-paketet för Unity](install-remote-rendering-unity-package.md)
* [Självstudie: Konfigurera ett Unity-projekt från grunden](../../tutorials/unity/project-setup.md)
