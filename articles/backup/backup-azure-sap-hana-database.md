---
title: Säkerhetskopiera en SAP HANA-databas till Azure med Azure Backup
description: I den här artikeln lär du dig hur du säkerhetskopierar en SAP HANA-databas till virtuella Azure-datorer med tjänsten Azure Backup.
ms.topic: conceptual
ms.date: 11/12/2019
ms.openlocfilehash: deedd4d2553b3b06f76f698fdb2425a8d3878d23
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79248065"
---
# <a name="back-up-sap-hana-databases-in-azure-vms"></a>Säkerhetskopiera SAP HANA-databaser i virtuella Azure-datorer

SAP HANA-databaser är kritiska arbets belastningar som kräver återställnings punkt mål och långsiktig kvarhållning. Du kan säkerhetskopiera SAP HANA databaser som körs på virtuella datorer i Azure med hjälp av [Azure Backup](backup-overview.md).

Den här artikeln visar hur du säkerhetskopierar SAP HANA databaser som körs på virtuella Azure-datorer till ett Azure Backup Recovery Services-valv.

I den här artikeln får du lära dig att:
> [!div class="checklist"]
>
> * Skapa och konfigurera ett valv
> * Identifiera databaser
> * Konfigurera säkerhets kopior
> * Köra ett säkerhets kopierings jobb på begäran

>[!NOTE]
>**Mjuk borttagning för SQL Server i Azure VM och mjuk borttagning för SAP HANA i Azure VM-arbetsbelastningar** finns nu i för hands version.<br>
>Registrera dig för för hands versionen genom att skriva till oss påAskAzureBackupTeam@microsoft.com

## <a name="prerequisites"></a>Krav

