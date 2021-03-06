---
title: Hantera app-grupper för Windows Virtual Desktop Portal – Azure
description: Hantera Windows-appar för virtuella skriv bord med Azure Portal.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: tutorial
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f072ed8a758173645c886cabf0b20f9e123cbbab
ms.sourcegitcommit: 50ef5c2798da04cf746181fbfa3253fca366feaa
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/30/2020
ms.locfileid: "82612217"
---
# <a name="tutorial-manage-app-groups-with-the-azure-portal"></a>Självstudie: hantera program grupper med Azure Portal

>[!IMPORTANT]
>Det här innehållet gäller för våren 2020-uppdateringen med Azure Resource Manager virtuella Windows Desktop-objekt. Om du använder den virtuella Windows-datorn med version 2019 utan Azure Resource Manager objekt, se [den här artikeln](./virtual-desktop-fall-2019/manage-app-groups-2019.md).
>
> Den virtuella Windows-skrivbordets våren 2020-uppdateringen är för närvarande en offentlig för hands version. Den här för hands versionen tillhandahålls utan service nivå avtal och vi rekommenderar inte att du använder den för produktions arbets belastningar. Vissa funktioner kanske inte stöds eller kan vara begränsade. 
> Mer information finns i [Kompletterande villkor för användning av Microsoft Azure-förhandsversioner](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Standard-app-gruppen som skapas för en ny Windows-pool för virtuella skriv bord publicerar också hela Skriv bordet. Dessutom kan du skapa en eller flera RemoteApp-programgrupper för poolen. Följ den här självstudien för att skapa en RemoteApp-app-grupp och publicera enskilda Start meny-appar.

I den här självstudiekursen får du lära du dig att:

> [!div class="checklist"]
> * Skapa en RemoteApp-grupp.
> * Bevilja åtkomst till RemoteApp-program.

## <a name="create-a-remoteapp-group"></a>Skapa en RemoteApp-grupp

Om du redan har skapat en adresspool och en session som är värd för en virtuell dator med hjälp av Azure Portal eller PowerShell kan du lägga till program grupper från Azure Portal med följande process:

1.  Logga in på [Azure Portal](https://portal.azure.com/).

2.  Sök efter och välj **virtuellt skriv bord i Windows**.

3.  Välj **program grupper** i menyn på vänster sida av sidan och välj sedan **+ Lägg till**.

4. På fliken **grundläggande** väljer du den prenumerations grupp och resurs grupp som du vill skapa app-gruppen för. Du kan också välja att skapa en ny resurs grupp i stället för att välja en befintlig.

5. Välj den modempool som ska associeras med program gruppen från den nedrullningsbara menyn bredvid **poolen värd**.

    >[!NOTE]
    >Du måste välja den modempool som är associerad med program gruppen. App-grupper har appar eller skriv bord som hanteras från en sessions värd och sessions-värdar är en del av värd-pooler. App-gruppen måste vara kopplad till en adresspool när den skapas.

    > [!div class="mx-imgBorder"]
    > ![En skärm bild av fliken grundläggande i Azure Portal.](media/basics-tab.png)

6. Om du vill lägga till program grupper i din värddator väljer du **pooler** på menyn till vänster på skärmen.
   
    Välj sedan namnet på den värddator som du vill lägga till program grupper i.
   
    Sedan väljer du **program grupper** på menyn till vänster på skärmen och väljer sedan **+ Lägg till**.

    Välj slutligen den prenumerations grupp och resurs grupp som du vill skapa app-gruppen i. Du kan antingen välja namnet på en befintlig resurs grupp från den nedrullningsbara menyn eller välja **Skapa ny** för att skapa en ny.

      >[!NOTE]
      >När du lägger till program grupper i din värd pool är den värddator som är relaterad till program gruppen redan markerad eftersom du har navigerat från den.
      > 
      > [!div class="mx-imgBorder"]
      >![En skärm bild av fliken grundläggande med förvaltad värd för poolen.](media/host-pool-selected.png)

7. Välj **RemoteApp** under program grupps typ och ange ett namn för RemoteApp.

      > [!div class="mx-imgBorder"]
      > ![En skärm bild av fälten för program grupp typ. "RemoteApp" är markerat.](media/remoteapp-button.png)

8.  Välj fliken **tilldelningar** .

9.  Om du vill publicera enskilda användare eller användar grupper i app-gruppen väljer du **+ Lägg till Azure AD-användare eller användar grupper**.

10.  Välj det antal användare som du vill lägga till apparna i. Du kan välja en eller flera användare och användar grupper.

     > [!div class="mx-imgBorder"]
     > ![En skärm bild av menyn för val av användare.](media/select-users.png)

11.  Välj **Välj**.

12.  Välj fliken **program** och välj sedan **+ Lägg till program**.

13.  Så här lägger du till ett program på Start-menyn: 

      - Gå till **program källa** och välj **Start-menyn** på den nedrullningsbara menyn. Gå sedan till **program** och välj programmet i den nedrullningsbara menyn.

     > [!div class="mx-imgBorder"]
     > ![En skärm bild av skärmen Lägg till program med Start-menyn markerad.](media/add-app-start.png)

      - I **visnings namn**anger du namnet på det program som ska visas för användaren på klienten.

      - Lämna de andra alternativen som de är och välj **Spara**.

14. Så här lägger du till ett program från en specifik fil Sök väg:

      - Gå till **program källa** och välj **fil Sök väg** på den nedrullningsbara menyn.

      - Ange sökvägen till programmet på värd för sessionen som är registrerad på den associerade poolen.

      - Ange programmets information i fälten **program namn**, **visnings namn**, **sökväg**och **ikon index** .

      - Välj **Spara**.

     > [!div class="mx-imgBorder"]
     > ![En skärm bild av sidan Lägg till program med fil Sök väg vald.](media/add-app-file.png)

     Upprepa den här processen för alla program som du vill lägga till i program gruppen.

15.  Välj sedan fliken **arbets yta** .

16.  Om du vill registrera app-gruppen på en arbets yta går du till **Registrera program grupp** och väljer **Ja**. Om du hellre vill registrera app-gruppen vid ett senare tillfälle väljer du **Nej**.

17.  Om du väljer **Ja**kan du välja en befintlig arbets yta för att registrera din app-grupp på.
       
       >[!NOTE]
       >Du kan bara registrera app-gruppen på arbets ytor som har skapats på samma plats som poolen. Dessutom. Om du tidigare har registrerat en annan app-grupp från samma adresspool som den nya app-gruppen till en arbets yta, kommer den att markeras och du kan inte redigera den. Alla app-grupper från en adresspool måste vara registrerade på samma arbets yta.

     > [!div class="mx-imgBorder"]
     > ![En skärm bild av sidan Registrera program grupp för en befintlig arbets yta. Host-poolen är förvalt.](media/register-existing.png)

18. Alternativt kan du, om du vill skapa taggar för att göra din arbets yta lätt att ordna, välja fliken **taggar** och ange dina taggnamn.

19. När du är klar väljer du fliken **Granska + skapa** .

20. Vänta en stund tills verifierings processen har slutförts. När du är färdig väljer du **skapa** för att distribuera din app-grupp.

Distributions processen gör följande saker för dig:

- Skapa gruppen RemoteApp-app.
- Lägg till de valda apparna i app-gruppen.
- Publicera app-gruppen som publicerats till användare och användar grupper som du har valt.
- Registrera app-gruppen, om du har valt att göra det.
- Skapa en länk till en Azure Resource Manager-mall baserat på din konfiguration som du kan hämta och spara för senare.

## <a name="next-steps"></a>Nästa steg

I den här självstudien har du lärt dig hur du skapar en app-grupp, fyller den med RemoteApp-program och tilldelar användare till app-gruppen. Information om hur du skapar en pool för validerings värdar finns i följande självstudie. Du kan använda en pool för validerings värdar för att övervaka tjänst uppdateringar innan du återställer dem till din produktions miljö.

> [!div class="nextstepaction"]
> [Skapa en värdpool för att verifiera tjänstuppdateringar](./create-validation-host-pool.md)
