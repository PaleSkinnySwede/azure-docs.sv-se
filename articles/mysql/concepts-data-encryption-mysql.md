---
title: Data kryptering med kundhanterad nyckel-Azure Database for MySQL
description: Azure Database for MySQL data kryptering med en kundhanterad nyckel kan du Bring Your Own Key (BYOK) för data skydd i vila. Det gör det också möjligt för organisationer att implementera separering av uppgifter i hanteringen av nycklar och data.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: conceptual
ms.date: 01/13/2020
ms.openlocfilehash: a97fee619858aa024ff208b72d3b2594c30d2fd5
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79299132"
---
# <a name="azure-database-for-mysql-data-encryption-with-a-customer-managed-key"></a>Azure Database for MySQL data kryptering med en kundhanterad nyckel

> [!NOTE]
> För tillfället måste du begära åtkomst för att använda den här funktionen. Det gör du genom att AskAzureDBforMySQL@service.microsoft.comkontakta.

Med data kryptering med Kundhanterade nycklar för Azure Database for MySQL kan du ta med din egen nyckel (BYOK) för data skydd i vila. Det gör det också möjligt för organisationer att implementera separering av uppgifter i hanteringen av nycklar och data. Med kundhanterad kryptering ansvarar du för och i en fullständig kontroll av, en nyckels livs cykel, nyckel användnings behörigheter och granskning av åtgärder på nycklar.

Data kryptering med Kundhanterade nycklar för Azure Database for MySQL anges på server nivå. För en specifik server används en kundhanterad nyckel, som kallas nyckel krypterings nyckel (KEK), för att kryptera data krypterings nyckeln (DEK) som används av tjänsten. KEK är en asymmetrisk nyckel som lagras i en kundägda och kundhanterad [Azure Key Vault](../key-vault/key-Vault-secure-your-key-Vault.md) instans. Nyckel krypterings nyckeln (KEK) och data krypterings nyckeln (DEK) beskrivs mer detaljerat längre fram i den här artikeln.

Key Vault är ett molnbaserad, externt nyckel hanterings system. Den har hög tillgänglighet och ger skalbar och säker lagring för kryptografiska RSA-nycklar, eventuellt backas upp av FIPS 140-2 nivå 2, verifierade HSM: er (Hardware Security modules). Den tillåter inte direkt åtkomst till en lagrad nyckel, men tillhandahåller tjänster för kryptering och dekryptering till auktoriserade entiteter. Key Vault kan generera nyckeln, importera den eller [låta den överföras från en lokal HSM-enhet](../key-vault/key-Vault-hsm-protected-keys.md).

> [!NOTE]
> Den här funktionen är tillgänglig i alla Azure-regioner där Azure Database for MySQL stöder pris nivåerna "Generell användning" och "Minnesoptimerade".

## <a name="benefits"></a>Fördelar

Data kryptering för Azure Database for MySQL ger följande fördelar:

* Data åtkomsten styrs helt av dig genom möjligheten att ta bort nyckeln och göra databasen otillgänglig 
* Fullständig kontroll över nyckel livs cykeln, inklusive rotation av nyckeln för att passa företags principer
* Central hantering och organisation av nycklar i Azure Key Vault
* Möjlighet att implementera separering av uppgifter mellan säkerhets chefer och DBA-och system administratörer


## <a name="terminology-and-description"></a>Terminologi och beskrivning

**Data krypterings nyckel (DEK)**: en symmetrisk AES256-nyckel som används för att kryptera en partition eller data block. Kryptering av varje data block med en annan nyckel gör det svårare att analysera krypteringen. Åtkomst till DEKs krävs av resurs leverantören eller program instansen som krypterar och dekrypterar ett särskilt block. När du ersätter en DEK med en ny nyckel måste endast data i det associerade blocket krypteras igen med den nya nyckeln.

**Nyckel krypterings nyckel (KEK)**: en krypterings nyckel som används för att kryptera DEKs. En KEK som aldrig lämnar Key Vault gör att DEKs själva krypteras och kontrol leras. Entiteten som har åtkomst till KEK kan skilja sig från den entitet som kräver DEK. Eftersom KEK krävs för att dekryptera DEKs är KEK en enda punkt med vilken DEKs kan tas bort effektivt genom borttagning av KEK.

