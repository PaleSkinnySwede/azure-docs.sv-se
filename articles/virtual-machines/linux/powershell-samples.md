---
title: PowerShell-exempel för Azure Virtual Machine
description: PowerShell-exempel för Azure Virtual Machine
author: cynthn
ms.service: virtual-machines
ms.topic: article
ms.workload: infrastructure
ms.date: 03/01/2019
ms.author: cynthn
ms.openlocfilehash: 8d7db5fe88890b7f807263e50757e637ad808eb1
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81759317"
---
# <a name="azure-virtual-machine-powershell-samples"></a>PowerShell-exempel för Azure Virtual Machine

Följande tabell innehåller länkar till exempel på PowerShell-skript som skapar och hanterar virtuella Linux-datorer.

| | |
|---|---|
|**Skapa virtuella datorer**||
| [Skapa en fullständigt konfigurerad virtuell dator](./../scripts/virtual-machines-linux-powershell-sample-create-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en resurs grupp, virtuell dator och alla relaterade resurser.|
| [Skapa en virtuell dator med Docker aktive rad](./../scripts/virtual-machines-linux-powershell-sample-create-docker-host.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator, konfigurerar den här virtuella datorn som en Docker-värd och kör en NGINX-behållare. |
| [Skapa en virtuell dator och kör konfigurations skript](./../scripts/virtual-machines-linux-powershell-sample-create-vm-nginx.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator och använder Azures anpassade skript tillägg för att installera NGINX. |
| [Skapa en virtuell dator med WordPress installerat](./../scripts/virtual-machines-linux-powershell-sample-create-vm-wordpress.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator och använder Azures anpassade skript tillägg för att installera WordPress. |
| [Skapa en virtuell dator från en hanterad OS-disk](./../scripts/virtual-machines-linux-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator genom att koppla en befintlig hanterad disk som operativ system disk. |
| [Skapa en virtuell dator från en ögonblicks bild](./../scripts/virtual-machines-linux-powershell-sample-create-vm-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator från en ögonblicks bild genom att först skapa en hanterad disk från ögonblicks bilden och sedan koppla den nya hanterade disken som operativ system disk. |
|**Hantera lagring**||
| [Skapa en hanterad disk från en virtuell hård disk i samma eller en annan prenumeration](../scripts/virtual-machines-linux-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en hanterad disk från en specialiserad virtuell hård disk som en OS-disk eller från en data-VHD som en datadisk, i samma eller en annan prenumeration.  |
| [Skapa en hanterad disk från en ögonblicksbild](../scripts/virtual-machines-linux-powershell-sample-create-managed-disk-from-snapshot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en hanterad disk från en ögonblicks bild. |
| [Exportera en ögonblicksbild som en virtuell hårddisk till ett lagringskonto](../scripts/virtual-machines-linux-powershell-sample-copy-snapshot-to-storage-account.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporterar en hanterad ögonblicks bild som en virtuell hård disk till ett lagrings konto i en annan region. |
| [Exportera den virtuella hårddisken för en hanterad disk till ett lagringskonto](../scripts/virtual-machines-linux-powershell-sample-copy-managed-disks-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Exporterar den underliggande virtuella hård disken för en hanterad disk till ett lagrings konto i en annan region. |
| [Skapa en ögonblicksbild från en virtuell hårddisk](../scripts/virtual-machines-linux-powershell-sample-create-snapshot-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en ögonblicks bild från en virtuell hård disk och använder sedan ögonblicks bilden för att skapa flera identiska hanterade diskar snabbt.  |
| [Kopiera en ögonblicks bild till samma eller en annan prenumeration](../scripts/virtual-machines-linux-powershell-sample-copy-snapshot-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Kopierar ögonblicks bild till samma eller en annan prenumeration i samma region som den överordnade ögonblicks bilden. |
|**Övervakning av virtuella datorer**||
| [Övervaka en virtuell dator med Azure Monitor loggar](./../scripts/virtual-machines-linux-powershell-sample-create-vm-oms.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en virtuell dator, installerar Log Analytics agent och registrerar den virtuella datorn i en Log Analytics arbets yta.  |
| [Kopiera en hanterad disk till samma eller en annan prenumeration](../scripts/virtual-machines-linux-powershell-sample-copy-managed-disks-to-same-or-different-subscription.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Kopierar en hanterad disk till samma eller en annan prenumeration som finns i samma region som den överordnade hanterade disken.
| [Samla in information om alla virtuella datorer i en prenumeration med PowerShell](../scripts/virtual-machines-powershell-sample-collect-vm-details.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) | Skapar en CSV som innehåller namnet på den virtuella datorn, resurs gruppens namn, region, Virtual Network, undernät, privat IP-adress, OS-typ och den offentliga IP-adressen för de virtuella datorerna i den angivna prenumerationen.
| | |
