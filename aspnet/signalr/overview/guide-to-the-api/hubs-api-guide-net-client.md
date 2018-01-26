---
uid: signalr/overview/guide-to-the-api/hubs-api-guide-net-client
title: Guia de API de Hubs de SignalR do ASP.NET - cliente .NET (c#) | Microsoft Docs
author: pfletcher
description: "Este documento fornece uma introdução ao uso da API de Hubs de SignalR versão 2 em clientes .NET, como o armazenamento do Windows (WinRT), WPF, Silverlight e contras..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/10/2014
ms.topic: article
ms.assetid: 6d02d9f7-94e5-4140-9f51-5a6040f274f6
ms.technology: dotnet-signalr
ms.prod: .net-framework
msc.legacyurl: /signalr/overview/guide-to-the-api/hubs-api-guide-net-client
msc.type: authoredcontent
ms.openlocfilehash: c52a02291e18b1dd8a9d95b33fe466d17aae835f
ms.sourcegitcommit: 060879fcf3f73d2366b5c811986f8695fff65db8
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/24/2018
---
<a name="aspnet-signalr-hubs-api-guide---net-client-c"></a>Guia de API de Hubs de SignalR do ASP.NET - cliente .NET (c#)
====================
por [Patrick Fletcher](https://github.com/pfletcher), [Tom Dykstra](https://github.com/tdykstra)

> Este documento fornece uma introdução ao uso da API de Hubs de SignalR versão 2 em clientes do .NET, como aplicativos de console, WPF, Silverlight e da Windows Store (WinRT).
> 
> A API de Hubs de SignalR permite fazer chamadas de procedimento remoto (RPCs) de um servidor para clientes conectados e de clientes para o servidor. No código do servidor, você define métodos que podem ser chamados por clientes e chamar os métodos que são executados no cliente. No código do cliente, você define métodos que podem ser chamados do servidor e chamar os métodos que são executados no servidor. SignalR cuida de todos os detalhes do cliente para servidor para você.
> 
> O SignalR também oferece uma API de nível inferior chamada conexões persistentes. Para obter uma introdução para o SignalR, Hubs e conexões persistentes, ou para um tutorial que mostra como criar um aplicativo SignalR completo, consulte [SignalR - Introdução](../getting-started/index.md).
> 
> ## <a name="software-versions-used-in-this-topic"></a>Versões de software usadas neste tópico
> 
> 
> - [Visual Studio 2013](https://www.microsoft.com/visualstudio/eng/2013-downloads)
> - .NET 4.5
> - SignalR versão 2
>   
> 
> 
> ## <a name="previous-versions-of-this-topic"></a>Versões anteriores deste tópico
> 
> Para obter informações sobre versões anteriores do SignalR, consulte [versões mais antigas do SignalR](../older-versions/index.md).
> 
> ## <a name="questions-and-comments"></a>Perguntas e comentários
> 
> Deixe comentários em como você gostou neste tutorial e o que podemos melhorar nos comentários na parte inferior da página. Se você tiver dúvidas que não estão diretamente relacionadas ao tutorial, você poderá postá-los para o [ASP.NET SignalR fórum](https://forums.asp.net/1254.aspx/1?ASP+NET+SignalR) ou [StackOverflow.com](http://stackoverflow.com/).


## <a name="overview"></a>Visão geral

Este documento contém as seguintes seções:

- [Instalação do cliente](#clientsetup)
- [Como estabelecer uma conexão](#establishconnection)

    - [Conexões entre domínios de clientes do Silverlight](#slcrossdomain)
- [Como configurar a conexão](#configureconnection)

    - [Como definir o número máximo de conexões simultâneas em clientes do WPF](#maxconnections)
    - [Como especificar parâmetros de cadeia de caracteres de consulta](#querystring)
    - [Como especificar o método de transporte](#transport)
    - [Como especificar cabeçalhos HTTP](#httpheaders)
    - [Como especificar os certificados de cliente](#clientcertificate)
- [Como criar o proxy do Hub](#proxy)
- [Como definir métodos no cliente que o servidor pode chamar](#callclient)

    - [Métodos sem parâmetros](#clientmethodswithoutparms)
    - [Métodos com parâmetros, especificando os tipos de parâmetro](#clientmethodswithparmtypes)
    - [Métodos com parâmetros, especificando objetos dinâmicos para os parâmetros](#clientmethodswithdynamparms)
    - [Como remover um manipulador](#removehandler)
- [Como chamar métodos do servidor do cliente](#callserver)
- [Como manipular eventos de tempo de vida da conexão](#connectionlifetime)
- [Como tratar erros](#handleerrors)
- [Como habilitar o registro de cliente](#logging)
- [Exemplos de código de aplicativo de console para métodos de cliente que o servidor pode chamar, Silverlight e WPF](#wpfsl)

Um exemplo .NET para projetos de cliente, consulte os seguintes recursos:

- [Gustavo armenta / SignalR exemplos](https://github.com/gustavo-armenta/SignalR-Samples) em GitHub.com (WinRT, Silverlight, exemplos de aplicativo de console).
- [DamianEdwards / SignalR MoveShapeDemo / MoveShape.Desktop](https://github.com/DamianEdwards/SignalR-MoveShapeDemo/tree/master/MoveShape/MoveShape.Desktop) em GitHub.com (exemplo WPF).
- [SignalR / Microsoft.AspNet.SignalR.Client.Samples](https://github.com/SignalR/SignalR/tree/master/samples/Microsoft.AspNet.SignalR.Client.Samples) em GitHub.com (exemplo de aplicativo de Console).

Para obter a documentação sobre como programar o servidor ou os clientes JavaScript, consulte os seguintes recursos:

- [Guia de API de Hubs de SignalR - servidor](hubs-api-guide-server.md)
- [Guia de API de Hubs de SignalR - cliente JavaScript](hubs-api-guide-javascript-client.md)

Links para tópicos de referência de API são para a versão do .NET 4.5 da API. Se você estiver usando o .NET 4, consulte [a versão do .NET 4 dos tópicos da API](https://msdn.microsoft.com/library/jj891075(v=vs.100).aspx).

<a id="clientsetup"></a>

## <a name="client-setup"></a>Instalação do cliente

Instalar o [SignalR](http://nuget.org/packages/Microsoft.AspNet.SignalR.Client) pacote do NuGet (não o [SignalR](http://nuget.org/packages/microsoft.aspnet.signalr) pacote). Este pacote suporta WinRT, Silverlight, WPF, aplicativo de console e os clientes do Windows Phone, para .NET 4 e o .NET 4.5.

Se a versão do SignalR que têm o cliente é diferente da versão que você tem no servidor, SignalR geralmente é capaz de se adaptar à diferença. Por exemplo, um servidor que executa a versão 2 do SignalR suportará clientes que têm 1.1 instalado, bem como os clientes que têm a versão 2 instalado. Se a diferença entre a versão no servidor e a versão do cliente for muito grande, ou se o cliente for mais recente do que o servidor, SignalR lança um `InvalidOperationException` exceção quando o cliente tenta estabelecer uma conexão. A mensagem de erro é "`You are using a version of the client that isn't compatible with the server. Client version X.X, server version X.X`".

<a id="establishconnection"></a>

## <a name="how-to-establish-a-connection"></a>Como estabelecer uma conexão

Antes de estabelecer uma conexão, você precisa criar um `HubConnection` de objeto e criar um proxy. Para estabelecer a conexão, chame o `Start` método sobre o `HubConnection` objeto.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample1.cs?highlight=1,4)]

> [!NOTE]
> Para clientes JavaScript que você precisa registrar pelo menos um manipulador de eventos antes de chamar o `Start` método para estabelecer a conexão. Isso não é necessário para clientes do .NET. Para clientes de JavaScript, o código de proxy gerado automaticamente cria proxies para todos os Hubs que existem no servidor e registrar um manipulador é como você indicar quais Hubs o cliente pretende usar. Mas para um cliente .NET criar proxies Hub manualmente, para que o SignalR pressupõe que você usará qualquer Hub que você criar um proxy para.


O código de exemplo usa o padrão "/ signalr" URL para se conectar ao seu serviço SignalR. Para obter informações sobre como especificar uma URL base diferente, consulte [ASP.NET SignalR Hubs API guia - Server - a URL /signalr](hubs-api-guide-server.md#signalrurl).

O `Start` método é executado de forma assíncrona. Para certificar-se de que as linhas subsequentes de código não executar até depois que a conexão é estabelecida, use `await` em um método assíncrono do ASP.NET 4.5 ou `.Wait()` em um método síncrono. Não use `.Wait()` em um cliente do WinRT.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample2.cs?highlight=1)]

[!code-css[Main](hubs-api-guide-net-client/samples/sample3.css?highlight=1)]

<a id="slcrossdomain"></a>

### <a name="cross-domain-connections-from-silverlight-clients"></a>Conexões entre domínios de clientes do Silverlight

Para obter informações sobre como habilitar conexões entre domínios de clientes do Silverlight, consulte [tornando um serviço disponível através dos limites do domínio](https://msdn.microsoft.com/library/cc197955(v=vs.95).aspx).

<a id="configureconnection"></a>

## <a name="how-to-configure-the-connection"></a>Como configurar a conexão

Antes de estabelecer uma conexão, você pode especificar qualquer uma das seguintes opções:

- Limite de conexões simultâneas.
- Parâmetros de cadeia de caracteres de consulta.
- O método de transporte.
- Cabeçalhos HTTP.
- Certificados de cliente.

<a id="maxconnections"></a>

### <a name="how-to-set-the-maximum-number-of-concurrent-connections-in-wpf-clients"></a>Como definir o número máximo de conexões simultâneas em clientes do WPF

Em clientes do WPF, você talvez precise aumentar o número máximo de conexões simultâneas de seu valor padrão de 2. O valor recomendado é 10.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample4.cs?highlight=4)]

Para obter mais informações, consulte [ServicePointManager.DefaultConnectionLimit](https://msdn.microsoft.com/library/system.net.servicepointmanager.defaultconnectionlimit.aspx).

<a id="querystring"></a>

### <a name="how-to-specify-query-string-parameters"></a>Como especificar parâmetros de cadeia de caracteres de consulta

Se você deseja enviar dados para o servidor quando o cliente se conecta, você pode adicionar parâmetros de cadeia de caracteres de consulta para o objeto de conexão. O exemplo a seguir mostra como definir um parâmetro de cadeia de caracteres de consulta no código do cliente.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample5.cs)]

O exemplo a seguir mostra como ler um parâmetro de cadeia de caracteres de consulta no código do servidor.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample6.cs?highlight=5)]

<a id="transport"></a>

### <a name="how-to-specify-the-transport-method"></a>Como especificar o método de transporte

Como parte do processo de conexão, um cliente SignalR normalmente negocia com o servidor para determinar o melhor transporte com suporte pelo servidor e cliente. Se você já souber qual transporte você deseja usar, você pode ignorar esse processo de negociação. Para especificar o método de transporte, passe um objeto de transporte para o método Start. O exemplo a seguir mostra como especificar o método de transporte no código do cliente.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample7.cs?highlight=4)]

O [Transports](https://msdn.microsoft.com/library/jj918090(v=vs.111).aspx) namespace inclui as seguintes classes que você pode usar para especificar o transporte.

- [LongPollingTransport](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.client.transports.longpollingtransport(v=vs.111).aspx)
- [ServerSentEventsTransport](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.client.transports.serversenteventstransport(v=vs.111).aspx)
- [WebSocketTransport](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.client.transports.websockettransport(v=vs.111).aspx) (disponível somente quando o servidor e o cliente usam o .NET 4.5).
- [AutoTransport](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.client.transports.autotransport(v=vs.111).aspx) (escolhe automaticamente o melhor transporte com suporte pelo cliente e o servidor. Este é o transporte padrão. Transmitindo-para o `Start` método tem o mesmo efeito que não está transmitindo nada.)

O transporte ForeverFrame não está incluído nesta lista porque ele é usado somente por navegadores.

Para obter informações sobre como verificar o método de transporte no código do servidor, consulte [guia de API de Hubs do ASP.NET SignalR - Server - como obter informações sobre o cliente da propriedade de contexto](hubs-api-guide-server.md#contextproperty). Para obter mais informações sobre os transportes e restaurações, consulte [Introdução ao SignalR - transportes e Fallbacks](../getting-started/introduction-to-signalr.md#transports).

<a id="httpheaders"></a>

### <a name="how-to-specify-http-headers"></a>Como especificar cabeçalhos HTTP

Para definir cabeçalhos HTTP, use o `Headers` propriedade no objeto de conexão. O exemplo a seguir mostra como adicionar um cabeçalho HTTP.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample8.cs?highlight=2)]

<a id="clientcertificate"></a>

### <a name="how-to-specify-client-certificates"></a>Como especificar os certificados de cliente

Para adicionar certificados de cliente, use o `AddClientCertificate` método no objeto de conexão.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample9.cs?highlight=2)]

<a id="proxy"></a>

## <a name="how-to-create-the-hub-proxy"></a>Como criar o proxy do Hub

Para definir métodos no cliente que pode chamar um Hub do servidor e para invocar métodos em um Hub no servidor, crie um proxy para o Hub chamando `CreateHubProxy` no objeto de conexão. A cadeia de caracteres para o qual você passar `CreateHubProxy` é o nome da classe do Hub ou o nome especificado pelo `HubName` atributo se estava sendo usado no servidor. Correspondência de nome diferencia maiusculas de minúsculas.

**Classe de Hub no servidor**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample10.cs?highlight=1)]

**Criar proxy de cliente para a classe de Hub**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample11.cs?highlight=2)]

Se você decora sua classe de Hub com um `HubName` de atributo, use esse nome.

**Classe de Hub no servidor**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample12.cs)]

**Criar proxy de cliente para a classe de Hub**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample13.cs?highlight=2)]

Se você chamar `HubConnection.CreateHubProxy` várias vezes com o mesmo `hubName`, você obtém a mesma em cache `IHubProxy` objeto.

<a id="callclient"></a>

## <a name="how-to-define-methods-on-the-client-that-the-server-can-call"></a>Como definir métodos no cliente que o servidor pode chamar

Para definir um método que o servidor pode chamar, use o proxy `On` método para registrar um manipulador de eventos.

Correspondência de nome do método diferencia maiusculas de minúsculas. Por exemplo, `Clients.All.UpdateStockPrice` no servidor executará `updateStockPrice`, `updatestockprice`, ou `UpdateStockPrice` no cliente.

Plataformas de cliente diferentes têm requisitos diferentes como escrever o código do método para atualizar a interface do usuário. Os exemplos mostrados são para clientes do WinRT (Windows Store .NET). WPF, Silverlight e exemplos de aplicativos de console são fornecidos nos [uma seção separada neste tópico](#wpfsl).

<a id="clientmethodswithoutparms"></a>

### <a name="methods-without-parameters"></a>Métodos sem parâmetros

Se o método que você está tratando não tiver parâmetros, use a sobrecarga de não-genérica do `On` método:

**Código do servidor chamando o método de cliente sem parâmetros**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample14.cs?highlight=5)]

**Código do cliente do WinRT para método de chamada do servidor sem parâmetros ([consulte WPF e Silverlight exemplos mais adiante neste tópico](#wpfsl))**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample15.cs)]

<a id="clientmethodswithparmtypes"></a>

### <a name="methods-with-parameters-specifying-the-parameter-types"></a>Métodos com parâmetros, especificando os tipos de parâmetro

Se o método que você está lidando com tiver parâmetros, especifique os tipos dos parâmetros como os tipos genéricos do `On` método. Há sobrecargas genéricas do `On` método para permitir que você especifique parâmetros até 8 (4 no Windows Phone 7). No exemplo a seguir, um parâmetro é enviado para o `UpdateStockPrice` método.

**Chamar o método de cliente com um parâmetro de código do servidor**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample16.cs?highlight=3)]

**A classe de estoque usada para o parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample17.cs)]

**Código do cliente do WinRT para um método de chamada do servidor com um parâmetro ([consulte WPF e Silverlight exemplos mais adiante neste tópico](#wpfsl))**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample18.cs?highlight=1,5)]

<a id="clientmethodswithdynamparms"></a>

### <a name="methods-with-parameters-specifying-dynamic-objects-for-the-parameters"></a>Métodos com parâmetros, especificando objetos dinâmicos para os parâmetros

Como uma alternativa para especificar parâmetros como tipos genéricos do `On` método, você pode especificar parâmetros como objetos dinâmicos:

**Chamar o método de cliente com um parâmetro de código do servidor**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample19.cs?highlight=3)]

**A classe de estoque usada para o parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample20.cs)]

**Código do cliente do WinRT para um método chamado a partir do servidor com um parâmetro, usando um objeto dinâmico para o parâmetro ([consulte WPF e Silverlight exemplos mais adiante neste tópico](#wpfsl))**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample21.cs?highlight=1,5)]

<a id="removehandler"></a>

### <a name="how-to-remove-a-handler"></a>Como remover um manipulador

Para remover um manipulador, chame seu `Dispose` método.

**Código do cliente para um método chamado a partir do servidor**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample22.cs?highlight=1)]

**Código do cliente para remover o manipulador**

[!code-css[Main](hubs-api-guide-net-client/samples/sample23.css?highlight=1)]

<a id="callserver"></a>

## <a name="how-to-call-server-methods-from-the-client"></a>Como chamar métodos do servidor do cliente

Para chamar um método no servidor, use o `Invoke` método no proxy do Hub.

Se o método de servidor não tem valor retornado, use a sobrecarga de não-genérica do `Invoke` método.

**Código do servidor para um método que não tem nenhum valor de retorno**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample24.cs?highlight=3)]

**Código do cliente chamar um método que não tem nenhum valor de retorno**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample25.cs?highlight=1)]

Se o método de servidor tem um valor de retorno, especifique o tipo de retorno do tipo genérico do `Invoke` método.

**Código do servidor para um método que tem um valor de retorno e usa um parâmetro de tipo complexo**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample26.cs?highlight=1)]

