---
title: Interceptación de mensajes | Microsoft Docs
description: Aprenda a crear registros mediante la interceptación y el procesamiento de intercambios de información con el SDK de Bot Builder para Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f7470fa3b3702b9231bc03566aa85b23ee883a05
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904218"
---
# <a name="intercept-messages"></a>Interceptación de mensajes

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>Ejemplo

En el ejemplo de código siguiente se muestra cómo interceptar los mensajes intercambiados entre un usuario y un bot por medio del concepto de **middleware** en el SDK de Bot Builder para Node.js. 

En primer lugar, configure el controlador de mensajes entrantes (`botbuilder`) y mensajes salientes (`send`).

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

A continuación, implemente cada uno de los controladores para definir la acción que se realizará con cada mensaje que se intercepte.

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

Ahora, cada mensaje entrante (del usuario al bot) desencadenará `logIncomingMessage`, y cada mensaje saliente (del bot al usuario) desencadenará `logOutgoingMessage`.
En este ejemplo, el bot simplemente imprime información sobre cada mensaje, pero puede actualizar `logIncomingMessage` y `logOutgoingMessage` según sea necesario para definir las acciones que quiere realizar con cada mensaje. 

## <a name="sample-code"></a>Código de ejemplo

Para ver un ejemplo completo que muestra cómo interceptar y registrar mensajes mediante el SDK de Bot Builder para Node.js, consulte el <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-middlewareLogging" target="_blank">ejemplo de middleware y registro</a> en GitHub.
