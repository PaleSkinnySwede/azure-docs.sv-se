---
title: Azure Cloud Shell snabb start – bash
description: Lär dig hur du använder bash-kommando raden i din webbläsare med Azure Cloud Shell.
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 03/12/2018
ms.author: damaerte
ms.openlocfilehash: 58d795a5aee79e4149864a79a923ce34950b31d6
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79458077"
---
# <a name="quickstart-for-bash-in-azure-cloud-shell"></a>Snabb start för bash i Azure Cloud Shell

Det här dokumentet innehåller information om hur du använder bash i Azure Cloud Shell i [Azure Portal](https://ms.portal.azure.com/).

> [!NOTE]
> Det finns även en [PowerShell i Azure Cloud Shell](quickstart-powershell.md) snabb start.

## <a name="start-cloud-shell"></a>Starta Cloud Shell
1. Starta **Cloud Shell** från det övre navigerings fältet i Azure Portal. <br>
![](media/quickstart/shell-icon.png)

2. Välj en prenumeration för att skapa ett lagrings konto och Microsoft Azure fil resurs.
3. Välj "skapa lagring"

> [!TIP]
> Du autentiseras automatiskt för Azure CLI i varje session.

### <a name="select-the-bash-environment"></a>Välj bash-miljön
Kontrol lera att den nedrullningsbara miljön i rutan till vänster i Shell-fönstret står `Bash`. <br>
![](media/quickstart/env-selector.png)

### <a name="set-your-subscription"></a>Ange din prenumeration
1. Lista över prenumerationer som du har åtkomst till.
   ```azurecli-interactive
   az account list
   ```

2. Ange önskad prenumeration: <br>
```azurecli-interactive
az account set --subscription 'my-subscription-name'
```

> [!TIP]
> Din prenumeration kommer att sparas i framtida sessioner med `/home/<user>/.azure/azureProfile.json`hjälp av.

### <a name="create-a-resource-group"></a>Skapa en resursgrupp
Skapa en ny resurs grupp i väster med namnet "MyRG".
```azurecli-interactive
az group create --location westus --name MyRG
```

### <a name="create-a-linux-vm"></a>Skapa en virtuell Linux-dator
Skapa en virtuell Ubuntu-dator i din nya resurs grupp. Azure CLI skapar SSH-nycklar och konfigurerar den virtuella datorn med dem. <br>

```azurecli-interactive
az vm create -n myVM -g MyRG --image UbuntuLTS --generate-ssh-keys
```

> [!NOTE]
> Använd `--generate-ssh-keys` instruerar Azure CLI för att skapa och konfigurera offentliga och privata nycklar i din virtuella dator `$Home` och katalog. Som standard placeras nycklar i Cloud Shell på `/home/<user>/.ssh/id_rsa` och. `/home/<user>/.ssh/id_rsa.pub` `.ssh` Mappen finns kvar i den bifogade fil resursens 5 GB-avbildning som används för att `$Home`Spara.

Ditt användar namn på den här virtuella datorn är ditt användar namn som användsUser@Azure:i Cloud Shell ($).

### <a name="ssh-into-your-linux-vm"></a>SSH till din virtuella Linux-dator
1. Sök efter namnet på den virtuella datorn i Azure Portal Sök fältet.
2. Klicka på Anslut för att hämta namnet på den virtuella datorn och den offentliga IP-adressen. <br>
   ![](media/quickstart/sshcmd-copy.png)

3. SSH till den virtuella datorn med `ssh` kommandot cmd.
   ```
   ssh username@ipaddress
   ```

När du har upprättat SSH-anslutningen bör du se Ubuntu Välkommen-prompten. <br>
![](media/quickstart/ubuntu-welcome.png)

## <a name="cleaning-up"></a>Rensa 
1. Avsluta SSH-sessionen.
   ```
   exit
   ```

2. Ta bort resurs gruppen och alla resurser i den.
   ```azurecli-interactive
   az group delete -n MyRG
   ```

## <a name="next-steps"></a>Nästa steg
[Lär dig mer om att spara filer för bash i Cloud Shell](persisting-shell-storage.md) <br>
[Lär dig mer om Azure CLI](https://docs.microsoft.com/cli/azure/) <br>
[Läs mer om Azure Files Storage](../storage/files/storage-files-introduction.md) <br>