**A classe de estoque usada para o parâmetro e valor de retorno**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample27.cs)]

**Código do cliente chamar um método que tem um valor de retorno e usa um parâmetro de tipo complexo, em um método assíncrono de ASP.NET 4.5**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample28.cs?highlight=1-2)]

**Código do cliente chamar um método que tem um valor de retorno e usa um parâmetro de tipo complexo, em um método síncrono**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample29.cs?highlight=1-2)]

O `Invoke` método executa de forma assíncrona e retorna um `Task` objeto. Se você não especificar `await` ou `.Wait()`, a próxima linha de código será executado antes do método que você chamar concluiu a execução.

<a id="connectionlifetime"></a>

## <a name="how-to-handle-connection-lifetime-events"></a>Como manipular eventos de tempo de vida da conexão

O SignalR fornece a seguinte conexão eventos de tempo de vida que você pode manipular:

- `Received`: Gerado quando nenhum dado é recebido na conexão. Fornece os dados recebidos.
- `ConnectionSlow`: Gerado quando o cliente detecta uma conexão lenta ou remoção com frequência.
- `Reconnecting`: Gerado quando o transporte subjacente inicia a reconexão.
- `Reconnected`: Gerado quando o transporte subjacente foi reconectada.
- `StateChanged`: Gerado quando o estado da conexão é alterado. Fornece o estado antigo e o novo estado. Para obter informações sobre conexão consulte valores de estado [enumeração ConnectionState](https://msdn.microsoft.com/library/microsoft.aspnet.signalr.client.connectionstate(v=vs.111).aspx).
- `Closed`: Gerado quando a conexão foi desconectado.

Por exemplo, se você quiser exibir mensagens de aviso para erros não fatais mas causam problemas intermitentes de conexão, como lentidão ou frequentes descartando de conexão, tratar o `ConnectionSlow` evento.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample30.cs)]

