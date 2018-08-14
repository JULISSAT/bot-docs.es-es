---
title: Protección del bot | Microsoft Docs
description: Obtenga información sobre cómo proteger el bot mediante el uso de la autenticación de Bot Framework y HTTPS.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 331effc0bf604388995288e5d7c3ca9d54537f94
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304907"
---
# <a name="secure-your-bot"></a>Protección del bot

Su bot puede conectarse a muchos canales de comunicación diferentes (Skype, SMS, correo electrónico, etc.) a través del servicio de Bot Framework Connector. En este artículo se describe cómo proteger su bot con la autenticación Bot Framework y HTTPS.

## <a name="use-https-and-bot-framework-authentication"></a>Usar la autenticación de Bot Framework y HTTPS

Para asegurarse de que solo el [conector](bot-builder-dotnet-concepts.md#connector) de Bot Framework Connector pueda acceder al punto de conexión del bot, configure el punto de conexión del bot para usar únicamente HTTPS y habilite la autenticación de Bot Framework [registrando](~/bot-service-quickstart-registration.md) el bot para que adquiera su appID y contraseña. 

## <a name="configure-authentication-for-your-bot"></a>Configurar la autenticación para el bot

Especifique el appID y la contraseña del bot en el archivo web.config del bot. 

> [!NOTE]
> Para buscar el **AppID** y **AppPassword** del bot, consulte la sección [MicrosoftAppID and MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword) (MicrosoftAppID y MicrosoftAppPassword).

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

A continuación, utilice el atributo `[BotAuthentication]` para especificar las credenciales de autenticación al usar el SDK de Bot Builder para .NET para crear su bot. 

Para usar las credenciales de autenticación que se almacenan en el archivo web.config, especifique el atributo `[BotAuthentication]` sin parámetros.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

Para usar otros valores para las credenciales de autenticación, especifique el atributo `[BotAuthentication]` e inserte esos valores.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>Recursos adicionales

- [Bot Builder SDK for .NET](bot-builder-dotnet-overview.md) (SDK de Bot Builder para .NET)
- [Key concepts in the bot Builder SDK for .NET](bot-builder-dotnet-concepts.md) (Conceptos clave del SDK de Bot Builder para .NET)
- [Register a bot with the Bot Framework](~/bot-service-quickstart-registration.md) (Registrar un bot con Bot Framework)