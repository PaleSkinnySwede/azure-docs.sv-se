---
title: 'Azure Portal: skapa en hanterad instans'
description: Skapa en SQL Database Hanterad instans, nätverks miljö och virtuell klient dator för åtkomst.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: danimir
ms.author: danil
ms.reviewer: sstein, carlrab
ms.date: 09/26/2019
ms.openlocfilehash: 25128442cd922f6b9130586e245695b6880f661c
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "80257622"
---
# <a name="quickstart-create-an-azure-sql-database-managed-instance"></a>Snabb start: skapa en Azure SQL Database Hanterad instans

Den här snabb starten vägleder dig genom hur du skapar en Azure SQL Database [hanterad instans](sql-database-managed-instance.md) i Azure Portal.

> [!IMPORTANT]
> För begränsningar, se [regioner som stöds](sql-database-managed-instance-resource-limits.md#supported-regions) och [prenumerations typer som stöds](sql-database-managed-instance-resource-limits.md#supported-subscription-types).

## <a name="sign-in-to-azure-portal"></a>Logga in på Azure-portalen

Om du inte har en Azure-prenumeration kan du [skapa ett kostnads fritt konto](https://azure.microsoft.com/free/).

Logga in på [Azure Portal](https://portal.azure.com/).

## <a name="create-a-managed-instance"></a>Skapa en hanterad instans

Följande steg visar hur du skapar en hanterad instans:

1. Välj **Azure SQL** på den vänstra menyn i Azure Portal. Om **Azure SQL** inte finns i listan väljer du **alla tjänster**och anger sedan **Azure SQL** i sökrutan.
2. Välj **+ Lägg** till för att öppna **alternativ sidan Välj SQL-distribution** . Du kan visa ytterligare information om en Azure SQL Database Hanterad instans genom att välja **Visa information** på panelen **hanterade instanser** .
3. Välj **Skapa**.

   ![Skapa en hanterad instans](./media/sql-database-managed-instance-get-started/create-managed-instance.png)

4. Använd flikarna i etablerings formuläret **skapa Azure SQL Database Hanterad instans** för att lägga till obligatorisk och valfri information. I följande avsnitt beskrivs de här flikarna.

### <a name="basics"></a>Grundläggande inställningar

- Fyll i obligatorisk information som krävs på fliken **grundläggande** . Detta är en minsta uppsättning information som krävs för att etablera en hanterad instans.

   ![Fliken "grunder" för att skapa en hanterad instans](./media/sql-database-managed-instance-get-started/tabs/mi-create-tab-basics.png)

   Använd tabellen nedan som referens för den information som krävs på den här fliken.

   | Inställningen| Föreslaget värde | Beskrivning |
   | ------ | --------------- | ----------- |
   | **Prenumeration** | Din prenumeration. | En prenumeration som ger dig behörighet att skapa nya resurser. |
   | **Resursgrupp** | En ny eller befintlig resursgrupp.|Giltiga resursgruppnamn finns i [Namngivningsregler och begränsningar](/azure/architecture/best-practices/resource-naming).|
   | **Namn på hanterad instans** | Ett giltigt namn.|Giltiga namn finns i [Namngivningsregler och begränsningar](/azure/architecture/best-practices/resource-naming).|
   | **Region** |Den region där du vill skapa den hanterade instansen.|Information om regioner finns i [Azure-regioner](https://azure.microsoft.com/regions/).|
   | **Administratörsinloggning för hanterad instans** | Ett giltigt användar namn. | Giltiga namn finns i [Namngivningsregler och begränsningar](/azure/architecture/best-practices/resource-naming). Använd inte "ServerAdmin" eftersom det är en reserverad server nivå roll.|
   | **Lösenord** | Ett giltigt lösen ord.| Lösenordet måste vara minst 16 tecken långt och uppfylla [de definierade kraven på komplexitet](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).|

- Välj **Konfigurera hanterad instans** för att ändra storlek på beräknings-och lagrings resurser och granska pris nivåerna. Använd skjutreglagen eller textrutorna för att ange mängden lagringsutrymme och antalet virtuella kärnor. När du är klar väljer du **tillämpa** för att spara ditt val. 

   ![Formulär för hanterad instans](./media/sql-database-managed-instance-get-started/tabs/mi-create-tab-configure-performance.png)

- Om du vill granska dina val innan du skapar en hanterad instans kan du välja **Granska + skapa**. Du kan också konfigurera nätverks alternativ genom att välja **Nästa: nätverk**.

### <a name="networking"></a>Nätverk

- Fyll i valfri information på fliken **nätverk** . Om du utelämnar den här informationen kommer portalen att tillämpa standardinställningar.

   ![Fliken nätverk för att skapa en hanterad instans](./media/sql-database-managed-instance-get-started/tabs/mi-create-tab-networking.png)

   Använd tabellen nedan som referens för den information som krävs på den här fliken.

   | Inställningen| Föreslaget värde | Beskrivning |
   | ------ | --------------- | ----------- |
   | **Virtuellt nätverk** | Välj antingen **Skapa nytt virtuellt nätverk** eller ett giltigt virtuellt nätverk och undernät.| Om ett nätverk eller undernät inte är tillgängligt, måste det [ändras för att uppfylla nätverks kraven](sql-database-managed-instance-configure-vnet-subnet.md) innan du väljer det som mål för den nya hanterade instansen. Information om kraven för att konfigurera nätverks miljön för en hanterad instans finns i [Konfigurera ett virtuellt nätverk för en hanterad instans](sql-database-managed-instance-connectivity-architecture.md). |
   | **Anslutnings typ** | Välj mellan en proxy och en Anslutnings typ för omdirigering.|Mer information om anslutnings typer finns i [Azure SQL Database anslutnings princip](sql-database-connectivity-architecture.md#connection-policy).|
   | **Offentlig slutpunkt**  | Välj **Aktivera**. | För att en hanterad instans ska kunna nås via den offentliga data slut punkten måste du aktivera det här alternativet. | 
   | **Tillåt åtkomst från** (om den **offentliga slut punkten** är aktive rad) | Välj ett av alternativen.   |Med Portal upplevelsen kan du konfigurera en säkerhets grupp med en offentlig slut punkt. </br> </br> Välj något av följande alternativ baserat på ditt scenario: </br> <ul> <li>**Azure-tjänster**: Vi rekommenderar det här alternativet när du ansluter från Power BI eller en annan tjänst för flera innehavare. </li> <li> **Internet**: används i test syfte när du snabbt vill skapa en hanterad instans. Vi rekommenderar det inte för produktions miljöer. </li> <li> **Ingen åtkomst**: det här alternativet skapar en säkerhets regel för **neka** . Ändra den här regeln för att göra en hanterad instans tillgänglig via en offentlig slut punkt. </li> </ul> </br> Mer information om säkerhet för offentliga slut punkter finns i [använda en Azure SQL Database Hanterad instans på ett säkert sätt med en offentlig slut punkt](sql-database-managed-instance-public-endpoint-securely.md).|

- Välj **Granska + skapa** för att granska dina val innan du skapar en hanterad instans. Du kan också konfigurera fler anpassade inställningar genom att välja **Nästa: ytterligare inställningar**.

### <a name="additional-settings"></a>Ytterligare inställningar

- Fyll i valfri information på fliken **ytterligare inställningar** . Om du utelämnar den här informationen kommer portalen att tillämpa standardinställningar.

   ![Fliken Ytterligare inställningar för att skapa en hanterad instans](./media/sql-database-managed-instance-get-started/tabs/mi-create-tab-additional-settings.png)

   Använd tabellen nedan som referens för den information som krävs på den här fliken.

   | Inställningen| Föreslaget värde | Beskrivning |
   | ------ | --------------- | ----------- |
   | **Sortering** | Välj den sortering som du vill använda för din hanterade instans. Om du migrerar databaser från SQL Server kontrollerar du käll sorteringen genom att `SELECT SERVERPROPERTY(N'Collation')` använda och använda det värdet.| Information om sorteringar finns i [Ange eller ändra Server sorteringen](https://docs.microsoft.com/sql/relational-databases/collations/set-or-change-the-server-collation).|   
   | **Tidszon** | Välj den tidszon som din hanterade instans ska observera.|Mer information finns i [tids zoner](sql-database-managed-instance-timezone.md).|
   | **Använd som sekundär redundans** | Välj **Ja**. | Aktivera det här alternativet om du vill använda den hanterade instansen som en sekundär grupp för redundans.|
   | **Primär hanterad instans** (om **Använd som sekundär redundans** är inställt på **Ja**) | Välj en befintlig primär hanterad instans som ska anslutas till samma DNS-zon med den hanterade instans som du skapar. | I det här steget aktive ras konfigurationen efter skapandet av gruppen redundans. Mer information finns i [Självstudier: Lägg till en SQL Database Hanterad instans i en failover-grupp](sql-database-managed-instance-failover-group-tutorial.md).|

### <a name="review--create"></a>Granska + skapa

5. Välj **Granska + fliken Skapa** för att granska dina val innan du skapar den hanterade instansen.

   ![Flik för att granska och skapa en hanterad instans](./media/sql-database-managed-instance-get-started/tabs/mi-create-tab-review-create.png)

6. Välj **skapa** för att starta etableringen av den hanterade instansen.

> [!IMPORTANT]
> Att distribuera en hanterad instans är en tids krävande åtgärd. Distribution av den första instansen i under nätet tar vanligt vis mycket längre tid än att distribuera till ett undernät med befintliga hanterade instanser. För genomsnittlig etablerings tid, se [hanterade instans hanterings åtgärder](sql-database-managed-instance.md#managed-instance-management-operations).

### <a name="monitor-deployment-progress"></a>Övervaka distributions förlopp

7. Välj ikonen **meddelanden** om du vill visa status för distributionen.

   ![Distributions förlopp för en hanterad instans distribution](./media/sql-database-managed-instance-get-started/in-progress/mi-create-deployment-in-progress.png)

8. Välj **distribution pågår** i meddelandet för att öppna fönstret hanterad instans och övervaka distributions förloppet ytterligare. 

> [!TIP]
> Om du stängde webbläsaren eller flyttat bort från distributions förlopps skärmen följer du stegen nedan för att gå tillbaka till skärmen distributions förlopp:
> 1. I Azure Portal öppnar du resurs gruppen (på fliken **grundläggande** ) där du distribuerar en hanterad instans.
> 2. Välj **distributioner**.
> 3. Välj den hanterade instansen distributions åtgärd pågår.

> [!IMPORTANT]
> För att kunna hämta statusen för skapandet av hanterade instanser måste du ha **Läs behörighet** över resurs gruppen. Om du inte har den här behörigheten eller återkalla den medan den hanterade instansen håller på att skapas, kan det leda till att hanterad instans inte visas i listan över distributioner av resurs grupper.
>

## <a name="post-deployment-operations"></a>Åtgärder efter distribution

Granska de resurser som skapats genom att finjustera nätverks inställningarna och hämta information om värd anslutning (FQDN) genom att följa stegen som beskrivs i det här avsnittet.

### <a name="view-resources-created"></a>Visa resurser som skapats

Vid lyckad distribution av hanterad instans för att visa resurser som skapats:

1. Öppna resurs gruppen för din hanterade instans. Visa dess resurser som har skapats för dig i snabb starten för att [skapa en hanterad instans](#create-a-managed-instance) .

   ![Resurser för hanterad instans](./media/sql-database-managed-instance-get-started/resources.png)

### <a name="view-and-fine-tune-network-settings"></a>Visa och finjustera nätverks inställningar

Om du vill finjustera nätverks inställningarna bör du kontrol lera följande:

1. Välj routningstabellen för att granska den användardefinierade väg (UDR) som skapades åt dig.

   ![Routningstabell](./media/sql-database-managed-instance-get-started/route-table.png)

2. I routningstabellen granskar du posterna för att dirigera trafik från och inom den hanterade instansens virtuella nätverk. Om du skapar eller konfigurerar din routningstabell manuellt, se till att skapa dessa poster i routningstabellen för hanterad instans.

   ![Post för ett undernät för hanterad instans till lokalt](./media/sql-database-managed-instance-get-started/udr.png)

3. Gå tillbaka till resurs gruppen och välj nätverks säkerhets gruppen.

   ![Nätverkssäkerhetsgrupp](./media/sql-database-managed-instance-get-started/network-security-group.png)

4. Granska inkommande och utgående säkerhetsregler. 

   ![Säkerhetsregler](./media/sql-database-managed-instance-get-started/security-rules.png)

> [!IMPORTANT]
> Om du har konfigurerat en offentlig slut punkt för din hanterade instans måste du öppna portar för att tillåta nätverks trafik att tillåta anslutningar till hanterad instans från det offentliga Internet. mer information finns i [Konfigurera en offentlig slut punkt för en hanterad instans](sql-database-managed-instance-public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group) .
>

### <a name="retrieve-connection-details-to-managed-instance"></a>Hämta anslutnings information till hanterad instans

Följ dessa steg för att hämta värdnamn och fullständigt kvalificerat domän namn (FQDN) för att ansluta till hanterad instans:

1. Gå tillbaka till resursgruppen och välj din hanterade instans.

   ![Hanterad instans i resurs gruppen](./media/sql-database-managed-instance-get-started/managed-instance.png)

2. Leta upp **värd** egenskapen på fliken **Översikt** . Kopiera värd namnet för den hanterade instansen för användning i nästa snabb start.

   ![Värdnamn](./media/sql-database-managed-instance-get-started/host-name.png)

   Värdet som kopieras representerar ett fullständigt kvalificerat domän namn (FQDN) som kan användas för att ansluta till en hanterad instans. Det liknar följande adress exempel: *your_host_name. a1b2c3d4e5f6. Database. Windows. net*.

## <a name="next-steps"></a>Nästa steg

Lär dig mer om hur du ansluter till en hanterad instans:
- En översikt över anslutnings alternativen för program finns i [ansluta dina program till en hanterad instans](sql-database-managed-instance-connect-app.md).
- En snabb start som visar hur du ansluter till en hanterad instans från en virtuell Azure-dator finns i [Konfigurera en anslutning till en virtuell Azure-dator](sql-database-managed-instance-configure-vm.md).
- En snabb start som visar hur du ansluter till en hanterad instans från en lokal klient dator med hjälp av en punkt-till-plats-anslutning finns i [Konfigurera en punkt-till-plats](sql-database-managed-instance-configure-p2s.md)-anslutning.

Så här återställer du en befintlig SQL Server databas från en lokal plats till en hanterad instans: 
- Använd [Azure Database migration service för migrering](../dms/tutorial-sql-server-to-managed-instance.md) för att återställa från en databas säkerhets kopia. 
- Använd [kommandot T-SQL REstore](sql-database-managed-instance-get-started-restore.md) för att återställa från en databas säkerhets kopia.

Avancerad övervakning av hanterade instanser av databas prestanda med inbyggd fel söknings information finns i [övervaka Azure SQL Database med hjälp av Azure SQL-analys](../azure-monitor/insights/azure-sql.md).
