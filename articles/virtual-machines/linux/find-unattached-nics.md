---
title: Hitta och ta bort frånkopplade Azure-nätverkskort
description: Hitta och ta bort Azure-nätverkskort som inte är anslutna till virtuella datorer med Azure CLI
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.workload: infrastructure-services
ms.topic: article
ms.date: 04/10/2018
ms.author: cynthn
ms.openlocfilehash: 8142b95ee666e205a8328eafd5930f1f386e49af
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "78945125"
---
# <a name="how-to-find-and-delete-unattached-network-interface-cards-nics-for-azure-vms"></a>Hitta och ta bort anslutna nätverkskort för virtuella Azure-datorer
När du tar bort en virtuell dator i Azure tas nätverkskorten (nätverkskorten) inte bort som standard. Om du skapar och tar bort flera virtuella datorer fortsätter de oanvända nätverkskorten att använda de interna IP-adresslån. När du skapar andra nätverkskort för virtuella datorer kan de inte erhålla ett IP-lån i under nätets adress utrymme. Den här artikeln visar hur du hittar och tar bort ej anslutna nätverkskort.

## <a name="find-and-delete-unattached-nics"></a>Hitta och ta bort ej anslutna nätverkskort

Egenskapen *virtualMachine* för ett nätverkskort lagrar ID och resurs grupp för den virtuella dator som nätverkskortet är anslutet till. Följande skript upprepas genom alla nätverkskort i en prenumeration och kontrollerar om egenskapen *virtualMachine* är null. Om den här egenskapen är null är NÄTVERKSKORTet inte anslutet till en virtuell dator.

Om du vill visa alla ej anslutna nätverkskort rekommenderar vi starkt att du först kör skriptet med variabeln *deleteUnattachedNics* till *0*. Om du vill ta bort alla ej anslutna nätverkskort när du har granskat listans utdata kör du skriptet med *deleteUnattachedNics* till *1*.

```azurecli
# Set deleteUnattachedNics=1 if you want to delete unattached NICs
# Set deleteUnattachedNics=0 if you want to see the Id(s) of the unattached NICs
deleteUnattachedNics=0

unattachedNicsIds=$(az network nic list --query '[?virtualMachine==`null`].[id]' -o tsv)
for id in ${unattachedNicsIds[@]}
do
   if (( $deleteUnattachedNics == 1 ))
   then

       echo "Deleting unattached NIC with Id: "$id
       az network nic delete --ids $id
       echo "Deleted unattached NIC with Id: "$id
   else
       echo $id
   fi
done
```

## <a name="next-steps"></a>Nästa steg

Mer information om hur du skapar och hanterar virtuella nätverk i Azure finns i [skapa och hantera virtuella dator nätverk](tutorial-virtual-network.md).
