---
title: 'Säker kommunikation via fjärr kommunikation med C #'
description: Lär dig hur du skyddar kommunikation mellan tjänster för tillförlitliga C#-tjänster som körs i ett Azure Service Fabric-kluster.
author: suchiagicha
ms.topic: conceptual
ms.date: 04/20/2017
ms.author: pepogors
ms.openlocfilehash: ee2f1d70f4094ccc7d80edbfaf16509b5124f607
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/28/2020
ms.locfileid: "75609629"
---
# <a name="secure-service-remoting-communications-in-a-c-service"></a>Säker kommunikation mellan tjänster i en C#-tjänst
> [!div class="op_single_selector"]
> * [C# i Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java i Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

Säkerhet är en av de viktigaste aspekterna av kommunikationen. Reliable Services Application Framework innehåller några inbyggda kommunikations stackar och verktyg som du kan använda för att förbättra säkerheten. Den här artikeln beskriver hur du kan förbättra säkerheten när du använder tjänst-Remoting i en C#-tjänst. Det bygger på ett befintligt [exempel](service-fabric-reliable-services-communication-remoting.md) som förklarar hur du konfigurerar fjärr kommunikation för pålitliga tjänster som skrivits i C#. 

Följ dessa steg om du vill skydda en tjänst när du använder service Remoting med C#-tjänster:

1. Skapa ett gränssnitt, `IHelloWorldStateful`som definierar de metoder som ska vara tillgängliga för ett fjärran rop på tjänsten. Din tjänst kommer att `FabricTransportServiceRemotingListener`använda, som deklareras i `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime` namn området. Det här är `ICommunicationListener` en implementering som tillhandahåller funktioner för fjärr kommunikation.

    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }

    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }

        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. Lägg till Listener-inställningar och säkerhets uppgifter.

    Kontrol lera att det certifikat som du vill använda för att skydda tjänst kommunikationen är installerat på alla noder i klustret. 
    
    > [!NOTE]
    > På Linux-noder måste certifikatet finnas som PEM-formaterade filer i katalogen */var/lib/sfcerts* . Mer information finns i [plats och format för X. 509-certifikat på Linux-noder](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes). 

    Det finns två sätt som du kan ange inställningar för lyssnare och säkerhets uppgifter:

   1. Ange dem direkt i Service koden:

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportRemotingListenerSettings  listenerSettings = new FabricTransportRemotingListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }

       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           x509Credentials.RemoteCertThumbprints.Add("9FEF3950642138446CC364A396E1E881DB76B483");
           return x509Credentials;
       }
       ```
   2. Ange dem med hjälp av ett [konfigurations paket](service-fabric-application-and-service-manifests.md):

       Lägg till ett `TransportSettings` namngivet avsnitt i filen Settings. xml.

       ```xml
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```

       I det här fallet kommer `CreateServiceReplicaListeners` metoden att se ut så här:

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportRemotingListenerSettings .LoadFrom("HelloWorldStatefulTransportSettings")))
           };
       }
       ```

        Om du lägger till `TransportSettings` ett avsnitt i Settings. xml- `FabricTransportRemotingListenerSettings` filen, kommer alla inställningar från det här avsnittet att läsas in som standard.

        ```xml
        <!--"TransportSettings" section .-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        I det här fallet kommer `CreateServiceReplicaListeners` metoden att se ut så här:

        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. När du anropar metoder på en säker tjänst med hjälp av fjärrstacken, i stället `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` för att använda klassen för att skapa en `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`Tjänstproxy, använder du. Skicka i `FabricTransportRemotingSettings`, som innehåller `SecurityCredentials`.

    ```csharp

    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "9FEF3950642138446CC364A396E1E881DB76B483",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
    x509Credentials.RemoteCertThumbprints.Add("4FEF3950642138446CC364A396E1E881DB76B48C");

    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        SecurityCredentials = x509Credentials,
    };

    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Om klient koden körs som en del av en tjänst kan du läsa in `FabricTransportRemotingSettings` från filen Settings. xml. Skapa ett HelloWorldClientTransportSettings-avsnitt som liknar tjänst koden, som du ser ovan. Gör följande ändringar i klient koden:

    ```csharp
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.LoadFrom("HelloWorldClientTransportSettings")));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Om klienten inte körs som en del av en tjänst kan du skapa en client_name. Settings. XML-fil på samma plats som client_name. exe är. Skapa sedan en TransportSettings-sektion i filen.

    På samma sätt som för tjänsten, om du `TransportSettings` lägger till ett avsnitt i klient inställningar. xml/client_name. Settings. xml `FabricTransportRemotingSettings` läser in alla inställningar från det här avsnittet som standard.

    I så fall är den tidigare koden ännu enklare:  

    ```csharp

    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```


I nästa steg ska du läsa [webb-API med OWIN i Reliable Services](service-fabric-reliable-services-communication-webapi.md).
