---
title: Habilitación de voz en un chat en web | Microsoft Docs
description: Obtenga información sobre cómo habilitar la característica de voz en un control de chat en web para un bot conectado al canal Chat en web.
keywords: característica de voz, chat en web, voz, audio, micrófono
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b83dff7969c58451e5752938f74b682b2163c49d
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298205"
---
# <a name="enable-speech-in-web-chat"></a>Habilitación de voz en un chat en web
Puede habilitar una interfaz de voz en el control de chat en web. Los usuarios interactúan con la interfaz de voz utilizando el micrófono en el control de chat en web.

![Muestra de voz en el chat en web](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

Si el usuario escribe en lugar de decir la respuesta, Chat en web desactiva la funcionalidad de voz y el bot solo da una respuesta textual en lugar de hablar en voz alta. Para volver a habilitar la respuesta hablada, el usuario puede usar el micrófono para responder al bot la próxima vez. Si el micrófono está aceptando entradas, aparecerá oscurecido o rellenado. Si está atenuado, el usuario hace clic en él para habilitarlo.

## <a name="prerequisites"></a>Requisitos previos

  Antes de ejecutar la muestra, debe tener un secreto o un token de línea directa para el bot que quiera ejecutar con el control de Chat en web. 
  * Consulte [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Conectar un bot a la línea directa) para obtener información sobre cómo obtener un secreto de línea directa asociado al bot.
  * Consulte [Generate a Direct Line token](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Generar un token de línea directa) para obtener información sobre cómo intercambiar el secreto de un token.

## <a name="customizing-web-chat-for-speech"></a>Personalizar el chat en web para la característica de voz
Para habilitar la funcionalidad de voz en Chat en web, debe personalizar el código de JavaScript que invoca el control de Chat en web. Puede probar el Chat en web habilitado para la característica de voz de forma local, mediante los siguientes pasos.

1. Descargue el [ejemplo index.html](https://aka.ms/web-chat-speech-sample). <!-- this aka.ms link needs to be updated if the sample location changes -->
2. Edite el código en `index.html` según el tipo de soporte de voz que desee agregar. Los tipos de implementaciones de voz se describen en [Habilitar servicios de voz](#enable-speech-services). 
3. Inicie un servidor web. Una forma de hacerlo es usar `npm http-server` en el símbolo del sistema Node.js.

   * Para instalar `http-server` de forma global para que pueda ejecutarse desde la línea de comandos, ejecute este comando:

     ```
     npm install http-server -g
     ```

   * Para iniciar un servidor web usando el puerto 8000, desde el directorio que contiene `index.html`, ejecute este comando:

     ```
     http-server -p 8000
     ```
4. Dirija el explorador a `http://localhost:8000/samples?parameters`. Por ejemplo, `http://localhost:8000/samples?s=YOURDIRECTLINESECRET` invoca el bot con un secreto de línea directa. Los parámetros que se pueden establecer en la cadena de consulta se describen en la tabla siguiente:

   | Parámetro | DESCRIPCIÓN |
   |-----------|-------------|
   | s | Secreto de línea directa. Consulte [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Conectar un bot a la línea directa) para obtener información sobre cómo obtener un secreto de línea directa. |
   | t | Token de línea directa. Consulte [Generate a Direct Line token](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Crear un token de línea directa) para obtener información sobre cómo generar este token. |
   | dominio | Opcional. Dirección URL de un punto de conexión de línea directa alternativo.  |
   | WebSocket | Opcional. Establecer en "true" para usar WebSocket para recibir mensajes. El valor predeterminado es `false`. |
   | userid | Opcional. Id. del usuario del bot.  |
   | username | Opcional. Nombre de usuario del usuario del bot.  |
   | botid | Opcional. Id. del bot. |
   | botname | Opcional. Nombre del bot. |


## <a name="enable-speech-services"></a>Habilitar servicios de voz
La personalización le permite agregar la funcionalidad de voz de cualquiera de las maneras siguientes:

* **Característica de voz que proporciona el explorador**: utilice la funcionalidad de voz integrada en el explorador web. En este momento, esta funcionalidad solo está disponible en el explorador Chrome.
<!--* **Use Bing Speech service** - You can use the Bing Speech service to provide speech recognition and synthesis. This way of access speech functionality is supported by a variety of browsers. In this case, the processing is done on a server instead of on the browser.-->
* **Crear un servicio de voz personalizado**: puede crear sus propios componentes personalizados de reconocimiento y síntesis de voz.

### <a name="browser-provided-speech"></a>Característica de voz que proporciona el explorador

El siguiente código crea una instancia del reconocedor de voz y los componentes de síntesis de voz que vienen con el explorador. Este método de agregar la funcionalidad de voz no es compatible con todos los exploradores. 

> [!NOTE] 
> Google Chrome admite el reconocedor de voz del explorador. Sin embargo, Chrome puede bloquear el micrófono en los siguientes casos:
> * Si la dirección URL de la página que contiene el Chat en web comienza con `http://` en lugar de `https://`.
> * Si la dirección URL es un archivo local que usa el protocolo `file://` en lugar de `http://localhost:8000`.

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

<!--### Bing Speech service

The following code instantiates speech recognizer and speech synthesis components that use the Bing Speech service. The recognition and generation of speech is performed on the server. This mechanism is supported in multiple browsers. 

> [!TIP]
> You can use speech recognition priming to improve your bot's speech recognition accuracy if you use the Bing Speech service. For more information, check out the [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text) blog post.

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### Use the Bing Speech service with a token

You also have the option to enable Cognitive Services speech recognition using a token. The token is generated in a secure back end using your API key.

The following example code shows how the token fetch is done from a secure back end to avoid exposing the API key.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]
-->
### <a name="custom-speech-service"></a>Custom Speech Service

También puede proporcionar su propio reconocimiento de voz personalizado que implementa ISpeechRecognizer, o la síntesis de voz que implementa ISpeechSynthesis. 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>Pasar las opciones de voz al Chat en web

El código siguiente pasa las opciones de voz al control de Chat en web:

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>Pasos siguientes
Ahora que puede habilitar la interacción de voz con el Chat en web, puede ver el método que usa el bot para crear los mensajes de voz y ajustar el estado del micrófono:
* [Agregar voz a los mensajes (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [Agregar voz a los mensajes (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>Recursos adicionales

* También puede [descargar el código fuente](https://github.com/Microsoft/BotFramework-WebChat) para el control de Chat en web en GitHub.
* Asimismo, la [documentación de Bing Speech API](https://docs.microsoft.com/azure/cognitive-services/speech/home) proporciona más información sobre Bing Speech API.

