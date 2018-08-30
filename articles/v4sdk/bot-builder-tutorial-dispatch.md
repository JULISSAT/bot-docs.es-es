---
title: Uso de la herramienta de distribución para LUIS y QnA Maker | Microsoft Docs
description: Obtenga información sobre cómo usar LUIS y QnA Maker en su bot.
keywords: LUIS, QnA, herramienta de distribución, varios servicios
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d3c9355a0e87d31029b92614dc182f3d7010c736
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2018
ms.locfileid: "39514945"
---
## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>Integración de varias aplicaciones de LUIS y servicios QnA con la herramienta de distribución

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

En este tutorial se muestra cómo usar un modelo de LUIS generado por la herramienta de distribución, a fin de integrar el bot con varias aplicaciones de Language Understanding (LUIS) y servicios QnA Maker. 

Imagine que ha desarrollado los servicios siguientes y que quiere crear un bot que se integre con todos ellos.

| Tipo de servicio | NOMBRE | DESCRIPCIÓN |
|------|------|------|
| Aplicación de LUIS | HomeAutomation | Reconoce las intenciones HomeAutomation.TurnOn HomeAutomation.TurnOff y HomeAutomation.None.|
| Aplicación de LUIS | Tiempo | Reconoce las intenciones Weather.GetForecast y Weather.GetCondition.|
| Servicio QnA Maker | Preguntas más frecuentes  | Proporciona respuestas a preguntas sobre un sistema de iluminación de automatización del hogar |

Primero vamos a crear las aplicaciones y los servicios y luego los integraremos.

> [!NOTE]
> Las tres aplicaciones deben crearse en la misma ubicación de Azure para que la herramienta de distribución acceda a ellas correctamente. En el siguiente ejemplo de código de distribución se usa la ubicación del Oeste de EE. UU.

## <a name="create-the-luis-apps"></a>Crear las aplicaciones de LUIS

