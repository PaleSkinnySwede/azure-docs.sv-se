---
title: Använda en Azure-filresurs med Windows | Microsoft Docs
description: Lär dig hur du använder en Azure-filresurs med Windows och Windows Server.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 06/07/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 2694e0c1536064267faad10517ae58d0709ad1c8
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82231772"
---
# <a name="use-an-azure-file-share-with-windows"></a>Använda en Azure-filresurs med Windows
[Azure Files](storage-files-introduction.md) är Microsofts lättanvända filsystem i molnet. Azure-filresurser kan användas smidigt i Windows och Windows Server. Den här artikeln beskriver överväganden för att använda en Azure-filresurs med Windows och Windows Server.

För att kunna använda en Azure-filresurs utanför den Azure-region som den finns i, till exempel lokalt eller i en annan Azure-region, krävs det att operativsystemet har stöd för SMB 3.0. 

Du kan använda Azure-filresurser i en Windows-installation som körs antingen i en virtuell Azure-dator eller lokalt. Tabellen nedan visar vilka operativsystemversioner som har stöd för användning av filresurser och i vilken miljö:

| Windows-version        | SMB-version | Monteras i Azure VM | Monteras lokalt |
|------------------------|-------------|-----------------------|-----------------------|
| Windows Server 2019 | SMB 3.0 | Ja | Ja |
| Windows 10<sup>1</sup> | SMB 3.0 | Ja | Ja |
| Windows Server, halvårskanal<sup>2</sup> | SMB 3.0 | Ja | Ja |
| Windows Server 2016 | SMB 3.0 | Ja | Ja |
| Windows 8,1 | SMB 3.0 | Ja | Ja |
| Windows Server 2012 R2 | SMB 3.0 | Ja | Ja |
| Windows Server 2012 | SMB 3.0 | Ja | Ja |
| Windows 7<sup>3</sup> | SMB 2.1 | Ja | Nej |
| Windows Server 2008 R2<sup>3</sup> | SMB 2.1 | Ja | Nej |

