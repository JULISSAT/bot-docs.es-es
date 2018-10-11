---
title: Implementación de plantillas de bot de empresa | Microsoft Docs
description: Aprenda a implementar todos los recursos de Azure auxiliares para el bot de empresa.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a4ac88872e11cd32a9de96d52dcf9e917bb3488f
ms.sourcegitcommit: 87b5b0ca9b0d5e028ece9f7cc4948c5507062c2b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/24/2018
ms.locfileid: "47029803"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>Plantilla de bot de empresa: Implementación del bot

> [!NOTE]
> Este tema se aplica a la versión v4 del SDK. 

## <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que el [administrador de paquetes de Node](https://nodejs.org/en/) está instalado.

- Instale las herramientas de la línea de comandos (CLI) de Azure Bot Service. Es importante que lo haga incluso si ya ha utilizado las herramientas anteriormente para asegurarse de que tiene las versiones más recientes.

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- Instale las herramientas de la línea de comandos (CLI) de Azure desde [aquí](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest).

- Instale la extensión de AZ para Bot Service.
```shell
az extension add -n botservice
```

## <a name="configuration"></a>Configuración

- Recupere su clave de creación de LUIS.
   - Repase [esta](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) página de documentación para más información sobre el portal de LUIS correcto para la región en la que tiene planeado realizar la implementación. 
   - Una vez que haya iniciado sesión, haga clic en el nombre que aparece en la esquina superior derecha.
   - Seleccione Configuración y anote la clave de creación para el paso siguiente.

## <a name="deployment"></a>Implementación

>Si tiene varias suscripciones de Azure y desea asegurarse de que la implementación selecciona la correcta, ejecute los comandos siguientes antes de continuar.

 Siga el proceso de inicio de sesión del explorador en su cuenta de Azure.
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

Los bots de plantilla de empresa requieren las siguientes dependencias para una operación de extremo a extremo.
- Aplicación web de Azure
- Cuenta de Azure Storage (transcripciones)
- Azure Application Insights (datos de telemetría)
- Azure Cosmos Database (estado)
- Azure Cognitive Services: Language Understanding
- Azure Cognitive Services: QnAMaker (incluido Azure Search, Azure Web App)
- Azure Cognitive Services: Content Moderator (paso opcional manual)

El nuevo proyecto de bot tiene un método de implementación que permite que el comando `msbot clone services` automatice la implementación de todos los servicios mencionados anteriormente en la suscripción de Azure y garantiza que el archivo .bot del proyecto se actualiza con todos los servicios, incluyendo las claves que permiten el funcionamiento ininterrumpido del bot.

> Una vez implementado, revise los planes de tarifas de los servicios creados y adáptelos a su escenario.

El archivo README.md del proyecto creado contiene una línea de comandos de servicios de clonación de msbot de ejemplo que se ha actualizado con el nombre del bot creado y una versión genérica como se muestra a continuación. Asegúrese de que actualiza la clave de creación del paso anterior y elija la ubicación del centro de datos de Azure que desee usar (por ejemplo, westus o westeurope).

> Asegúrese de que la clave de creación de LUIS que se recuperó en el paso anterior es para la región que va a especificar a continuación.

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

La herramienta msbot resumirá el plan de implementación incluyendo la ubicación y la SKU. Asegúrese de revisarlo antes de continuar.

![Confirmación de implementación](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>Una vez completada la implementación, es **obligatorio** tomar nota del secreto del archivo .bot proporcionado ya que lo necesitará en pasos posteriores.

- Actualice el archivo `appsettings.json` con el nombre y el secreto del archivo .bot recién creado.
- Ejecute el siguiente comando y recupere la clave InstrumentationKey de la instancia de Application Insights y actualícela en el archivo `appsettings.json`.

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>Prueba

Una vez completada, ejecute el proyecto de bot dentro de su entorno de desarrollo y abra el emulador de Bot Framework. En el emulador, elija Abrir bot en el menú Archivo y vaya al archivo .bot en el directorio.

A continuación, escriba ```hi``` para comprobar que todo funciona.

## <a name="deploy-to-azure"></a>Implementar en Azure

Las pruebas pueden realizarse localmente por completo. Cuando esté listo para implementar el bot en Azure para realizar pruebas adicionales puede usar el comando siguiente para publicar el código fuente, esto se puede ejecutar cada vez que desee insertar actualizaciones de código fuente.

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>Habilitación de escenarios adicionales

El proyecto de bot proporciona funcionalidad adicional que se puede habilitar mediante los pasos siguientes.

### <a name="authentication"></a>Autenticación

Para habilitar la autenticación siga estos pasos después de configurar un nombre de conexión de autenticación dentro de la configuración del bot en Azure Portal. Se puede encontrar más información en la [documentación](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0).

Registre `SignInDialog` en el constructor MainDialog:
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

Agregue lo siguiente al código en la ubicación que desee para probar un flujo sencillo de inicio de sesión:
    
`var signInResult = await dc.BeginAsync(SignInDialog.Name);`

### <a name="content-moderation"></a>Moderación de contenido

La moderación de contenido puede usarse para identificar información de identificación personal (PII) y contenido para adultos en los mensajes enviados al bot. Para habilitar esta funcionalidad, vaya a Azure Portal y cree un nuevo servicio Content Moderator. Recopile su clave de suscripción y región para configurar el archivo .bot. 

> Este paso se automatizará en el futuro.

Agregue el siguiente código en la parte inferior del método service.AddBot<>() en el inicio para habilitar la moderación de contenido en cada turno. Se puede acceder al resultado de la moderación de contenido a través del estado del bot. 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
Acceda al resultado del middleware mediante una llamada a este desde la pila de diálogos
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>Personalización del bot

Después de comprobar que ha implementado correctamente el bot desde el principio, puede personalizarlo para adaptarlo a sus necesidades y al escenario. Continúe con [Personalización del bot](bot-builder-enterprise-template-customize.md).
