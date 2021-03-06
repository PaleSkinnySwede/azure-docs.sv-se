---
title: Leverantörer av moln lösningar | Azure Marketplace
description: Utgivare kan nu sälja sina erbjudanden via partner kanalen för Microsoft Cloud Solution Provider (CSP).
author: dsindona
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 04/16/2020
ms.author: dsindona
ms.openlocfilehash: b962610c585df288a9cb3297ed8e09c8abc5ac0a
ms.sourcegitcommit: be32c9a3f6ff48d909aabdae9a53bd8e0582f955
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/26/2020
ms.locfileid: "82160655"
---
# <a name="cloud-solution-providers"></a>Leverantörer av molnlösningar

Program varu erbjudanden kan uppnå miljon tals kvalificerade Microsoft-kunder som betjänas av partners i Cloud Solution Provider (CSP)-programmet, utöver offentlig tillgänglighet för erbjudanden via [Microsoft-butiker](https://docs.microsoft.com/azure/marketplace/comparing-appsource-azure-marketplace).

Utgivare konfigurerar erbjudanden för tillgänglighet i CSP-programmet för en valbarhet, för ett nytt erbjudande eller ett befintligt erbjudande, och gör det möjligt för partner att sälja dina produkter och skapa sammanställda lösningar för kunder.

Utgivare ansvarar för att tillhandahålla stöd för att ge support för att ge slutanvändarna möjlighet att ge support för partner i CSP-programmet och/eller kunder att kontakta dig. Det är en bra idé att tillhandahålla partner i CSP-programmet med användar dokumentation, utbildning och tjänst hälso-och avbrotts meddelanden (i förekommande fall) så att partner i CSP-programmet är utrustade för att hantera support förfrågningar på nivå 1 från kunder.  

Följande erbjudanden är berättigade att välja att säljas av partner i CSP-programmet:

- SaaS-erbjudanden (Software-as-a-Service)
- Virtual Machines (VM)
- Lösningsmallar
- Hanterade program

> [!NOTE]
> Behållare och ta med din egen licens (BYOL) VM SKU: er väljs att säljas av partner i CSP-programmet som standard.

## <a name="how-to-configure-an-offering"></a>Så här konfigurerar du ett erbjudande

Inställningen för att välja CSP-program konfigureras i Partner Center eller i Cloud Partner Portal skapa ett erbjudande. [Lär dig mer om den ändrade utgivarens upplevelse](https://www.microsoftpartnercommunity.com/t5/Azure-Marketplace-and-AppSource/Cloud-Marketplace-In-Partner-Center/m-p/9738#M293).

### <a name="partner-center-opt-in"></a>Deltagande i Partner Center

I Partner Center hittar du deltagande upplevelsen under modulen CSP åter försäljare.

![CSP åter försäljarens publik](media/marketplace-publishers-guide/csp-reseller-audience.png)

I Audience-modulen för CSP-åter försäljaren finns tre alternativ att välja mellan:

- Alternativ ett: alla partner i CSP-programmet
- Alternativ två: vissa partner i CSP-programmet som jag väljer
- Alternativ tre: inga partner i CSP-programmet

#### <a name="option-one-any-partner-in-the-csp-program"></a>Alternativ ett: alla partner i CSP-programmet

![Alla partner i CSP-programmet](media/marketplace-publishers-guide/csp-reseller-option-one.png)

 Genom att välja det här alternativet är alla partner i CSP-programmet berättigade till att sälja erbjudandet till sina kunder.

#### <a name="option-two-specific-partners-in-the-csp-program-i-select"></a>Alternativ två: vissa partner i CSP-programmet som jag väljer

![Vissa partner i CSP-programmet som jag väljer](media/marketplace-publishers-guide/csp-reseller-option-two.png)

Genom att välja det här alternativet godkänner du vilka partner i CSP-programmet som är berättigade att sälja erbjudandet.

Om du vill godkänna partner klickar du på **Välj CSP-partner** och en meny visas där du kan söka efter partner namn eller CSP Azure Active Directory (AAD) klient-ID.

![Välj CSP-menyn](media/marketplace-publishers-guide/csp-pop-up-module.png)

Du kan använda Sök filter, till exempel **land**, **kompetens**eller **kunskap**.

![Lands-, kompetens-och kunskaps filter för partner sökning](media/marketplace-publishers-guide/csp-add-resellers.png)

När du har valt listan över partners väljer du **Lägg till**.

![Exempel lista över behöriga partner i CSP-programmet](media/marketplace-publishers-guide/csp-add-resellers-details.png)

En tabell som visar listan över partner som du har valt visas på mål sidan för CSP-åter försäljaren.

![Tabell med lista över partner på sidan för CSP-åter försäljaren](media/marketplace-publishers-guide/csp-option-two-add-reseller.png)

Välj **Spara utkast** för att registrera dina ändringar.

Om erbjudandet är avpublicerat måste du publicera ditt erbjudande för att göra det tillgängligt för dina valda partner.

>[!NOTE]
>Om du godkänner en partner i CSP-programmet i en viss region kan de sälja erbjudandet till alla kunder som tillhör den aktuella regionen. Mer information om hur CSP-erbjudanden klassificeras finns i avsnittet om [program i Cloud Solution Provider-program, regionala marknader och valutor](https://docs.microsoft.com/partner-center/regional-authorization-overview) .

Om du uppdaterar CSP-listan för ett redan publicerat erbjudande lägger du till ytterligare partner och väljer **Synkronisera CSP-mål**.

Om du har ett erbjudande som redan har en lista över behöriga partner och du vill använda samma lista för ett annat erbjudande, använder du **import/export**. Gå till det erbjudande som innehåller CSP-listan och välj **Exportera kryptografiproviders**. Funktionen utvecklar en. csv-fil som kan importeras till ett annat erbjudande.

#### <a name="option-three-no-partners-in-the-csp-program"></a>Alternativ tre: inga partner i CSP-programmet

![Inga partner i CSP-programmet](media/marketplace-publishers-guide/csp-reseller-option-three.png)

Genom att välja det här alternativet är du väljer ditt erbjudande från CSP-programmet. Du kan ändra det här alternativet när som helst.

### <a name="cloud-partner-portal-opt-in"></a>Cloud Partner Portal Anmäl dig

I Cloud Partner Portal anges opt-in på fliken Marketplace eller butik. Möjligheten att välja vissa partner i CSP-programmet är bara tillgänglig i Partner Center.

![Upplevelse för att välja CSP i CPP](media/marketplace-publishers-guide/csp-opt-in.png)

## <a name="deauthorize-partners-in-the-csp-program"></a>Deauktorisera partner i CSP-programmet

Om du har auktoriserat en partner i CSP-programmet och den partnern redan har sålt produkten till sina kunder, kommer du inte att kunna godkänna den partnern.

Om en partner i CSP-programmet inte har sålt produkten till kunderna och du vill ta bort CSP: n när ditt erbjudande har publicerats, använder du följande anvisningar:

1. Gå till [sidan supportbegäran](https://partner.microsoft.com/support/v2/?stage=1). De första menyerna i listan fylls i automatiskt.

   > [!NOTE]
   > Ändra inte markeringarna i list rutan i förväg.

2. För **Välj produkt version väljer du hantering av** **Live-erbjudandet**.
3. För **Välj en kategori som bäst beskriver problemet**väljer du den kategori som refererar till ditt erbjudande.
4. För **Välj ett problem som bäst beskriver problemet väljer du** **uppdatera befintligt erbjudande**.
5. Välj **Nästa** för att komma till **sidan ärende information** och ange mer information om problemet.
6. Använd **deauktorisera CSP** som ärende rubrik och fyll i resten av de nödvändiga avsnitten.




## <a name="navigate-between-options"></a>Navigera mellan alternativ

Använd det här avsnittet för att navigera mellan de tre alternativen för CSP-åter försäljare.

### <a name="navigate-from-option-one-any-partner-in-the-csp-program"></a>Navigera från alternativ ett: alla partner i CSP-programmet

Om erbjudandet för närvarande är **alternativ 1: någon partner i CSP-programmet** och du vill navigera till något av de andra två alternativen, använder du följande instruktioner för att skapa en begäran:

1. Gå till [sidan supportbegäran](https://partner.microsoft.com/support/v2/?stage=1). De första menyerna i listan fylls i automatiskt.

   > [!NOTE]
   > Ändra inte markeringarna i list rutan i förväg.

2. För **Välj produkt version väljer du hantering av** **Live-erbjudandet**.
3. För **Välj en kategori som bäst beskriver problemet**väljer du den kategori som refererar till ditt erbjudande.
4. För **Välj ett problem som bäst beskriver problemet väljer du** **uppdatera befintligt erbjudande**.
5. Välj **Nästa** för att komma till **sidan ärende information** och ange mer information om problemet.
6. Använd **deauktorisera CSP** som ärende rubrik och fyll i resten av de nödvändiga avsnitten.

> [!NOTE]
> Om du försöker navigera till alternativ två och en partner i CSP-programmet redan har sålt erbjudandet till sina kunder, är den partnern som standard redan i listan över behöriga partner.  

### <a name="navigate-from-option-two-specific-partners-in-the-csp-program-i-select"></a>Navigera från alternativ två: vissa partner i CSP-programmet som jag väljer

Om erbjudandet för närvarande är **Alternativ 2: vissa partner i CSP-programmet som jag väljer** och du vill navigera till **alternativ ett: alla partner i CSP-programmet**använder följande instruktioner för att skapa en begäran:

1. Gå till [sidan supportbegäran](https://partner.microsoft.com/support/v2/?stage=1). De första menyerna i listan fylls i automatiskt.

   > [!NOTE]
   > Ändra inte markeringarna i list rutan i förväg.

2. För **Välj produkt version väljer du hantering av** **Live-erbjudandet**.
3. För **Välj en kategori som bäst beskriver problemet**väljer du den kategori som refererar till ditt erbjudande.
4. För **Välj ett problem som bäst beskriver problemet väljer du** **uppdatera befintligt erbjudande**.
5. Välj **Nästa** för att komma till **sidan ärende information** och ange mer information om problemet.
6. Använd **deauktorisera CSP** som ärende rubrik och fyll i resten av de nödvändiga avsnitten.

 Om erbjudandet för närvarande är **Alternativ 2: särskilda partner i CSP-programmet som jag väljer** och du vill navigera till **Alternativ 3: inga partner i CSP-programmet**. du kommer bara att kunna navigera till det alternativet om de partner i CSP-programmet som du tidigare har godkänt inte har sålt ditt erbjudande till slutanvändare. Använd följande instruktioner för att skapa en begäran:

1. Gå till [sidan supportbegäran](https://partner.microsoft.com/support/v2/?stage=1). De första menyerna i listan fylls i automatiskt.

   > [!NOTE]
   > Ändra inte markeringarna i list rutan i förväg.

2. För **Välj produkt version väljer du hantering av** **Live-erbjudandet**.
3. För **Välj en kategori som bäst beskriver problemet**väljer du den kategori som refererar till ditt erbjudande.
4. För **Välj ett problem som bäst beskriver problemet väljer du** **uppdatera befintligt erbjudande**.
5. Välj **Nästa** för att komma till **sidan ärende information** och ange mer information om problemet.
6. Använd **deauktorisera CSP** som ärende rubrik och fyll i resten av de nödvändiga avsnitten.

### <a name="navigate-from-option-3-no-partners-in-the-csp-program"></a>Navigera från alternativ 3: inga partner i CSP-programmet

Om erbjudandet för närvarande är **Alternativ 3: inga partner i CSP-programmet**, kan du navigera till något av de andra två alternativen när som helst.

## <a name="sharing-sales-and-support-materials-with-partners-in-the-csp-program"></a>Dela försäljnings-och support material med partner i CSP-programmet

För att hjälpa partner i Cloud Solution Provider-programmet att representera ditt erbjudande och engagera med din organisation måste du skicka in försäljnings-och support material som kommer att vara tillgängliga för åter försäljarna. De här resurserna kommer inte att exponeras för kunder i Marketplace-butiker.

### <a name="partner-center-csp-channel"></a>Partner Center CSP-kanal

Om du har valt CSP-kanalen i Partner Center måste utgivare ange en URL som är värd för relevant marknadsförings material och kanal kontakt information till CSP-kanalen under modulen för erbjudande listan:

![Information om CSP-information för partner Center](media/marketplace-publishers-guide/pc-csp-channel.png)

### <a name="cloud-partner-portal-csp-channel"></a>Cloud Partner Portal CSP-kanal

Om du har valt KRYPTOGRAFIPROVIDER-kanalen i Cloud Partner Portal måste utgivare ange en URL som är värd för relevant marknadsförings material och kanal kontakt information till CSP-kanalen:

![Information om Cloud Partner Portal CSP-säkerhet](media/marketplace-publishers-guide/cpp-csp-information.png)

## <a name="next-steps"></a>Nästa steg

Besök [Azure Marketplace och AppSource Publisher guide](https://docs.microsoft.com/azure/marketplace/marketplace-publishers-guide).

Mer information om Marketplace GTM-tjänster finns i [Go-to-Marketing-tjänster](https://partner.microsoft.com/reach-customers/gtm).

Logga in på [partner Center](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) för att skapa och konfigurera ditt erbjudande.
