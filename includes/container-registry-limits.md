---
title: ta med fil
description: ta med fil
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 03/11/2020
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: 0090f02382e024e5539383328b55d58798002d63
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "79117130"
---
| Resurs | Basic | Standard | Premium |
|---|---|---|---|
| Lagring<sup>1</sup> | 10 GiB | 100 GiB| 500 GiB |
| Största bild skikt storlek | 200 GiB | 200 GiB | 200 GiB |
| ReadOps per minut<sup>2, 3</sup> | 1,000 | 3 000 | 10 000 |
| WriteOps per minut<sup>2, 4</sup> | 100 | 500 | 2 000 |
| Ladda ned bandbredd MBps<sup>2</sup> | 30 | 60 | 100 |
| Ladda upp bandbredd MBps<sup>2</sup> | 10 | 20 | 50 |
| Webhooks | 2 | 10 | 500 |
| Geo-replikering | E.t. | E.t. | [Stöds][geo-replication] |
| Förtroende för innehåll | E.t. | E.t. | [Stöds][content-trust] |
| Åtkomst till virtuellt nätverk | E.t. | E.t. | [Förhandsgranskning][vnet] |
| Integrering av privata länkar | E.t. | E.t. | [Förhandsgranskning][plink] |
| Kundhanterade nycklar | E.t. | E.t. | [Förhandsgranskning][cmk] |
| Databas – begränsade behörigheter | E.t. | E.t. | [Förhandsgranskning][token]|
| &bull;Token | E.t. | E.t. | 20 000 |
| &bull;Omfångs kartor | E.t. | E.t. | 20 000 |
| &bull;Mappning av databaser per område | E.t. | E.t. | 500 |


<sup>1</sup> De angivna lagrings gränserna är mängden lagrings utrymme som *ingår* för varje nivå. Du debiteras ytterligare en daglig taxa per GiB för avbildnings lagring över dessa gränser. Mer information om priser finns i [Azure Container Registry prissättning][pricing].

<sup>2</sup>*ReadOps*, *WriteOps*och *bandbredd* är minimala uppskattningar. Azure Container Registry strävar efter att förbättra prestandan när användningen kräver.

<sup>3</sup> En [Docker-pull](https://docs.docker.com/registry/spec/api/#pulling-an-image) översätter sig till flera Läs åtgärder baserat på antalet lager i bilden, plus manifest hämtning.

<sup>4</sup> En [Docker push](https://docs.docker.com/registry/spec/api/#pushing-an-image) översätts till flera Skriv åtgärder baserat på antalet lager som måste flyttas. En `docker push` innehåller *ReadOps* för att hämta ett manifest för en befintlig avbildning.

<!-- LINKS - External -->
[pricing]: https://azure.microsoft.com/pricing/details/container-registry/

<!-- LINKS - Internal -->
[geo-replication]: ../articles/container-registry/container-registry-geo-replication.md
[content-trust]: ../articles/container-registry/container-registry-content-trust.md
[vnet]: ../articles/container-registry/container-registry-vnet.md
[plink]: ../articles/container-registry/container-registry-private-link.md
[cmk]: ../articles/container-registry/container-registry-customer-managed-keys.md
[token]: ../articles/container-registry/container-registry-repository-scoped-permissions.md