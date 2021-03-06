---
title: Lägga till och hantera TLS/SSL-certifikat
description: Skapa ett kostnads fritt certifikat, importera ett App Service certifikat, importera ett Key Vault certifikat eller köp ett App Service-certifikat i Azure App Service.
tags: buy-ssl-certificates
ms.topic: tutorial
ms.date: 10/25/2019
ms.reviewer: yutlin
ms.custom: seodec18
ms.openlocfilehash: 9f9fcc0b3d8dfe19961668e77da91bc9f14ff2d1
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "81453914"
---
# <a name="add-a-tlsssl-certificate-in-azure-app-service"></a>Lägg till ett TLS/SSL-certifikat i Azure App Service

[Azure App Service](overview.md) ger en mycket skalbar och automatisk korrigering av webb värd tjänst. Den här artikeln visar hur du skapar, överför eller importerar ett privat certifikat eller ett offentligt certifikat till App Service. 

När certifikatet har lagts till i din App Service app eller [Function-app](https://docs.microsoft.com/azure/azure-functions/)kan du [skydda ett anpassat DNS-namn med det](configure-ssl-bindings.md) eller [använda det i din program kod](configure-ssl-certificate-in-code.md).

I följande tabell visas de alternativ som du har för att lägga till certifikat i App Service:

|Alternativ|Beskrivning|
|-|-|
| Skapa ett kostnads fritt App Service-hanterat certifikat (förhands granskning) | Ett privat certifikat som är enkelt att använda om du bara behöver skydda din `www` [anpassade domän](app-service-web-tutorial-custom-domain.md) eller någon annan icke-blott domän i App Service. |
| Köp ett App Service-certifikat | Ett privat certifikat som hanteras av Azure. Den kombinerar den automatiserade certifikat hanteringen och flexibiliteten i förnyelse-och export alternativen. |
| Importera ett certifikat från Key Vault | Användbart om du använder [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/) för att hantera dina [PKCS12-certifikat](https://wikipedia.org/wiki/PKCS_12). Se [krav för privata certifikat](#private-certificate-requirements). |
| Ladda upp ett privat certifikat | Om du redan har ett privat certifikat från en tredje part kan du ladda upp det. Se [krav för privata certifikat](#private-certificate-requirements). |
| Ladda upp ett offentligt certifikat | Offentliga certifikat används inte för att skydda anpassade domäner, men du kan läsa in dem i koden om du behöver dem för att få åtkomst till fjär resurser. |

## <a name="prerequisites"></a>Krav

För att följa den här instruktions guiden:

- [Skapa en app service-app](/azure/app-service/).
- Endast kostnads fria certifikat: mappa en under domän (till exempel `www.contoso.com`) för att app service med en [CNAME-post](app-service-web-tutorial-custom-domain.md#map-a-cname-record).

## <a name="private-certificate-requirements"></a>Krav för privata certifikat

> [!NOTE]
> Azure Web Apps har **inte** stöd för AES256 och alla PFX-filer bör krypteras med TripleDES.

Det [kostnads fria app service-hanterade certifikatet](#create-a-free-certificate-preview) eller [app Services certifikatet](#import-an-app-service-certificate) uppfyller redan kraven i App Service. Om du väljer att överföra eller importera ett privat certifikat till App Service måste certifikatet uppfylla följande krav:

* Exporterad som en [lösenordsskyddad PFX-fil](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Certificate_filename_extensions)
* Innehålla en privat nyckel som är minst 2 048 bitar lång
* Innehålla alla mellanliggande certifikat i certifikatkedjan

För att skydda en anpassad domän i en TLS-bindning har certifikatet ytterligare krav:

* Innehåller en [utökad nyckel användning](https://en.wikipedia.org/w/index.php?title=X.509&section=4#Extensions_informing_a_specific_usage_of_a_certificate) för SERVERAUTENTISERING (OID = 1.3.6.1.5.5.7.3.1)
* Vara signerat av en betrodd certifikatutfärdare

> [!NOTE]
> **ECC-certifikat (Elliptic Curve Cryptography)** kan fungera med App Service, men behandlas inte i den här artikeln. Din certifikatutfärdare kan visa hur du skapar ECC-certifikat.

[!INCLUDE [Prepare your web app](../../includes/app-service-ssl-prepare-app.md)]

## <a name="create-a-free-certificate-preview"></a>Skapa ett kostnads fritt certifikat (förhands granskning)

Det kostnads fria App Service-hanterade certifikatet är en lösning för att skydda ditt anpassade DNS-namn i App Service. Det är ett fullständigt fungerande TLS/SSL-certifikat som hanteras av App Service och förnyas automatiskt. Det kostnads fria certifikatet levereras med följande begränsningar:

- Har inte stöd för certifikat med jokertecken.
- Stöder inte blott-domäner.
- Kan inte exporteras.
- Stöder inte DNS A-poster.

> [!NOTE]
> Det kostnads fria certifikatet utfärdas av DigiCert. För vissa toppnivå domäner måste du uttryckligen tillåta DigiCert som en certifikat utfärdare genom att skapa en [CAA-domän post](https://wikipedia.org/wiki/DNS_Certification_Authority_Authorization) med värdet: `0 issue digicert.com`.
> 

Så här skapar du ett kostnads fritt App Service-hanterat certifikat:

I <a href="https://portal.azure.com" target="_blank">Azure Portal</a>väljer du **app Services** > **\<App-Name>** på menyn till vänster.

Välj **TLS/SSL-inställningar** > **privat nyckel certifikat (. pfx)** > i den vänstra navigeringen i din app.**Skapa App Service hanterat certifikat**.

![Skapa ett kostnads fritt certifikat i App Service](./media/configure-ssl-certificate/create-free-cert.png)

En icke-blott domän som är korrekt mappad till din app med en CNAME-post visas i dialog rutan. Välj den anpassade domän som du vill skapa ett kostnads fritt certifikat för och välj **skapa**. Du kan bara skapa ett certifikat för varje anpassad domän som stöds.

När åtgärden har slutförts visas certifikatet i listan med certifikat för **privat nyckel** .

![Ett kostnads fritt certifikat har skapats](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Om du vill skydda en anpassad domän med det här certifikatet måste du fortfarande skapa en certifikat bindning. Följ stegen i [skapa bindning](configure-ssl-bindings.md#create-binding).
>

## <a name="import-an-app-service-certificate"></a>Importera en App Service Certificate

Om du köper en App Service Certificate från Azure, hanterar Azure följande aktiviteter:

- Tar hand om inköps processen från GoDaddy.
- Utför domän verifiering av certifikatet.
- Underhåller certifikatet i [Azure Key Vault](../key-vault/general/overview.md).
- Hanterar certifikat förnyelse (se [Förnya certifikat](#renew-certificate)).
- Synkronisera certifikatet automatiskt med de importerade kopiorna i App Service appar.

Om du vill köpa ett App Service certifikat går du till [Starta certifikat ordning](#start-certificate-order).

Om du redan har ett arbets App Service certifikat kan du:

- [Importera certifikatet till App Service](#import-certificate-into-app-service).
- [Hantera certifikatet](#manage-app-service-certificates), till exempel förnya, uppdatera nycklar och exportera det.

### <a name="start-certificate-order"></a>Starta certifikat ordning

Starta en App Service certifikat ordning på <a href="https://portal.azure.com/#create/Microsoft.SSL" target="_blank">sidan App Service Certificate skapa</a>.

![Starta App Service certifikat Köp](./media/configure-ssl-certificate/purchase-app-service-cert.png)

Använd följande tabell som hjälp för att konfigurera certifikatet. Klicka på **Skapa** när du är klar.

| Inställningen | Beskrivning |
|-|-|
| Name | Ett eget namn på ditt App Service certifikat. |
| Värddator namn för blott-domän | Ange rot domänen här. Det utfärdade certifikatet skyddar *både* rot domänen och under `www` domänen. I det utfärdade certifikatet innehåller fältet eget namn rot domänen och fältet Alternativt namn på certifikat mottagare innehåller `www` domänen. Om du bara vill skydda en under domän anger du det fullständigt kvalificerade domän namnet för under domänen här (till exempel `mysubdomain.contoso.com`).|
| Prenumeration | Den prenumeration som ska innehålla certifikatet. |
| Resursgrupp | Den resurs grupp som ska innehålla certifikatet. Du kan använda en ny resurs grupp eller välja samma resurs grupp som App Service-appen, till exempel. |
| Certifikat-SKU | Bestämmer vilken typ av certifikat som ska skapas, om ett standard certifikat eller ett [jokertecken](https://wikipedia.org/wiki/Wildcard_certificate). |
| Juridiska villkor | Klicka för att bekräfta att du godkänner de juridiska villkoren. Certifikaten hämtas från GoDaddy. |

### <a name="store-in-azure-key-vault"></a>Lagra i Azure Key Vault

När certifikat köpet har slutförts finns det några fler steg du måste slutföra innan du kan börja använda det här certifikatet. 

Välj certifikatet på sidan [app service certifikat](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) och klicka sedan på **certifikat konfiguration** > **steg 1: Store**.

![Konfigurera Key Vault lagring av App Service certifikat](./media/configure-ssl-certificate/configure-key-vault.png)

[Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) är en Azure-tjänst som hjälper till att skydda kryptografiska nycklar och hemligheter som används av moln program och-tjänster. Det är det lagrings utrymme som du väljer för App Service certifikat.

På sidan **Key Vault status** klickar du på **Key Vault lagrings plats** för att skapa ett nytt valv eller välja ett befintligt valv. Om du väljer att skapa ett nytt valv använder du följande tabell som hjälp för att konfigurera valvet och klicka på Skapa. Skapa den nya Key Vault inuti samma prenumeration och resurs grupp som din App Service-app.

| Inställningen | Beskrivning |
|-|-|
| Name | Ett unikt namn som består av alfanumeriska tecken och bindestreck. |
| Resursgrupp | Som en rekommendation väljer du samma resurs grupp som ditt App Service certifikat. |
| Plats | Välj samma plats som App Service-appen. |
| Prisnivå | Mer information finns [Azure Key Vault pris information](https://azure.microsoft.com/pricing/details/key-vault/). |
| Åtkomstprinciper| Definierar program och tillåten åtkomst till valv resurserna. Du kan konfigurera den senare genom att följa stegen i [bevilja flera program åtkomst till ett nyckel valv](../key-vault/general/group-permissions-for-apps.md). |
| Virtual Network åtkomst | Begränsa valv åtkomst till vissa virtuella Azure-nätverk. Du kan konfigurera den senare genom att följa stegen i [konfigurera Azure Key Vault brand väggar och virtuella nätverk](../key-vault/general/network-security.md) |

När du har valt valvet stänger du sidan **Key Vault-lagringsplats** . **Steg 1: Store** -alternativet ska visa en grön bock markering för lyckad. Låt sidan vara öppen för nästa steg.

### <a name="verify-domain-ownership"></a>Verifiera domän ägarskap

På sidan samma **certifikat konfiguration** som du använde i det sista steget klickar du på **steg 2: verifiera**.

![Verifiera domän för App Service certifikat](./media/configure-ssl-certificate/verify-domain.png)

Välj **App Service verifiering**. Eftersom du redan har mappat domänen till din webbapp (se [krav](#prerequisites)) är den redan verifierad. Klicka bara på **Verifiera** för att slutföra det här steget. Klicka på knappen **Uppdatera** tills meddelande **certifikatet är domän verifierat** visas.

> [!NOTE]
> Fyra typer av domän verifierings metoder stöds: 
> 
> - **App Service** – det enklaste alternativet när domänen redan är mappad till en app service-app i samma prenumeration. Det drar nytta av det faktum att App Service-appen redan har verifierat domänens ägarskap.
> - **Domän** – verifiera en [App Service-domän som du har köpt från Azure](manage-custom-dns-buy-domain.md). Azure lägger automatiskt till verifierings-TXT-posten åt dig och slutför processen.
> - **E-post** – verifiera domänen genom att skicka ett e-postmeddelande till domän administratören. Instruktioner finns när du väljer alternativet.
> - **Manuellt** – verifiera domänen genom att antingen använda en HTML-sida (endast**standard** certifikat) eller en DNS TXT-post. Instruktioner finns när du väljer alternativet.

### <a name="import-certificate-into-app-service"></a>Importera certifikat till App Service

I <a href="https://portal.azure.com" target="_blank">Azure Portal</a>väljer du **app Services** > **\<App-Name>** på menyn till vänster.

Välj **TLS/SSL-inställningar** > **privat nyckel certifikat (. pfx)** > **Importera App Service Certificate**i det vänstra navigerings fältet.

![Importera App Service certifikat i App Service](./media/configure-ssl-certificate/import-app-service-cert.png)

Välj det certifikat som du just har köpt och välj **OK**.

När åtgärden har slutförts visas certifikatet i listan med certifikat för **privat nyckel** .

![Import av App Service certifikat avslutad](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> Om du vill skydda en anpassad domän med det här certifikatet måste du fortfarande skapa en certifikat bindning. Följ stegen i [skapa bindning](configure-ssl-bindings.md#create-binding).
>

## <a name="import-a-certificate-from-key-vault"></a>Importera ett certifikat från Key Vault

Om du använder Azure Key Vault för att hantera dina certifikat kan du importera ett PKCS12-certifikat från Key Vault till App Service så länge det [uppfyller kraven](#private-certificate-requirements).

I <a href="https://portal.azure.com" target="_blank">Azure Portal</a>väljer du **app Services** > **\<App-Name>** på menyn till vänster.

Välj **TLS/SSL-inställningar** > **privat nyckel certifikat (. pfx)** > **Importera Key Vault certifikat**i det vänstra navigerings fältet i appen.

![Importera Key Vault certifikat i App Service](./media/configure-ssl-certificate/import-key-vault-cert.png)

Använd följande tabell för att få hjälp att välja certifikatet.

| Inställningen | Beskrivning |
|-|-|
| Prenumeration | Den prenumeration som Key Vault tillhör. |
| Key Vault | Valvet med det certifikat som du vill importera. |
| Certifikat | Välj i listan över PKCS12-certifikat i valvet. Alla PKCS12-certifikat i valvet visas med deras tumavtrycken, men alla stöds inte i App Service. |

När åtgärden har slutförts visas certifikatet i listan med certifikat för **privat nyckel** . Om importen Miss lyckas med ett fel uppfyller certifikatet inte [kraven för App Service](#private-certificate-requirements).

![Import av Key Vault certifikat avslutad](./media/configure-ssl-certificate/import-app-service-cert-finished.png)

> [!IMPORTANT] 
> Om du vill skydda en anpassad domän med det här certifikatet måste du fortfarande skapa en certifikat bindning. Följ stegen i [skapa bindning](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-private-certificate"></a>Ladda upp ett privat certifikat

När du har fått ett certifikat från din certifikat leverantör följer du stegen i det här avsnittet för att göra det klart för App Service.

### <a name="merge-intermediate-certificates"></a>Sammanfoga mellanliggande certifikat

Om du får flera certifikat av din certifikatutfärdare i certifikatkedjan, måste du sammanfoga certifikaten i ordning.

Gör detta genom att öppna varje certifikat som du fick i ett redigeringsprogram.

Skapa en fil för det sammanfogade certifikatet med namnet _mergedcertificate.crt_. I redigeringsprogrammet kopierar du innehållet i varje certifikat till den här filen. Ordningen på dina certifikat ska följa ordningen i certifikatkedjan, först med ditt certifikat och sist med rotcertifikatet. Det ser ut som i följande exempel:

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

### <a name="export-certificate-to-pfx"></a>Exportera certifikat till PFX

Exportera det kopplade TLS/SSL-certifikatet med den privata nyckel som din certifikatbegäran genererades med.

Om du genererade din certifikatbegäran med hjälp av OpenSSL, har du skapat en privat nyckelfil. Kör följande kommando för att exportera certifikatet till PFX. Ersätt plats hållarna för den _ &lt;privata nyckel filen>_ och _ &lt;sammanfogade certifikat fil>_ med Sök vägarna till din privata nyckel och den sammanslagna certifikat filen.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>  
```

När du uppmanas till det anger du ett exportlösenord. Du använder det här lösen ordet när du överför TLS/SSL-certifikatet till App Service senare.

Om du använder IIS eller _Certreq.exe_ till att generera din certifikatbegäran, installerar du certifikatet på den lokala datorn och [exporterar sedan certifikatet till PFX](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

### <a name="upload-certificate-to-app-service"></a>Ladda upp certifikat till App Service

Nu är du klar med att ladda upp certifikatet till App Service.

I <a href="https://portal.azure.com" target="_blank">Azure Portal</a>väljer du **app Services** > **\<App-Name>** på menyn till vänster.

Välj **TLS/SSL-inställningar** > **certifikat för privat nyckel certifikat (. pfx)** > i den vänstra navigeringen i**appen.**

![Ladda upp privat certifikat i App Service](./media/configure-ssl-certificate/upload-private-cert.png)

I **PFX-certifikatsfil** väljer du din PFX-fil. I **Certifikatslösenord** skriver du lösenordet som du skapade när du exporterade PFX-filen. När du är färdig klickar du på **överför**. 

När åtgärden har slutförts visas certifikatet i listan med certifikat för **privat nyckel** .

![Uppladdningen av certifikat avslutad](./media/configure-ssl-certificate/create-free-cert-finished.png)

> [!IMPORTANT] 
> Om du vill skydda en anpassad domän med det här certifikatet måste du fortfarande skapa en certifikat bindning. Följ stegen i [skapa bindning](configure-ssl-bindings.md#create-binding).
>

## <a name="upload-a-public-certificate"></a>Ladda upp ett offentligt certifikat

Offentliga certifikat stöds i *. cer* -format. 

I <a href="https://portal.azure.com" target="_blank">Azure Portal</a>väljer du **app Services** > **\<App-Name>** på menyn till vänster.

Klicka på **TLS/SSL-inställningar** > **offentliga certifikat (. cer)** > **Ladda upp certifikat för offentlig nyckel**i den vänstra navigeringen i appen.

I **namn**anger du ett namn för certifikatet. I **CER-certifikatfil**väljer du CER-filen.

Klicka på **Överför**.

![Ladda upp ett offentligt certifikat i App Service](./media/configure-ssl-certificate/upload-public-cert.png)

När certifikatet har laddats upp kopierar du tumavtrycket för certifikatet och ser [till att det blir tillgängligt för certifikatet](configure-ssl-certificate-in-code.md#make-the-certificate-accessible).

## <a name="manage-app-service-certificates"></a>Hantera App Service certifikat

I det här avsnittet visas hur du hanterar ett App Service-certifikat som du har köpt i [Importera ett app service certifikat](#import-an-app-service-certificate).

- [Nyckel förnyelse certifikat](#rekey-certificate)
- [Förnya certifikat](#renew-certificate)
- [Exportera certifikatet](#export-certificate)
- [Ta bort certifikat](#delete-certificate)

### <a name="rekey-certificate"></a>Nyckel förnyelse certifikat

Om du tror att certifikatets privata nyckel har komprometterats kan du uppdatera ditt certifikat. Välj certifikatet på sidan [app service certifikat](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) och välj sedan nyckel uppdatering **och synkronisera** i det vänstra navigerings fältet.

Starta processen genom att klicka på **nyckel förnyelse** . Den här processen kan ta 1-10 minuter att slutföra.

![Nyckel förnyelse ett App Service-certifikat](./media/configure-ssl-certificate/rekey-app-service-cert.png)

Omnyckelering av certifikatet rullar certifikatet med ett nytt certifikat utfärdat av certifikat utfärdaren.

När uppdaterings åtgärden har slutförts klickar du på **Synkronisera**. Synkroniseringsåtgärden uppdaterar automatiskt värd namns bindningarna för certifikatet i App Service utan att orsaka avbrott i dina appar.

> [!NOTE]
> Om du inte klickar på **Synkronisera**synkroniserar App Service automatiskt ditt certifikat inom 48 timmar.

### <a name="renew-certificate"></a>Förnya certifikat

Om du vill aktivera automatisk förnyelse av certifikatet väljer du certifikatet på sidan [app service certifikat](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) och klickar sedan på **Inställningar för automatisk förnyelse** i det vänstra navigerings fältet. App Service certifikat har som standard en giltighets tid på ett år.

Välj **på** och klicka på **Spara**. Certifikat kan börja förnyas automatiskt 60 dagar före förfallo datum om du har aktiverat automatisk förnyelse.

![Förnya App Service certifikat automatiskt](./media/configure-ssl-certificate/auto-renew-app-service-cert.png)

Om du vill förnya certifikatet manuellt i stället klickar du på **manuell förnyelse**. Du kan begära att förnya ditt certifikat 60 dagar manuellt innan det upphör att gälla.

När förnyelse åtgärden är klar klickar du på **Synkronisera**. Synkroniseringsåtgärden uppdaterar automatiskt värd namns bindningarna för certifikatet i App Service utan att orsaka avbrott i dina appar.

> [!NOTE]
> Om du inte klickar på **Synkronisera**synkroniserar App Service automatiskt ditt certifikat inom 48 timmar.

### <a name="export-certificate"></a>Exportera certifikatet

Eftersom en App Service Certificate är en [Key Vault hemlighet](../key-vault/about-keys-secrets-and-certificates.md#key-vault-secrets), kan du exportera en PFX-kopia av den och använda den för andra Azure-tjänster eller utanför Azure.

Om du vill exportera App Service Certificate som en PFX-fil kör du följande kommandon i [Cloud Shell](https://shell.azure.com). Du kan också köra det lokalt om du har [installerat Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli). Ersätt plats hållarna med de namn som du använde när du [skapade App Service-certifikatet](#start-certificate-order).

```azurecli-interactive
secretname=$(az resource show \
    --resource-group <group-name> \
    --resource-type "Microsoft.CertificateRegistration/certificateOrders" \
    --name <app-service-cert-name> \
    --query "properties.certificates.<app-service-cert-name>.keyVaultSecretName" \
    --output tsv)

az keyvault secret download \
    --file appservicecertificate.pfx \
    --vault-name <key-vault-name> \
    --name $secretname \
    --encoding base64
```

Den hämtade *appservicecertificate. pfx* -filen är en RAW PKCS12-fil som innehåller både offentliga och privata certifikat. I varje prompt använder du en tom sträng för import lösen ordet och PEM-pass frasen.

### <a name="delete-certificate"></a>Ta bort certifikat 

Att ta bort ett App Service certifikat är slutgiltigt och oåterkalleligt. Borttagning av en App Service Certificate resurs resulterar i att certifikatet återkallas. Alla bindningar i App Service med det här certifikatet blir ogiltiga. För att förhindra oavsiktlig borttagning sätter Azure ett lås på certifikatet. Om du vill ta bort ett App Service-certifikat måste du först ta bort borttagnings låset för certifikatet.

Välj certifikatet på sidan [app service certifikat](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.CertificateRegistration%2FcertificateOrders) och välj sedan **Lås** i det vänstra navigerings fältet.

Hitta låset på certifikatet med lås typen **ta bort**. Till höger om det väljer du **ta bort**.

![Ta bort lås för App Service certifikat](./media/configure-ssl-certificate/delete-lock-app-service-cert.png)

Nu kan du ta bort App Service-certifikatet. Välj **Översikt** > **ta bort**i det vänstra navigerings fältet. I bekräftelse dialog rutan skriver du in certifikat namnet och väljer **OK**.

## <a name="automate-with-scripts"></a>Automatisera med skript

### <a name="azure-cli"></a>Azure CLI

[!code-azurecli[main](../../cli_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.sh?highlight=3-5 "Bind a custom TLS/SSL certificate to a web app")] 

### <a name="powershell"></a>PowerShell

[!code-powershell[main](../../powershell_scripts/app-service/configure-ssl-certificate/configure-ssl-certificate.ps1?highlight=1-3 "Bind a custom TLS/SSL certificate to a web app")]

## <a name="more-resources"></a>Fler resurser

* [Skydda ett anpassat DNS-namn med en TLS/SSL-bindning i Azure App Service](configure-ssl-bindings.md)
* [Använda HTTPS](configure-ssl-bindings.md#enforce-https)
* [Använda TLS 1.1/1.2](configure-ssl-bindings.md#enforce-tls-versions)
* [Använd ett TLS/SSL-certifikat i koden i Azure App Service](configure-ssl-certificate-in-code.md)
* [Vanliga frågor och svar: App Service certifikat](https://docs.microsoft.com/azure/app-service/faq-configuration-and-management/)
