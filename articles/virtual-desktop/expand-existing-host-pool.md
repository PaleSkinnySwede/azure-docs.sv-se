---
title: Expandera den befintliga poolen med nya sessionsbaserade värdar – Azure
description: Hur du expanderar en befintlig adresspool med nya sessionsbaserade värdar i Windows Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: d46d5618d7e3dc26775401f4a90d0c98d75ea31a
ms.sourcegitcommit: a6d477eb3cb9faebb15ed1bf7334ed0611c72053
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/08/2020
ms.locfileid: "82929221"
---
# <a name="expand-an-existing-host-pool-with-new-session-hosts"></a>Expandera en befintlig adresspool med nya värdbaserade sessioner

>[!IMPORTANT]
>Det här innehållet gäller för våren 2020-uppdateringen med Azure Resource Manager virtuella Windows Desktop-objekt. Om du använder den virtuella Windows-datorn med version 2019 utan Azure Resource Manager objekt, se [den här artikeln](./virtual-desktop-fall-2019/expand-existing-host-pool-2019.md).
>
> Den virtuella Windows-skrivbordets våren 2020-uppdateringen är för närvarande en offentlig för hands version. Den här för hands versionen tillhandahålls utan service nivå avtal och vi rekommenderar inte att du använder den för produktions arbets belastningar. Vissa funktioner kanske inte stöds eller kan vara begränsade. 
> Mer information finns i [Kompletterande villkor för användning av Microsoft Azure-förhandsversioner](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

När du ramperar användningen i din värd pool kan du behöva expandera den befintliga poolen med nya sessionsbaserade värdar för att hantera den nya belastningen.

I den här artikeln får du lära dig hur du kan expandera en befintlig adresspool med nya sessionsbaserade värdar.

## <a name="what-you-need-to-expand-the-host-pool"></a>Vad du behöver för att expandera poolen

Innan du börjar ska du kontrol lera att du har skapat en adresspool och en sessions värd för virtuella datorer (VM) med någon av följande metoder:

- [Azure Portal](./create-host-pools-azure-marketplace.md)
- [Skapa en värdpool med PowerShell](./create-host-pools-powershell.md)

Du behöver också följande information från första gången du skapade värddatorn och de virtuella datorerna i sessionen:

- Namn på virtuell dator, avbildning och namn
- Autentiseringsuppgifter för domän anslutning
- Namn på virtuellt nätverk och undernät

## <a name="add-virtual-machines-with-the-azure-portal"></a>Lägg till virtuella datorer med Azure Portal

Expandera din värddator genom att lägga till virtuella datorer:

1. Logga in på Azure Portal.

2. Sök efter och välj **virtuellt skriv bord i Windows**.

3. I menyn till vänster på skärmen väljer du **lagringspooler**och väljer sedan namnet på den värddator som du vill lägga till virtuella datorer i.

4. Välj **virtuella datorer** på menyn på vänster sida av skärmen.

5. Välj **+ Lägg** till för att börja skapa din värd pool.

6. Ignorera fliken grundläggande inställningar och välj sedan fliken **VM-information** . Här kan du Visa och redigera information om den virtuella datorn (VM) som du vill lägga till i poolen.

7. Välj den resurs grupp som du vill skapa de virtuella datorerna under och välj sedan regionen. Du kan välja den aktuella region som du använder eller en ny region.
   
8. Ange antalet värdar för sessionen som du vill lägga till i din värddator i **antal virtuella datorer**. Om du till exempel expanderar din värd-pool med fem värdar anger du **5**.
   
    >[!NOTE]
    >Du kan inte redigera storleken eller avbildningen av de virtuella datorerna eftersom det är viktigt att se till att alla virtuella datorer i poolen har samma storlek.
    
9. För information om det **virtuella nätverket**väljer du det virtuella nätverk och undernät som du vill att de virtuella datorerna ska anslutas till. Du kan välja samma virtuella nätverk som befintliga datorer som används eller välja en annan som passar den region som du valde i steg 7.

10. För **Administratörs kontot**anger du Active Directory domän användar namn och lösen ord som är associerat med det virtuella nätverk som du har valt. Dessa autentiseringsuppgifter används för att ansluta de virtuella datorerna till det virtuella nätverket.

      >[!NOTE]
      >Se till att dina administratörs namn överensstämmer med informationen som lämnas här. Och det finns inget MFA aktiverat för kontot.

11. Välj fliken **taggar** om du har några taggar som du vill gruppera de virtuella datorerna med. Annars hoppar du över den här fliken. 

12. Välj fliken **Granska + skapa** . granska dina val och om allt ser bra ut väljer du **skapa**. 

## <a name="next-steps"></a>Nästa steg

Nu när du har expanderat den befintliga poolen kan du logga in på en Windows Virtual Desktop-klient för att testa dem som en del av en användarsession. Du kan ansluta till en session med någon av följande klienter:

- [Ansluta med Windows-skrivbordsklienten](./connect-windows-7-and-10.md)
- [Ansluta med webbklienten](./connect-web.md)
- [Ansluta med Android-klienten](./connect-android.md)
- [Ansluta med macOS-klienten](./connect-macos.md)
- [Ansluta med iOS-klienten](./connect-ios.md)
