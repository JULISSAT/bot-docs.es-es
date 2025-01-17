---
title: Incorporación de autenticación al bot mediante Azure Bot Service | Microsoft Docs
description: Obtenga información sobre cómo usar las características de autenticación de Azure Bot Service para agregar el inicio de sesión único al bot.
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ROBOTS: NOINDEX
ms.date: 10/04/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 3bd411da4edd30b6045654884aeae5ae1cc4239f
ms.sourcegitcommit: 9e1034a86ffdf2289b0d13cba2bd9bdf1958e7bc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2019
ms.locfileid: "69890530"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Incorporación de autenticación al bot mediante Azure Bot Service

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]  

En este tutorial se usan las nuevas funciones de autenticación de bot de Azure Bot Service, que proporciona características para facilitar el desarrollo de un bot que autentica a los usuarios en varios proveedores de identidades como Azure AD (Azure Active Directory), GitHub, Uber y así sucesivamente. Estas actualizaciones también adoptan pasos hacia una mejor experiencia del usuario mediante la eliminación de la _comprobación del código mágico_ para algunos clientes.

Antes de esto, era necesario que el bot incluyera controladores de OAuth y vínculos de inicio de sesión, que almacenara los identificadores de cliente de destino y los secretos, y que realizara la administración de los tokens de usuario.
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

Ahora, ya no es necesario que los desarrolladores de bots hospeden controladores de OAuth o administren el ciclo de vida de los tokens, ya que todo esto se puede realizar mediante Azure Bot Service.

Las características incluyen:

- Mejoras en los canales para admitir características de autenticación nuevas, como las bibliotecas WebChat y DirectLineJS nuevas para eliminar la necesidad de la comprobación de código mágico de seis dígitos.
- Mejoras en Azure Portal para agregar, eliminar y configurar opciones de conexión a distintos proveedores de identidades de OAuth.
- Compatibilidad con una variedad de proveedores de identidades estándar como Azure AD (los puntos de conexión v1 y v2), GitHub y otros.
- Actualizaciones en Bot Framework SDK para C# y Node.js con el fin de poder recuperar los tokens, crear OAuthCards y controlar eventos de TokenResponse.
- Ejemplos de cómo crear un bot que se autentique en Azure AD (puntos de conexión v1 y v2) y GitHub.

Puede extrapolar a partir de los pasos descritos en este artículo para agregar estas características a un bot existente. Estos son los bots de ejemplo que muestran las nuevas características de autenticación

