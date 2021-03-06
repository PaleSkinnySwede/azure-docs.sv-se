---
title: Lär dig hur du skyddar åtkomsten till data i Azure Cosmos DB
description: Lär dig mer om åtkomst kontroll koncept i Azure Cosmos DB, inklusive huvud nycklar, skrivskyddade nycklar, användare och behörigheter.
author: thomasweiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 01/21/2020
ms.openlocfilehash: f62ad6952170f22fe0f94a792a137f991a0e5026
ms.sourcegitcommit: 34a6fa5fc66b1cfdfbf8178ef5cdb151c97c721c
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "82208728"
---
# <a name="secure-access-to-data-in-azure-cosmos-db"></a>Säker åtkomst till data i Azure Cosmos DB

Den här artikeln ger en översikt över hur du skyddar åtkomsten till data som lagras i [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).

Azure Cosmos DB använder två typer av nycklar för att autentisera användare och ge åtkomst till dess data och resurser. 

|Nyckeltyp|Resurser|
|---|---|
|[Huvud nycklar](#master-keys) |Används för administrativa resurser: databas konton, databaser, användare och behörigheter|
|[Resurs-token](#resource-tokens)|Används för program resurser: behållare, dokument, bifogade filer, lagrade procedurer, utlösare och UDF: er|

<a id="master-keys"></a>

## <a name="master-keys"></a>Huvud nycklar

Huvud nycklar ger åtkomst till alla administrativa resurser för databas kontot. Huvud nycklar:

- Ge åtkomst till konton, databaser, användare och behörigheter. 
- Kan inte användas för att ge detaljerad åtkomst till behållare och dokument.
- Skapas när ett konto skapas.
- Kan återskapas när som helst.

Varje konto består av två huvud nycklar: en primär nyckel och en sekundär nyckel. Syftet med dubbla nycklar är att du kan skapa om eller registrera nycklar, vilket ger kontinuerlig åtkomst till ditt konto och dina data.

Förutom de två huvud nycklarna för Cosmos DB-kontot finns det två skrivskyddade nycklar. Dessa skrivskyddade nycklar tillåter bara Läs åtgärder på kontot. Skrivskyddade nycklar ger inte åtkomst till Läs behörighets resurser.

Primära, sekundära, skrivskyddade och Läs-och skriv huvud nycklar kan hämtas och återskapas med hjälp av Azure Portal. Instruktioner finns i [Visa, kopiera och återskapa åtkomst nycklar](manage-with-cli.md#regenerate-account-key).

![Åtkomst kontroll (IAM) i Azure Portal – demonstrera NoSQL Database-säkerhet](./media/secure-access-to-data/nosql-database-security-master-key-portal.png)

### <a name="key-rotation"></a>Nyckel rotation<a id="key-rotation"></a>

Processen att rotera huvud nyckeln är enkel. 

1. Navigera till Azure Portal för att hämta den sekundära nyckeln.
2. Ersätt din primära nyckel med din sekundära nyckel i ditt program. Se till att alla Cosmos DB-klienter i alla distributioner startas om och kommer att börja använda den uppdaterade nyckeln.
3. Rotera primär nyckeln i Azure Portal.
4. Verifiera att den nya primär nyckeln fungerar mot alla resurser. Nyckel rotations processen kan ta var som helst om från mindre än en minut till timmar beroende på storleken på Cosmos DB kontot.
5. Ersätt den sekundära nyckeln med den nya primära nyckeln.

![Huvud nyckel rotation i Azure Portal – demonstrera NoSQL Database-säkerhet](./media/secure-access-to-data/nosql-database-security-master-key-rotate-workflow.png)

### <a name="code-sample-to-use-a-master-key"></a>Kod exempel för att använda en huvud nyckel

Följande kod exempel illustrerar hur du använder en Cosmos DB konto slut punkt och huvud nyckel för att instansiera en DocumentClient och skapa en databas.

```csharp
//Read the Azure Cosmos DB endpointUrl and authorization keys from config.
//These values are available from the Azure portal on the Azure Cosmos DB account blade under "Keys".
//Keep these values in a safe and secure location. Together they provide Administrative access to your Azure Cosmos DB account.

private static readonly string endpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
private static readonly string authorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];

CosmosClient client = new CosmosClient(endpointUrl, authorizationKey);
```

## <a name="resource-tokens"></a>Resurs-token<a id="resource-tokens"></a>

Resurs-token ger åtkomst till program resurserna i en databas. Resurs-token:

- Ge åtkomst till vissa behållare, partitionsalternativ, dokument, bifogade filer, lagrade procedurer, utlösare och UDF: er.
- Skapas när en [användare](#users) beviljas [behörigheter](#permissions) till en speciell resurs.
- Återskapas när en behörighets resurs behandlas enligt POST, GET eller parkera samtal.
- Använd en hash-token som är specifikt konstruerad för användaren, resursen och behörigheten.
- Är den tid som är kopplad till en anpassningsbar giltighets period. Standardvärdet för en giltig tidsrymd är en timme. Livs längd för token kan dock uttryckligen anges, upp till högst fem timmar.
- Ange ett säkert alternativ för att ge huvud nyckeln.
- Gör det möjligt för klienter att läsa, skriva och ta bort resurser i Cosmos DB konto enligt de behörigheter som de har beviljats.

Du kan använda en resurs-token (genom att skapa Cosmos DB användare och behörigheter) när du vill ge åtkomst till resurser i ditt Cosmos DB-konto till en klient som inte är betrodd med huvud nyckeln.  

Cosmos DB-resurs-token är ett säkert alternativ som gör att klienter kan läsa, skriva och ta bort resurser i ditt Cosmos DB konto enligt de behörigheter som du har beviljat, och utan att behöva en huvud-eller skrivskyddad nyckel.

Här är ett typiskt design mönster där du kan begära, generera och leverera resurs-token till klienter:

1. En tjänst på mellan nivå har kon figurer ATS för att hantera ett mobilt program för att dela användar foton.
2. Tjänsten på mellan nivå har Cosmos DB kontots huvud nyckel.
3. Foto appen installeras på mobila enheter slutanvändare.
4. Vid inloggningen upprättar Foto appen identiteten för användaren med tjänsten på mellan nivå. Den här mekanismen för identitets etablering är helt upp till programmet.
5. När identiteten har upprättats begär mellanskikts tjänsten behörigheter baserat på identiteten.
6. Tjänsten mellan nivå skickar en Resource-token tillbaka till telefon-appen.
7. Phone-appen kan fortsätta att använda resurs-token för att direkt komma åt Cosmos DB resurser med behörigheterna som definierats av resurs-token och för intervallet som tillåts av resurs-token.
8. När resursens token upphör att gälla får efterföljande begär Anden ett 401 obehörigt undantag.  I det här läget upprättar Phone-appen identiteten igen och begär en ny resurs-token.

    ![Arbets flöde för Azure Cosmos DB-resurs-token](./media/secure-access-to-data/resourcekeyworkflow.png)

Generering och hantering av resurs-token hanteras av de interna Cosmos DB klient biblioteken. men om du använder REST måste du skapa huvudena för begäran/autentisering. Mer information om hur du skapar autentiseringsscheman för REST finns i [Access Control på Cosmos DB resurser](https://docs.microsoft.com/rest/api/cosmos-db/access-control-on-cosmosdb-resources) eller käll koden för vår [.NET SDK](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos/src/AuthorizationHelper.cs) -eller [Node. js SDK](https://github.com/Azure/azure-cosmos-js/blob/master/src/auth.ts).

Ett exempel på en tjänst mellan nivåer som används för att generera eller Broker-resursfiler finns i [ResourceTokenBroker-appen](https://github.com/Azure/azure-documentdb-dotnet/tree/master/samples/xamarin/UserItems/ResourceTokenBroker/ResourceTokenBroker/Controllers).

## <a name="users"></a>Användare<a id="users"></a>

Azure Cosmos DB användare är associerade med en Cosmos-databas.  Varje databas kan innehålla noll eller flera Cosmos DB användare. Följande kod exempel visar hur du skapar en Cosmos DB användare med hjälp av [Azure Cosmos dB .NET SDK v3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement).

```csharp
//Create a user.
Database database = benchmark.client.GetDatabase("SalesDatabase");

User user = await database.CreateUserAsync("User 1");
```

> [!NOTE]
> Varje Cosmos DB användare har en ReadAsync ()-metod som kan användas för att hämta listan över [behörigheter](#permissions) som är associerade med användaren.

## <a name="permissions"></a>Åtkomst<a id="permissions"></a>

En behörighets resurs är kopplad till en användare och tilldelad till behållaren samt partitionens nyckel nivå. Varje användare kan innehålla noll eller fler behörigheter. En behörighets resurs ger åtkomst till en säkerhetstoken som användaren behöver när de försöker komma åt en speciell behållare eller data i en angiven partitionsnyckel. Det finns två tillgängliga åtkomst nivåer som kan tillhandahållas av en behörighets resurs:

- Alla: användaren har fullständig behörighet till resursen.
- Läsa: användaren kan bara läsa innehållet i resursen, men kan inte utföra Skriv-, uppdaterings-eller borttagnings åtgärder på resursen.

> [!NOTE]
> För att kunna köra lagrade procedurer måste användaren ha behörigheten alla för behållaren där den lagrade proceduren ska köras.

### <a name="code-sample-to-create-permission"></a>Kod exempel för att skapa behörighet

Följande kod exempel visar hur du skapar en behörighets resurs, läser resurs-token för behörighets resursen och associerar behörigheterna med [användaren](#users) som skapades ovan.

```csharp
// Create a permission on a container and specific partition key value
Container container = client.GetContainer("SalesDatabase", "OrdersContainer");
user.CreatePermissionAsync(
    new PermissionProperties(
        id: "permissionUser1Orders",
        permissionMode: PermissionMode.All,
        container: benchmark.container,
        resourcePartitionKey: new PartitionKey("012345")));
```

### <a name="code-sample-to-read-permission-for-user"></a>Kod exempel för Läs behörighet för användare

Följande kodfragment visar hur du hämtar behörigheten som är kopplad till den användare som skapats ovan och instansierar en ny CosmosClient för användarens räkning, som är begränsad till en enda partitionsnyckel.

```csharp
//Read a permission, create user client session.
PermissionProperties permissionProperties = await user.GetPermission("permissionUser1Orders")

CosmosClient client = new CosmosClient(accountEndpoint: "MyEndpoint", authKeyOrResourceToken: permissionProperties.Token);
```

## <a name="add-users-and-assign-roles"></a>Lägg till användare och tilldela roller

Om du vill lägga till Azure Cosmos DB konto läsar åtkomst till ditt användar konto, har du en prenumerations ägare som utför följande steg i Azure Portal.

1. Öppna Azure Portal och välj ditt Azure Cosmos DB-konto.
2. Klicka på fliken **åtkomst kontroll (IAM)** och klicka sedan på **+ Lägg till roll tilldelning**.
3. I fönstret **Lägg till roll tilldelning** väljer du **Cosmos DB konto läsar roll**i rutan **roll** .
4. I **rutan tilldela åtkomst till väljer du** **Azure AD-användare, grupp eller program**.
5. Välj den användare, grupp eller det program i din katalog som du vill bevilja åtkomst till.  Du kan söka i katalogen efter visnings namn, e-postadress eller objekt identifierare.
    Den valda användaren, gruppen eller programmet visas i listan med valda medlemmar.
6. Klicka på **Spara**.

Entiteten kan nu läsa Azure Cosmos DB-resurser.

## <a name="delete-or-export-user-data"></a>Ta bort eller exportera användar data

Med Azure Cosmos DB kan du söka efter, välja, ändra och ta bort personliga data som finns i databasen eller samlingarna. Azure Cosmos DB innehåller API: er för att hitta och ta bort personliga data men det är ditt ansvar att använda API: erna och definiera den logik som krävs för att radera personliga data. Varje API för flera modeller (SQL, MongoDB, Gremlin, Cassandra, Table) innehåller olika språk-SDK: er som innehåller metoder för att söka efter och ta bort personliga data. Du kan också aktivera funktionen [Time to Live (TTL)](time-to-live.md) för att ta bort data automatiskt efter en angiven period, utan att detta medför ytterligare kostnader.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="next-steps"></a>Nästa steg

- Mer information om Cosmos Database-säkerhet finns i [Cosmos DB databas säkerhet](database-security.md).
- Information om hur du skapar Azure Cosmos DB tokens finns [Access Control på Azure Cosmos DB resurser](https://docs.microsoft.com/rest/api/cosmos-db/access-control-on-cosmosdb-resources).
- Exempel på användar hantering med användare och behörigheter, [.NET SDK v3 användar hanterings exempel](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/master/Microsoft.Azure.Cosmos.Samples/Usage/UserManagement/UserManagementProgram.cs)
