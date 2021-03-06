---
author: rothja
ms.service: cost-management-billing
ms.topic: include
ms.date: 04/21/2020
ms.author: jroth
ms.openlocfilehash: 8247b1cedc2c5ebc8577af6be485aed0fcd5d6af
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81768771"
---
### <a name="key-transactions-maximum-transactions-allowed-in-10-seconds-per-vault-per-regionsup1sup"></a>Nyckel transaktioner (maximalt antal transaktioner som tillåts om 10 sekunder, per valv per region<sup>1</sup>):

|Nyckeltyp|HSM-nyckel<br>SKAPA nyckel|HSM-nyckel<br>Alla andra transaktioner|Program varu nyckel<br>SKAPA nyckel|Program varu nyckel<br>Alla andra transaktioner|
|:---|---:|---:|---:|---:|
|RSA 2 048-bitars|5|1,000|10|2 000|
|RSA 3 072-bitars|5|250|10|500|
|RSA 4 096-bitars|5|125|10|250|
|ECC P-256|5|1,000|10|2 000|
|ECC P-384|5|1,000|10|2 000|
|ECC P-521|5|1,000|10|2 000|
|ECC-SECP256K1|5|1,000|10|2 000|

> [!NOTE]
> I den föregående tabellen ser vi att för RSA 2 048-bitars program varu nycklar tillåts 2 000 Hämta transaktioner per 10 sekunder. För RSA 2 048-bitars HSM-Keys tillåts 1 000 Hämta transaktioner per 10 sekunder.
>
> Tröskelvärdena för begränsningen viktas och tvången är på deras summa. Som du ser i föregående tabell när du utför åtgärder på RSA HSM-nycklar, är det till exempel åtta gånger dyrare att använda 4 096-bitars nycklar jämfört med 2 048-bitars nycklar. Det beror på att 1000/125 = 8.
>
> I ett angivet intervall på 10 sekunder kan en Azure Key Vault-klient *bara utföra en* av följande åtgärder innan den påträffar en `429` begränsning för http-status kod:
> - 2 000 RSA 2 048-bitars program varu nyckel Hämta transaktioner
> - 1 000 RSA 2 048-bitars HSM-nyckel Hämta transaktioner
> - 125 RSA 4 096-bitars HSM-nyckel Hämta transaktioner
> - 124 RSA 4 096-bitars HSM-nyckel Hämta transaktioner och 8 RSA 2 048-bit HSM-nyckel Hämta transaktioner

### <a name="secrets-managed-storage-account-keys-and-vault-transactions"></a>Hemligheter, hanterade lagrings konto nycklar och valv transaktioner:

| Transaktions typ | Maximalt antal transaktioner som tillåts inom 10 sekunder, per valv per region<sup>1</sup> |
| --- | --- |
| Alla transaktioner |2 000 |

Information om hur du hanterar begränsning när gränserna överskrids finns i [Azure Key Vault begränsnings vägledning](../articles/key-vault/key-vault-ovw-throttling.md).

<sup>1</sup> en begränsning för hela prenumerationen för alla transaktions typer är fem gånger per begränsning för nyckel valvet. Till exempel är HSM-andra transaktioner per prenumeration begränsad till 5 000 transaktioner på 10 sekunder per prenumeration.

### <a name="azure-private-link-integration"></a>Azure Private Link-integrering

| Resurs | Gräns |
| -------- | ----- |
| Privata slut punkter per nyckel valv | 64 |
| Nyckel valv med privata slut punkter per prenumeration | 64 |