| Muestra | Versión de BotBuilder | DESCRIPCIÓN |
|:---|:---:|:---|
| [AadV1Bot](https://aka.ms/AadV1Bot) | v3 | Muestra la compatibilidad de OAuthCard en el SDK v3 de C#, mediante el punto de conexión v1 de Azure AD |
| [AadV2Bot](https://aka.ms/AadV2Bot) | v3 |  Muestra la compatibilidad de OAuthCard en el SDK v3 de C#, mediante el punto de conexión v2 de Azure AD |
| [GitHubBot](https://aka.ms/GitHubBot) | v3 |  Muestra la compatibilidad de OAuthCard en el SDK v3 de C#, mediante GitHub |
| [BasicOAuth](https://aka.ms/BasicOAuth) | v3 |  Muestra la compatibilidad de OAuth 2.0 en el SDK v3 de C# |

> [!NOTE]
> Las características de autenticación también funcionan con Node.js con BotBuilder v3. Pero en este artículo solo se describe código de C# de ejemplo.

Para obtener más información y soporte técnico, vea [Bot Framework additional resources](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) (Recursos adicionales de Bot Framework).

## <a name="overview"></a>Información general

En este tutorial se crea un bot de ejemplo que se conecta a Microsoft Graph mediante un token v1 o v2 de Azure AD. <!--verify this info and fix wording--> Como parte de este proceso, usará código de un repositorio de GitHub; en este tutorial se describe cómo configurarlo, incluida la aplicación de bot.

- [Crear el bot y una aplicación de autenticación](#create-your-bot-and-an-authentication-application)
- [Preparar el código de ejemplo del bot](#prepare-the-bot-sample-code)
- [Usar el emulador para probar el bot](#use-the-emulator-to-test-your-bot)

Para completar estos pasos, necesitará instalar Visual Studio 2017, npm, node y gitt. También debe tener cierta familiaridad con Azure, OAuth 2.0 y el desarrollo de bots.

Cuando haya terminado, tendrá un bot que puede responder a una serie de tareas simples en una aplicación de Azure AD, como las de comprobar y enviar un correo electrónico, o bien mostrar quién es usted y quién es el administrador. Para ello, el bot usará un token de una aplicación de Azure AD con la biblioteca Microsoft.Graph.

En la sección final se desglosa parte del código del bot.

- [Notas sobre el flujo de recuperación de tokens](#notes-on-the-token-retrieval-flow)

## <a name="create-your-bot-and-an-authentication-application"></a>Crear el bot y una aplicación de autenticación

Deberá crear un bot de registro, donde se establece el punto de conexión de mensajería en el código del bot, y tendrá que crear una aplicación de Azure AD (v1 o v2) para permitir que el bot acceda a Office 365.

> [!NOTE]
> Estas características de autenticación funcionan con otros tipos de bots. Pero en este tutorial solo se usa un bot de registro.

### <a name="register-an-application-in-azure-ad"></a>Registrar una aplicación en Azure AD

Necesita una aplicación de Azure AD que el bot pueda usar para conectarse a Microsoft Graph API.

Para este bot se pueden usar puntos de conexión v1 o v2 de Azure AD.
Para obtener información sobre las diferencias entre los puntos de conexión v1 y v2, vea la [comparación entre v1 y v2](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) y la [introducción al punto de conexión v2.0 de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

#### <a name="to-create-an-azure-ad-application"></a>Creación de una aplicación de Azure AD

Siga estos pasos para crear una aplicación de Azure AD. Puede usar los puntos de conexión v1 o v2 con la aplicación que cree.

> [!TIP]
> Deberá crear y registrar la aplicación de Azure AD en un inquilino en el que pueda dar su consentimiento para delegar los permisos solicitados por una aplicación.

1. Abra el panel [Azure Active Directory][azure-aad-blade] en Azure Portal.
    Si no está en el inquilino correcto, haga clic en **Cambiar directorio** para cambiar el inquilino. (Para obtener instrucciones sobre cómo crear un inquilino, consulte [Acceso al portal y creación de un inquilino](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant)).
1. Abra el panel **Registros de aplicaciones**.
1. En el panel **Registros de aplicaciones**, haga clic en **Nuevo registro**.
1. Rellene los campos obligatorios y cree el registro de aplicaciones.

   1. Asigne un nombre a la aplicación.
   1. Seleccione los **tipos de cuenta admitidos** para la aplicación. (Cualquiera de estas opciones funcionará con este ejemplo).
   1. Para el **URI de redirección**
       1. Seleccione **Web**.
       1. Establezca la dirección URL en `https://token.botframework.com/.auth/web/redirect`.
   1. Haga clic en **Registrar**.

      - Una vez creada, Azure muestra la página **Información general** de la aplicación.
      - Registre el valor del **identificador de aplicación (cliente)** . Usará este valor más adelante como _Id. de cliente_ al registrar la aplicación de Azure AD con el bot.
      - Registre también el valor del **identificador de directorio (inquilino)** . También lo usará para registrar esta aplicación con el bot.

1. En el panel de navegación, haga clic en **Certificados y secretos** para crear un secreto para la aplicación.

   1. En **Secretos de cliente**, haga clic en **Nuevo secreto de cliente**.
   1. Agregue una descripción para diferenciar a este secreto de otros que puede que tenga que crear para esta aplicación, como `bot login`.
   1. Establezca la opción **Expira** en **Nunca**.
   1. Haga clic en **Agregar**.
   1. Antes de salir de esta página, registre el secreto. Usará este valor más adelante como _Secreto de cliente_ al registrar la aplicación de Azure AD con el bot.

1. En el panel de navegación, haga clic en **Permisos de API** para que se abra el panel **Permisos de API**. Es un procedimiento recomendado establecer explícitamente los permisos de API de la aplicación.

   1. Haga clic en **Agregar un permiso** para que aparezca el panel **Solicitud de permisos de API**.
   1. Para este ejemplo, seleccione **Microsoft APIs** y **Microsoft Graph**.
   1. Elija **Permisos delegados** y asegúrese de que se seleccionan los permisos que necesita. Este ejemplo requiere estos permisos.

      > [!NOTE]
      > Los permisos marcados como **SE NECESITA EL CONSENTIMIENTO DEL ADMINISTRADOR** requerirán un usuario y un administrador de inquilinos para iniciar sesión, por lo que evite usarlos para el bot.

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. Haga clic en **Agregar permisos**. (La primera vez que un usuario acceda a esta aplicación mediante el bot deberá conceder su consentimiento).

Ahora tiene una aplicación de Azure AD configurada.

### <a name="create-your-bot-on-azure"></a>Creación de un bot en Azure

Cree un **Registro de canales de bot** mediante [Azure Portal](https://portal.azure.com/).

### <a name="register-your-azure-ad-application-with-your-bot"></a>Registro de la aplicación de Azure AD con el bot

El paso siguiente consiste en registrar la aplicación de Azure AD que acaba de crear con el bot.

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD v1](#tab/aadv1)

1. Vaya hasta la página de recursos del bot en [Azure Portal](http://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Este nombre se usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación (cliente) que registró para la aplicación v1 de Azure AD.
    1. En **Secreto de cliente**, escriba el secreto que ha creado para conceder al bot acceso a la aplicación de Azure AD.
    1. En **Tipo de concesión**, escriba `authorization_code`.
    1. Para **Dirección URL de inicio de sesión**, escriba `https://login.microsoftonline.com`.
    1. En **Id. de inquilino**, especifique el identificador del directorio (inquilino) que registró anteriormente para la aplicación de Azure AD.

       Este será el inquilino asociado a los usuarios que se pueden autenticar.

    1. En **URL de recursos**, escriba `https://graph.microsoft.com/`.
    1. Deje **Ámbitos** en blanco.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD v2](#tab/aadv2)

1. Vaya a la página de registro de canales del bot en [Azure Portal](http://portal.azure.com/).
1. Haga clic en **Configuración**.
1. En **Configuración de conexión de OAuth**, cerca de la parte inferior de la página, haga clic en **Agregar configuración**.
1. Rellene el formulario de la siguiente manera:

    1. Para **Nombre**, escriba un nombre para la conexión. Lo usará en el código del bot.
    1. En **Proveedor de servicios**, seleccione **Azure Active Directory v2**. Después de seleccionar esta opción, se mostrarán los campos específicos de Azure AD.
    1. Para **Id. de cliente**, escriba el identificador de aplicación (cliente) que registró para la aplicación v1 de Azure AD.
    1. En **Secreto de cliente**, escriba el secreto que ha creado para conceder al bot acceso a la aplicación de Azure AD.
    1. En **Id. de inquilino**, especifique el identificador del directorio (inquilino) que registró anteriormente para la aplicación de Azure AD.

       Este será el inquilino asociado a los usuarios que se pueden autenticar.

    1. En **Ámbitos**, escriba los nombres de los permisos que eligió en el registro de la aplicación: `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Para Azure AD v2, **Ámbitos** toma una lista de valores que distingue mayúsculas de minúsculas, separados por espacios.

1. Haga clic en **Save**(Guardar).

> [!NOTE]
> Estos valores permiten que la aplicación acceda a datos de Office 365 a través de Microsoft Graph API.

---

#### <a name="to-test-your-connection"></a>Para probar la conexión

1. Abra la conexión que acaba de crear.
1. Haga clic en **Probar conexión** en la parte superior del panel **Configuración de conexión del proveedor de servicios**.
1. La primera vez, se debería abrir una pestaña del explorador nueva con los permisos que solicita la aplicación y en la que se le pide que acepte.
1. Haga clic en **Aceptar**.
1. Después, esto debería redirigirle a una página **Prueba de conexión a "<nombreDeLaConexión>" correcta**.

## <a name="prepare-the-bot-sample-code"></a>Preparar el código de ejemplo del bot

1. Clone el repositorio de GitHub en https://github.com/Microsoft/BotBuilder.
1. Abra y compile la solución, `BotBuilder\CSharp\Microsoft.Bot.Builder.sln`.
1. Cierre esa solución y abra `BotBuilder\CSharp\Samples\Microsoft.Bot.Builder.Samples.sln`.
1. Establezca el proyecto de inicio.
    - Para un bot en el que se use la aplicación v1 de Azure AD, use el proyecto `Microsoft.Bot.Sample.AadV1Bot`.
    - Para un bot en el que se use la aplicación v2 de Azure AD, use el proyecto `Microsoft.Bot.Sample.AadV2Bot`.
1. Abra el archivo `Web.config` y modifique la configuración de la aplicación como sigue:
    1. Establezca `ConnectionName` en el valor que usó al configurar la configuración de conexión de OAuth 2.0 del bot.
    1. Establezca el valor `MicrosoftAppId` en el identificador de aplicación del bot.
    1. Establezca el valor `MicosoftAppPassword` en el secreto del bot.

    > [!IMPORTANT]
    > En función de los caracteres del secreto, es posible que sea necesario aplicar secuencias de escape XML a la contraseña. Por ejemplo, cualquier símbolo de Y comercial (&) tendrá que codificarse como `&amp;`.

    ```xml
    <appSettings>
        <add key="ConnectionName" value="<your-AAD-connection-name>"/>
        <add key="MicrosoftAppId" value="<your-bot-appId>" />
        <add key="MicrosoftAppPassword" value="<your-bot-password>" />
    </appSettings>
    ```

    Si no sabe cómo obtener su **identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft**, puede crear una nueva contraseña tal y como se describe aquí: [bot-channels-registration-password](bot-service-quickstart-registration.md#get-registration-password). O bien, puede recuperar el **identificador de aplicación de Microsoft** y la **contraseña de aplicación de Microsoft** aprovisionados con el **registro de canales de bot** desde la implementación que se describe aquí:  [find-your-azure-bots-appid-and-appsecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret)

    > [!NOTE]
    > Ahora podría publicar el código de este bot en la suscripción de Azure (haciendo clic con el botón derecho en el proyecto y seleccionando **Publicar**), pero para este tutorial no es necesario. Tendría que establecer una configuración de publicación que usara la aplicación y el plan de hospedaje que usó durante la configuración del bot en Azure Portal.

## <a name="use-the-emulator-to-test-your-bot"></a>Usar el emulador para probar el bot

Deberá instalar el [Emulador de bot](https://github.com/Microsoft/BotFramework-Emulator) para probar el bot de forma local. Puede usar el emulador v3 o v4.

1. Inicie el bot (con o sin depuración).
1. Anote el número de puerto de host local de la página. Necesitará esta información para interactuar con el bot.
1. Inicie el emulador.
1. Conéctese al bot.

   Si todavía no ha configurado la conexión, proporcione la dirección y el id. de aplicación y la contraseña de Microsoft del bot. Agregue `/api/messages` a la dirección URL del bot. La dirección URL se parecerá a esta `http://localhost:portNumber/api/messages`.

1. Escriba `help` para ver una lista de los comandos disponibles para el bot y probar las características de autenticación.
1. Una vez que haya iniciado sesión, no es necesario que vuelva a proporcionar las credenciales hasta que cierre la sesión.
1. Para cerrar la sesión y cancelar la autenticación, escriba `signout`.

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> La autenticación del bot requiere el uso de Bot Connector Service. El servicio accede a la información de registro de los canales del bot, motivo por el que es necesario establecer el punto de conexión de mensajería del bot en el portal. La autenticación también requiere el uso de HTTPS, por lo que fue necesario crear una dirección de reenvío HTTPS para el bot que se ejecuta localmente.

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>Notas sobre el flujo de recuperación de tokens

Cuando un usuario le pide al bot que haga algo para lo que es necesario que el usuario haya iniciado sesión, el bot puede usar `Microsoft.Bot.Builder.Dialogs.GetTokenDialog` para iniciar la recuperación de un token para una conexión determinada. Los dos fragmentos de código siguientes se toman de la clase `GetTokenDialog`.

### <a name="check-for-a-cached-token"></a>Buscar un token en caché

En este código, en primer lugar el bot realiza una comprobación rápida para determinar si Azure Bot Service ya tiene un token para el usuario (que se identifica mediante el remitente de la actividad actual) y el nombre de la conexión dado (que es el nombre de conexión que se usa en la configuración). Azure Bot Service tendrá ya un token en caché o no. La llamada a GetUserTokenAsync realiza esta "comprobación rápida". Si Azure Bot Service tiene un token y lo devuelve, el token se puede usar inmediatamente. Si Azure Bot Service no tiene un token, este método devolverá NULL. En este caso, el bot puede enviar una OAuthCard personalizada para que el usuario inicie sesión.

```csharp
// First ask Bot Service if it already has a token for this user
var token = await context.GetUserTokenAsync(ConnectionName).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
    await SendOAuthCardAsync(context, (Activity)context.Activity);
}
```

### <a name="send-an-oauthcard-to-the-user"></a>Envío de OAuthCard al usuario

Puede personalizar la OAuthCard con el texto y el texto de botón que quiera. Las partes importantes son las siguientes:

- Establecer `ContentType` en `OAuthCard.ContentType`.
- Establecer la propiedad `ConnectionName` en el nombre de la conexión que se quiere usar.
- Incluir un botón con un elemento `CardAction` de `Type` `ActionTypes.Signin`; tenga en cuenta que no es necesario especificar ningún valor para el vínculo de inicio de sesión.

Al final de esta llamada, el bot debe "esperar a que el token" regrese. Esta espera tiene lugar en la secuencia de actividad principal porque es posible que el usuario tenga que hacer muchas cosas para iniciar sesión.

```csharp
private async Task SendOAuthCardAsync(IDialogContext context, Activity activity)
{
    await context.PostAsync($"To do this, you'll first need to sign in.");

    var reply = await context.Activity.CreateOAuthReplyAsync(_connectionName, _signInMessage, _buttonLabel).ConfigureAwait(false);
    await context.PostAsync(reply);

    context.Wait(WaitForToken);
}
```

### <a name="wait-for-a-tokenresponseevent"></a>Esperar a TokenResponseEvent

En este código, la clase de cuadro de diálogo del bot está esperando un `TokenResponseEvent` (a continuación se muestra más información sobre cómo se enruta esto a la pila del cuadro de diálogo). Primero, el método `WaitForToken` determina si este evento se ha enviado. Si se ha enviado, el bot lo puede usar. En caso contrario, el método `WaitForToken` toma el texto que se haya enviado al bot y lo pasa a `GetUserTokenAsync`. El motivo es que algunos clientes (como WebChat) no necesitan el código de comprobación y pueden enviar el token directamente en `TokenResponseEvent`. Otros clientes requerirán el código mágico (como Facebook o Slack). Azure Bot Service presentará a estos clientes un código mágico de seis dígitos y pedirá al usuario que lo escriba en la ventana de chat. Aunque no es lo más adecuado, es el comportamiento "de reserva", de modo que si `WaitForToken` recibe un código, el bot lo puede enviar a Azure Bot Service y obtener un token. Si también se produce un error en esta llamada, puede decidir si notificar un error, o bien hacer otra cosa. Pero en la mayoría de los casos, el bot tendrá un token de usuario.

Si examina el archivo **MessageController.cs**, verá que las actividades `Event` de este tipo también se enrutan a la pila del cuadro de diálogo.

```csharp
private async Task WaitForToken(IDialogContext context, IAwaitable<object> result)
{
    var activity = await result as Activity;

    var tokenResponse = activity.ReadTokenResponseContent();
    if (tokenResponse != null)
    {
        // Use the token to do exciting things!
    }
    else
    {
        if (!string.IsNullOrEmpty(activity.Text))
        {
            tokenResponse = await context.GetUserTokenAsync(ConnectionName,
                                                               activity.Text);
            if (tokenResponse != null)
            {
                // Use the token to do exciting things!
                return;
            }
        }
        await context.PostAsync($"Hmm. Something went wrong. Let's try again.");
        await SendOAuthCardAsync(context, activity);
    }
}
```

### <a name="message-controller"></a>Controlador de mensajes

En las llamadas posteriores al bot, tenga en cuenta que este bot de ejemplo nunca almacena el token en caché. Se debe a que el bot siempre puede pedir el token a Azure Bot Service. Esto evita que el bot tenga que administrar el ciclo de vida del token, actualizar el token, etc., ya Azure Bot Service se encarga de todo esto de forma automática.

```csharp
else if(message.Type == ActivityTypes.Event)
{
    if(message.IsTokenResponseEvent())
    {
        await Conversation.SendAsync(message, () => new Dialogs.RootDialog());
    }
}
```
## <a name="additional-resources"></a>Recursos adicionales
[Bot Framework SDK](https://github.com/microsoft/botbuilder)