Para obter mais informações, consulte [compreensão e tratamento de eventos de tempo de vida de Conexão no SignalR](handling-connection-lifetime-events.md).

<a id="handleerrors"></a>

## <a name="how-to-handle-errors"></a>Como tratar erros

Se você não habilitar explicitamente as mensagens de erro detalhadas no servidor, o objeto de exceção SignalR retorna após um erro contém algumas informações sobre o erro. Por exemplo, se uma chamada para `newContosoChatMessage` falhar, a mensagem de erro no objeto de erro contém "`There was an error invoking Hub method 'contosoChatHub.newContosoChatMessage'.`" enviar mensagens de erro detalhadas para clientes de produção não é recomendado por motivos de segurança, mas se você deseja habilitar as mensagens de erro detalhadas para solução de problemas, use o seguinte código no servidor.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample31.cs?highlight=2)]

<a id="handleerrors"></a>

Para tratar erros que gera o SignalR, você pode adicionar um manipulador para o `Error` evento no objeto de conexão.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample32.cs)]

Para tratar erros de invocações do método, encapsule o código em um bloco try-catch.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample33.cs)]

<a id="logging"></a>

## <a name="how-to-enable-client-side-logging"></a>Como habilitar o registro de cliente

Para habilitar o registro em log do lado do cliente, defina o `TraceLevel` e `TraceWriter` propriedades no objeto de conexão.

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample34.cs?highlight=2-3)]

