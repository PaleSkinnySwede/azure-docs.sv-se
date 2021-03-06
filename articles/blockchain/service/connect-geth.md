---
title: Använda Geth för att ansluta till Azure blockchain-tjänsten
description: Anslut till en Geth-instans på Azure blockchain service Transaction-noden
ms.date: 11/20/2019
ms.topic: quickstart
ms.reviewer: janders
ms.openlocfilehash: 9da78eac1dc429bcc0ad52bb9cb2f1fb743a90d4
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/29/2020
ms.locfileid: "74455825"
---
# <a name="quickstart-use-geth-to-attach-to-an-azure-blockchain-service-transaction-node"></a>Snabb start: Använd Geth för att ansluta till en Azure blockchain service Transaction-nod

I den här snabb starten använder du Geth-klienten för att ansluta till en Geth-instans på en Azure blockchain-tjänst transaktions nod. När du är ansluten använder du Geth JavaScript-konsolen för att anropa ett Web3 JavaScript-Dapp-API.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Krav

* Installera [Geth](https://github.com/ethereum/go-ethereum/wiki/geth)
* Slutför [snabb start: skapa en blockchain-medlem med hjälp av Azure Portal](create-member.md) eller [snabb start: skapa en Azure blockchain service blockchain-medlem med Azure CLI](create-member-cli.md)

## <a name="get-geth-connection-string"></a>Hämta Geth-anslutningssträng

Du kan hämta anslutnings strängen Geth för en Azure blockchain service Transaction-nod i Azure Portal.

1. Logga in på [Azure-portalen](https://portal.azure.com).
1. Gå till din Azure blockchain service-medlem. Välj **Transactions-noder** och noden standard transaktions nod.

    ![Välj noden standard transaktion](./media/connect-geth/transaction-nodes.png)

1. Välj **anslutnings strängar**.
1. Kopiera anslutnings strängen från **https (åtkomst nyckel 1)**. Du behöver strängen för nästa avsnitt.

    ![Anslutningssträng](./media/connect-geth/connection-string.png)

## <a name="connect-to-geth"></a>Anslut till Geth

1. Öppna en kommando tolk eller ett gränssnitt.
1. Använd Geth Attach-underkommandot för att koppla till den aktiva Geth-instansen på Transaction-noden. Klistra in anslutnings strängen som ett argument för under kommandot Attach. Ett exempel:

    ``` bash
    geth attach <connection string>
    ```

1. När du har anslutit till Ethereum-konsolen för Transaction-noden kan du anropa Web3 Java Script Dapp API eller admin-API.

    Använd till exempel följande API för att ta reda på chainId.

    ``` bash
    admin.nodeInfo.protocols.istanbul.config.chainId
    ```

    I det här exemplet är chainId 661.

    ![Azure blockchain service-alternativ](./media/connect-geth/geth-attach.png)

1. Om du vill koppla från konsolen skriver `exit`du.

## <a name="next-steps"></a>Nästa steg

I den här snabb starten använde du Geth-klienten för att ansluta till en Geth-instans på en Azure blockchain service Transaction-nod. Testa nästa självstudie för att använda Azure blockchain Development Kit för Ethereum för att skapa, bygga, distribuera och köra en smart kontrakts funktion via en transaktion.

> [!div class="nextstepaction"]
> [Skapa, skapa och distribuera smarta avtal i Azure blockchain-tjänsten](send-transaction.md)
