---
title: Usando os hubs de SignalR do ASP.NET Core
author: tdykstra
description: Saiba como usar os hubs do SignalR do ASP.NET Core.
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 05/01/2018
uid: signalr/hubs
ms.openlocfilehash: e583676ab0eed45aeaf6391d8cdf8c1485aa914e
ms.sourcegitcommit: e7e1e531b80b3f4117ff119caadbebf4dcf5dcb7
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/12/2018
ms.locfileid: "44510331"
---
# <a name="use-hubs-in-signalr-for-aspnet-core"></a>Usar os hubs no SignalR do ASP.NET Core

Por [Rachel Appel](https://twitter.com/rachelappel) e [Kevin Griffin](https://twitter.com/1kevgriff)

[Exibir ou baixar o código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/signalr/hubs/sample/ ) [(como fazer o download)](xref:tutorials/index#how-to-download-a-sample)

## <a name="what-is-a-signalr-hub"></a>O que é um hub SignalR

A API de Hubs de SignalR permite chamar métodos em clientes conectados do servidor. No código do servidor, você define métodos que são chamados pelo cliente. O código do cliente, você define métodos que são chamados do servidor. SignalR cuida de tudo o que nos bastidores que possibilita a comunicação de servidor para cliente e servidor-cliente em tempo real.

## <a name="configure-signalr-hubs"></a>Configurar os hubs de SignalR

O middleware SignalR requer alguns serviços, que são configurados por meio da chamada `services.AddSignalR`.

[!code-csharp[Configure service](hubs/sample/startup.cs?range=38)]

Ao adicionar a funcionalidade do SignalR para um aplicativo ASP.NET Core, configurar as rotas do SignalR chamando `app.UseSignalR` no `Startup.Configure` método.

[!code-csharp[Configure routes to hubs](hubs/sample/startup.cs?range=57-60)]

## <a name="create-and-use-hubs"></a>Criar e usar os hubs

Criar um hub, declarando uma classe que herda de `Hub`e adicione os métodos públicos a ele. Os clientes poderão chamar métodos que são definidos como `public`.

[!code-csharp[Create and use hubs](hubs/sample/hubs/chathub.cs?range=8-37)]

Você pode especificar um tipo de retorno e parâmetros, incluindo tipos complexos e matrizes, como você faria em qualquer método em c#. O SignalR lida com a serialização e desserialização de objetos complexos e matrizes em seus valores de retorno e parâmetros.

## <a name="the-context-object"></a>O objeto de contexto

O `Hub` classe tem um `Context` propriedade que contém as propriedades a seguir com informações sobre a conexão:

| Propriedade | Descrição |
| ------ | ----------- |
| `ConnectionId` | Obtém a ID exclusiva para a conexão, atribuído pelo SignalR. Há uma ID de conexão para cada conexão.|
| `UserIdentifier` | Obtém o [identificador de usuário](xref:signalr/groups). Por padrão, o SignalR usa o `ClaimTypes.NameIdentifier` do `ClaimsPrincipal` associado com a conexão como o identificador de usuário. |
| `User` | Obtém o `ClaimsPrincipal` associado ao usuário atual. |
| `Items` | Obtém uma coleção de chave/valor que pode ser usada para compartilhar dados dentro do escopo dessa conexão. Dados podem ser armazenados nessa coleção e ela será mantida para a conexão entre as invocações de método de hub diferentes. |
| `Features` | Obtém a coleção de recursos disponíveis sobre a conexão. Por enquanto, essa coleção não é necessária na maioria dos cenários, portanto, ele ainda não está documentado em detalhes. |
| `ConnectionAborted` | Obtém um `CancellationToken` que notifica quando a conexão será anulada. |

`Hub.Context` também contém os seguintes métodos:

| Método | Descrição |
| ------ | ----------- |
| `GetHttpContext` | Retorna o `HttpContext` para a conexão, ou `null` se a conexão não está associado uma solicitação HTTP. Para conexões HTTP, você pode usar esse método para obter informações como cadeias de caracteres de consulta e cabeçalhos HTTP. |
| `Abort` | Anula a conexão. |

## <a name="the-clients-object"></a>O objeto de clientes

O `Hub` classe tem um `Clients` propriedade que contém as seguintes propriedades para a comunicação entre cliente e servidor:

| Propriedade | Descrição |
| ------ | ----------- |
| `All` | Chama um método em todos os clientes conectados |
| `Caller` | Chama um método no cliente que invocou o método de hub |
| `Others` | Chama um método em todos os clientes conectados, exceto o cliente que invocou o método |


`Hub.Clients` também contém os seguintes métodos:

| Método | Descrição |
| ------ | ----------- |
| `AllExcept` | Chama um método em todos os clientes conectados, exceto para as conexões especificadas |
| `Client` | Chama um método em um cliente conectado específico |
| `Clients` | Chama um método em clientes conectados específicos |
| `Group` | Chama um método para todas as conexões no grupo especificado  |
| `GroupExcept` | Chama um método para todas as conexões do grupo especificado, exceto as conexões especificadas |
| `Groups` | Chama um método para vários grupos de conexões  |
| `OthersInGroup` | Chama um método a um grupo de conexões, excluindo o cliente que invocou o método de hub  |
| `User` | Chama um método para todas as conexões associadas a um usuário específico |
| `Users` | Chama um método para todas as conexões associadas com os usuários especificados |

Cada propriedade ou método nas tabelas anteriores retorna um objeto com um `SendAsync` método. O `SendAsync` método permite que você forneça o nome e parâmetros do método de cliente para chamar.

## <a name="send-messages-to-clients"></a>Enviar mensagens para os clientes

Para fazer chamadas para clientes específicos, use as propriedades do `Clients` objeto. No exemplo a seguir, o `SendMessageToCaller` método demonstra como enviar uma mensagem para a conexão que invocou o método de hub. O `SendMessageToGroups` método envia uma mensagem para os grupos armazenados em um `List` denominado `groups`.

[!code-csharp[Send messages](hubs/sample/hubs/chathub.cs?range=15-24)]

## <a name="handle-events-for-a-connection"></a>Manipular eventos para uma conexão

A API de Hubs de SignalR fornece o `OnConnectedAsync` e `OnDisconnectedAsync` métodos virtuais para gerenciar e controlar conexões. Substituir o `OnConnectedAsync` método virtual para executar ações quando um cliente se conecta ao Hub, como adicioná-lo a um grupo.

[!code-csharp[Handle events](hubs/sample/hubs/chathub.cs?range=26-36)]

## <a name="handle-errors"></a>Tratar erros

As exceções geradas em seus métodos de hub são enviadas ao cliente que invocou o método. No cliente JavaScript, o `invoke` método retorna um [promessa JavaScript](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises). Quando o cliente recebe um erro com um manipulador anexado à promessa usando `catch`, ele tem chamado e passado como um JavaScript `Error` objeto.

[!code-javascript[Error](hubs/sample/wwwroot/js/chat.js?range=23)]

## <a name="related-resources"></a>Recursos relacionados

* [Introdução ao SignalR do ASP.NET Core](xref:signalr/introduction)
* [Cliente JavaScript](xref:signalr/javascript-client)
* [Publicar no Azure](xref:signalr/publish-to-azure-web-app)