Se [kraven](tutorial-backup-sap-hana-db.md#prerequisites) och [Vad skriptet för för registrering gör](tutorial-backup-sap-hana-db.md#what-the-pre-registration-script-does) i avsnitten för att konfigurera databasen för säkerhets kopiering.

### <a name="set-up-network-connectivity"></a>Konfigurera nätverks anslutning

För alla åtgärder kräver den virtuella datorn för SAP HANA anslutning till offentliga Azure-IP-adresser. VM-åtgärder (databas identifiering, konfigurera säkerhets kopiering, schemalägga säkerhets kopiering, återställning av återställnings punkter osv.) kan inte köras utan anslutning till offentliga Azure-IP-adresser.

Upprätta anslutningar genom att använda något av följande alternativ:

#### <a name="allow-the-azure-datacenter-ip-ranges"></a>Tillåt IP-intervall för Azure-datacenter

Med det här alternativet tillåts [IP-intervall](https://www.microsoft.com/download/details.aspx?id=41653) i den nedladdade filen. Använd cmdleten Set-AzureNetworkSecurityRule för att få åtkomst till en nätverks säkerhets grupp (NSG). Om listan med betrodda mottagare bara innehåller landsspecifika IP-adresser, måste du också uppdatera de säkra mottagarna visar tjänst tag gen Azure Active Directory (Azure AD) för att aktivera autentisering.

#### <a name="allow-access-using-nsg-tags"></a>Tillåt åtkomst med NSG-Taggar

Om du använder NSG för att begränsa anslutningen bör du använda AzureBackup service tag för att tillåta utgående åtkomst till Azure Backup. Dessutom bör du även tillåta anslutning för autentisering och data överföring genom att använda [regler](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags) för Azure AD och Azure Storage. Detta kan göras från Azure Portal eller via PowerShell.

Så här skapar du en regel med hjälp av portalen:

  1. I **alla tjänster**går du till **nätverks säkerhets grupper** och väljer Nätverks säkerhets gruppen.
  2. Välj **utgående säkerhets regler** under **Inställningar**.
  3. Välj **Lägg till**. Ange all information som krävs för att skapa en ny regel enligt beskrivningen i [säkerhets regel inställningar](https://docs.microsoft.com/azure/virtual-network/manage-network-security-group#security-rule-settings). Se till att alternativet **destination** har angetts till **service tag** och **mål tjänst tag gen** är inställt på **AzureBackup**.
  4. Klicka på **Lägg till**för att spara den nyligen skapade utgående säkerhets regeln.

Så här skapar du en regel med hjälp av PowerShell:

 1. Lägg till autentiseringsuppgifter för Azure-kontot och uppdatera de nationella molnen<br/>
      `Add-AzureRmAccount`<br/>

 2. Välj prenumerationen NSG<br/>
      `Select-AzureRmSubscription "<Subscription Id>"`

 3. Välj NSG<br/>
    `$nsg = Get-AzureRmNetworkSecurityGroup -Name "<NSG name>" -ResourceGroupName "<NSG resource group name>"`

 4. Lägg till Tillåt utgående regel för Azure Backup service tag<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AzureBackupAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "AzureBackup" -DestinationPortRange 443 -Description "Allow outbound traffic to Azure Backup service"`

 5. Lägg till Tillåt utgående regel för lagrings tjänst tag gen<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "StorageAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "Storage" -DestinationPortRange 443 -Description "Allow outbound traffic to Azure Backup service"`

 6. Lägg till Tillåt utgående regel för AzureActiveDirectory service tag<br/>
    `Add-AzureRmNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AzureActiveDirectoryAllowOutbound" -Access Allow -Protocol * -Direction Outbound -Priority <priority> -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix "AzureActiveDirectory" -DestinationPortRange 443 -Description "Allow outbound traffic to AzureActiveDirectory service"`

 7. Spara NSG<br/>
    `Set-AzureRmNetworkSecurityGroup -NetworkSecurityGroup $nsg`

**Tillåt åtkomst med hjälp av Azure Firewall-Taggar**. Om du använder Azure-brandväggen kan du skapa en program regel med hjälp av AzureBackup [FQDN-taggen](https://docs.microsoft.com/azure/firewall/fqdn-tags). Detta tillåter utgående åtkomst till Azure Backup.

**Distribuera en HTTP-proxyserver för att dirigera trafik**. När du säkerhetskopierar en SAP HANA-databas på en virtuell Azure-dator använder säkerhets kopierings tillägget på den virtuella datorn HTTPS-API: er för att skicka hanterings kommandon till Azure Backup och data till Azure Storage. Säkerhets kopierings tillägget använder också Azure AD för autentisering. Dirigera trafiken för säkerhetskopieringstillägget för dessa tre tjänster via HTTP-proxyn. Tilläggen är den enda komponenten som är konfigurerad för åtkomst till det offentliga Internet.

Anslutnings alternativen omfattar följande fördelar och nack delar:

**Alternativet** | **Fördelar** | **Nackdelar**
--- | --- | ---
Tillåta IP-intervall | Inga ytterligare kostnader | Komplext att hantera eftersom IP-adressintervall ändras med tiden <br/><br/> Ger åtkomst till hela Azure, inte bara Azure Storage
Använda NSG service-Taggar | Enklare att hantera när intervall ändringar slås samman automatiskt <br/><br/> Inga ytterligare kostnader <br/><br/> | Kan endast användas med NSG: er <br/><br/> Ger åtkomst till hela tjänsten
Använd Azure Firewall FQDN-Taggar | Enklare att hantera eftersom nödvändiga FQDN-namn hanteras automatiskt | Kan endast användas med Azure brand vägg
Använda en HTTP-proxy | Detaljerad kontroll i proxyn över lagrings-URL: er tillåts <br/><br/> En enda punkt i Internet åtkomst till virtuella datorer <br/><br/> Ändringar i Azure IP-adress ingår inte | Ytterligare kostnader för att köra en virtuell dator med proxy-programvaran

#### <a name="private-endpoints"></a>Privata slut punkter

[!INCLUDE [Private Endpoints](../../includes/backup-private-endpoints.md)]

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="discover-the-databases"></a>Identifiera databaserna

1. Klicka på **säkerhetskopiera**i **komma igång**i valvet. I **var kör din arbets belastning?** väljer du **SAP HANA i virtuell Azure-dator**.
2. Klicka på **Starta identifiering**. Detta initierar identifiering av oskyddade virtuella Linux-datorer i valv regionen.

   * Efter identifieringen visas oskyddade virtuella datorer i portalen, listade efter namn och resurs grupp.
   * Om en virtuell dator inte visas som förväntat kontrollerar du om den redan har säkerhetskopierats i ett valv.
   * Flera virtuella datorer kan ha samma namn, men de tillhör olika resurs grupper.

3. I **välj Virtual Machines**klickar du på länken för att hämta skriptet som ger behörighet till tjänsten Azure Backup för att få åtkomst till de SAP HANA virtuella datorerna för databas identifiering.
4. Kör skriptet på varje virtuell dator som är värd för SAP HANA databaser som du vill säkerhetskopiera.
5. När skriptet har körts på de virtuella datorerna väljer du de virtuella datorerna i **välj Virtual Machines**. Klicka sedan på **identifiera databaser**.
6. Azure Backup identifierar alla SAP HANA-databaser på den virtuella datorn. Under identifieringen registrerar Azure Backup den virtuella datorn med valvet och installerar ett tillägg på den virtuella datorn. Ingen agent är installerad på databasen.

    ![Identifiera SAP HANA databaser](./media/backup-azure-sap-hana-database/hana-discover.png)

## <a name="configure-backup"></a>Konfigurera säkerhetskopiering  

Aktivera nu säkerhets kopiering.

1. I steg 2 klickar du på **Konfigurera säkerhets kopiering**.

    ![Konfigurera säkerhetskopiering](./media/backup-azure-sap-hana-database/configure-backup.png)
2. I **Välj objekt att säkerhetskopiera**väljer du alla databaser som du vill skydda > **OK**.

    ![Välj objekt som ska säkerhets kopie ras](./media/backup-azure-sap-hana-database/select-items.png)
3. I **säkerhets kopierings policy** > **väljer du säkerhets kopierings princip**, skapar en ny säkerhets kopierings princip för databaserna enligt anvisningarna nedan.

    ![Välj säkerhets kopierings princip](./media/backup-azure-sap-hana-database/backup-policy.png)
4. När du har skapat principen klickar du på **Aktivera säkerhets kopiering**på menyn **säkerhets kopiering** .

    ![Aktivera säkerhets kopiering](./media/backup-azure-sap-hana-database/enable-backup.png)
5. Spåra förloppet för säkerhets kopierings konfigurationen i området **meddelanden** i portalen.

### <a name="create-a-backup-policy"></a>Skapa en säkerhetskopieringspolicy

En säkerhets kopierings princip definierar när säkerhets kopieringar görs och hur länge de ska behållas.

* En policy skapas på valvnivå.
* Flera valv kan använda samma säkerhetskopieringspolicy, men du måste tillämpa säkerhetskopieringspolicyn på varje valv.

Ange princip inställningarna enligt följande:

1. I **Policynamn** anger du ett namn för den nya policyn.

   ![Ange princip namn](./media/backup-azure-sap-hana-database/policy-name.png)
2. I **Policy för fullständig säkerhetskopia** väljer du en **Säkerhetskopieringsfrekvens** och väljer **Dagligen** eller **Varje vecka**.
   * **Varje dag**: Välj den timme och den tidszon som säkerhets kopierings jobbet ska starta.
       * Du måste köra en fullständig säkerhets kopiering. Du kan inte inaktivera det här alternativet.
       * Klicka på **Fullständig säkerhetskopia** för att visa policyn.
       * Du kan inte skapa differentiella säkerhetskopior för dagliga fullständiga säkerhetskopior.
   * **Varje vecka**: Välj den veckodag, timme och tidszon som säkerhets kopierings jobbet körs i.

   ![Välj säkerhets kopierings frekvens](./media/backup-azure-sap-hana-database/backup-frequency.png)

3. Konfigurera inställningar för kvarhållning för fullständig säkerhets kopiering i **kvarhållningsintervall**.
    * Som standard är alla alternativ markerade. Rensa eventuella gränser för kvarhållningsintervall som du inte vill använda och ange de som du gör.
    * Den minsta Retentions perioden för någon typ av säkerhets kopiering (fullständig/differentiell/log) är sju dagar.
    * Återställningspunkter taggas för kvarhållning baserat på deras kvarhållningsintervall. Om du till exempel väljer en daglig fullständig säkerhetskopia utlöses endast en fullständig säkerhetskopia varje dag.
    * Säkerhets kopian för en angiven dag märks och bevaras baserat på vecko kvarhållningsintervallet och-inställningen.
    * De månatliga och årliga kvarhållningsintervallen fungerar på liknande sätt.

4. I menyn **fullständig säkerhets kopierings policy** klickar du på **OK** för att godkänna inställningarna.
5. Välj **differentiell säkerhets kopiering** för att lägga till en differentiell princip.
6. I **Policy för differentiell säkerhetskopia** väljer du **Aktivera** för att öppna kontrollerna för frekvens och kvarhållning.
    * Du kan endast utlösa en differentiell säkerhetskopia per dag.
    * Differentiella säkerhetskopior kan behållas i upp till 180 dagar. Om du behöver längre kvarhållning måste du använda fullständiga säkerhetskopior.

    ![Princip för differentiell säkerhets kopiering](./media/backup-azure-sap-hana-database/differential-backup-policy.png)

    > [!NOTE]
    > Stegvisa säkerhets kopieringar stöds inte för närvarande.

7. Klicka på **OK** för att spara principen och återgå till huvud menyn för **säkerhets kopierings policyn** .
8. Välj **logg säkerhets kopiering** för att lägga till en transaktions logg princip för säkerhets kopiering
    * I **logg säkerhets kopiering**väljer du **Aktivera**.  Detta kan inte inaktive ras eftersom SAP HANA hanterar alla logg säkerhets kopior.
    * Ange frekvens och bevarande kontroller.

    > [!NOTE]
    > Logg säkerhets kopior börjar bara att flyta efter en lyckad fullständig säkerhets kopiering har slutförts.

9. Klicka på **OK** för att spara principen och återgå till huvud menyn för **säkerhets kopierings policyn** .
10. När du är klar med att definiera säkerhets kopierings principen klickar du på **OK**.

> [!NOTE]
> Varje logg säkerhets kopia länkas till den tidigare fullständiga säkerhets kopian för att skapa en återställnings kedja. Den fullständiga säkerhets kopian kommer att behållas tills kvarhållning av den senaste logg säkerhets kopian har upphört att gälla. Detta kan betyda att den fullständiga säkerhets kopian behålls under en längre period för att se till att alla loggar kan återställas. Vi antar att användaren har en veckovis fullständig säkerhets kopiering, dagliga differentiella och 2 timmars loggar. Alla finns kvar i 30 dagar. Men den fullständiga veckan kan bara rensas/tas bort efter att nästa fullständiga säkerhets kopiering är tillgängligt, d.v.s. efter 30 och 7 dagar. Anta att en fullständig säkerhets kopiering varje vecka sker på den 16 november. Enligt bevarande principen bör den behållas tills den 16 december. Den senaste logg säkerhets kopian för denna fullständiga aktivitet sker före nästa schemalagda, på Nov 22. Tills den här loggen är tillgänglig fram till Dec 22, kan den sextonde sexton fullständiga filen inte tas bort. Därför behålls den sextonde sexton fullständiga till och med Dec 22.

## <a name="run-an-on-demand-backup"></a>Köra en säkerhets kopiering på begäran

Säkerhets kopieringar körs enligt princip schemat. Du kan köra en säkerhets kopiering på begäran på följande sätt:

1. I menyn valv klickar du på **säkerhets kopierings objekt**.
2. I **säkerhets kopierings objekt**väljer du den virtuella dator som kör SAP HANA databasen och klickar sedan på **Säkerhetskopiera nu**.
3. I **Säkerhetskopiera nu**använder du kalender kontrollen för att välja den sista dagen som återställnings punkten ska behållas. Klicka sedan på **OK**.
4. Övervaka Portal meddelanden. Du kan övervaka jobb förloppet i valv instrument panelen > **säkerhets kopierings jobb** > **pågår**. Det kan ta en stund att skapa den första säkerhets kopieringen, beroende på databasens storlek.

## <a name="run-sap-hana-studio-backup-on-a-database-with-azure-backup-enabled"></a>Köra säkerhets kopiering av SAP HANA Studio på en databas med Azure Backup aktiverat

Om du vill ta en lokal säkerhets kopia (med HANA Studio) av en databas som säkerhets kopie ras med Azure Backup gör du följande:

1. Vänta tills alla säkerhets kopieringar eller loggar för databasen har slutförts. Kontrol lera statusen i SAP HANA Studio/cockpit.
2. Inaktivera logg säkerhets kopior och ange säkerhets kopierings katalogen till fil systemet för relevant databas.
3. Det gör du genom att dubbelklicka på **systemdb** > **konfiguration** > **Välj databas** > **filter (logg)**.
4. Ange **enable_auto_log_backup** till **Nej**.
5. Ange **log_backup_using_backint** till **false**.
6. Utför en fullständig säkerhets kopiering på begäran av databasen.
7. Vänta tills den fullständiga säkerhets kopieringen och katalog säkerhets kopieringen har slutförts.
8. Återställ tidigare inställningar tillbaka till dem för Azure:
    * Ange **enable_auto_log_backup** till **Ja**.
    * Ange **log_backup_using_backint** till **Sant**.

## <a name="next-steps"></a>Nästa steg

* Lär dig hur du [återställer SAP HANA databaser som körs på virtuella Azure-datorer](https://docs.microsoft.com/azure/backup/sap-hana-db-restore)
* Lär dig hur du [hanterar SAP HANA databaser som säkerhets kopie ras med Azure Backup](https://docs.microsoft.com/azure/backup/sap-hana-db-manage)