DEKs, som krypteras med KeyExchange, lagras separat. Endast en entitet med åtkomst till KEK kan dekryptera dessa DEKs. Mer information finns i [säkerhet i kryptering i vila](../security/fundamentals/encryption-atrest.md).

## <a name="how-data-encryption-with-a-customer-managed-key-works"></a>Hur data kryptering med en kundhanterad nyckel fungerar

![Diagram som visar en översikt över Bring Your Own Key](media/concepts-data-access-and-security-data-encryption/mysqloverview.png)

För att en MySQL-server ska kunna använda Kundhanterade nycklar som lagras i Key Vault för kryptering av DEK ger en Key Vault administratör följande åtkomst behörighet till servern:

* **Hämta**: för att hämta den offentliga delen och egenskaperna i nyckel valvet.
* **wrapKey**: för att kunna kryptera Dek.
* **unwrapKey**: för att kunna dekryptera Dek.

Nyckel valvs administratören kan också [Aktivera loggning av Key Vault gransknings händelser](../azure-monitor/insights/azure-key-vault.md), så att de kan granskas senare.

När servern har kon figurer ATS för att använda den Kundhanterade nyckeln som lagras i nyckel valvet skickar servern DEK till nyckel valvet för kryptering. Key Vault returnerar den krypterade DEK som lagras i användar databasen. På samma sätt skickar servern det skyddade DEK till nyckel valvet för dekryptering när det behövs. Granskare kan använda Azure Monitor för att granska Key Vault gransknings händelse loggar om loggning är aktiverat.

## <a name="requirements-for-configuring-data-encryption-for-azure-database-for-mysql"></a>Krav för att konfigurera data kryptering för Azure Database for MySQL

Följande är krav för att konfigurera Key Vault:

* Key Vault och Azure Database for MySQL måste tillhöra samma Azure Active Directory-klient (Azure AD). Key Vault mellan klienter och Server interaktioner stöds inte. När du flyttar resurser måste du konfigurera om data krypteringen.
* Du måste aktivera funktionen för mjuk borttagning i nyckel valvet för att skydda mot data förlust om en oavsiktlig nyckel (eller Key Vault) tas bort. Mjuka, borttagna resurser behålls i 90 dagar, om inte användaren återställer eller tar bort dem under tiden. Åtgärder för att återställa och rensa har sina egna behörigheter som är kopplade till en Key Vault åtkomst princip. Funktionen mjuk borttagning är inaktive rad som standard, men du kan aktivera den via PowerShell eller Azure CLI (Observera att du inte kan aktivera den via Azure Portal).
* Bevilja Azure Database for MySQL åtkomst till nyckel valvet med behörigheterna get, wrapKey och unwrapKey med hjälp av dess unika hanterade identitet. I Azure Portal skapas den unika identiteten automatiskt när data kryptering är aktiverat på MySQL. Se [Konfigurera data kryptering för MySQL](howto-data-encryption-portal.md) för detaljerade, stegvisa anvisningar när du använder Azure Portal.

* När du använder en brand vägg med Key Vault måste du aktivera alternativet Tillåt att **betrodda Microsoft-tjänster kringgår brand väggen**.

Följande är krav för att konfigurera den Kundhanterade nyckeln:

* Den Kundhanterade nyckeln som ska användas för att kryptera DEK kan bara vara asymmetrisk, RSA 2028.
* Aktiverings datumet (om det är inställt) måste vara datum och tid tidigare. Utgångs datumet (om det är inställt) måste vara ett framtida datum och en framtida tidpunkt.
* Nyckeln måste vara i *aktiverat* läge.
* Om du importerar en befintlig nyckel till nyckel valvet ska du se till att tillhandahålla den i de fil format som stöds`.pfx`( `.byok`, `.backup`,).

## <a name="recommendations"></a>Rekommendationer

När du använder data kryptering med hjälp av en kundhanterad nyckel är det här rekommendationer för att konfigurera Key Vault:

* Ange ett resurs lås på Key Vault för att kontrol lera vem som kan ta bort den här kritiska resursen och förhindra oavsiktlig eller obehörig borttagning.
* Aktivera granskning och rapportering på alla krypterings nycklar. Key Vault innehåller loggar som är lätta att mata in i andra säkerhets informations-och händelse hanterings verktyg. Azure Monitor Log Analytics är ett exempel på en tjänst som redan är integrerad.

* Se till att Key Vault och Azure Database for MySQL finns i samma region och se till att det går snabbare att få åtkomst till DEK-åtgärder för omslutning och brytning.

