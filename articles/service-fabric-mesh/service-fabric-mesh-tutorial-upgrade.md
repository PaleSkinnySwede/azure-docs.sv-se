---
title: Självstudie – uppgradera ett Azure Service Fabric nät-program
description: Den här självstudien är del fyra i en serie och visar hur du uppgraderar ett Azure Service Fabric-nätprogram direkt från Visual Studio.
author: dkkapur
ms.topic: conceptual
ms.date: 11/29/2018
ms.author: dekapur
ms.custom: mvc, devcenter
ms.openlocfilehash: 7cdb8868f760ef0f35ab90c06b411110f871738c
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75351723"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>Självstudie: Lär dig hur du uppgraderar ett Service Fabric program med hjälp av Visual Studio

Den här självstudien är del fyra i en serie och visar hur du uppgraderar ett Azure Service Fabric-nätprogram direkt från Visual Studio. Uppgraderingen kommer att innehålla både en kod uppdatering och en konfigurations uppdatering. Du ser att stegen för att uppgradera och publicera från Visual Studio är desamma.

I den här guiden får du lära du dig hur man:
> [!div class="checklist"]
> * Uppgradera en Service Fabric nät tjänst med Visual Studio

I den här självstudieserien får du lära du dig att:
> [!div class="checklist"]
> * [Skapa en Service Fabric Mesh-app i Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Felsöka en Service Fabric Mesh-app som körs i ditt lokala utvecklingskluster](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Distribuera en Service Fabric Mesh-app](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * Uppgradera en Service Fabric Mesh-app
> * [Rensa Service Fabric Mesh-resurser](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Krav

Innan du börjar den här självstudien:

* Om du inte har distribuerat att göra-appen följer du anvisningarna i [Publicera ett Service Fabric Mesh-webbprogram](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md).

## <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>Uppgradera en Service Fabric nät tjänst med Visual Studio

Den här artikeln visar hur du uppgraderar en mikrotjänst i ett program. I det här exemplet ska vi ändra `WebFrontEnd` tjänsten för att visa en aktivitets kategori och öka mängden processor som den har fått. Sedan uppgraderar vi den distribuerade tjänsten.

## <a name="modify-the-config"></a>Ändra konfigurationen

När du skapar en Service Fabric nätapp lägger Visual Studio till en **Parameters. yaml** -fil för varje distributions miljö (moln och lokal). I de här filerna kan du definiera parametrar och deras värden som sedan kan refereras från din nät *. yaml-filer som service. yaml eller Network. yaml.  Visual Studio innehåller några variabler åt dig, till exempel hur mycket CPU som tjänsten kan använda.

Vi uppdaterar `WebFrontEnd_cpu` parametern för att uppdatera CPU-resurserna till `1.5` i förväntat att **webfrontend** -tjänsten kommer att användas mer kraftigt.

1. I **todolistapp** -projektet, under **miljöer** > **Cloud**, öppnar du filen **Parameters. yaml** . Ändra värdet `WebFrontEnd_cpu`, värde till `1.5`. Parameter namnet föregås av tjänst namnet `WebFrontEnd_` som en bästa praxis att skilja den från parametrar med samma namn som gäller för olika tjänster.

    ```xml
    WebFrontEnd_cpu: 1.5
    ```

2. Öppna filen **service. yaml** för **webfrontend** -projekt under **webfrontend** > **service-resurser**.

    Observera att avsnittet i `resources:` `cpu:` är inställt på `"[parameters('WebFrontEnd_cpu')]"`. Om projektet skapas `'WebFrontEnd_cpu` för molnet hämtas värdet för från **miljöns** > **Cloud** > **Parameters. yaml** -fil och kommer att vara. `1.5` Om projektet skapas för att köras lokalt hämtas värdet från **miljöns** > **lokala** > **parametrar. yaml** -fil och blir "0,5".

> [!Tip]
> Som standard används parameter filen som är en peer i filen Profile. yaml för att ange värden för filen Profile. yaml.
> Till exempel miljöer > Cloud > Parameters. yaml innehåller parameter värden för miljöer > Cloud > Profile. yaml.
>
> Du kan åsidosätta detta genom att lägga till följande i filen Profile. yaml`parametersFilePath=”relative or full path to the parameters file”` : till exempel `parametersFilePath=”C:\MeshParms\CustomParameters.yaml”` eller`parametersFilePath=”..\CommonParameters.yaml”`

## <a name="modify-the-model"></a>Ändra modellen

Om du vill införa en kod ändring lägger `Category` du till en `ToDoItem` egenskap i- `ToDoItem.cs` klassen i filen.

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

Uppdatera sedan `Load()` metoden i samma fil för att ange kategorin till en standard sträng:

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>Ändra tjänsten

`WebFrontEnd` Projektet är ett ASP.net Core program med en webb sida som visar objekt i att göra-listan. I `WebFrontEnd` projektet öppnar `Index.cshtml` du och lägger till följande två rader, som anges nedan, för att Visa uppgiftens kategori:

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Skapa och kör appen för att kontrol lera att du ser en ny kategori kolumn på webb sidan som visar aktiviteterna.

## <a name="upgrade-the-app-from-visual-studio"></a>Uppgradera appen från Visual Studio

Oavsett om du gör en kod uppgradering eller en konfigurations uppgradering (i det här fallet gör vi båda), uppgraderar du Service Fabric nätappen i Azure genom att högerklicka på **todolistapp** i Visual Studio och sedan välja **publicera...**

Därefter visas dialogrutan **Publish Service Fabric Application** (Publicera Service Fabric-program).

Använd List rutan **mål profil** för att välja filen Profile. yaml som ska användas för den här distributionen. Vi uppgraderar appen i molnet så att vi väljer **Cloud. yaml** i list rutan, som använder `WebFrontEnd_cpu` värdet för 1,0 som definierats i filen.

![Visual Studio, dialogruta för Service Fabric Mesh-projekt](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Välj Azure-konto och Azure-prenumeration. Ange **platsen** för den plats som du använde när du ursprungligen publicerade att göra-appen till Azure. Den här artikeln användes **USA, östra**.

Ange **resurs gruppen** till den resurs grupp som du använde när du ursprungligen publicerade att göra-appen till Azure.

Ange **Azure Container Registry** till det Azure Container Registry-namn som du skapade när du ursprungligen publicerade att göra-appen till Azure.

I dialog rutan publicera trycker du på knappen **publicera** för att uppgradera att göra-appen på Azure.

Övervaka förloppet för uppgraderingen genom att välja fönstret **Service Fabric verktyg** i fönstret **utdata** i Visual Studio. 

När avbildningen har skapats och skickas till Azure Container Registry visas **statusen för status** länken i de utdata som du kan klicka på för att övervaka distributionen i Azure Portal.

När uppgraderingen är färdig visar Service Fabric- **verktygets** utdata IP-adressen och porten för ditt program i form av en URL.

```json
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Öppna en webbläsare och navigera till webbadressen för att se webbplatsen köras i Azure. Nu bör du se en webb sida som innehåller en kategori kolumn.

## <a name="next-steps"></a>Nästa steg

I den här självstudiedelen lärde du dig:
> [!div class="checklist"]
> * Så här uppgraderar du en Service Fabric nätappen med hjälp av Visual Studio

Gå vidare till nästa kurs:
> [!div class="nextstepaction"]
> [Rensa Service Fabric Mesh-resurser](service-fabric-mesh-tutorial-cleanup-resources.md)