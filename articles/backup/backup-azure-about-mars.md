---
title: Om MARS-agenten
description: Lär dig hur MARS-agenten stöder säkerhets kopierings scenarier
ms.reviewer: srinathv
ms.topic: conceptual
ms.date: 12/02/2019
ms.openlocfilehash: 5656c113a6823a1708854a547b199bd16c521b04
ms.sourcegitcommit: 50ef5c2798da04cf746181fbfa3253fca366feaa
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82611491"
---
# <a name="about-the-microsoft-azure-recovery-services-mars-agent"></a>Om Microsoft Azure Recovery Services (MARS)-agenten

I den här artikeln beskrivs hur Azure Backup tjänsten använder MARS-agenten (Microsoft Azure Recovery Services) för att säkerhetskopiera och återställa filer, mappar och volym eller system tillstånd från en lokal dator till Azure.

MARS-agenten stöder följande säkerhets kopierings scenarier:

![Scenarier med MARS-säkerhetskopiering](./media/backup-try-azure-backup-in-10-mins/backup-scenarios.png)

- **Filer och mappar**: selektivt skydda Windows-filer och mappar.
- **Volym nivå**: skydda en hel Windows-volym på din dator.
- **System nivå**: skydda ett helt Windows-system tillstånd.

MARS-agenten stöder följande återställnings scenarier:

![Scenarier för MARS-återställning](./media/backup-try-azure-backup-in-10-mins/restore-scenarios.png)

- **Samma server**: den server där säkerhets kopian skapades.
  - **Filer och mappar**: Välj de enskilda filer och mappar som du vill återställa.
  - **Volym nivå**: Välj den volym och återställnings punkt som du vill återställa och Återställ den sedan till samma plats eller en annan plats på samma dator.  Skapa en kopia av befintliga filer, Skriv över befintliga filer eller hoppa över att återskapa befintliga filer.
  - **System nivå**: Välj system tillstånd och återställnings punkt för att återställa till samma dator på en angiven plats.

- **Alternativ server**: en annan server än den server där säkerhets kopian gjordes.
  - **Filer och mappar**: Välj de enskilda filerna och mapparna vars återställnings punkt du vill återställa till mål datorn.
  - **Volym nivå**: Välj den volym och återställnings punkt som du vill återställa till en annan plats. Skapa en kopia av befintliga filer, Skriv över befintliga filer eller hoppa över att återskapa befintliga filer.
  - **System nivå**: Välj system tillstånd och återställnings punkt för att återställa som en system tillstånds fil till en annan dator.

## <a name="backup-process"></a>Säkerhetskopieringsprocessen

1. Skapa ett [Recovery Services-valv](install-mars-agent.md#create-a-recovery-services-vault)från Azure Portal och välj filer, mappar och system tillstånd från **säkerhets kopierings målen**.
2. [Ladda ned autentiseringsuppgifterna för Recovery Services valvet och agent installations programmet](https://docs.microsoft.com/azure/backup/install-mars-agent#download-the-mars-agent) till en lokal dator.

3. [Installera agenten](https://docs.microsoft.com/azure/backup/install-mars-agent#install-and-register-the-agent) och Använd de hämtade autentiseringsuppgifterna för valvet för att registrera datorn i Recovery Services-valvet.
4. I agent konsolen på klienten [konfigurerar du säkerhets kopieringen](https://docs.microsoft.com/azure/backup/backup-windows-with-mars-agent#create-a-backup-policy) för att ange vad som ska säkerhets kopie ras, när säkerhets kopieringen ska ske (schemat), hur länge säkerhets kopiorna ska behållas i Azure (bevarande principen) och börja skydda.

![Azure Backup Agent diagram](./media/backup-try-azure-backup-in-10-mins/backup-process.png)

### <a name="additional-information"></a>Ytterligare information

- Den **första säkerhets kopieringen** (första säkerhets kopieringen) körs enligt inställningarna för säkerhets kopiering.  MARS-agenten använder VSS för att ta en ögonblicks bild av de volymer som valts för säkerhets kopiering. Agenten använder bara Windows System Writer-åtgärden för att avbilda ögonblicks bilden. Den använder inte några VSS-skrivare för program och fångar inte in programkonsekventa ögonblicks bilder. När du har tagit ögonblicks bilden med VSS skapar MARS-agenten en virtuell hård disk (VHD) i cache-mappen som du angav när du konfigurerade säkerhets kopian. Agenten lagrar också kontroll summor för varje data block.

- **Stegvisa säkerhets kopieringar** (efterföljande säkerhets kopieringar) körs enligt det schema du anger. Under stegvisa säkerhets kopieringar identifieras ändrade filer och en ny virtuell hård disk skapas. Den virtuella hård disken komprimeras och krypteras och skickas sedan till valvet. När den stegvisa säkerhets kopieringen har slutförts slås den nya virtuella hård disken samman med den virtuella hård disk som skapades efter den inledande replikeringen. Denna sammanlagda virtuella hård disk tillhandahåller det senaste tillstånd som ska användas för att jämföra säkerhets kopior.

- MARS-agenten kan köra säkerhets kopierings jobbet i **optimerat läge** med hjälp av ändrings journalen för USN (Update Sequence Number) eller i **läget ej optimerad** genom att söka efter ändringar i kataloger eller filer genom att genomsöka hela volymen. Icke-optimerat läge är långsammare eftersom agenten måste genomsöka varje fil på volymen och jämföra den mot metadata för att fastställa de ändrade filerna.  Den **första säkerhets kopieringen** körs alltid i icke-optimerat läge. Om den tidigare säkerhets kopieringen misslyckades kommer nästa schemalagda säkerhets kopierings jobb att köras i icke-optimerat läge.

### <a name="additional-scenarios"></a>Fler scenarier

- **Säkerhetskopiera vissa filer och mappar i Azure Virtual Machines**: den primära metoden för att säkerhetskopiera virtuella datorer i Azure är att använda ett Azure Backup-tillägg på den virtuella datorn. Tillägget säkerhetskopierar hela den virtuella datorn. Om du vill säkerhetskopiera vissa filer och mappar i en virtuell dator kan du installera MARS-agenten på de virtuella Azure-datorerna. Mer information finns i [arkitektur: inbyggd virtuell Azure-säkerhetskopiering](https://docs.microsoft.com/azure/backup/backup-architecture#architecture-built-in-azure-vm-backup).

- **Offline-seeding**: inledande fullständiga säkerhets kopieringar av data till Azure överför vanligt vis stora mängder data och kräver mer nätverks bandbredd. Vid efterföljande säkerhets kopieringar överförs endast delta eller stegvis mängd data. Azure Backup komprimerar de första säkerhets kopiorna. Genom processen för *dirigering av dirigering*kan Azure Backup använda diskar för att ladda upp komprimerade första säkerhets kopierings data offline till Azure. Mer information finns i [Azure Backup offline-säkerhetskopiering med Azure Data Box](offline-backup-azure-data-box.md).

## <a name="next-steps"></a>Nästa steg

[Stödmatris för MARS-agenten](https://docs.microsoft.com/azure/backup/backup-support-matrix-mars-agent)

[Vanliga frågor och svar om MARS-agent](https://docs.microsoft.com/azure/backup/backup-azure-file-folder-backup-faq)
