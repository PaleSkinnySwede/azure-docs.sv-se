---
title: Azure-front dörr – stöd för domäner med jokertecken
description: Den här artikeln hjälper dig att förstå hur Azures front dörr stöder mappning och hantering av jokertecken i listan över anpassade domäner.
services: frontdoor
author: sharad4u
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/10/2020
ms.author: sharadag
ms.openlocfilehash: 6d8a6d6f0b05b9b7fd0144959c82b6a2c9e659a3
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81768314"
---
# <a name="wildcard-domains"></a>Domäner med jokertecken

Förutom Apex-domäner och under domäner kan du mappa ett domän namn med jokertecken till listan över klient dels värdar eller anpassade domäner i din Azures profil för klient del dörren. Att använda domäner med jokertecken i din Azures konfiguration för front dörr fören klar trafik Dirigerings beteendet för flera under domäner för ett API, program eller en webbplats från samma regel för routning. Du behöver inte ändra konfigurationen för att lägga till eller ange varje under domän separat. Som exempel kan du definiera `customer1.contoso.com`routning för, `customer2.contoso.com`och `customerN.contoso.com` med hjälp av samma regel för Routning och lägga till domänen `*.contoso.com`med jokertecken.

Viktiga scenarier som har förbättrats med stöd för domäner med jokertecken är:

- Du behöver inte publicera varje under domän i din Azures profil för front dörr och sedan Aktivera HTTPS för att binda ett certifikat för varje under domän.
- Du behöver inte längre ändra din produktions konfiguration för Azures front dörr om ett program lägger till en ny under domän. Tidigare var du tvungen att lägga till under domänen, binda ett certifikat till det, bifoga en WAF-princip (Web Application Firewall) och sedan lägga till domänen i olika regler för routning.

> [!NOTE]
> För närvarande stöds inte domäner med jokertecken via API, PowerShell och Azure CLI. Stöd för att lägga till och hantera jokertecken domäner i Azure Portal är inte tillgängligt.

## <a name="adding-wildcard-domains"></a>Lägga till domäner med jokertecken

Du kan lägga till en domän med jokertecken under avsnittet för klient dels värdar eller domäner. På samma sätt som under domäner, verifierar Azures front dörr att det finns en CNAME-Postmappning för din domän med jokertecken. Den här DNS-mappningen kan vara en direkt CNAME `*.contoso.com` -Postmappning som mappad till `contoso.azurefd.net`. Eller så kan du använda afdverify temporär mappning. Till exempel `afdverify.contoso.com` mappas för `afdverify.contoso.azurefd.net` att validera CNAME-postkartan för jokertecknet.

> [!NOTE]
> Azure DNS stöder poster med jokertecken.

Du kan lägga till så många under domäner på en nivå för domänen med jokertecken i klient dels värdar, upp till gränsen för klient dels värdar. Den här funktionen kan krävas för:

- Definiera en annan väg för en under domän än resten av domänerna (från domänen med jokertecken).

- Ha en annan WAF-princip för en särskild under domän. `*.contoso.com` Tillåter till exempel att du `foo.contoso.com` lägger till utan att behöva bevisa domän ägarskap igen. Men det är inte `foo.bar.contoso.com` tillåtet eftersom det inte är en under domän med `*.contoso.com`en nivå på. För att `foo.bar.contoso.com` lägga till utan ytterligare verifiering av `*.bar.contosonews.com` domän ägarskap måste läggas till.

Du kan lägga till domäner med jokertecken och deras under domäner med vissa begränsningar:

- Om en domän med jokertecken läggs till i en Azure-profil för front dörren:
  - Det går inte att lägga till domänen med jokertecken i någon annan Azure-profil för front dörren.
  - Det går inte att lägga till under domäner på den första nivån i domänen med jokertecken i en annan Azure-profil för front dörren eller en Azure Content Delivery Network-profil.
- Om en under domän till en domän med jokertecken läggs till i en Azure-profil för front dörr eller Azure Content Delivery Network-profil, kan inte domänen med jokertecken läggas till i andra profiler för Azures front dörr.
- Om två profiler (Azure frontend eller Azure Content Delivery Network) har olika under domäner i en rot domän, kan inte jokertecken domäner läggas till i någon av profilerna.

## <a name="certificate-binding"></a>Certifikat bindning

För att acceptera HTTPS-trafik i domänen med jokertecken måste du Aktivera HTTPS i domänen med jokertecken. Certifikat bindningen för en domän med jokertecken kräver ett certifikat med jokertecken. Det vill säga certifikatets ämnes namn bör även ha domänen med jokertecken.

> [!NOTE]
> För närvarande är det bara att använda ditt eget alternativ för anpassat SSL-certifikat tillgängligt för att aktivera HTTPS för domäner med jokertecken. Azure-hanterade certifikat för front dörren kan inte användas för domäner med jokertecken.

Du kan välja att använda samma jokertecken från Azure Key Vault eller från Azure frontend-hanterade certifikat för under domäner.

Om en under domän läggs till för en domän med jokertecken som redan har ett certifikat som är kopplat till den, kan HTTPS för under domänen inte inaktive ras. Under domänen använder certifikat bindningen för domänen med jokertecken, om inte en annan Key Vault eller Azure-hanterat certifikat för front dörren åsidosätter det.

## <a name="waf-policies"></a>WAF-principer

WAF-principer kan kopplas till domäner med jokertecken, ungefär som andra domäner. En annan WAF-princip kan tillämpas på en under domän till en domän med jokertecken. För under domänerna måste du ange den WAF-princip som ska användas, även om den är samma princip som domänen med jokertecken. Under domäner ärver *inte* automatiskt WAF-principen från domänen med jokertecken.

Om du inte vill att en WAF-princip ska köras för en under domän kan du skapa en tom WAF-princip utan några hanterade eller anpassade rulesets.

## <a name="routing-rules"></a>Dirigeringsregler

När du konfigurerar en regel för routning kan du välja en domän med jokertecken som en klient dels värd. Du kan också ha olika väg beteenden för domäner och under domäner med jokertecken. Som det beskrivs i [hur Azure-frontend använder väg matchning](front-door-route-matching.md), väljs den mest exakta matchningen för domänen över olika regler för routning vid körning.

> [!IMPORTANT]
> Du måste ha matchande Sök vägs mönster i dina routningsregler, annars kan klienterna se felen. Till exempel har du två regler för routning som Route 1 (`*.foo.com/*` mappas till backend-pool A) och routning 2 (`bar.foo.com/somePath/*` mappas till backend-pool B). Sedan skickas en begäran till `bar.foo.com/anotherPath/*`. Azures front dörr väljer väg 2 baserat på en mer bestämd domän matchning, bara för att hitta Inga matchande Sök vägs mönster över vägarna.

## <a name="next-steps"></a>Nästa steg

- Lär dig hur du [skapar en profil för Azures frontend-dörr](quickstart-create-front-door.md).
- Lär dig hur du [lägger till en anpassad domän i Azures frontend-dörr](front-door-custom-domain.md).
- Lär dig hur du [aktiverar https på en anpassad domän](front-door-custom-domain-https.md).