<sup>1</sup> Windows 10, version 1507, 1607, 1709, 1803, 1809, 1903 och 1909.  
<sup>2</sup> Windows Server, version 1809, 1903 och 1909.  
<sup>3</sup> Normalt Microsoft-Support för Windows 7 och Windows Server 2008 R2 har avslut ATS. Det går bara att köpa ytterligare stöd för säkerhets uppdateringar via [ESU-programmet (Extended Security Update)](https://support.microsoft.com/help/4497181/lifecycle-faq-extended-security-updates). Vi rekommenderar starkt att du migrerar dessa operativ system.

> [!Note]  
> Vi rekommenderar alltid den senaste uppdateringen för din version av Windows.

## <a name="prerequisites"></a>Krav 
* **Lagrings konto namn**: om du vill montera en Azure-filresurs behöver du namnet på lagrings kontot.

* **Lagrings konto nyckel**: om du vill montera en Azure-filresurs behöver du den primära (eller sekundära) lagrings nyckeln. SAS-nycklar stöds inte för montering.

* **Se till att port 445 är öppen**: SMB-protokollet kräver att TCP-port 445 är öppen; anslutningar misslyckas om port 445 är blockerad. Du kan kontrollera om din brandvägg blockerar port 445 med `Test-NetConnection`-cmdleten. Du kan lära dig om [olika sätt att lösa den blockerade port 445 här](https://docs.microsoft.com/azure/storage/files/storage-troubleshoot-windows-file-connection-problems#cause-1-port-445-is-blocked).

    Följande PowerShell-kod förutsätter att du har installerat Azure PowerShell-modulen. mer information finns i [installera Azure PowerShell-modulen](https://docs.microsoft.com/powershell/azure/install-az-ps) . Kom ihåg att ersätta `<your-storage-account-name>` och `<your-resource-group-name>` med gällande namn för ditt lagringskonto.

    ```powershell
    $resourceGroupName = "<your-resource-group-name>"
    $storageAccountName = "<your-storage-account-name>"

    # This command requires you to be logged into your Azure account, run Login-AzAccount if you haven't
    # already logged in.
    $storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName

    # The ComputerName, or host, is <storage-account>.file.core.windows.net for Azure Public Regions.
    # $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as sovereign clouds
    # or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
    Test-NetConnection -ComputerName ([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) -Port 445
    ```

    Om en anslutning upprättades bör du se följande utdata:

    ```
    ComputerName     : <storage-account-host-name>
    RemoteAddress    : <storage-account-ip-address>
    RemotePort       : 445
    InterfaceAlias   : <your-network-interface>
    SourceAddress    : <your-ip-address>
    TcpTestSucceeded : True
    ```

    > [!Note]  
    > Ovanstående kommando returnerar den aktuella IP-adressen för lagringskontot. Det är inte säkert att IP-adressen förblir densamma, och den kan ändras när som helst. Hårdkoda inte den här IP-adressen i några skript eller i en brandväggskonfiguration. 

## <a name="using-an-azure-file-share-with-windows"></a>Använda en Azure-filresurs med Windows
Om du vill använda en Azure-filresurs med Windows måste du antingen montera den, vilket innebär att tilldela den en enhetsbeteckning eller en sökväg för monteringspunkt, eller komma åt den via dess [UNC-sökväg](https://msdn.microsoft.com/library/windows/desktop/aa365247.aspx). 

Till skillnad från andra SMB-resurser som du kanske har använt, till exempel de som finns på en Windows Server, Linux Samba-server eller NAS-enhet, har Azure-filresurser för närvarande inte stöd för Kerberos-autentisering med Active Directory-identitet (AD) eller Azure Active Directory-identitet (AAD), men det är en funktion som vi [arbetar med](https://feedback.azure.com/forums/217298-storage/suggestions/6078420-acl-s-for-azurefiles). I stället måste du komma åt din Azure-filresurs med lagringskontonyckeln för det lagringskonto som innehåller Azure-filresursen. En lagrings konto nyckel är en administratörs nyckel för ett lagrings konto, inklusive administratörs behörighet till alla filer och mappar i fil resursen som du ansluter till, samt för alla fil resurser och andra lagrings resurser (blobbar, köer, tabeller osv.) som finns i ditt lagrings konto. Om detta inte är tillräckligt för din arbetsbelastning kan [Azure File Sync](storage-sync-files-planning.md) åtgärda bristen på Kerberos-autentisering och ACL-stöd under tiden tills stöd för AAD-baserad Kerberos-autentisering och ACL blir tillgängliga offentligt.

Ett vanligt mönster för lyftning och skiftande av verksamhetsspecifika program som förväntar sig en SMB-filresurs till Azure är att använda en Azure-filresurs som ett alternativ till att köra en dedikerad Windows-filserver i en Azure-dator. En viktig aspekt för en lyckad migrering av ett verksamhetsspecifikt program till att använda en Azure-filresurs är att många verksamhetsspecifika program kör i kontexten för ett dedikerat tjänstkonto med begränsade systembehörigheter i stället för den virtuella datorns administratörskonto. Därför måste du se till att du monterar/sparar autentiseringsuppgifterna för Azure-filresursen från kontexten för tjänstkontot i stället för ditt administratörskonto.

### <a name="persisting-azure-file-share-credentials-in-windows"></a>Spara autentiseringsuppgifterna för Azure-filresurser i Windows  
Med [cmdkey](https://docs.microsoft.com/windows-server/administration/windows-commands/cmdkey)-verktyget kan du lagra autentiseringsuppgifterna för lagringskontot i Windows. Det innebär att när du försöker få åtkomst till en Azure-filresurs via dess UNC-sökväg eller montera Azure-filresursen så behöver du inte ange autentiseringsuppgifter. För att spara autentiseringsuppgifterna för ditt lagringskonto kör du följande PowerShell-kommandon och ersätter `<your-storage-account-name>` och `<your-resource-group-name>` där det är lämpligt.

```powershell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"

# These commands require you to be logged into your Azure account, run Login-AzAccount if you haven't
# already logged in.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$storageAccountKeys = Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName

# The cmdkey utility is a command-line (rather than PowerShell) tool. We use Invoke-Expression to allow us to 
# consume the appropriate values from the storage account variables. The value given to the add parameter of the
# cmdkey utility is the host address for the storage account, <storage-account>.file.core.windows.net for Azure 
# Public Regions. $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as sovereign 
# clouds or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
Invoke-Expression -Command ("cmdkey /add:$([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) " + `
    "/user:AZURE\$($storageAccount.StorageAccountName) /pass:$($storageAccountKeys[0].Value)")
```

Du kan verifiera att cmdkey-verktyget har lagrat autentiseringsuppgifterna för lagringskontot med hjälp av listparametern:

```powershell
cmdkey /list
```

Om autentiseringsuppgifterna för din Azure-filresurs har lagrats korrekt är de utdata som förväntas enligt följande (det kan finnas ytterligare nycklar som lagras i listan):

```
Currently stored credentials:

Target: Domain:target=<storage-account-host-name>
Type: Domain Password
User: AZURE\<your-storage-account-name>
```

Du bör nu kunna montera eller få åtkomst till resursen utan att behöva ange ytterligare autentiseringsuppgifter.

#### <a name="advanced-cmdkey-scenarios"></a>Avancerade cmdkey-scenarier
Det finns två ytterligare scenarier att överväga med cmdkey: lagra autentiseringsuppgifter för en annan användare på datorn, till exempel ett tjänstkonto, och lagra autentiseringsuppgifter på en fjärrdator med hjälp av PowerShell-fjärrkommunikation.

Det är enkelt att lagra autentiseringsuppgifterna för en annan användare på datorn: när du är inloggad på ditt konto kör du bara följande PowerShell-kommando:

```powershell
$password = ConvertTo-SecureString -String "<service-account-password>" -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "<service-account-username>", $password
Start-Process -FilePath PowerShell.exe -Credential $credential -LoadUserProfile
```

Då öppnas ett nytt PowerShell-fönster under användarkontexten för ditt tjänstkonto (eller ett användarkonto). Du kan sedan använda cmdkey-verktyget enligt beskrivningen [ovan](#persisting-azure-file-share-credentials-in-windows).

Däremot går det inte att lagra autentiseringsuppgifterna på en fjärrdator med hjälp av PowerShell-fjärrkommunikation, eftersom cmdkey inte tillåter åtkomst, även för tillägg, till dess autentiseringsuppgiftsarkiv när användaren är inloggad via PowerShell-fjärrkommunikation. Vi rekommenderar inloggning på datorn via [Fjärrskrivbord](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/clients/windows).

### <a name="mount-the-azure-file-share-with-powershell"></a>Montera Azure-filresursen med PowerShell
Kör följande kommandon från en vanlig PowerShell-session (inte en upphöjd) för att montera Azure-filresursen. Kom ihåg att ersätta `<your-resource-group-name>`, `<your-storage-account-name>`, `<your-file-share-name>` och `<desired-drive-letter>` med rätt information.

```powershell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"
$fileShareName = "<your-file-share-name>"

# These commands require you to be logged into your Azure account, run Login-AzAccount if you haven't
# already logged in.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName
$storageAccountKeys = Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $storageAccountName
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object { 
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    throw [System.Exception]::new("Azure file share not found")
}

# The value given to the root parameter of the New-PSDrive cmdlet is the host address for the storage account, 
# <storage-account>.file.core.windows.net for Azure Public Regions. $fileShare.StorageUri.PrimaryUri.Host is 
# used because non-Public Azure regions, such as sovereign clouds or Azure Stack deployments, will have different 
# hosts for Azure file shares (and other storage resources).
$password = ConvertTo-SecureString -String $storageAccountKeys[0].Value -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential -ArgumentList "AZURE\$($storageAccount.StorageAccountName)", $password
New-PSDrive -Name <desired-drive-letter> -PSProvider FileSystem -Root "\\$($fileShare.StorageUri.PrimaryUri.Host)\$($fileShare.Name)" -Credential $credential -Persist
```

> [!Note]  
> Vid användning av alternativet `-Persist` på `New-PSDrive`-cmdleten kan filresursen endast återmonteras vid start om autentiseringsuppgifterna sparas. Du kan spara autentiseringsuppgifterna med hjälp av cmdkey enligt [tidigare beskrivning](#persisting-azure-file-share-credentials-in-windows). 

Om du vill kan du demontera Azure-filresursen med hjälp av följande PowerShell-cmdlet.

```powershell
Remove-PSDrive -Name <desired-drive-letter>
```

### <a name="mount-the-azure-file-share-with-file-explorer"></a>Montera Azure-filresursen med Utforskaren
> [!Note]  
> Observera att följande instruktioner visas på Windows 10 och kan skilja sig en aning på äldre versioner. 

1. Öppna Utforskaren. Du kan göra detta genom att öppna från startmenyn eller använda tangentbordsgenvägen Win+E.

1. Navigera till objektet **Den här datorn** på vänster sida av fönstret. Detta ändrar menyerna i menyfliksområdet. Under menyn Dator väljer du **Anslut nätverksenhet**.
    
    ![En skärmbild av den nedrullningsbara menyn "Anslut nätverksenhet"](./media/storage-how-to-use-files-windows/1_MountOnWindows10.png)

1. Välj enhets bokstaven och ange UNC-sökvägen. UNC-sökvägarna är `<storageAccountName>.file.core.windows.net/<fileShareName>`. Till exempel: `anexampleaccountname.file.core.windows.net/example-share-name`.
    
    ![En skärmbild av dialogrutan "Anslut nätverksenhet"](./media/storage-how-to-use-files-windows/2_MountOnWindows10.png)

1. Använd det lagringskontonamn som börjar med `AZURE\` som användarnamn och en lagringskontonyckel som lösenord.
    
    ![En skärmbild av dialogrutan med nätverksautentiseringsuppgifter](./media/storage-how-to-use-files-windows/3_MountOnWindows10.png)

1. Använd Azure-filresurser som du vill.
    
    ![Azure-filresursen är nu monterad](./media/storage-how-to-use-files-windows/4_MountOnWindows10.png)

1. När du är redo att demontera Azure-filresursen kan du göra det genom att högerklicka på posten för resursen under **Nätverksplatser** i Utforskaren och välja **Koppla från**.

### <a name="accessing-share-snapshots-from-windows"></a>Komma åt ögonblicksbilder av resurser från Windows
Om du har tagit en resursögonblicksbild, antingen manuellt eller automatiskt via ett skript eller en tjänst som Azure Backup, kan du visa tidigare versioner av en resurs, en katalog eller en viss fil från filresursen i Windows. Du kan ta en ögonblicks bild av en resurs från [Azure Portal](storage-how-to-use-files-portal.md), [Azure POWERSHELL](storage-how-to-use-files-powershell.md)och [Azure CLI](storage-how-to-use-files-cli.md).

#### <a name="list-previous-versions"></a>Lista över tidigare versioner
Bläddra till det objekt eller överordnade objekt som behöver återställas. Dubbelklicka för att gå till den önskade katalogen. Högerklicka och välj **Egenskaper** på menyn.

![Snabbmeny för en vald katalog](./media/storage-how-to-use-files-windows/snapshot-windows-previous-versions.png)

Välj **Tidigare versioner** för att visa en lista över resursögonblicksbilder för den här katalogen. Det kan ta några sekunder att läsa in listan, beroende på nätverkshastigheten och antalet resursögonblicksbilder i katalogen.

![Fliken Tidigare versioner](./media/storage-how-to-use-files-windows/snapshot-windows-list.png)

Du kan välja **Öppna** om du vill öppna en viss ögonblicksbild. 

![Öppnad ögonblicksbild](./media/storage-how-to-use-files-windows/snapshot-browse-windows.png)

#### <a name="restore-from-a-previous-version"></a>Återställa från en tidigare version
Välj **Återställ** för att rekursivt kopiera innehållet i hela katalogen vid tidpunkten då resursögonblicksbilden skapades till den ursprungliga platsen.

 ![Återställningsknapp i varningsmeddelande](./media/storage-how-to-use-files-windows/snapshot-windows-restore.png) 

## <a name="securing-windowswindows-server"></a>Skydda Windows/Windows Server
För att du ska kunna montera en Azure-filresurs i Windows måste port 445 vara tillgänglig. Många organisationer blockerar port 445 på grund av säkerhetsrisker med SMB 1. SMB 1, även kallat CIFS (Common Internet File System), är ett äldre filsystemprotokoll som medföljer Windows och Windows Server. SMB 1 är ett inaktuellt, ineffektivt och framför allt oskyddat protokoll. Den goda nyheten är att Azure Files inte har stöd för SMB 1, och alla versioner av Windows och Windows Server som stöds gör det möjligt att ta bort eller inaktivera SMB 1. Vi [rekommenderar starkt](https://aka.ms/stopusingsmb1) att du alltid tar bort eller inaktiverar SMB 1-klienten och -servern i Windows innan du använder Azure-filresurser i produktion.

I följande tabell finns detaljerad information om status för SMB 1 i varje version av Windows:

| Windows-version                           | SMB 1-standardstatus | Inaktivera/ta bort metod       | 
|-------------------------------------------|----------------------|-----------------------------|
| Windows Server 2019                       | Disabled             | Ta bort med Windows-funktionen |
| Windows Server, versioner 1709+            | Disabled             | Ta bort med Windows-funktionen |
| Windows 10, versioner 1709+                | Disabled             | Ta bort med Windows-funktionen |
| Windows Server 2016                       | Enabled              | Ta bort med Windows-funktionen |
| Windows 10, versionerna 1507, 1607 och 1703 | Enabled              | Ta bort med Windows-funktionen |
| Windows Server 2012 R2                    | Enabled              | Ta bort med Windows-funktionen | 
| Windows 8,1                               | Enabled              | Ta bort med Windows-funktionen | 
| Windows Server 2012                       | Enabled              | Inaktivera med registret       | 
| Windows Server 2008 R2                    | Enabled              | Inaktivera med registret       |
| Windows 7                                 | Enabled              | Inaktivera med registret       | 

### <a name="auditing-smb-1-usage"></a>Granskning av SMB 1-användning
> Gäller för Windows Server 2019, Windows Server halvårs kanal (version 1709 och 1803), Windows Server 2016, Windows 10 (version 1507, 1607, 1703, 1709 och 1803), Windows Server 2012 R2 och Windows 8,1

Innan du tar bort SMB 1 i din miljö kan det vara bra att granska användning av SMB 1 för att se om några klienter kommer att störas av ändringen. Om alla begäranden görs mot SMB-resurser med SMB 1 loggas en granskningshändelse i händelseloggen i `Applications and Services Logs > Microsoft > Windows > SMBServer > Audit`. 

> [!Note]  
> Om du vill aktivera stöd för granskning på Windows Server 2012 R2 och Windows 8.1 installerar du minst [KB4022720](https://support.microsoft.com/help/4022720/windows-8-1-windows-server-2012-r2-update-kb4022720).

Du aktiverar granskning genom att köra följande cmdlet från en upphöjd PowerShell-session:

```powershell
Set-SmbServerConfiguration –AuditSmb1Access $true
```

### <a name="removing-smb-1-from-windows-server"></a>Ta bort SMB 1 från Windows Server
> Gäller för Windows Server 2019, halvårs kanal för Windows Server (version 1709 och 1803), Windows Server 2016, Windows Server 2012 R2

Om du vill ta bort SMB 1 från en Windows Server-instans kör du följande cmdlet från en upphöjd PowerShell-session:

```powershell
Remove-WindowsFeature -Name FS-SMB1
```

Starta om servern för att slutföra borttagningsprocessen. 

> [!Note]  
> Från och med Windows 10 och Windows Server version 1709 installeras inte SMB 1 som standard, och det har separata Windows-funktioner för SMB 1-klienten och SMB 1-servern. Vi rekommenderar att du alltid lämnar både SMB 1-servern (`FS-SMB1-SERVER`) och SMB 1-klienten (`FS-SMB1-CLIENT`) avinstallerade.

### <a name="removing-smb-1-from-windows-client"></a>Ta bort SMB 1 från Windows-klient
> Gäller för Windows 10 (versionerna 1507, 1607, 1703, 1709 och 1803) samt Windows 8.1

Om du vill ta bort SMB 1 från din Windows-klient kör du följande cmdlet från en upphöjd PowerShell-session:

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

Starta om datorn för att slutföra borttagningsprocessen.

### <a name="disabling-smb-1-on-legacy-versions-of-windowswindows-server"></a>Inaktivera SMB 1 på äldre versioner av Windows/Windows Server
> Gäller för Windows Server 2012, Windows Server 2008 R2 och Windows 7

SMB 1 kan inte tas bort helt och hållet i äldre versioner av Windows/Windows Server, men det kan inaktiveras via registret. Om du vill inaktivera SMB 1 skapar du en ny registernyckel `SMB1` av typen `DWORD` med värdet `0` i `HKEY_LOCAL_MACHINE > SYSTEM > CurrentControlSet > Services > LanmanServer > Parameters`.

Det är även enkelt att göra detta med följande PowerShell-cmdlet:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" SMB1 -Type DWORD -Value 0 –Force
```

När du har skapat den här registernyckeln måste du starta om servern för att inaktivera SMB 1.

### <a name="smb-resources"></a>SMB-resurser
- [Sluta använda SMB 1](https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/)
- [Clearinghouse för SMB 1-produkt](https://blogs.technet.microsoft.com/filecab/2017/06/01/smb1-product-clearinghouse/)
- [Identifiera SMB 1 i din miljö med DSCEA](https://blogs.technet.microsoft.com/ralphkyttle/2017/04/07/discover-smb1-in-your-environment-with-dscea/)
- [Inaktivera SMB 1 via en grupprincip](https://blogs.technet.microsoft.com/secguide/2017/06/15/disabling-smbv1-through-group-policy/)

## <a name="next-steps"></a>Nästa steg
Mer information om Azure Files finns på följande länkar:
- [Planera för distribution av Azure Files](storage-files-planning.md)
- [VANLIGA FRÅGOR OCH SVAR](../storage-files-faq.md)
- [Felsökning i Windows](storage-troubleshoot-windows-file-connection-problems.md)      
