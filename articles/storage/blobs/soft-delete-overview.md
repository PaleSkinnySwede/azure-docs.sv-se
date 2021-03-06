---
title: Mjuk borttagning för Blob Storage
titleSuffix: Azure Storage
description: Aktivera mjuk borttagning för BLOB-objekt så att du enkelt kan återställa dina data när de är felaktigt ändrade eller borttagna av ett program eller annan lagrings konto användare.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 04/30/2020
ms.author: tamram
ms.subservice: blobs
ms.openlocfilehash: dd5d9c721c3e0204a66367b76654f9a917e26ba6
ms.sourcegitcommit: d815163a1359f0df6ebfbfe985566d4951e38135
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 05/07/2020
ms.locfileid: "82884636"
---
# <a name="soft-delete-for-blob-storage"></a>Mjuk borttagning för Blob Storage

Mjuk borttagning skyddar BLOB-data från att oavsiktligt eller felaktigt ändras eller tas bort. När mjuk borttagning har Aktiver ATS för ett lagrings konto kan blobbar, BLOB-versioner (för hands version) och ögonblicks bilder i det lagrings kontot återställas när de har tagits bort, inom en kvarhållningsperiod som du anger.

Om det finns en risk att dina data av misstag kan ändras eller tas bort av ett program eller en annan lagrings konto användare, rekommenderar Microsoft att du aktiverar mjuk borttagning.

