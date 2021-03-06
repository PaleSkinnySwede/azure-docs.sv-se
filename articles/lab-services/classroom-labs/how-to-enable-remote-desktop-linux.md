---
title: Aktivera fjärr skrivbord för Linux i Azure Lab Services | Microsoft Docs
description: Lär dig hur du aktiverar fjärr skrivbord för virtuella Linux-datorer i ett labb i Azure Lab Services.
services: lab-services
documentationcenter: na
author: spelluru
manager: ''
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/30/2020
ms.author: spelluru
ms.openlocfilehash: dfd16d0977504e598492225238d29c83fcb7a028
ms.sourcegitcommit: acc558d79d665c8d6a5f9e1689211da623ded90a
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82598070"
---
# <a name="enable-remote-desktop-for-linux-virtual-machines-in-a-lab-in-azure-lab-services"></a>Aktivera fjärr skrivbord för virtuella Linux-datorer i ett labb i Azure Lab Services
Den här artikeln visar hur du utför följande uppgifter:

- Aktivera fjärr skrivbord för virtuell Linux-dator
- Hur lärare kan ansluta till mallen VM via Anslutning till fjärrskrivbord (RDP).

## <a name="enable-remote-desktop-for-linux-vm"></a>Aktivera fjärr skrivbord för virtuell Linux-dator
Under labb skapandet kan lärare aktivera **anslutning till fjärr skrivbord** för **Linux** -avbildningar. Alternativet **aktivera anslutning till fjärrskrivbord** visas när en Linux-avbildning väljs för mallen. När det här alternativet är aktiverat kan lärare ansluta till mallen VM och student virtuella datorer via RDP (fjärr skrivbord). 

![Aktivera anslutning till fjärr skrivbord för en Linux-avbildning](../media/how-to-enable-remote-desktop-linux/enable-rdp-option.png)

I rutan **aktivera anslutning till fjärrskrivbord** meddelande väljer du **Fortsätt med fjärr skrivbord**. 

![Aktivera anslutning till fjärr skrivbord för en Linux-avbildning](../media/how-to-enable-remote-desktop-linux/enabling-remote-desktop-connection-dialog.png)

> [!IMPORTANT] 
> När du aktiverar **anslutning till fjärr skrivbord** öppnas **RDP** -porten på Linux-datorer. Om RDP redan har installerats och kon figurer ATS på avbildningen av den virtuella datorn kan du/studenter ansluta till virtuella datorer via RDP utan att följa ytterligare steg.
> 
> Om den virtuella dator avbildningen inte har RDP installerat och konfigurerad, måste du ansluta till Linux-datorn med SSH för första gången och installera RDP-och GUI-paket så att du/studenter kan ansluta till Linux-datorn med RDP senare. Mer information finns i [Installera och konfigurera fjärr skrivbord för att ansluta till en virtuell Linux-dator i Azure](../../virtual-machines/linux/use-remote-desktop.md). Sedan publicerar du avbildningen så att eleverna kan RDP i de virtuella student Linux-datorerna. 

## <a name="supported-operating-systems"></a>Operativsystem som stöds
Anslutning till fjärr skrivbord stöds för följande operativ system:

- openSUSE skottår 42,3
- CentOS-baserad 7,5
- Debian 9 "sträckning"
- Ubuntu Server 16,04 LTS

## <a name="connect-to-the-template-vm"></a>Ansluta till den virtuella malldatorn 
Lärare måste ansluta till mallen VM med SSH först och installera RDP-och GUI-paket på den. Sedan kan lärarna använda RDP för att ansluta till mallen VM: 

1. Om du ser **Anpassa mall** i verktygsfältet väljer du den. Välj sedan **Fortsätt** i dialog rutan **Anpassa mall** . Den här åtgärden startar mallen VM.  

    ![Anpassa mall](../media/how-to-enable-remote-desktop-linux/customize-template.png)
2. När mallen VM har startats kan du välja **Anslut mall** och sedan **ansluta via SSH** i verktygsfältet. 

    ![Anslut till Mall via RDP när labbet har skapats](../media/how-to-enable-remote-desktop-linux/rdp-after-lab-creation.png) 
3. Du ser följande dialog ruta **för att ansluta till din virtuella dator** . Välj **kopierings** knappen bredvid text rutan för att kopiera den till Urklipp. Spara SSH-anslutningssträngen. Använd den här anslutnings strängen från en SSH-Terminal (t. ex. [SparaTillFil](https://www.putty.org/)) för att ansluta till den virtuella datorn.
 
    ![SSH-anslutningssträng](../media/how-to-enable-remote-desktop-linux/ssh-connection-string.png)
4. Installera RDP-och GUI-paket så att du/studenter kan ansluta till Linux-datorn med RDP senare. Mer information finns i [Installera och konfigurera fjärr skrivbord för att ansluta till en virtuell Linux-dator i Azure](../../virtual-machines/linux/use-remote-desktop.md). Sedan publicerar du avbildningen så att eleverna kan RDP i de virtuella student Linux-datorerna.
5. När paketen har installerats kan du använda **mallen Anslut till** i verktygsfältet och sedan välja **Anslut via RDP** för att ansluta till mallen VM via RDP. Spara RDP-filen och Använd den för att ansluta till mallen VM via RDP. 

## <a name="next-steps"></a>Nästa steg
När en instruktör har aktiverat funktionen anslutning till fjärr skrivbord kan eleverna ansluta till sina virtuella datorer via RDP/SSH. Mer information finns i [använda fjärr skrivbord för virtuella Linux-datorer i ett klass rums labb](how-to-use-remote-desktop-linux-student.md). 