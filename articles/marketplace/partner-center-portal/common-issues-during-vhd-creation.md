---
title: Vanliga problem vid skapande av virtuell hård disk (FAQ)
description: Vanliga frågor och svar om vanliga problem när du skapar en virtuell hård disk (VHD).
author: emuench
ms.author: mingshen
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: guide
ms.date: 04/09/2020
ms.openlocfilehash: a47bece4df7263dbc29ec19510c3ea422bb3f4cf
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81266271"
---
# <a name="common-issues-during-vhd-creation"></a>Vanliga problem vid skapande av virtuell hård disk

> [!NOTE]
> Vi flyttar hanteringen av dina virtuella Azure-erbjudanden från Cloud Partner Portal till Partner Center. Innan du har migrerat dina erbjudanden fortsätter du att följa anvisningarna i [vanliga problem när du skapar en virtuell hård disk (FAQ)](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/virtual-machine/cpp-common-vhd-creation-issues) i Cloud Partner Portal för att hantera dina erbjudanden.

Vanliga frågor och svar (FAQ) omfattar vanliga problem som kan uppstå när du skapar en virtuell hård disk (VHD) för ditt erbjudande för virtuella Azure-datorer.

## <a name="how-do-i-create-a-vm-from-the-azure-portal-using-a-vhd-in-premium-storage"></a>Hur gör jag för att skapa en virtuell dator från Azure Portal med en virtuell hård disk i Premium Storage?

Azure Marketplace har för närvarande inte stöd för att skapa virtuella dator erbjudanden från avbildningar på hanterad lagring eller från Azure Premium Storage. Mer information finns i [Översikt över Azure Managed disks](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview).

## <a name="can-i-use-generation-2-vms-for-offers"></a>Kan jag använda virtuella datorer i generation 2 för erbjudanden?

Nej, endast generation 1-VHD: er stöds. Men vi arbetar för närvarande med Microsoft Azure Platform-teamet för att undersöka stöd för virtuella datorer i generation 2. Mer information finns i [skapa en virtuell dator i generation 1 eller 2 i Hyper-V?](https://docs.microsoft.com/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)

## <a name="how-do-i-change-the-name-of-the-host"></a>Hur gör jag för att ändra namnet på värden?

Du kan inte. När en virtuell dator har skapats kan inte användare (inklusive ägare) uppdatera värd namnet.

## <a name="how-do-i-reset-the-remote-desktop-service-or-its-sign-in-password"></a>Hur gör jag för att återställa fjärr skrivbords tjänsten eller inloggnings lösen ordet?

De här artiklarna förklarar hur du utför RDS-återställning för Windows-och Linux-baserade virtuella datorer:

* [Så här återställer du fjärrskrivbordstjänsten eller dess inloggningslösenord på en virtuell Windows-dator](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-reset-rdp/)
* [Så här återställer du ett Linux VM-lösenord eller SSH-nyckel, reparerar SSH-konfigurationen och kontrollerar disk konsekvens med VMAccess-tillägget](https://azure.microsoft.com/documentation/articles/virtual-machines-linux-classic-reset-access/)

## <a name="how-do-i-generate-new-ssh-certificates"></a>Hur gör jag för att skapa nya SSH-certifikat?

Generering av certifikat förklaras i [Azures avbildnings certifiering för virtuella datorer](https://aks.ms/CertifyVMimage).

## <a name="how-do-i-configure-a-virtual-private-network-vpn-to-work-with-my-vms"></a>Hur gör jag för att konfigurera ett virtuellt privat nätverk (VPN) för att fungera med mina virtuella datorer?

Om du använder Azure Resource Manager distributions modell har du tre alternativ:

* [Skapa en Route-baserad VPN-gateway med hjälp av Azure Portal](https://docs.microsoft.com/azure/vpn-gateway/create-routebased-vpn-gateway-portal)
* [Skapa en Route-baserad VPN-gateway med hjälp av Azure PowerShell](https://docs.microsoft.com/azure/vpn-gateway/create-routebased-vpn-gateway-powershell)
* [Skapa en Route-baserad VPN-gateway med CLI](https://docs.microsoft.com/azure/vpn-gateway/create-routebased-vpn-gateway-cli)

## <a name="what-are-microsoft-support-policies-for-running-microsoft-server-software-on-azure-based-vms"></a>Vad är Microsofts support principer för att köra Microsoft Server-program på Azure-baserade virtuella datorer?

Du hittar information på [Microsoft Server Software support för Microsoft Azure virtuella datorer](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

## <a name="do-virtual-machines-have-unique-identifiers-associated-with-them"></a>Har virtuella datorer unika identifierare som är associerade med dem?

Ja, om det finns på Azure. Azure tilldelar en unik identifierare, som kallas [unikt ID för den virtuella Azure-datorn](https://blogs.msdn.microsoft.com/wasimbloch/2016/10/20/azure-virtual-machine-unique-id/), till varje ny VM-resurs som skapats. Mer information finns i unikt ID för virtuell Azure-dator. Du kan också hämta den här identifieraren via [list-API: et](https://docs.microsoft.com/rest/api/compute/virtualmachines/list).

## <a name="in-a-vm-how-do-i-manage-the-custom-script-extension-in-the-startup-task"></a>Hur hanterar jag det anpassade skript tillägget i en virtuell dator i Start aktiviteten?

Mer information om hur du använder tillägget för anpassat skript med hjälp av Azure PowerShell modul, Azure Resource Manager mallar och fel söknings steg i Windows-system, finns i [anpassat skript tillägg för Windows](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-extensions-customscript/).

## <a name="are-32-bit-applications-or-services-supported-in-azure-marketplace"></a>Är 32-bitars program eller tjänster som stöds i Azure Marketplace?

Nej. I allmänhet inte. De operativ system och standard tjänster som stöds för virtuella Azure-datorer är alla 64-bitars. Även om de flesta 64-bitars operativ system stöder 32-bitars versioner av program för bakåtkompatibilitet, stöds inte användning av 32-bitars program som en del av din VM-lösning och rekommenderas inte. Återskapa ditt program som ett 64-bitars projekt.

Mer information finns i dessa artiklar:

* [Köra 32-bitars program](https://docs.microsoft.com/windows/desktop/WinProg64/running-32-bit-applications)
* [Stöd för 32-bitars operativsystem i Azure Virtual Machines](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines)
* [Microsofts serverprogramsupport för Microsoft Azure Virtual Machines](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)

## <a name="error-vhd-is-already-registered-with-image-repository-as-the-resource"></a>Fel: VHD har redan registrerats med avbildnings lagrings plats som resurs

Varje gång jag försöker skapa en avbildning från mina virtuella hård diskar visas felet "VHD har redan registrerats med avbildnings lagrings plats som resurs" i Azure PowerShell. Jag har inte skapat någon avbildning förut eller kunde inte hitta någon bild med det här namnet i Azure. Hur gör jag för att lösa det?

Det här problemet uppstår vanligt vis om du har skapat en virtuell dator från en virtuell hård disk som har låst den. Bekräfta att det inte finns någon virtuell dator som har allokerats från denna virtuella hård disk och försök sedan igen. Om problemet kvarstår öppnar du ett support ärende. Se [Support för partner Center](https://docs.microsoft.com/azure/marketplace/partner-center-portal/support).