[!INCLUDE [updated-for-az](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="about-soft-delete"></a>Om mjuk borttagning

När mjuk borttagning är aktiverat på ett lagrings konto kan du återställa objekt när de har tagits bort, inom den angivna data lagrings perioden. Det här skyddet utökar till alla BLOB-data (block-blobbar, tillägg av blobbar och sid-blobar) som raderas som en överskrivning.

Om data i en befintlig BLOB eller ögonblicks bild tas bort när mjuk borttagning har Aktiver ATS men BLOB-versioner (för hands version) inte är aktive rad genereras en mjuk borttagen ögonblicks bild för att spara statusen för de överskrivna data. När den angivna kvarhållningsperioden har upphört att gälla tas objektet bort permanent.

Om BLOB-versioner och mjuk borttagning både är aktiverade på lagrings kontot, skapar en BLOB en ny version i stället för en mjuk borttagen ögonblicks bild. Den nya versionen är inte mjuk och tas inte bort när kvarhållningsperioden för mjuk borttagning upphör att gälla. Borttagnings bara versioner av en BLOB kan återställas inom kvarhållningsperioden genom att anropa åtgärden [ta bort BLOB](/rest/api/storageservices/undelete-blob) . Blobben kan sedan återställas från en av dess versioner genom att anropa åtgärden [Kopiera BLOB](/rest/api/storageservices/copy-blob) . Mer information om hur du använder BLOB-versioner och mjuk borttagning finns i [BLOB-versioner och mjuk borttagning](versioning-overview.md#blob-versioning-and-soft-delete).

Mjuka borttagna objekt är osynliga om de inte uttryckligen anges.

Mjuk borttagning är bakåtkompatibelt, så du behöver inte göra några ändringar i dina program för att dra nytta av de skydd som den här funktionen erbjuder. Men [Data återställning](#recovery) introducerar en ny **Undelete-BLOB** -API.

Mjuk borttagning är tillgängligt för både nya och befintliga General-Purpose v2-, General-Purpose v1-och Blob Storage-konton. Det finns stöd för både standard-och Premium konto typer. Mjuk borttagning är tillgängligt för alla lagrings nivåer, inklusive frekvent, låg frekvent och Arkiv lag ring. Mjuk borttagning är tillgängligt för ohanterade diskar, vilket är Page blobbar under försättsblad, men inte tillgängligt för hanterade diskar.

### <a name="configuration-settings"></a>Konfigurationsinställningar

När du skapar ett nytt konto är mjuk borttagning inaktiverat som standard. Mjuk borttagning inaktive ras också som standard för befintliga lagrings konton. Du kan när som helst aktivera eller inaktivera mjuk borttagning för ett lagrings konto.

När du aktiverar mjuk borttagning måste du konfigurera kvarhållningsperioden. Kvarhållningsperioden anger hur lång tid som Soft Deleted data lagras och är tillgänglig för återställning. För objekt som uttryckligen tas bort, startar kvarhållningsperioden när data tas bort. För mjuka borttagna versioner eller ögonblicks bilder som genereras av funktionen för mjuk borttagning när data skrivs över, startar klockan när versionen eller ögonblicks bilden genereras. Kvarhållningsperioden kan vara mellan 1 och 365 dagar.

Du kan när som helst ändra lagrings perioden för mjuk borttagning. En uppdaterad kvarhållningsperiod gäller endast för nyligen borttagna data. Tidigare borttagna data går ut baserat på den kvarhållningsperiod som konfigurerades när dessa data togs bort. Försök att ta bort ett Soft borttaget objekt påverkar inte förfallo tiden.

Om du inaktiverar mjuk borttagning kan du fortsätta att komma åt och återställa mjuka borttagna data i ditt lagrings konto som sparades när funktionen var aktive rad.

### <a name="saving-deleted-data"></a>Sparar borttagna data

Mjuk borttagning bevarar dina data i många fall där objekt tas bort eller skrivs över.

När en BLOB skrivs över med hjälp av **Lägg till BLOB**, **Ange blockeringslistan**eller **Kopiera BLOB**, genereras en version eller ögonblicks bild av blobens tillstånd innan Skriv åtgärden genereras automatiskt. Det här objektet är osynligt om inte mjuka borttagna objekt anges explicit. I [återställnings](#recovery) avsnittet finns information om hur du visar mjuka borttagna objekt.

![](media/soft-delete-overview/storage-blob-soft-delete-overwrite.png)

*Mjuk borttagning av data är grått, medan aktiva data är blå. Mer nyligen skrivna data visas under äldre data. När B0 skrivs över med B1 genereras en mjuk borttagen ögonblicks bild av B0. När B1 skrivs över med B2 genereras en mjuk raderad ögonblicks bild av B1.*

> [!NOTE]  
> Mjuk borttagning ger endast överskrivning av skydd för kopierings åtgärder när den är aktive rad för mål-blobens konto.

> [!NOTE]  
> Mjuk borttagning ger inte överskrivning av blobar på Arkiv nivån. Om en BLOB i arkivet skrivs över med en ny BLOB på vilken nivå som helst upphör den överskrivna blobben att gälla permanent.

När **Delete BLOB** anropas för en ögonblicks bild markeras den ögonblicks bilden som mjuk borttagen. En ny ögonblicks bild skapas inte.

![](media/soft-delete-overview/storage-blob-soft-delete-explicit-delete-snapshot.png)

*Mjuk borttagning av data är grått, medan aktiva data är blå. Mer nyligen skrivna data visas under äldre data. När en **ögonblicks bilds-BLOB** anropas blir B0 en ögonblicks bild och B1 är det aktiva läget för blobben. När B0-ögonblicksbilden tas bort markeras den som mjuk borttagen.*

När **Delete BLOB** anropas på en bas-BLOB (en blob som inte är en ögonblicks bild) markeras denna blob som mjuk borttagen. I överensstämmelse med föregående beteende, anropar **ta bort BLOB** på en blob som har aktiva ögonblicks bilder som returnerar ett fel. Anrop av **Delete BLOB** i en blob med mjuka borttagna ögonblicks bilder returnerar inte ett fel. Du kan fortfarande ta bort en blob och alla dess ögonblicks bilder i en enskild åtgärd när mjuk borttagning är aktiverat. Då markeras bas-blob och ögonblicks bilder som mjuk borttagning.

![](media/soft-delete-overview/storage-blob-soft-delete-explicit-include.png)

*Mjuk borttagning av data är grått, medan aktiva data är blå. Mer nyligen skrivna data visas under äldre data. Här görs ett **Delete BLOB** -anrop för att ta bort B2 och alla associerade ögonblicks bilder. Den aktiva blobben, B2 och alla associerade ögonblicks bilder markeras som mjuk borttagen.*

> [!NOTE]  
> När en mjuk borttagen BLOB skrivs över skapas en mjuk raderad ögonblicks bild av blobens tillstånd innan Skriv åtgärden genereras automatiskt. Den nya blobben ärver den överskrivna blobens nivå.

Mjuk borttagning sparar inte dina data i fall av container eller konto borttagningar, eller när BLOB-metadata och blob-egenskaper skrivs över. Om du vill skydda ett lagrings konto från felaktig borttagning kan du konfigurera ett lås med hjälp av Azure Resource Manager. Se Azure Resource Manager artikel [Lås resurser för att förhindra oväntade ändringar](../../azure-resource-manager/management/lock-resources.md) av mer information.

Följande tabell information förväntas när mjuk borttagning är aktiverat:

| REST API åtgärd | Resurstyp | Beskrivning | Funktions förändring |
|--------------------|---------------|-------------|--------------------|
| [Ta bort](/rest/api/storagerp/StorageAccounts/Delete) | Konto | Tar bort lagrings kontot, inklusive alla behållare och blobbar som det innehåller.                           | Ingen ändring. Behållare och blobbar i det borttagna kontot går inte att återskapa. |
| [Ta bort container](/rest/api/storageservices/delete-container) | Container | Tar bort behållaren, inklusive alla blobbar som den innehåller. | Ingen ändring. Blobbar i den borttagna behållaren går inte att återskapa. |
| [Placera blob](/rest/api/storageservices/put-blob) | Blockera, lägga till och Page blobbar | Skapar en ny BLOB eller ersätter en befintlig BLOB i en behållare | Om den används för att ersätta en befintlig BLOB genereras en ögonblicks bild av blobens tillstånd före anropet automatiskt. Detta gäller även för en tidigare mjuk borttagen BLOB om den ersätts av en blob av samma typ (block, tillägg eller sida). Om den ersätts av en blob av en annan typ, kommer alla befintliga data för mjuk borttagning att upphöra att gälla permanent. |
| [Ta bort blob](/rest/api/storageservices/delete-blob) | Blockera, lägga till och Page blobbar | Markerar en BLOB-eller BLOB-ögonblicksbild för borttagning. Blobben eller ögonblicks bilden raderas senare under skräp insamlingen | Om den används för att ta bort en BLOB-ögonblicksbild, markeras den ögonblicks bilden som mjuk borttagen. Om den används för att ta bort en BLOB markeras denna blob som mjuk borttagning. |
| [Kopiera blob](/rest/api/storageservices/copy-blob) | Blockera, lägga till och Page blobbar | Kopierar en käll-blob till en mål-BLOB i samma lagrings konto eller i ett annat lagrings konto. | Om den används för att ersätta en befintlig BLOB genereras en ögonblicks bild av blobens tillstånd före anropet automatiskt. Detta gäller även för en tidigare mjuk borttagen BLOB om den ersätts av en blob av samma typ (block, tillägg eller sida). Om den ersätts av en blob av en annan typ, kommer alla befintliga data för mjuk borttagning att upphöra att gälla permanent. |
| [Spärra block](/rest/api/storageservices/put-block) | Blockblobar | Skapar ett nytt block som ska allokeras som en del av en Block-Blob. | Om det används för att genomföra ett block till en blob som är aktiv, sker ingen ändring. Om den används för att genomföra ett block till en blob som är mjuk borttagning, skapas en ny blob och en ögonblicks bild skapas automatiskt för att avbilda statusen för den mjuka borttagna blobben. |
| [Lista över blockerade](/rest/api/storageservices/put-block-list) | Blockblobar | Genomför en BLOB genom att ange den uppsättning block-ID: n som utgör block-bloben. | Om den används för att ersätta en befintlig BLOB genereras en ögonblicks bild av blobens tillstånd före anropet automatiskt. Detta gäller även för en tidigare mjuk borttagen BLOB om det är en Block-Blob. Om den ersätts av en blob av en annan typ, kommer alla befintliga data för mjuk borttagning att upphöra att gälla permanent. |
| [Placerings sida](/rest/api/storageservices/put-page) | Sidblobbar | Skriver ett intervall med sidor till en sid-blob. | Ingen ändring. Sid-BLOB-data som skrivs över eller rensas med den här åtgärden sparas inte och går inte att återskapa. |
| [Lägg till block](/rest/api/storageservices/append-block) | Tilläggsblobar | Skriver ett data block till slutet av en tilläggs-BLOB | Ingen ändring. |
| [Ange BLOB-egenskaper](/rest/api/storageservices/set-blob-properties) | Blockera, lägga till och Page blobbar | Anger värden för system egenskaper som definierats för en blob. | Ingen ändring. Det går inte att återskapa de överskrivna BLOB-egenskaperna. |
| [Ange BLOB-metadata](/rest/api/storageservices/set-blob-metadata) | Blockera, lägga till och Page blobbar | Anger användardefinierade metadata för angiven blob som ett eller flera namn/värde-par. | Ingen ändring. Överskrivna BLOB-metadata går inte att återskapa. |

Det är viktigt att Observera att anrops **sidan** för att skriva över eller rensa intervall av en sid-BLOB inte automatiskt genererar ögonblicks bilder. Virtuella dator diskar backas upp av Page blobbar och använder **sidan sätt** för att skriva data.

### <a name="recovery"></a>Återställning

Om du anropar [Undelete-blobben](/rest/api/storageservices/undelete-blob) i en mjuk borttagen bas-BLOB återställs den och alla tillhör ande mjuka borttagna ögonblicks bilder aktive rad. Om du anropar åtgärden **ångra borttagning av BLOB** i en aktiv bas-BLOB återställs alla tillhör ande mjuka borttagna ögonblicks bilder som aktiva. När ögonblicks bilder återställs som aktiva ser de ut som användardefinierade ögonblicks bilder. de skriver inte över bas-bloben.

Om du vill återställa en blob till en angiven mjuk, borttagen ögonblicks bild kan du anropa **Undelete BLOB** på bas-bloben. Sedan kan du kopiera ögonblicks bilden över den nu aktiva blobben. Du kan också kopiera ögonblicks bilden till en ny blob.

![](media/soft-delete-overview/storage-blob-soft-delete-recover.png)

*Mjuk borttagning av data är grått, medan aktiva data är blå. Mer nyligen skrivna data visas under äldre data. Här kallas **Undelete BLOB** i BLOB B, vilket återställer bas-bloben, B1 och alla associerade ögonblicks bilder, precis B0, som aktiva. I det andra steget kopieras B0 över bas-bloben. Den här kopierings åtgärden genererar en mjuk borttagen ögonblicks bild av B1.*

Om du vill visa mjuka borttagna blobbar och blob-ögonblicksbilder kan du välja att inkludera borttagna data i **list blobbar**. Du kan välja att bara Visa mjuka borttagna bas-blobar, eller inkludera mjuka borttagna BLOB-ögonblicksbilder. För all mjuk borttagning av data kan du Visa den tid då data togs bort samt antalet dagar innan data kommer att upphöra att gälla permanent.

### <a name="example"></a>Exempel

Följande är konsol resultatet av ett .NET-skript som laddar upp, skriver över, ögonblicks bilder, tar bort och återställer en blob med namnet *HelloWorld* när mjuk borttagning är aktiverat:

```bash
Upload:
- HelloWorld (is soft deleted: False, is snapshot: False)

Overwrite:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Snapshot:
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Delete (including snapshots):
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: False)

Undelete:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)

Copy a snapshot over the base blob:
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: True)
- HelloWorld (is soft deleted: True, is snapshot: True)
- HelloWorld (is soft deleted: False, is snapshot: False)
```

I avsnittet [Nästa steg](#next-steps) visas en pekare till det program som skapade utdata.

## <a name="pricing-and-billing"></a>Priser och fakturering

Alla mjuka borttagna data debiteras enligt samma taxa som aktiva data. Du kommer inte att debiteras för data som tas bort permanent efter den konfigurerade kvarhållningsperioden. En djupare inblick i ögonblicks bilder och hur de påförs debiteras finns i [förstå hur ögonblicks bilder debiteras](storage-blob-snapshots.md).

Du debiteras inte för transaktioner som rör automatisk generering av ögonblicks bilder. Du faktureras för att **ta bort BLOB** -transaktioner enligt priset för Skriv åtgärder.

Mer information om priser för Azure Blob Storage i allmänhet finns på [sidan med priser för azure Blob Storage](https://azure.microsoft.com/pricing/details/storage/blobs/).

När du först aktiverar mjuk borttagning rekommenderar Microsoft att du använder en kort kvarhållningsperiod för att bättre förstå hur funktionen kommer att påverka din faktura.

Om du aktiverar mjuk borttagning för ofta överskrivna data kan det leda till ökade kapacitets kostnader för lagring och ökad fördröjning vid registrering av blobbar. Du kan minimera denna ytterligare kostnad och svars tid genom att lagra ofta skrivna data i ett separat lagrings konto där mjuk borttagning har inaktiverats.

## <a name="faq"></a>VANLIGA FRÅGOR OCH SVAR

### <a name="can-i-use-the-set-blob-tier-api-to-tier-blobs-with-soft-deleted-snapshots"></a>Kan jag använda API-API: et för att ta bort blobar med mjuk borttagna ögonblicks bilder?

Ja. De mjuka borttagna ögonblicks bilderna blir kvar på den ursprungliga nivån, men bas-bloben flyttas till den nya nivån.

### <a name="premium-storage-accounts-have-a-per-blob-snapshot-limit-of-100-do-soft-deleted-snapshots-count-toward-this-limit"></a>Premium Storage-konton har en gräns per BLOB-ögonblicksbild på 100. Räknas den mjuka borttagna ögonblicks bilder mot den här gränsen?

Nej, mjuka borttagna ögonblicks bilder räknas inte mot den här gränsen.

### <a name="if-i-delete-an-entire-account-or-container-with-soft-delete-turned-on-will-all-associated-blobs-be-saved"></a>Om jag tar bort ett helt konto eller en behållare med mjuk borttagning aktiverat, kommer alla associerade blobbar att sparas?

Nej, om du tar bort ett helt konto eller en behållare tas alla kopplade blobbar bort permanent. Mer information om hur du skyddar ett lagrings konto från oavsiktliga borttagningar finns i [Lås resurser för att förhindra oväntade ändringar](../../azure-resource-manager/management/lock-resources.md).

### <a name="can-i-view-capacity-metrics-for-deleted-data"></a>Kan jag Visa kapacitets mått för borttagna data?

Soft Deleted data ingår som en del av din totala kapacitet för lagrings kontot. Mer information om att spåra och övervaka lagrings kapacitet finns [Lagringsanalys](../common/storage-analytics.md).

### <a name="can-i-read-and-copy-out-soft-deleted-snapshots-of-my-blob"></a>Kan jag läsa och kopiera mjuk borttagna ögonblicks bilder av min BLOB?  

Ja, men du måste anropa Undelete i blobben först.

### <a name="is-soft-delete-available-for-virtual-machine-disks"></a>Är mjuk borttagning tillgängligt för virtuella dator diskar?  

Mjuk borttagning är tillgängligt för både Premium-och ohanterade icke-hanterade diskar, som är sid-blobar under försättsblad. Med mjuk borttagning kan du bara återställa data som tagits bort genom att **ta bort BLOB**, **Skicka BLOB**, **lagra blockeringslistan**och **Kopiera BLOB** -åtgärder. Data som skrivs över av ett anrop till en **sida** kan inte återställas.

En virtuell Azure-dator skriver till en ohanterad disk med hjälp av anrop till att använda **sidan**, så Använd mjuk borttagning för att ångra skrivningar till en ohanterad disk från en virtuell Azure-dator är inte ett scenario som stöds.

### <a name="do-i-need-to-change-my-existing-applications-to-use-soft-delete"></a>Måste jag ändra mina befintliga program för att använda mjuk borttagning?

Det är möjligt att dra nytta av mjuk borttagning oavsett vilken API-version du använder. Men om du vill visa och återställa mjuka borttagna blobbar och blob-ögonblicksbilder måste du använda version 2017-07-29 av [Azure Storage REST API](/rest/api/storageservices/Versioning-for-the-Azure-Storage-Services) eller senare. Microsoft rekommenderar att alltid använda den senaste versionen av Azure Storage API.

## <a name="next-steps"></a>Nästa steg

- [Aktivera mjuk borttagning för blobbar](soft-delete-enable.md)
- [Blob-version (för hands version)](versioning-overview.md)
