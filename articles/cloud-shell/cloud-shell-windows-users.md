---
title: Azure Cloud Shell för Windows-användare | Microsoft Docs
description: Guide för användare som inte är bekanta med Linux-system
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/03/2018
ms.author: damaerte
ms.openlocfilehash: 4fc4f6523eb19294cabdf6b5b910dd346a877502
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "67204147"
---
# <a name="powershell-in-azure-cloud-shell-for-windows-users"></a>PowerShell i Azure Cloud Shell för Windows-användare

I maj 2018 [annonserades](https://azure.microsoft.com/blog/pscloudshellrefresh/) ändringarna till PowerShell i Azure Cloud Shell.
PowerShell-upplevelsen i Azure Cloud Shell kör nu [PowerShell Core 6](https://github.com/powershell/powershell) i en Linux-miljö.
Med den här ändringen kan det finnas skillnader i PowerShell-upplevelsen i Cloud Shell jämfört med vad som förväntas i Windows PowerShell-upplevelsen.

## <a name="file-system-case-sensitivity"></a>Skift läges känslighet för fil system

Fil systemet är Skift läges okänsligt i Windows, men i Linux är fil systemet Skift läges känsligt.
Tidigare `file.txt` och `FILE.txt` ansågs vara samma fil, men nu anses de vara olika filer.
Rätt Skift läge måste användas `tab-completing` i fil systemet.
PowerShell-vissa upplevelser, till `tab-completing` exempel cmdlet-namn, parametrar och värden, är inte Skift läges känsliga.

## <a name="windows-powershell-aliases-vs-linux-utilities"></a>Windows PowerShell-alias vs Linux-verktyg

Vissa befintliga PowerShell-alias har samma namn som inbyggda Linux-kommandon, till `cat`exempel,`ls` `sort`,, `sleep`, osv. I PowerShell Core 6 har alias som kolliderar med inbyggda Linux-kommandon tagits bort.
Nedan visas vanliga alias som har tagits bort och motsvarande kommandon:  

|Borttaget alias   |Motsvarande kommando   |
|---|---|
|`cat`    | `Get-Content` |
|`curl`   | `Invoke-WebRequest` |
|`diff`   | `Compare-Object` |
|`ls`     | `dir` <br> `Get-ChildItem` |
|`mv`     | `Move-Item`   |
|`rm`     | `Remove-Item` |
|`sleep`  | `Start-Sleep` |
|`sort`   | `Sort-Object` |
|`wget`   | `Invoke-WebRequest` |

## <a name="persisting-home"></a>Bevara $HOME

Tidigare användare kunde bara spara skript och andra filer i moln enheten.
Nu sparas användarens $HOME katalog också över flera sessioner.

## <a name="powershell-profile"></a>PowerShell-profil

Som standard skapas inte en användares PowerShell-profil.
Skapa din profil genom att skapa en `PowerShell` katalog under `$HOME/.config`.

```azurepowershell-interactive
mkdir (Split-Path $profile.CurrentUserAllHosts)
```

Under `$HOME/.config/PowerShell`kan du skapa dina profilmappar – `profile.ps1` och/eller. `Microsoft.PowerShell_profile.ps1`

## <a name="whats-new-in-powershell-core-6"></a>Vad är nytt i PowerShell Core 6

Om du vill ha mer information om vad som är nytt i PowerShell Core 6 refererar du till [PowerShell-dokumenten](https://docs.microsoft.com/powershell/scripting/whats-new/what-s-new-in-powershell-core-60?view=powershell-6) och blogg inlägget [komma igång med PowerShell Core](https://blogs.msdn.microsoft.com/powershell/2017/06/09/getting-started-with-powershell-core-on-windows-mac-and-linux/) .
