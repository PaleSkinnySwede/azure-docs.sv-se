---
title: Hämta en lista med grupper i Azure Active Directory Portal | Microsoft Docs
description: Hämta grupp egenskaper i grupp i Azure administrations centret i Azure Active Directory.
services: active-directory
author: curtand
ms.author: curtand
manager: mtillman
ms.date: 04/16/2020
ms.topic: conceptual
ms.service: active-directory
ms.subservice: users-groups-roles
ms.workload: identity
ms.custom: it-pro
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 59983678c1b14d6aa87a7b500605e3abeb6a9b85
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81533554"
---
# <a name="bulk-download-a-list-of-groups-in-azure-active-directory"></a>Hämta en lista över grupper i Azure Active Directory

Med hjälp av Azure Active Directory-portalen (Azure AD) kan du hämta en lista över alla grupper i organisationen till en fil med kommaavgränsade värden (CSV).

## <a name="to-download-a-list-of-groups"></a>Hämta en lista över grupper

1. Logga in på [Azure Portal](https://portal.azure.com) med ett administratörs konto i organisationen.
1. I Azure AD väljer du **grupper** > **hämtnings grupper**.
1. På sidan **hämtning av grupper** väljer du **Start** för att ta emot en CSV-fil som visar dina grupper.

   ![Kommandot ladda ned grupper finns på sidan alla grupper](./media/groups-bulk-download/bulk-download.png)

## <a name="check-download-status"></a>Kontrol lera hämtnings status

Du kan se statusen för alla väntande Mass förfrågningar på resultat sidan för **Mass åtgärder** .

[![](media/groups-bulk-download/bulk-center.png "Check status in the Bulk Operations Results page")](media/groups-bulk-download/bulk-center.png#lightbox)

## <a name="bulk-download-service-limits"></a>Begränsningar för Mass nedladdnings tjänst

Varje Mass aktivitet för att hämta en grupp lista kan köras i upp till en timme. På så sätt kan du ladda ned en lista med minst 300 000 grupper.

## <a name="next-steps"></a>Nästa steg

- [Mass borttagning av grupp medlemmar](groups-bulk-remove-members.md)
- [Ladda ned medlemmar i en grupp](groups-bulk-download-members.md)
