---
title: Incorporación de sugerencias de entrada a mensajes | Microsoft Docs
description: Obtenga información sobre cómo agregar sugerencias de entrada a mensajes mediante Bot Builder SDK.
keywords: Sugerencias de entrada, aceptar entradas, esperar entradas, ignorar entradas, voz
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: acf119a05d4c9f37b74c4fcaf2ad944978504560
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707251"
---
# <a name="add-input-hints-to-messages"></a>Incorporación de sugerencias de entrada a mensajes

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Al especificar una sugerencia de entrada para un mensaje, puede indicar si el bot acepta, espera o ignora la entrada del usuario después de que el mensaje se haya entregado al cliente. En muchos canales, esto permite a los clientes establecer en consecuencia el estado de los controles de entrada de usuario. Por ejemplo, si la sugerencia de entrada de un mensaje indica que el bot ignora la entrada de usuario, el cliente podría desactivar el micrófono y deshabilitar el cuadro de entrada para impedir que el usuario proporcione una entrada.

Asegúrese de que se incluyen las bibliotecas necesarias para las sugerencias de entrada.

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

En el siguiente espacio de nombres se define la clase **MessageFactory**, que se usa en estos ejemplos.

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>Aceptar entradas

Para indicar que el bot está preparado pasivamente para la entrada, pero que no espera ninguna respuesta del usuario, establezca la sugerencia de entrada del mensaje en _accepting input_. En muchos canales, esto hará que se habilite el cuadro de entrada del cliente y que el micrófono se desactive, sin dejar de estar accesible para el usuario. Por ejemplo, Cortana activará el micrófono para aceptar la entrada del usuario si este mantiene presionado el botón de micrófono. En el código siguiente se crea un mensaje que indica que el bot acepta la entrada del usuario.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>Esperar entradas

Para indicar que el bot espera una respuesta del usuario, establezca la sugerencia de entrada del mensaje en _expecting input_. En muchos canales, esto hará que se habilite el cuadro de entrada del cliente y que el micrófono se active. En el ejemplo de código siguiente se crea un mensaje que indica que el bot espera la entrada del usuario.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>Ignorar entradas

Para indicar que el bot no está preparado para recibir la entrada del usuario, establezca la sugerencia de entrada del mensaje en _ignoring input_. En muchos canales, esto hará que se deshabilite el cuadro de entrada del cliente y que el micrófono se desactive. En el ejemplo de código siguiente se crea un mensaje que indica que el bot ignora la entrada del usuario.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>Valores predeterminados para las sugerencias de entrada

Si no establece la sugerencia de entrada para un mensaje, Bot Builder SDK la establecerá automáticamente mediante esta lógica:

- Si el bot envía un aviso, en la sugerencia de entrada del mensaje se especificará que el bot **espera una entrada**.</li>
- Si el bot envía un solo mensaje, en la sugerencia de entrada del mensaje se especificará que el bot **acepta entradas**.</li>
- Si el bot envía una serie de mensajes consecutivos, la sugerencia de entrada de todos los mensajes excepto el último de la serie especificará que el bot **ignora las entradas**, mientras que la sugerencia de entrada del último mensaje de la serie especificará que el bot **acepta entradas**.

