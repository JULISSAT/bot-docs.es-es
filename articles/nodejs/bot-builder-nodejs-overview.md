---
title: Bot Builder SDK para Node.js | Microsoft Docs
description: Explore el Bot Builder SDK para Node.js, un marco de creación de bots eficaz y fácil de usar.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 24da2cc90908a1f22bee9d2cd007607b116ac2cc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904208"
---
# <a name="bot-builder-sdk-for-nodejs"></a>Bot Builder SDK para Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Bot Builder SDK para Node.js es un marco eficaz y fácil de usar que proporciona una manera conocida para que los desarrolladores de Node.js escriban bots.
Puede usarlo para crear una amplia variedad de interfaces de usuario conversacionales, desde avisos sencillos hasta conversaciones de forma libre.

La lógica conversacional para el bot se aloja como servicio web. El Bot Builder SDK usa <a href="http://restify.com">restify</a>, un marco conocido para la creación de servicios web, para crear el servidor web del bot. El SDK también es compatible con <a href="http://expressjs.com/">Express</a>, y el uso de otros marcos de aplicación web es posible con algunas adaptaciones. 

Mediante el SDK, puede sacar provecho de las características siguientes: 

- Sistema eficaz para crear cuadros de diálogo para encapsular la lógica conversacional.
- Avisos integrados para cosas sencillas, como Sí/No, cadenas, números y enumeraciones, así como compatibilidad con mensajes que contienen imágenes y adjuntos, y tarjetas enriquecidas que contiene botones.
- Compatibilidad integrada con plataformas de inteligencia artificial como eficaces, como <a href="http://luis.ai" target="_blank">LUIS</a>.
- Reconocedores integrados y controladores de eventos que guían al usuario a través de la conversación, brindando ayuda, navegación, aclaraciones y confirmaciones según sea necesario.

## <a name="get-started"></a>Introducción

Si no está familiarizado con la escritura de bots, [cree su primer bot con Node.js](bot-builder-nodejs-quickstart.md) con instrucciones paso a paso que le ayudarán a configurar el proyecto, instalar el SDK y ejecutar su primer bot. 

Si no está familiarizado con el Bot Builder SDK para Node.js, puede comenzar con los conceptos clave que le ayudarán a comprender los componentes principales del Bot Builder SDK, consulte [Conceptos clave](bot-builder-nodejs-concepts.md).

Para asegurarse de que el bot aborde los escenarios de usuario principales, revise los [principios de diseño](../bot-service-design-principles.md) y [explore los patrones](../bot-service-design-pattern-task-automation.md) para obtener instrucciones.

## <a name="get-samples"></a>Obtención de ejemplos

En los [ejemplos del Bot Builder SDK para Node.js](bot-builder-nodejs-samples.md) se describen bots basados en tareas en los que se muestra cómo aprovechar las características del Bot Builder SDK para Node.js. Puede usar los ejemplos para empezar a trabajar rápidamente en la compilación de bots excelentes con funcionalidades enriquecidas.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Conceptos clave](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>Recursos adicionales

Las siguientes guías de procedimientos enfocada en tareas demuestran diversas características del Bot Builder SDK para Node.js.

* [Responder a mensajes](bot-builder-nodejs-use-default-message-handler.md)
* [Control de acciones del usuario](bot-builder-nodejs-dialog-actions.md)
* [Reconocimiento de intenciones del usuario](bot-builder-nodejs-recognize-intent-messages.md)
* [Envío de una tarjeta enriquecida](bot-builder-nodejs-send-rich-cards.md)
* [Envío de archivos adjuntos](bot-builder-nodejs-send-receive-attachments.md)
* [Guardado de datos de usuario](bot-builder-nodejs-save-user-data.md)


Si tiene problemas o sugerencias sobre el Bot Builder SDK para Node.js, consulte el [soporte técnico](../bot-service-resources-links-help.md) para obtener una lista de los recursos disponibles. 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 
