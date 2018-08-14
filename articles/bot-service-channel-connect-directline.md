---
title: Conectar un bot con Direct Line | Microsoft Docs
description: Obtenga información sobre cómo configurar la conexión de un bot con Direct Line.
keywords: Direct Line, canales de bot, cliente personalizado, conexión con canales, configuración
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 2ec11c4cc0e12b6a130b2f703f30660c4bc9f072
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/19/2018
ms.locfileid: "39306493"
---
# <a name="connect-a-bot-to-direct-line"></a>Conectar un bot con Direct Line

Para permitir que su propia aplicación cliente se comunique con su bot, puede usar el canal de Direct Line. 

## <a name="add-the-direct-line-channel"></a>Agregar el canal de Direct Line

Para agregar el canal de Direct Line, abra el bot en [Azure Portal](https://portal.azure.com/), haga clic en la pestaña **Canales** y, luego, en **Direct Line**.

![Agregar el canal de Direct Line](~/media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>Agregar un nuevo sitio

Después, agregue un nuevo sitio que represente la aplicación cliente que quiere conectar con el bot. Haga clic en **Add new site** (Agregar nuevo sitio), escriba un nombre para el sitio y haga clic en **Listo**.

![Agregar un sitio de Direct Line](~/media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>Administrar claves secretas

Cuando se cree el sitio, Bot Framework generará unas claves secretas que la aplicación cliente puede usar para [autenticar](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) las solicitudes de Direct Line API que emite para comunicarse con el bot. Para ver una clave en texto sin formato, haga clic en **Mostrar** junto a la clave correspondiente.

![Mostrar la clave de Direct Line](~/media/bot-service-channel-connect-directline/directline-showkey.png)

Copie y almacene de forma segura la clave que se muestra. Después, use la clave para [autenticar](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) las solicitudes de Direct Line API que el cliente emite para comunicarse con el bot.
También puede usar Direct Line API para [intercambiar la clave por un token](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token) que el cliente pueda usar para autenticar sus solicitudes posteriores en el ámbito de una sola conversación.

![Copiar la clave de Direct Line](~/media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>Definición de la configuración

Por último, configure los valores del sitio.

- Seleccione la versión de protocolo de Direct Line que la aplicación cliente usará para comunicarse con el bot.

> [!TIP]
> Si va a crear una conexión entre la aplicación cliente y un bot, use Direct Line API 3.0.

Cuando termine, haga clic en **Listo** para guardar la configuración del sitio. Puede repetir este proceso desde el paso [Agregar un nuevo sitio](#add-new-site) para cada aplicación cliente que quiera conectar al bot.
