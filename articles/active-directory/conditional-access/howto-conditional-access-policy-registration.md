---
title: Villkorlig åtkomst – kombinerad säkerhets information – Azure Active Directory
description: Skapa en anpassad princip för villkorlig åtkomst för registrering av säkerhets information
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 03/25/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9c8081bb8145a6654c168fb2d664e1666b32dc18
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "81457917"
---
# <a name="conditional-access-securing-security-info-registration"></a>Villkorlig åtkomst: säkra registrering av säkerhets information

Att skydda när och hur användare registrerar sig för Azure Multi-Factor Authentication och återställning av lösen ord för självbetjäning är nu möjligt med användar åtgärder i princip för villkorlig åtkomst. Den här förhands gransknings funktionen är tillgänglig för organisationer som har aktiverat den [kombinerade förhands granskningen](../authentication/concept-registration-mfa-sspr-combined.md). Den här funktionen kan vara aktive rad i organisationer där de vill använda villkor som betrott nätverks plats för att begränsa åtkomsten till registrering för Azure Multi-Factor Authentication och lösen ords återställning via självbetjäning (SSPR). Mer information om användbara villkor finns i artikeln [villkorlig åtkomst: villkor](concept-conditional-access-conditions.md).

## <a name="create-a-policy-to-require-registration-from-a-trusted-location"></a>Skapa en princip för att kräva registrering från en betrodd plats

Följande princip gäller för alla valda användare, som försöker registrera sig med den kombinerade registrerings upplevelsen och blockerar åtkomsten om de inte ansluter från en plats som har marker ATS som ett betrott nätverk.

1. I **Azure Portal**bläddrar du till **Azure Active Directory** > **säkerhet** > för**villkorlig åtkomst**.
1. Välj **ny princip**.
1. I namn anger du ett namn för den här principen. Till exempel **kombinerad säkerhets informations registrering på betrodda nätverk**.
1. Under **tilldelningar**väljer du **användare och grupper**och väljer de användare och grupper som du vill att den här principen ska tillämpas på.

   > [!WARNING]
   > Användare måste aktive ras för den [kombinerade registreringen](../authentication/howto-registration-mfa-sspr-combined.md).

1. Under **molnappar eller åtgärder**väljer du **användar åtgärder**, markera **Registrera säkerhets information**.
1. Under **villkor** > **platser**.
   1. Konfigurera **Ja**.
   1. Ta med **vilken plats som helst**.
   1. Undanta **alla betrodda platser**.
   1. Välj **gör** på bladet platser.
   1. Välj **färdig** på bladet villkor.
1. Under **villkor** > **klient program (för hands version)** anger du **Konfigurera** till **Ja**och väljer **klart**.
1. Under **åtkomst kontroller** > **beviljar**.
   1. Välj **blockera åtkomst**.
   1. Klicka sedan på **Välj**.
1. Ange **Aktivera princip** till **På**.
1. Välj sedan **Spara**.

I steg 6 i den här policyn har organisationer alternativ som de kan göra. Principen ovan kräver registrering från en betrodd nätverks plats. Organisationer kan välja att använda alla tillgängliga villkor i stället för **platser**. Kom ihåg att den här principen är en spärr princip, så att allt som inkluderas blockeras och allt som inte matchar include är tillåtet. 

Vissa kan välja att använda enhets tillstånd i stället för plats i steg 6 ovan:

6. Under **villkor** > **enhets tillstånd (för hands version)**.
   1. Konfigurera **Ja**.
   1. Ta med **all enhets status**.
   1. Undanta **enheten hybrid Azure AD** och/eller **enhet markerad som kompatibel**
   1. Välj **gör** på bladet platser.
   1. Välj **färdig** på bladet villkor.

> [!WARNING]
> Om du använder enhets tillstånd som ett villkor i principen kan det påverka gäst användare i katalogen. [Endast rapport läge](concept-conditional-access-report-only.md) kan hjälpa dig att fastställa konsekvenserna av princip beslut.

## <a name="next-steps"></a>Nästa steg

[Vanliga principer för villkorlig åtkomst](concept-conditional-access-policy-common.md)

[Bestäm inverkan med endast rapport läge för villkorlig åtkomst](howto-conditional-access-report-only.md)

[Simulera inloggnings beteende med hjälp av What If verktyget för villkorlig åtkomst](troubleshoot-conditional-access-what-if.md)