Här är rekommendationer för att konfigurera en kundhanterad nyckel:

* Behåll en kopia av den Kundhanterade nyckeln på en säker plats eller depositions den till depositions-tjänsten.

* Om Key Vault genererar nyckeln skapar du en nyckel säkerhets kopia innan du använder nyckeln för första gången. Du kan bara återställa säkerhets kopian till Key Vault. Mer information om säkerhets kopierings kommandot finns i [Backup-AzKeyVaultKey](https://docs.microsoft.com/powershell/module/az.keyVault/backup-azkeyVaultkey).

## <a name="inaccessible-customer-managed-key-condition"></a>Otillgängligt kund hanterat nyckel villkor

När du konfigurerar data kryptering med en kundhanterad nyckel i Key Vault, krävs kontinuerlig åtkomst till den här nyckeln för att servern ska vara online. Om servern förlorar åtkomsten till den Kundhanterade nyckeln i Key Vault börjar servern neka alla anslutningar inom 10 minuter. Servern utfärdar ett motsvarande fel meddelande och ändrar Server tillstånd till *otillgängligt*. Den enda åtgärden som tillåts för en databas i det här läget är att ta bort den.

### <a name="accidental-key-access-revocation-from-key-vault"></a>Återkallning av åtkomst till oavsiktlig nyckel från Key Vault

Det kan hända att någon med tillräckliga åtkomst rättigheter för Key Vault oavsiktligt inaktiverar Server åtkomst till nyckeln av:

* Återkalla nyckel valvets get-, wrapKey-och unwrapKey-behörigheter från servern.
* Tar bort nyckeln.
* Tar bort nyckel valvet.
* Ändra nyckel valvets brand Väggs regler.

* Tar bort den hanterade identiteten för servern i Azure AD.

## <a name="monitor-the-customer-managed-key-in-key-vault"></a>Övervaka den Kundhanterade nyckeln i Key Vault

Konfigurera följande Azure-funktioner för att övervaka databasens tillstånd och för att aktivera aviseringar för förlust av transparent data krypterings skydds åtkomst:

* [Azure Resource Health](../service-health/resource-health-overview.md): en oåtkomlig databas som har förlorat åtkomst till kund nyckeln visas som "oåtkomlig" när den första anslutningen till databasen har nekats.
* [Aktivitets logg](../service-health/alerts-activity-log-service-notifications.md): när åtkomst till kund nyckeln i den kundhanterade Key Vault Miss lyckas, läggs poster till i aktivitets loggen. Du kan återställa åtkomst så snart som möjligt, om du skapar aviseringar för dessa händelser.

* [Åtgärds grupper](../azure-monitor/platform/action-groups.md): definiera dessa för att skicka aviseringar och aviseringar baserat på dina inställningar.

## <a name="restore-and-replicate-with-a-customers-managed-key-in-key-vault"></a>Återställa och replikera med en kunds hanterade nyckel i Key Vault

När Azure Database for MySQL har krypterats med en kunds hanterade nyckel som lagras i Key Vault krypteras även alla nyligen skapade kopior av servern. Du kan göra den här nya kopian antingen via en lokal eller geo-återställning eller via Läs repliker. Kopian kan dock ändras för att avspegla en ny kunds hanterade nyckel för kryptering. När den Kundhanterade nyckeln ändras börjar gamla säkerhets kopieringar av servern med den senaste nyckeln.

För att undvika problem när du konfigurerar kundhanterad data kryptering under återställningen eller skapa en läsning av repliken är det viktigt att följa dessa steg på huvud servern och de återställda/replikerade servrarna:

* Initiera processen för att återställa eller läsa replikering från huvud Azure Database for MySQL.
* Behåll den nyligen skapade servern (återställd/replik) i ett otillgängligt tillstånd eftersom dess unika identitet ännu inte har fått behörighet att Key Vault.
* På den återställda/replik servern verifierar du om den Kundhanterade nyckeln på data krypterings inställningarna. På så sätt ser du till att den nya servern har fått behörighet att radbryta och packa upp den nyckel som lagras i Key Vault.

## <a name="next-steps"></a>Nästa steg

Lär dig hur du [konfigurerar data kryptering med en kundhanterad nyckel för Azure Database för MySQL med hjälp av Azure Portal](howto-data-encryption-portal.md).
