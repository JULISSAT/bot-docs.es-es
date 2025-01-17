---
title: Reconocimiento de intenciones a partir del contenido del mensaje | Microsoft Docs
description: Aprenda a reconocer la intención del usuario mediante expresiones regulares o comprobando el contenido del mensaje.
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 370ba1fc756ecea8fd745067109fdb13b6e825a9
ms.sourcegitcommit: 008aa6223aef800c3abccda9a7f72684959ce5e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2019
ms.locfileid: "70026314"
---
# <a name="recognize-user-intent-from-message-content"></a>Reconocimiento de intenciones del usuario a partir del contenido del mensaje

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Cuando su bot recibe un mensaje de un usuario, el bot puede usar un **reconocedor** para examinar el mensaje y determinar la intención. La intención proporciona una asignación de mensajes a diálogos que se puede invocar. En este artículo se explica cómo reconocer la intención mediante expresiones regulares o inspeccionando el contenido de un mensaje. Por ejemplo, un bot puede utilizar expresiones regulares para comprobar si un mensaje contiene la palabra "ayuda" e invocar un diálogo de ayuda. Un bot también puede comprobar las propiedades del mensaje del usuario, por ejemplo, para ver si el usuario envió una imagen en lugar de texto e invocar un diálogo de procesamiento de imágenes.

> [!NOTE]
> Para más información acerca del reconocimiento de intenciones mediante LUIS, consulte [Reconocimiento de intenciones y entidades con LUIS](bot-builder-nodejs-recognize-intent-luis.md).

## <a name="use-the-built-in-regular-expression-recognizer"></a>Uso del reconocedor de expresiones regulares integrado

Use [RegExpRecognizer][RegExpRecognizer] para detectar la intención del usuario mediante una expresión regular. Puede pasar varias expresiones al reconocedor para admitir varios idiomas.

> [!TIP]
> Consulte [Compatibilidad con la localización](bot-builder-nodejs-localization.md) para más información sobre la localización del bot.

El código siguiente crea un reconocedor de expresiones regulares denominado `CancelIntent` y lo agrega al bot. El reconocedor de este ejemplo proporciona expresiones regulares para las configuraciones regionales `en_us` y `ja_jp`.

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

Una vez que se agrega el reconocedor al bot, adjunte una acción [triggerAction][triggerAction] al diálogo que desea que el bot invoque cuando el reconocedor detecte la intención. Use la opción `matches` para especificar el nombre de la intención, tal como se muestra en el código siguiente:

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

Los reconocedores de intenciones son globales, lo que significa que el reconocedor se ejecutará para cada mensaje recibido del usuario. Si un reconocedor detecta una intención que está enlazada a un diálogo mediante `triggerAction`, puede desencadenar la interrupción del diálogo que está activo actualmente. Permitir y controlar interrupciones es un diseño flexible que representa lo que el usuario hace realmente.

> [!TIP]
> Para aprender como funciona un `triggerAction` con diálogos, consulte [Administración del flujo de conversación](bot-builder-nodejs-manage-conversation-flow.md). Para más información sobre las diversas acciones que puede asociar con una intención reconocida, consulte [Controlar acciones del usuario](bot-builder-nodejs-dialog-actions.md).

## <a name="register-a-custom-intent-recognizer"></a>Registro de un reconocedor de intenciones personalizado

También puede implementar un reconocedor personalizado. En este ejemplo se agrega un reconocedor simple que comprueba si el usuario ha dicho "ayuda" o "adiós", pero podría agregar fácilmente un reconocedor que realizara un procesamiento más complejo como, por ejemplo, uno que reconozca cuándo envía el usuario una imagen.

[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

Una vez que haya registrado un reconocedor, puede asociarlo con una acción mediante una cláusula `matches`.

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>Eliminación de ambigüedad entre varias intenciones

El bot puede registrar más de un reconocedor. Tenga en cuenta que el ejemplo del reconocedor personalizado incluye la asignación de una puntuación numérica a cada intención. Esto se hace porque el bot puede tener más de un reconocedor y Bot Framework SDK proporciona una lógica integrada para eliminar la ambigüedad entre las intenciones que devuelven los diversos reconocedores. La puntuación asignada a una intención suele estar comprendida entre 0.0 y 1.0, pero un reconocedor personalizado puede definir una intención superior a 1.1 para asegurarse de que la lógica de eliminación de ambigüedades de Bot Framework SDK la elija siempre.

De forma predeterminada, los reconocedores se ejecutan en paralelo, pero puede establecer recognizeOrder en [IIntentRecognizerSetOptions][IIntentRecognizerSetOptions] para que el proceso se cierre en cuanto el bot encuentre uno con una puntuación de 1.0.

Bot Framework SDK incluye un [ejemplo][DisambiguationSample] que muestra cómo proporcionar una lógica personalizada de eliminación de ambigüedades en el bot mediante la implementación de [IDisambiguateRouteHandler][IDisambiguateRouteHandler].

## <a name="next-steps"></a>Pasos siguientes

La lógica para usar expresiones regulares e inspeccionar contenidos de mensajes puede ser compleja, especialmente si el flujo de conversación del bot es de final abierto. Para ayudar a su bot a controlar una amplia variedad de entradas de texto y voz de los usuarios, puede usar un servicio de reconocimiento de intenciones como [LUIS][LUIS] para agregar comprensión del lenguaje natural al bot.

> [!div class="nextstepaction"]
> [Reconocimiento de intenciones y entidades con LUIS](bot-builder-nodejs-recognize-intent-luis.md)

[LUIS]: https://www.luis.ai/

[IDisambiguateRouteHandler]:   https://docs.microsoft.com/javascript/api/botbuilder/idisambiguateroutehandler?view=botbuilder-ts-3.0
[IIntentRecognizerSetOptions]: https://docs.microsoft.com/javascript/api/botbuilder/iintentrecognizersetoptions?view=botbuilder-ts-3.0
[RegExpRecognizer]:            https://docs.microsoft.com/javascript/api/botbuilder/regexprecognizer?view=botbuilder-ts-3.0
[triggerAction]:               https://docs.microsoft.com/javascript/api/botbuilder/dialog?view=botbuilder-ts-3.0#triggeraction-itriggeractionoptions-

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
