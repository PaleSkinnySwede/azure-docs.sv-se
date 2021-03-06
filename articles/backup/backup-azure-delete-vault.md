---
title: Ta bort ett Microsoft Azure Recovery Services-valv
description: I den här artikeln lär du dig hur du tar bort beroenden och sedan tar bort ett Azure Backup Recovery Services-valv.
ms.topic: conceptual
ms.date: 09/20/2019
ms.openlocfilehash: 5fcf8004cd5792b30ec57537d5d8ab0bc085dfb3
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82183763"
---
# <a name="delete-an-azure-backup-recovery-services-vault"></a>Ta bort ett Azure Backup Recovery Services-valv

I den här artikeln beskrivs hur du tar bort ett [Azure Backup](backup-overview.md) Recovery Services-valv. Den innehåller instruktioner för att ta bort beroenden och sedan ta bort ett valv.

## <a name="before-you-start"></a>Innan du börjar

Du kan inte ta bort ett Recovery Services-valv som har beroenden, till exempel skyddade servrar eller säkerhets kopierings hanterings servrar som är kopplade till den.

- Det går inte att ta bort valv som innehåller säkerhets kopierings data (även om du har stoppat skyddet men behåller säkerhets kopierings data).

- Om du tar bort ett valv som innehåller beroenden visas följande meddelande:

  ![Ta bort valvet-felet.](./media/backup-azure-delete-vault/error.png)

- Om du tar bort ett lokalt skyddat objekt från en portal som innehåller beroenden visas ett varnings meddelande:

  ![Ta bort det skyddade Server felet.](./media/backup-azure-delete-vault/error-message.jpg)