<a id="wpfsl"></a>

## <a name="wpf-silverlight-and-console-application-code-samples-for-client-methods-that-the-server-can-call"></a>Exemplos de código de aplicativo de console para métodos de cliente que o servidor pode chamar, Silverlight e WPF

Os exemplos de código mostrados anteriormente para definir métodos de cliente que o servidor pode chamar se aplicam aos clientes do WinRT. Os exemplos a seguir mostram o código equivalente para WPF, Silverlight e clientes de aplicativo de console.

### <a name="methods-without-parameters"></a>Métodos sem parâmetros

**Código do cliente WPF para o método chamado a partir do servidor sem parâmetros**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample35.cs?highlight=1)]

**Código do cliente Silverlight para o método chamado a partir do servidor sem parâmetros**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample36.cs?highlight=1)]

**Código de cliente do aplicativo de console para o método chamado a partir do servidor sem parâmetros**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample37.cs?highlight=1)]

### <a name="methods-with-parameters-specifying-the-parameter-types"></a>Métodos com parâmetros, especificando os tipos de parâmetro

**Código de cliente do WPF para um método chamado a partir do servidor com um parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample38.cs?highlight=1,4)]

**Código do cliente Silverlight para um método chamado a partir do servidor com um parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample39.cs?highlight=1,5)]

**Código de cliente do aplicativo de console para um método chamado a partir do servidor com um parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample40.cs?highlight=1-2)]

### <a name="methods-with-parameters-specifying-dynamic-objects-for-the-parameters"></a>Métodos com parâmetros, especificando objetos dinâmicos para os parâmetros

**Código de cliente do WPF para um método chamado a partir do servidor com um parâmetro, usando um objeto dinâmico para o parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample41.cs?highlight=1,4)]

**Código do cliente Silverlight para um método chamado a partir do servidor com um parâmetro, usando um objeto dinâmico para o parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample42.cs?highlight=1,5)]

**Código de cliente do aplicativo de console para um método chamado a partir do servidor com um parâmetro, usando um objeto dinâmico para o parâmetro**

[!code-csharp[Main](hubs-api-guide-net-client/samples/sample43.cs?highlight=1-2)]