La forma más rápida de crear las aplicaciones de LUIS HomeAutomation y Weather consiste en descargar los archivos [homeautomation.json][HomeAutomationJSON] y [weather.json][WeatherJSON]. Después, vaya al [sitio web de LUIS](https://www.luis.ai/home) e inicie sesión. Haga clic en **Mis aplicaciones** > **Import new app** (Importar nueva aplicación) y elija el archivo homeautomation.json. Asigne el nombre `homeautomation` a la nueva aplicación. Haga clic en **Mis aplicaciones** > **Import new app** (Importar nueva aplicación) y elija el archivo weather.json. Asigne el nombre `weather` a esta otra aplicación.

## <a name="create-the-qna-cognitive-service-in-azure"></a>Crear el servicio cognitivo de QnA en Azure

Un servicio QnA Maker está formado por dos partes: el servicio cognitivo de Azure y la base de conocimiento de pares de preguntas y respuestas que se publica mediante el servicio cognitivo.

Para crear el servicio cognitivo en Azure, inicie sesión en Azure Portal en https://portal.azure.com y haga lo siguiente:

1. Haga clic en **Todos los servicios**.
1. Busque en `Cognitive` y seleccione **Cognitive Services**.
1. Haga clic en **Agregar**.
1. Busque en `QnA` y seleccione **QnA Maker**.
1. En la hoja de QnA Maker, haga clic en **Crear**.
1. Rellene la información y cree el servicio QnA Maker.

    <!-- TODO: Add screenshot.-->

    * Escriba un nombre para el servicio. Para este tutorial usaremos `SmartLightQnA`.
    * Seleccione la suscripción que se va a usar.
    * Seleccione el plan de tarifa que se va a usar. Ahora usaremos el plan F0 (gratis).
    * Cree el grupo de recursos que se va a usar o seleccione uno nuevo. En este tutorial, vamos a crear el grupo de recursos `SmartLightQnA`.
    * Seleccione un plan de tarifa de búsqueda; aquí usaremos el plan B (básico).
    * Seleccione una ubicación de búsqueda; aquí usaremos `West US`.
    * Escriba el nombre que se usará para la aplicación; en este caso, mantendremos el valor predeterminado `SmartLightQnA`.
    * Seleccione la ubicación del sitio web; aquí usaremos `West US`.
    * Mantenga el valor predeterminado consistente en habilitar Application Insights.
    * Seleccione una ubicación de Application Insights; aquí usaremos `West US 2`.
    * Haga clic en **Crear** para crear el servicio QnA Maker.
    * Azure crea y empieza a implementar el servicio.

1. Una vez que se haya implementado, vea la notificación y haga clic en **Go to resource** (Ir al recurso) para ir a la hoja del servicio.
1. Haga clic en **Claves** para obtener las claves.

    * Copie el nombre del servicio y la primera clave. Los necesitará en los pasos siguientes.
    * Obtendrá dos claves para poder volver a generar una de ellas de cada vez sin tener que interrumpir el servicio.

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>Crear y publicar la base de conocimiento de QnA Maker

Vaya al [sitio web de QnA Maker](https://qnamaker.ai) e inicie sesión. Seleccione **Create a knowledge base** (Crear una base de conocimiento) y cree una base de conocimiento denominada "FAQ". Haga clic en el botón **Seleccionar archivo** y cargue el [archivo TSV de ejemplo][FAQ_TSV]. Haga clic en **Crear** y, una vez que se haya creado el servicio, haga clic en **Publicar**.

## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>Usar la herramienta de distribución para crear la aplicación de LUIS de distribuidor

Ahora, vamos a crear una aplicación de LUIS para combinar los servicios que hemos creado.

Instale la [herramienta de distribución][DispatchTool]. Para ello, ejecute este comando en un símbolo del sistema de Node.js.

```
npm install -g botdispatch
```

Ejecute el comando siguiente para inicializar la herramienta de distribución del nombre `CombineWeatherAndLights`. Sustituya su [clave de creación de LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys) por `"YOUR-LUIS-AUTHORING-KEY"`.

```
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

Para cada una de las aplicaciones de LUIS que ha creado, obtenga el identificador de aplicación de LUIS. Los encontrará para cada aplicación en **Mis aplicaciones**, en el [sitio de LUIS](https://www.luis.ai/home). Haga clic en el nombre de la aplicación y, luego, en **Configuración** para ver el identificador de aplicación. 

Después, ejecute el comando `dispatch add` para cada una de las aplicaciones de LUIS que ha creado.

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

Ejecute el comando `dispatch add` para el servicio QnA Maker que ha creado. El parámetro `-key` debería ser la clave de Azure Portal, que guardó al completar los pasos necesarios para [crear el servicio cognitivo de QnA en Azure](./bot-builder-tutorial-dispatch.md#create-the-qna-cognitive-service-in-azure).

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-QNA-SUBSCRIPTION-KEY"
```

Ejecute `dispatch create`:

```
dispatch create
```

De este modo se crea la aplicación de LUIS de distribuidor denominada **CombineWeatherAndLights**. Puede ver la nueva aplicación en [https://www.luis.ai/home](https://www.luis.ai/home). 

![Aplicación de distribuidor en LUIS.ai](media/tutorial-dispatch/dispatch-app-in-luis.png)

Haga clic en la nueva aplicación. En **Intenciones** puede ver tiene las intenciones `l_homeautomation`, `l_weather` y `q_faq`.

![Intenciones del distribuidor en LUIS.ai](media/tutorial-dispatch/dispatch-intents-in-luis.png)

Haga clic en el botón **Entrenar** para entrenar la aplicación de LUIS y use la pestaña **PUBLICAR** para [publicarla](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp). Haga clic en **Configuración** para copiar el identificador de la nueva aplicación y usarlo en su bot.

## <a name="create-the-bot"></a>Crear el bot

Ahora puede enlazar las intenciones de la aplicación de distribuidor con la lógica de su bot, que enruta los mensajes al servicio QnA Maker y las aplicaciones de LUIS originales.

Puede usar como punto de partida el ejemplo que se incluye con Bot Builder SDK.

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

Comience con el código del [ejemplo de distribución de LUIS][DispatchBotCS]. En Visual Studio, [actualice los paquetes Nuget](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package) a las últimas versiones preliminares de lo siguiente:

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA` (necesario para QnA Maker)
* `Microsoft.Bot.Builder.Ai.Luis` (necesario para LUIS)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

Descargue el [ejemplo de distribución de LUIS][DispatchBotJs].  Instale los paquetes necesarios, incluido el paquete `botbuilder-ai` para LUIS y QnA Maker, mediante npm:

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


Configure el ejemplo de modo que use la aplicación de distribuidor.

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

En **appsettings.json**, en el [ejemplo de distribución de LUIS][DispatchBotCS], edite los campos siguientes.

| NOMBRE | DESCRIPCIÓN |
|------|------|
| `Luis-SubscriptionKey` |  Clave de suscripción de LUIS. Puede ser una clave de punto de conexión o una clave de creación, como se describe [aquí](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys). | 
| `Luis-ModelId-Dispatcher` | Identificador de la aplicación de LUIS generada por la herramienta de distribución. | 
| `Luis-ModelId-HomeAutomation` | Identificador de la aplicación que ha creado a partir de homeautomation.json.  | 
| `Luis-ModelId-Weather` | Identificador de la aplicación que ha creado a partir de weather.json. | 
| `QnAMaker-Endpoint-Url` | Debe establecerse en https://westus.api.cognitive.microsoft.com/qnamaker/v2.0 para los servicios QnA Maker de versión preliminar. <br/>Establezca esta opción en https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker para los servicios QnA Maker nuevos (disponibilidad general).|
| `QnAMaker-SubscriptionKey` | Clave de suscripción de QnA Maker. | 
| `QnAMaker-KnowledgeBaseId` | Identificador de la base de conocimiento que cree en el [portal de QnA Maker](https://qnamaker.ai).| 



En **Startup.cs**, eche un vistazo al método `ConfigureServices`. Contiene código para inicializar `LuisRecognizerMiddleware` con el identificador de la aplicación de LUIS que acaba de generar.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
Los métodos `GetLuisConfiguration` y `GetQnAMakerConfiguration` obtienen de appsettings.json la configuración de LUIS y de QnA.
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>Distribuir el mensaje

Eche un vistazo a **LuisDispatchBot.cs**, donde el bot distribuye el mensaje a la aplicación de LUIS o de QnA Maker para un subcomponente. 

En `DispatchToTopIntent`, si la aplicación de distribuidor detecta la intención `l_homeautomation` o `l_weather`, llama a un método `DispatchToTopIntent` que crea un `LuisRecognizer` para llamar a las aplicaciones originales `homeautomation` y `weather`. Si el bot detecta la intención `q_faq`, o bien la intención `none` que se usa como caso de reserva, llama a un método que realiza una consulta a QnA Maker.

> [!NOTE] 
> Si los nombres de intención `l_homeautomation`, `l_weather` o `q_faq` no coinciden con la aplicación de LUIS que ha creado mediante la herramienta de distribución, modifíquelos para que coincidan con la versión en minúsculas de los nombres de intención que se ven en el [portal de LUIS](https://www.luis.ai).

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

El método `DispatchToQnAMaker` envía el mensaje del usuario al servicio QnA Maker. Asegúrese de que ha publicado ese servicio en el [portal de QnA Maker](https://qnamaker.ai) antes de ejecutar el bot.

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

El método `DispatchToLuisModel` envía el mensaje del usuario a las aplicaciones de LUIS originales `homeautomation` y `weather`. Asegúrese de que ha publicado esas aplicaciones de LUIS en el [portal de LUIS](https://www.luis.ai) antes de ejecutar el bot.

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

El método `RecognizeAsync` llama a un `LuisRecognizer` para obtener resultados de una aplicación de LUIS.

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

Comience con el código del [ejemplo de bot de distribución][DispatchBotJs]. Abra **app.js** y, opcionalmente, reemplace los campos `appId` por los identificadores de las aplicaciones de LUIS que ha creado. Si deja los campos `appId` como estaban originalmente, usará aplicaciones de LUIS públicas creadas con fines de demostración.

```javascript
// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

En el código siguiente, reemplace `endpointKey` y `knowledgeBaseID` por la clave y el identificador de la base de conocimiento de QnA Maker. El `host` debe establecerse en `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0` para los servicios QnA Maker de versión preliminar, y `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker` para los servicios QnA Maker nuevos (disponibilidad general).

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

El resto del código en **app.js** controla las intenciones `l_homeautomation`, `l_weather` y `None`, para lo que inicia los cuadros de diálogo pertinentes. En el caso de la intención `q_faq`, responde con la respuesta desde el servicio QnA Maker.

> [!NOTE] 
> Si los nombres de intención `l_homeautomation`, `l_weather` o `q_faq` no coinciden con la aplicación de LUIS que ha creado mediante la herramienta de distribución, modifíquelos para que coincidan con los nombres de intención que se ven en el [portal de LUIS](https://www.luis.ai).

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>Ejecutar el bot

Pruebe el bot con [Bot Framework Emulator](../bot-service-debug-emulator.md). Envíele un mensaje como "encender las luces" para distribuir el mensaje a la aplicación de LUIS de automatización del hogar, y un mensaje como "consultar la información meteorológica en Seattle" para enviarlo a la aplicación de LUIS de información meteorológica.

> [!NOTE] 
> Antes de ejecutar el bot, asegúrese de que ha publicado todas las aplicaciones LUIS que ha creado en el [portal de LUIS](https://www.luis.ai) y compruebe que ha publicado el servicio QnA Maker en el [portal de QnA Maker](https://qnamaker.ai).

![Enviar mensajes al bot de distribución](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>Evaluar el rendimiento del distribuidor

Algunos mensajes de usuario se proporcionan como ejemplos en las aplicaciones de LUIS y los servicios QnA Maker, y la aplicación de LUIS combinada que la herramienta de distribución genera no funcionará bien para esas entradas. Para comprobar el rendimiento de su aplicación, use la opción `eval`. 

```
dispatch eval
```

Al ejecutar `dispatch eval` se genera un archivo **Summary.html** que proporciona estadísticas sobre el rendimiento del nuevo modelo de lenguaje.

> [!TIP] 
> Puede ejecutar `dispatch eval` en cualquier aplicación de LUIS, no solo en las aplicaciones de LUIS que ha creado la herramienta de distribución.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Editar las intenciones para los duplicados y las superposiciones

Revise las expresiones de ejemplo marcadas como duplicados en **Summary.html** y quite los ejemplos similares o que se superponen. Por ejemplo, supongamos que en la aplicación de LUIS `homeautomation` para la automatización del hogar, las solicitudes como "encender las luces" se asignan a una intención "TurnOnLights", pero las solicitudes como "¿Por qué no se encienden las luces?" se asignan a una intención "None" para que se puedan pasar a QnA Maker. Al combinar la aplicación de LUIS y el servicio QnA Maker mediante la herramienta de distribución, es preciso llevar a cabo una de las acciones siguientes: 

* Quitar la intención "None" de la aplicación de LUIS `homeautomation` original y agregar las expresiones de dicha intención a la intención "None" de la aplicación de distribuidor.
* Si no quita la intención "None" de la aplicación de LUIS original, deberá agregar lógica en su bot para pasar los mensajes que coinciden con dicha intención al servicio QnA Maker.

> [!TIP] 
> Vea [Best practices for Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) (Procedimientos recomendados para Language Understanding) para obtener sugerencias sobre cómo mejorar el rendimiento del modelo de lenguaje.

## <a name="additional-resources"></a>Recursos adicionales

* [Uso de LUIS para el flujo de conversación][luis-v4-how-to]

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js


