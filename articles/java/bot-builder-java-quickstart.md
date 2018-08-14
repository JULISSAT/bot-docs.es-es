---
title: Creación de un bot con el SDK de Bot Builder para Java | Microsoft Docs
description: Cree un bot de forma rápida con el SDK de Bot Builder para Java.
keywords: SDK de Bot Builder, crear un bot, inicio rápido, introducción
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ca4714d3b3988fd08021f55a4905d9426996b7eb
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305116"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-java"></a>Creación de un bot con el SDK de Bot Builder para Java
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

El SDK de Bot Builder para Java proporciona a los desarrolladores de Java una manera conocida de escribir bots. El SDK v4 está en versión preliminar; para más información, visite el [repositorio de GitHub](https://github.com/Microsoft/botbuilder-java) para Java.

> [!NOTE]
> Nuestros ejemplos de código y documentos van dirigidos actualmente a Java versión 1.8.

## <a name="getting-started"></a>Introducción

El SDK v4 está formado por una serie de [bibliotecas](https://github.com/Microsoft/botbuilder-java/tree/master/libraries). Para compilarlas localmente, consulte [Building the SDK](https://github.com/Microsoft/botbuilder-java/wiki/building-the-sdk) (Compilación del SDK).

- Instale [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases)

### <a name="create-echobot"></a>Creación de EchoBot

En el archivo App.java, agregue lo siguiente:

```Java
package com.microsoft.bot.connector.sample;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.microsoft.aad.adal4j.AuthenticationException;
import com.microsoft.bot.connector.customizations.CredentialProvider;
import com.microsoft.bot.connector.customizations.CredentialProviderImpl;
import com.microsoft.bot.connector.customizations.JwtTokenValidation;
import com.microsoft.bot.connector.customizations.MicrosoftAppCredentials;
import com.microsoft.bot.connector.implementation.ConnectorClientImpl;
import com.microsoft.bot.schema.models.Activity;
import com.microsoft.bot.schema.models.ActivityTypes;
import com.microsoft.bot.schema.models.ResourceResponse;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpServer;

import java.io.IOException;
import java.io.InputStream;
import java.net.InetSocketAddress;
import java.net.URLDecoder;
import java.util.logging.Level;
import java.util.logging.Logger;

public class App {
    private static final Logger LOGGER = Logger.getLogger( App.class.getName() );
    private static String appId = "";       // <-- app id -->
    private static String appPassword = ""; // <-- app password -->

    public static void main( String[] args ) throws IOException {
        CredentialProvider credentialProvider = new CredentialProviderImpl(appId, appPassword);
        HttpServer server = HttpServer.create(new InetSocketAddress(3978), 0);
        server.createContext("/api/messages", new MessageHandle(credentialProvider));
        server.setExecutor(null);
        server.start();
    }

    static class MessageHandle implements HttpHandler {
        private ObjectMapper objectMapper;
        private CredentialProvider credentialProvider;
        private MicrosoftAppCredentials credentials;

        MessageHandle(CredentialProvider credentialProvider) {
            this.objectMapper = new ObjectMapper()
                    .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false)
                    .findAndRegisterModules();
            this.credentialProvider = credentialProvider;
            this.credentials = new MicrosoftAppCredentials(appId, appPassword);
        }

        public void handle(HttpExchange httpExchange) throws IOException {
            if (httpExchange.getRequestMethod().equalsIgnoreCase("POST")) {
                Activity activity = getActivity(httpExchange);
                String authHeader = httpExchange.getRequestHeaders().getFirst("Authorization");
                try {
                    JwtTokenValidation.assertValidActivity(activity, authHeader, credentialProvider);

                    // send ack to user activity
                    httpExchange.sendResponseHeaders(202, 0);
                    httpExchange.getResponseBody().close();

                    if (activity.type().equals(ActivityTypes.MESSAGE)) {
                        // reply activity with the same text
                        ConnectorClientImpl connector = new ConnectorClientImpl(activity.serviceUrl(), this.credentials);
                        ResourceResponse response = connector.conversations().sendToConversation(activity.conversation().id(),
                                new Activity()
                                        .withType(ActivityTypes.MESSAGE)
                                        .withText("Echo: " + activity.text())
                                        .withRecipient(activity.from())
                                        .withFrom(activity.recipient())
                        );
                    }
                } catch (AuthenticationException ex) {
                    httpExchange.sendResponseHeaders(401, 0);
                    httpExchange.getResponseBody().close();
                    LOGGER.log(Level.WARNING, "Auth failed!", ex);
                } catch (Exception ex) {
                    LOGGER.log(Level.WARNING, "Execution failed", ex);
                }
            }
        }

        private String getRequestBody(HttpExchange httpExchange) throws IOException {
            StringBuilder buffer = new StringBuilder();
            InputStream stream = httpExchange.getRequestBody();
            int rByte;
            while ((rByte = stream.read()) != -1) {
                buffer.append((char)rByte);
            }
            stream.close();
            if (buffer.length() > 0) {
                return URLDecoder.decode(buffer.toString(), "UTF-8");
            }
            return "";
        }

        private Activity getActivity(HttpExchange httpExchange) {
            try {
                String body = getRequestBody(httpExchange);
                LOGGER.log(Level.INFO, body);
                return objectMapper.readValue(body, Activity.class);
            } catch (Exception ex) {
                LOGGER.log(Level.WARNING, "Failed to get activity", ex);
                return null;
            }

        }
    }
}
```

Si usa Maven, puede copiar el archivo pom.xml de la carpeta de ejemplos en este repositorio. Una vez que ha comenzado la ejecución del ejecutable, inicie Bot Framework Emulator.

### <a name="start-the-emulator-and-connect-your-bot"></a>Inicio del emulador y conexión del bot

En este momento, el bot se ejecuta de forma local.
A continuación, inicie el emulador y, después, conéctese al bot en el emulador:

1. Haga clic en el vínculo **create a new bot configuration** (Crear configuración de bot) en la pestaña de bienvenida del emulador. 

2. Escriba un **nombre de bot** y la ruta de acceso del directorio al código del bot. El archivo de configuración del bot se guarda en esta ruta de acceso.

3. Escriba `http://localhost:port-number/api/messages` en el campo **Endpoint URL** (Dirección URL del punto de conexión), donde *port-number* coincide con el número de puerto que se muestra en el explorador en el que se ejecuta la aplicación.

4. Haga clic en **Connect** (Conectar) para conectarse al bot. No tendrá que especificar el **identificador de aplicación de Microsoft** ni la **contraseña de aplicación de Microsoft**. Por ahora puede dejar estos campos en blanco. Más adelante obtendrá esta información al registrar el bot.

### <a name="interact-with-your-bot"></a>Interacción con el bot
Envíe "Hola" a su bot y el bot repetirá el mensaje.

## <a name="next-steps"></a>Pasos siguientes

A continuación, vaya a los conceptos que explican qué es un bot y cómo funciona.

> [!div class="nextstepaction"]
> [Conceptos básicos de bot](../v4sdk/bot-builder-basics.md)