- Om säkerhets kopierings objekt är i tyst borttaget läge nedan visas varnings meddelandet och du måste vänta tills de tas bort permanent. Mer information finns i den här [artikeln](https://docs.microsoft.com/azure/backup/backup-azure-security-feature-cloud).

   ![Ta bort valvet-felet.](./media/backup-azure-delete-vault/error-message-soft-delete.png)

- Det går inte att ta bort valv som har registrerade lagrings konton. Information om hur du avregistrerar kontot finns i [avregistrera ett lagrings konto](manage-afs-backup.md#unregister-a-storage-account).
  
Om du vill ta bort valvet väljer du det scenario som matchar konfigurationen och följer de rekommenderade stegen:

Scenario | Steg för att ta bort beroenden för att ta bort valv |
-- | --
Jag har lokala filer och mappar som skyddas med hjälp av Azure Backup agenten, säkerhetskopiera till Azure | Utför stegen i [ta bort säkerhets kopierings objekt från mars Management Console](#delete-backup-items-from-the-mars-management-console)
Jag har lokala datorer som skyddas med hjälp av MABS (Microsoft Azure Backup Server) eller DPM (System Center Data Protection Manager) till Azure | Utför stegen i [ta bort säkerhets kopierings objekt från hanterings konsolen för Mabs](#delete-backup-items-from-the-mabs-management-console)
Jag har skyddat objekt i molnet (till exempel en virtuell laaS-dator eller en Azure Files-resurs)  | Utför stegen i [ta bort skyddade objekt i molnet](#delete-protected-items-in-the-cloud)
Jag har skyddade objekt både lokalt och i molnet | Utför stegen i följande avsnitt i följande ordning: <br> 1. [ta bort skyddade objekt i molnet](#delete-protected-items-in-the-cloud)<br> 2. [ta bort säkerhets kopierings objekt från mars Management Console](#delete-backup-items-from-the-mars-management-console) <br> 3. [ta bort säkerhets kopierings objekt från hanterings konsolen för Mabs](#delete-backup-items-from-the-mabs-management-console)
Jag har inga skyddade objekt lokalt eller i molnet. men jag får fortfarande valvet borttagnings fel | Utför stegen i [ta bort Recovery Services valvet med Azure Resource Manager](#delete-the-recovery-services-vault-by-using-azure-resource-manager) <br><br> Se till att inga lagrings konton har registrerats med valvet. Information om hur du avregistrerar kontot finns i [avregistrera ett lagrings konto](manage-afs-backup.md#unregister-a-storage-account).

## <a name="delete-protected-items-in-the-cloud"></a>Ta bort skyddade objekt i molnet

Börja med att läsa avsnittet **[innan du börjar](#before-you-start)** och förstå borttagnings processen för beroenden och valvet.

Utför följande steg för att stoppa skyddet och ta bort säkerhetskopierade data:

1. Gå till **Recovery Services valv**från portalen och gå sedan till **säkerhets kopierings objekt**. Välj sedan de skyddade objekten i molnet (till exempel Azure Virtual Machines Azure Storage [tjänsten Azure Files] eller SQL Server på Azure Virtual Machines).

    ![Välj typ av säkerhets kopiering.](./media/backup-azure-delete-vault/azure-storage-selected.png)

2. Högerklicka för att välja säkerhets kopierings objekt. Beroende på om objektet för säkerhets kopian är skyddat eller inte, visar menyn antingen rutan **stoppa säkerhets kopiering** eller fönstret **ta bort säkerhets kopierings data** .

    - Om fönstret **stoppa säkerhets kopiering** visas väljer du **ta bort säkerhets kopierings data** på den nedrullningsbara menyn. Ange namnet på det säkerhetskopierade objektet (det här fältet är Skift läges känsligt) och välj sedan en orsak i den nedrullningsbara menyn. Ange dina kommentarer, om du har några. Välj sedan **stoppa säkerhets kopiering**.

        ![Fönstret stoppa säkerhets kopiering.](./media/backup-azure-delete-vault/stop-backup-item.png)

    - Om fönstret **ta bort säkerhets kopierings data** visas anger du namnet på objektet i säkerhets kopian (det här fältet är Skift läges känsligt) och väljer sedan en orsak i den nedrullningsbara menyn. Ange dina kommentarer, om du har några. Välj sedan **ta bort**.

         ![Rutan ta bort säkerhets kopierings data.](./media/backup-azure-delete-vault/stop-backup-blade-delete-backup-data.png)

3. Kontrol lera **meddelande** ikonen: ![meddelande ikonen.](./media/backup-azure-delete-vault/messages.png) När processen har slutförts visar tjänsten följande meddelande: *stoppar säkerhets kopiering och tar bort säkerhetskopierade data för*säkerhets kopierings*objekt.* *Åtgärden har slutförts*.
4. Välj **Uppdatera** på menyn **säkerhets kopierings objekt** för att se till att säkerhets kopierings objekt har tagits bort.

      ![Sidan ta bort säkerhets kopierings objekt.](./media/backup-azure-delete-vault/empty-items-list.png)

## <a name="delete-protected-items-on-premises"></a>Ta bort skyddade objekt lokalt

Börja med att läsa avsnittet **[innan du börjar](#before-you-start)** och förstå borttagnings processen för beroenden och valvet.

1. På instrument panelen för valv väljer du **infrastruktur för säkerhets kopiering**.
2. Välj något av följande alternativ beroende på ditt lokala scenario:

      - För MARS väljer du **skyddade servrar** och **Azure Backup sedan agent**. Välj sedan den server som du vill ta bort.

        ![För MARS väljer du ditt valv för att öppna instrument panelen.](./media/backup-azure-delete-vault/identify-protected-servers.png)

      - För MABS eller DPM väljer du **säkerhets kopierings hanterings servrar**. Välj sedan den server som du vill ta bort.

          ![För MABS väljer du ditt valv för att öppna instrument panelen.](./media/backup-azure-delete-vault/delete-backup-management-servers.png)

3. Rutan **ta bort** visas med ett varnings meddelande.

     ![Ta bort-fönstret.](./media/backup-azure-delete-vault/delete-protected-server.png)

     Läs varnings meddelandet och instruktionerna i kryss rutan medgivande.
    > [!NOTE]
    >
    >- Om den skyddade servern är synkroniserad med Azure-tjänster och säkerhets kopierings objekt finns, visar kryss rutan medgivande antalet beroende säkerhets kopierings objekt och länken för att Visa säkerhets kopierings objekt.
    >- Om den skyddade servern inte är synkroniserad med Azure-tjänster och det finns säkerhets kopierings objekt visas bara antalet säkerhets kopierings objekt i kryss rutan medgivande.
    >- Om det inte finns några säkerhets kopierings objekt uppmanas du att ta bort den i kryss rutan medgivande.

4. Markera kryss rutan medgivande och välj sedan **ta bort**.

5. Kontrol lera **meddelande** ikonen ![ta bort säkerhetskopierade data](./media/backup-azure-delete-vault/messages.png). När åtgärden har slutförts visar tjänsten meddelandet: *stoppar säkerhets kopiering och tar bort säkerhetskopierade data för säkerhets kopierings objekt.* *Åtgärden har slutförts*.
6. Välj **Uppdatera** på menyn **säkerhets kopierings objekt** för att se till att säkerhets kopierings objekt tas bort.

När den här processen är klar kan du ta bort säkerhets kopierings objekt från hanterings konsolen:

- [Ta bort säkerhets kopierings objekt från MARS Management Console](#delete-backup-items-from-the-mars-management-console)
- [Ta bort säkerhets kopierings objekt från hanterings konsolen för MABS](#delete-backup-items-from-the-mabs-management-console)

### <a name="delete-backup-items-from-the-mars-management-console"></a>Ta bort säkerhets kopierings objekt från MARS Management Console

1. Öppna hanterings konsolen för MARS, gå till **Åtgärds** fönstret och välj **Schemalägg säkerhets kopiering**.
2. På sidan **ändra eller stoppa en schemalagd säkerhets kopiering** väljer du **sluta använda detta schema för säkerhets kopiering och tar bort alla lagrade säkerhets kopior**. Välj sedan **Nästa**.

    ![Ändra eller stoppa en schemalagd säkerhets kopiering.](./media/backup-azure-delete-vault/modify-schedule-backup.png)

3. På sidan **stoppa en schemalagd säkerhets kopiering** väljer du **Slutför**.

    ![Stoppa en schemalagd säkerhets kopiering.](./media/backup-azure-delete-vault/stop-schedule-backup.png)
4. Du uppmanas att ange en PIN-kod för säkerhet (personal Identification Number) som du måste generera manuellt. Det gör du genom att först logga in på Azure Portal.
5. Gå till**Inställningar** > **Properties**för **Recovery Services valv** > .
6. Under **säkerhets-PIN**väljer du **generera**. Kopiera den här PIN-koden. PIN-koden är bara giltig i fem minuter.
7. I hanterings konsolen klistrar du in PIN-koden och väljer sedan **OK**.

    ![Skapa en säkerhets kod.](./media/backup-azure-delete-vault/security-pin.png)

8. Följande meddelande visas på sidan **ändra säkerhets kopierings förlopp** : *borttagna säkerhets kopierings data bevaras i 14 dagar. Efter den tiden tas säkerhetskopierade data bort permanent.*  

    ![Ta bort infrastruktur för säkerhets kopiering.](./media/backup-azure-delete-vault/deleted-backup-data.png)

När du har tagit bort de lokala säkerhets kopierings objekten följer du stegen i portalen.

### <a name="delete-backup-items-from-the-mabs-management-console"></a>Ta bort säkerhets kopierings objekt från hanterings konsolen för MABS

Det finns två metoder som du kan använda för att ta bort säkerhets kopierings objekt från hanterings konsolen för MABS.

#### <a name="method-1"></a>Metod 1

Gör så här om du vill stoppa skyddet och ta bort säkerhetskopierade data:

1. Öppna DPM-administratörskonsol och välj sedan **skydd** i navigerings fältet.
2. I visningsfönstret väljer du den skydds grupps medlem som du vill ta bort. Högerklicka för att markera alternativet **sluta skydda grupp medlemmar** .
3. I dialog rutan **stoppa skydd** väljer du **ta bort skyddade data**och markerar sedan kryss rutan **ta bort lagring online** . Välj sedan **stoppa skydd**.

    ![Välj Ta bort skyddade data från fönstret sluta skydda.](./media/backup-azure-delete-vault/delete-storage-online.png)

    Den skyddade medlems statusen ändras till *inaktiv replikering är tillgänglig*.

4. Högerklicka på den inaktiva skydds gruppen och välj **ta bort inaktivt skydd**.

    ![Ta bort inaktivt skydd.](./media/backup-azure-delete-vault/remove-inactive-protection.png)

5. I fönstret **ta bort inaktivt skydd** markerar du kryss rutan **ta bort onlinelagring** och väljer sedan **OK**.

    ![Ta bort online-lagring.](./media/backup-azure-delete-vault/remove-replica-on-disk-and-online.png)

#### <a name="method-2"></a>Metod 2

Öppna **hanterings** konsolen för Mabs. Under **Välj data skydds metod**avmarkerar du kryss rutan **Jag vill ha onlineskydd** .

  ![Välj data skydds metod.](./media/backup-azure-delete-vault/data-protection-method.png)

När du har tagit bort de lokala säkerhets kopierings objekten följer du stegen i portalen.

## <a name="delete-the-recovery-services-vault"></a>Ta bort Recovery Services-valvet

1. När alla beroenden har tagits bort bläddrar du till fönstret **Essentials** i menyn valv.
2. Kontrol lera att det inte finns några säkerhets kopierings objekt, säkerhets kopierings hanterings servrar eller replikerade objekt i listan. Om objekten fortfarande visas i valvet, se avsnittet [innan du börjar](#before-you-start) .

3. När det inte finns några fler objekt i valvet väljer du **ta bort** på instrument panelen för valvet.

    ![Välj Ta bort på instrument panelen för valvet.](./media/backup-azure-delete-vault/vault-ready-to-delete.png)

4. Välj **Ja** för att kontrol lera att du vill ta bort valvet. Valvet har tagits bort. Portalen återgår till den **nya** tjänst menyn.

## <a name="delete-the-recovery-services-vault-by-using-powershell"></a>Ta bort Recovery Services valvet med PowerShell

Börja med att läsa avsnittet **[innan du börjar](#before-you-start)** och förstå borttagnings processen för beroenden och valvet.

Stoppa skyddet och ta bort säkerhets kopierings data:

- Om du använder SQL i Azure VM backup och aktiverat automatiskt skydd för SQL-instanser ska du först inaktivera det automatiska skyddet.

    ```PowerShell
        Disable-AzRecoveryServicesBackupAutoProtection
           [-InputItem] <ProtectableItemBase>
           [-BackupManagementType] <BackupManagementType>
           [-WorkloadType] <WorkloadType>
           [-PassThru]
           [-VaultId <String>]
           [-DefaultProfile <IAzureContextContainer>]
           [-WhatIf]
           [-Confirm]
           [<CommonParameters>]
    ```

  [Läs mer](https://docs.microsoft.com/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection?view=azps-2.6.0) om hur du inaktiverar skydd för ett Azure Backup-skyddat objekt.

- Stoppa skyddet och ta bort data för alla skyddade objekt i molnet (t. ex. laaS VM, Azure-filresurs osv.):

    ```PowerShell
       Disable-AzRecoveryServicesBackupProtection
       [-Item] <ItemBase>
       [-RemoveRecoveryPoints]
       [-Force]
       [-VaultId <String>]
       [-DefaultProfile <IAzureContextContainer>]
       [-WhatIf]
       [-Confirm]
       [<CommonParameters>]
    ```

    [Lär dig mer](https://docs.microsoft.com/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection?view=azps-2.6.0&viewFallbackFrom=azps-2.5.0) om att inaktivera skydd för ett skyddat objekt med säkerhets kopiering.

- För lokala filer och mappar som skyddas med hjälp av Azure Backup Agent (MARS) säkerhetskopiera till Azure använder du följande PowerShell-kommando för att ta bort säkerhetskopierade data från varje MARS PowerShell-modul:

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Post där följande meddelande visas:

    *Microsoft Azure Backup är du säker på att du vill ta bort den här säkerhets kopierings principen? Borttagna säkerhets kopierings data sparas i 14 dagar. Efter den tiden tas säkerhetskopierade data bort permanent. <br/> [J] Ja [A] ja till alla [N] nej [L] nej till alla [S] gör uppehåll [?] Hjälp (standard är "Y"):*

- För lokala datorer som skyddas med MABS (Microsoft Azure Backup Server) eller DPM till Azure (System Center Data Protection Manager) använder du följande kommando för att ta bort säkerhetskopierade data i Azure.

    ```powershell
    Get-OBPolicy | Remove-OBPolicy -DeleteBackup -SecurityPIN <Security Pin>
    ```

    Post där följande meddelande visas:

   *Microsoft Azure Backup* Är du säker på att du vill ta bort den här säkerhets kopierings principen? Borttagna säkerhets kopierings data sparas i 14 dagar. Efter den tiden tas säkerhetskopierade data bort permanent. <br/>
   [J] Ja [A] ja till alla [N] nej [L] nej till alla [S] gör uppehåll [?] Hjälp (standard är "Y"):*

När du har tagit bort säkerhetskopierade data avregistrerar du alla lokala behållare och hanterings servrar.

- För lokala filer och mappar som skyddas med hjälp av Azure Backup Agent (MARS) säkerhetskopiera till Azure:

    ```PowerShell
    Unregister-AzRecoveryServicesBackupContainer
              [-Container] <ContainerBase>
              [-PassThru]
              [-VaultId <String>]
              [-DefaultProfile <IAzureContextContainer>]
              [-WhatIf]
              [-Confirm]
              [<CommonParameters>]
    ```

    [Läs mer](https://docs.microsoft.com/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer?view=azps-2.6.0) om att avregistrera en Windows-Server eller annan behållare från valvet.

- För lokala datorer som skyddas med MABS (Microsoft Azure Backup Server) eller DPM till Azure (System Center Data Protection hantera:

    ```PowerShell
        Unregister-AzRecoveryServicesBackupManagementServer
          [-AzureRmBackupManagementServer] <BackupEngineBase>
          [-PassThru]
          [-VaultId <String>]
          [-DefaultProfile <IAzureContextContainer>]
          [-WhatIf]
          [-Confirm]
          [<CommonParameters>]
    ```

    [Läs mer](https://docs.microsoft.com/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer?view=azps-2.6.0) om att avregistrera en behållare för säkerhets kopierings hantering från valvet.

När du har tagit bort säkerhetskopierade data permanent och avregistrerat alla behållare fortsätter du att ta bort valvet.

Ta bort ett Recovery Services-valv:

   ```PowerShell
       Remove-AzRecoveryServicesVault
      -Vault <ARSVault>
      [-DefaultProfile <IAzureContextContainer>]
      [-WhatIf]
      [-Confirm]
      [<CommonParameters>]
   ```

[Läs mer](https://docs.microsoft.com/powershell/module/az.recoveryservices/remove-azrecoveryservicesvault) om att ta bort ett Recovery Services-valv.

## <a name="delete-the-recovery-services-vault-by-using-cli"></a>Ta bort Recovery Services valvet med CLI

Börja med att läsa avsnittet **[innan du börjar](#before-you-start)** och förstå borttagnings processen för beroenden och valvet.

> [!NOTE]
> För närvarande stöder Azure Backup CLI bara hantering av virtuella Azure-säkerhetskopieringar, så följande kommando för att ta bort valvet fungerar bara om valvet innehåller virtuella Azure-säkerhetskopieringar. Du kan inte ta bort ett valv som använder Azure Backup CLI, om valvet innehåller några säkerhets kopierings objekt av andra typer än virtuella Azure-datorer.

Utför följande om du vill ta bort det befintliga Recovery Services-valvet:

- Stoppa skyddet och ta bort säkerhetskopierade data

    ```azurecli
    az backup protection disable --container-name
                             --item-name
                             [--delete-backup-data {false, true}]
                             [--ids]
                             [--resource-group]
                             [--subscription]
                             [--vault-name]
                             [--yes]
    ```

    Mer information finns i den här [artikeln](/cli/azure/backup/protection#az-backup-protection-disable).

- Ta bort ett befintligt Recovery Services-valv:

    ```azurecli
    az backup vault delete [--force]
                       [--ids]
                       [--name]
                       [--resource-group]
                       [--subscription]
                       [--yes]
    ```

    Mer information finns i den här [artikeln](https://docs.microsoft.com/cli/azure/backup/vault?view=azure-cli-latest)

## <a name="delete-the-recovery-services-vault-by-using-azure-resource-manager"></a>Ta bort Recovery Services valvet med Azure Resource Manager

Det här alternativet för att ta bort Recovery Services-valvet rekommenderas endast om alla beroenden tas bort och du fortfarande får ett *fel meddelande om borttagning av valvet*. Prova någon eller några av följande tips:

- I fönstret **Essentials** i menyn valv kontrollerar du att det inte finns några säkerhets kopierings objekt, säkerhets kopierings hanterings servrar eller replikerade objekt i listan. Om det finns säkerhets kopierings objekt, se avsnittet [innan du börjar](#before-you-start) .
- Försök [att ta bort valvet från portalen](#delete-the-recovery-services-vault) igen.
- Om alla beroenden tas bort och du fortfarande får *borttagnings felet för valvet*, använder du verktyget ARMClient för att utföra följande steg (efter kommentaren).

1. Gå till [Chocolatey.org](https://chocolatey.org/) för att ladda ned och installera choklad. Installera sedan ARMClient genom att köra följande kommando:

   `choco install armclient --source=https://chocolatey.org/api/v2/`
2. Logga in på ditt Azure-konto och kör sedan följande kommando:

    `ARMClient.exe login [environment name]`

3. I Azure Portal kan du samla in prenumerations-ID och resurs gruppens namn för valvet som du vill ta bort.

Mer information om kommandot ARMClient finns i README för [ARMClient](https://github.com/projectkudu/ARMClient/blob/master/README.md).

### <a name="use-the-azure-resource-manager-client-to-delete-a-recovery-services-vault"></a>Använd Azure Resource Manager-klienten för att ta bort ett Recovery Services-valv

1. Kör följande kommando med ditt prenumerations-ID, resurs gruppens namn och namnet på valvet. Om du inte har några beroenden tas valvet bort när du kör följande kommando:

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<recovery services vault name>?api-version=2015-03-15
   ```

2. Om valvet inte är tomt får du följande fel meddelande: det *går inte att ta bort valvet eftersom det finns befintliga resurser i det här valvet.* Om du vill ta bort ett skyddat objekt eller en behållare i ett valv kör du följande kommando:

   ```azurepowershell
   ARMClient.exe delete /subscriptions/<subscriptionID>/resourceGroups/<resourcegroupname>/providers/Microsoft.RecoveryServices/vaults/<recovery services vault name>/registeredIdentities/<container name>?api-version=2016-06-01
   ```

3. I Azure Portal kontrollerar du att valvet har tagits bort.

## <a name="next-steps"></a>Nästa steg

[Lär dig mer om Recovery Services valv](backup-azure-recovery-services-vault-overview.md)<br/>
[Lär dig mer om att övervaka och hantera Recovery Services valv](backup-azure-manage-windows-server.md)
