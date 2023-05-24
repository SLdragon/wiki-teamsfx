> We appreciate your feedback, please report any issues to us [here](https://github.com/OfficeDev/TeamsFx/issues/new/choose).

Microsoft Teams allows you to automate simple and repetitive tasks right inside a conversation. You can build a Teams bot that can respond to simple commands sent in chats with adaptive cards.

In this tutorial, you will learn:

Get started with Teams Toolkit and TeamsFx SDK:
* [How to create a command bot with Teams Toolkit](#How-to-create-a-command-response-bot)
* [How to understand the command bot project](#Take-a-tour-of-your-app-source-code)
* [How to understand the command handler](#How-command-and-response-works)

Customize the scaffolded app template:
* [How to customize the initialization](#Customize-initialization)
* [How to customize the installation](#Customize-installation)
* [How to customize the adapter](#Customize-adapter)
* [How to customize your app to add more command and response](#How-to-add-more-command-and-response)
* [How to customize the trigger pattern](#Customize-the-trigger-pattern)
* [How to build dynamic content in response with adaptive cards](#How-to-build-command-response-using-adaptive-card-with-dynamic-content)

Connect your app with Graph or other APIs:
* [How to access Microsoft Graph from your workflow bot](#Access-Microsoft-Graph)
* [How to connect to existing APIs from your command bot](#Connect-to-existing-APIs)

Extend command bot to other bot scenarios:
* [How to extend command bot with notification](#how-to-extend-my-command-and-response-bot-to-support-notification)
* [How to extend command bot with workflow](#how-to-extend-my-command-bot-to-support-adaptive-card-actions)

## How to create a command-response bot

### In Visual Studio Code

1. From Teams Toolkit sidebar click `Create a new Teams app` or select `Teams: Create a new Teams app` from command palette.

  ![image](https://github.com/OfficeDev/TeamsFx/assets/10163840/933116a5-c0f8-44e7-a327-49191edbd20b)

2. Select `Bot`.

  ![image](https://github.com/OfficeDev/TeamsFx/assets/10163840/2a2bd256-5bd7-42aa-b7fa-7307e161cfa1)

3. Select the `Chat Command` from Scenario-based Teams app section.
 
  ![image](https://github.com/OfficeDev/TeamsFx/assets/10163840/22135ffe-ba18-4151-9dbd-2e438e4bb931)

4. Select programming language

  ![image](https://github.com/OfficeDev/TeamsFx/assets/10163840/278187df-bf8b-4dfe-9d20-d717733824f8)

5. Enter an application name and then press enter.

  ![image](https://github.com/OfficeDev/TeamsFx/assets/10163840/f6cfc494-aa64-442d-a17e-42545925f130)

### In Visual Studio
1. Make sure you have installed ASP.NET workloads and "Microsoft Teams development tools".
![image](https://user-images.githubusercontent.com/25972542/180788786-95a98752-1207-41e8-9095-613a6b21b78d.png)

2. Create a new project and select "Microsoft Teams App".
![image](https://user-images.githubusercontent.com/25972542/180789311-d0d74a1a-b27e-4d79-b26c-1a839ec48371.png)

3. In next window enter your project name.

4. In next window select Command Bot.
![image](https://user-images.githubusercontent.com/25972542/180798463-eb91c825-aada-48bb-8f7c-83b34d46bc9e.png)

### In TeamsFx CLI

* If you prefer interactive mode, execute `teamsfx new` command, then use the keyboard to go through the same flow as in Visual Studio Code.

* If you prefer non-interactive mode, enter all required parameters in one command.

`teamsfx new --interactive false --capabilities "command-bot" --programming-language "typescript" --folder "./" --app-name myAppName`

After you successfully created the project, you can quickly start local debugging via `F5` in VSCode. Select `Debug (Edge)` or `Debug (Chrome)` debug option of your preferred browser. You can send a `helloWorld` command after running this template and get a response as below:

![command-response](https://user-images.githubusercontent.com/11220663/165891754-16916b68-c1b5-499d-b6a8-bdfb195f1fd0.png)

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Take a tour of your app source code

### For JS/TS project (In Visual Studio Code)

The created app is a normal TeamsFx project that will contain following folders:
| Folder / File | Contents |
| - | - |
| `teamsapp.yml` | Main project file describes your application configuration and defines the set of actions to run in each lifecycle stages |
| `teamsapp.local.yml`| This overrides `teamsapp.yml` with actions that enable local execution and debugging |
| `env/`| Name / value pairs are stored in environment files and used by `teamsapp.yml` to customize the provisioning and deployment rules |
| `.vscode/` | VSCode files for debugging |
| `appPackage/` | Templates for the Teams application manifest |
| `infra/` | Templates for provisioning Azure resources |
| `src/` | The source code for the application |

The following files can be customized and demonstrate an example command app implementation to get you started:
| File | Contents |
| - | - |
| `src/index.js(ts)` | Application entry point and `restify` handlers for command and response |
| `src/teamsBot.js(ts)` | An empty teams activity handler for bot customization |
| `src/adaptiveCards/helloworldCommand.json` | A generated Adaptive Card that is sent to Teams |
| `src/helloworldCommandHandler.js(ts)` | The business logic to handle a command  |

### For CSharp project (In Visual Studio)
| File name | Contents |
|- | -|
| `teamsapp.yml` | Main project file describes your application configuration and defines the set of actions to run in each lifecycle stages |
| `teamsapp.local.yml`| This overrides `teamsapp.yml` with actions that enable local execution and debugging |
| `appPackage/` | Templates for the Teams application manifest |
| `infra/` | Templates for provisioning Azure resources |
| `Properties/` | LaunchSetting file for local debug |
| `Controllers/` | BotController and NotificationControllers to handle the conversation with user |
| `Commands/` | Define the commands and how your Bot will react to these commands |
| `Models/` | Adaptive card data models |
| `Resources/` | Adaptive card templates |
| `appsettings.*.json` | The runtime settings |
| `GettingStarted.txt` | Instructions on minimal steps to wonderful|
| `Program.cs` | Create the Teams Bot instance |
| `TeamsBot.cs` | An empty Bot handler |

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## How command-and-response works
The TeamsFx Command-Response Bots are created using the [Bot Framework SDK](https://docs.microsoft.com/azure/bot-service/bot-builder-basics?view=azure-bot-service-4.0). The Bot Framework SDK provides [built-in message handler](https://docs.microsoft.com/microsoftteams/platform/bots/bot-basics?tabs=javascript#teams-activity-handlers) to handle the incoming message activity, which requires learning curve to understand the concept of Bot Framework (e.g. the [event-driven conversation model](https://docs.microsoft.com/azure/bot-service/bot-activity-handler-concept?view=azure-bot-service-4.0&tabs=javascript)). To simplify the development, the TeamsFx SDK provides command-response abstraction layer to let developers only focus on the development of business logic to handle the command request without learning the Bot Framework SDK.

Behind the scenes, the TeamsFx SDK leverages [Bot Framework Middleware](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0) to handle the integration with the underlying activity handlers. This middleware handles the incoming message activity and invokes the corresponding `handlerCommandReceived` function if the received message text matches the command pattern provided in a `TeamsFxBotCommandHandler` instance. After processing, the middleware will call `context.sendActivity` to send the command response returned from the `handlerCommandReceived` function to the user.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Customize initialization
To respond to command in chat, you need to create `ConversationBot` first. (Code already generated in project)

### For JavaScript / TypeScript Command app: 
``` typescript
/** JavaScript/TypeScript: src/internal/initialize.js(ts) **/
const commandApp = new ConversationBot({
  // The bot id and password to create CloudAdapter.
  // See https://aka.ms/about-bot-adapter to learn more about adapters.
  adapterConfig: {
    MicrosoftAppId: config.botId,
    MicrosoftAppPassword: config.botPassword,
    MicrosoftAppType: "MultiTenant",
  },
  command: {
    enabled: true,
    commands: [new HelloWorldCommandHandler()],
  },
});
```

### For CSharp Command App:
```csharp
/** .NET: Program.cs **/
builder.Services.AddSingleton<HelloWorldCommandHandler>();
builder.Services.AddSingleton(sp =>
{
    var options = new ConversationOptions()
    {
        Adapter = sp.GetService<CloudAdapter>(),
        Command = new CommandOptions()
        {
            Commands = new List<ITeamsCommandHandler> { sp.GetService<HelloWorldCommandHandler>() }
        }
    };

    return new ConversationBot(options);
});
```

## Customize installation
A Teams bot can be installed into a team, or a group chat, or as personal app, depending on difference scopes. You can choose the installation target when adding the App.
- See [Distribute your app](https://docs.microsoft.com/microsoftteams/platform/concepts/deploy-and-publish/apps-publish-overview) for more install options.

  ![Installation Target](https://github.com/OfficeDev/TeamsFx/wiki/notification/addanapp.png)

- See [Remove an app from Teams](https://support.microsoft.com/office/remove-an-app-from-teams-0bc48d54-e572-463c-a7b7-71bfdc0e4a9d) for uninstallation.

## Customize adapter
```typescript
/** Typescript **/
// Create your own adapter
const adapter = new CloudAdapter(...);

// Customize your adapter, e.g., error handling
adapter.onTurnError = ...

const bot = new ConversationBot({
    // use your own adapter
    adapter: adapter;
    ...
});

// Or, customize later
bot.adapter.onTurnError = ...
```

## How to add more command and response

You can use the following 4 steps to add more command and response:
1. [Step 1: Add a command definition in manifest](#Step-1-Add-a-command-definition-in-manifest)
1. [Step 2: Respond with an Adaptive Card](#Step-2-Respond-with-an-Adaptive-Card)
1. [Step 3: Handle the command](#Step-3-Handle-the-command)
1. [Step 4: Register the new command](#Step-4-Register-the-new-command)

### Step 1: Add a command definition in manifest

You can edit the manifest template file `appPackage\manifest.json` to include definitions of a `doSomething` command with its title and description in the `commands` array:

```json
"commandLists": [
  {
    "commands": [
        {
            "title": "helloWorld",
            "description": "A helloworld command to send a welcome message"
        },
        {
            "title": "doSomething",
            "description": "A sample do something command"
        }
    ]
  }
]
```

### Step 2: Respond with an Adaptive Card

To respond with an Adaptive Card, define your card in its JSON format:
* For JavaScript/TypeScript: create a new file `src/adaptiveCards/doSomethingCommandResponse.json`.
* For .NET: create a new file `Resources/DoSomethingCommandResponse.json`.

```json
{
  "type": "AdaptiveCard",
  "body": [
    {
      "type": "TextBlock",
      "size": "Medium",
      "weight": "Bolder",
      "text": "Your doSomething Command is added!"
    },
    {
      "type": "TextBlock",
      "text": "Congratulations! Your hello world bot now includes a new DoSomething Command",
      "wrap": true
    }
  ],
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "version": "1.4"
}
```

You can use the [Adaptive Card Designer](https://adaptivecards.io/designer/) to help visually design your Adaptive Card UI.

> Please note:

> - Respond with an Adaptive Card is optional, you can simply respond with plain texts.
> - If you'd like to send adaptive card with dynamic data, please refer to [this document](#how-to-build-command-response-using-adaptive-card-with-dynamic-content).

### Step 3: Handle the command

#### JavaScript / TypeScript Command handler
The TeamsFx SDK provides a convenient class, `TeamsFxBotCommandHandler`, to handle when an command is triggered from Teams conversation message. Create a new file, `src/doSomethingCommandHandler.js(ts)`:

```javascript
/** JavaScript **/
const doSomethingCard = require("./adaptiveCards/doSomethingCommandResponse.json");
const { AdaptiveCards } = require("@microsoft/adaptivecards-tools");
const { CardFactory, MessageFactory } = require("botbuilder");

class DoSomethingCommandHandler {
  triggerPatterns = "doSomething";

  async handleCommandReceived(context, message) {
    // verify the command arguments which are received from the client if needed.
    console.log(`App received message: ${message.text}`);

    const cardData = {
      title: "doSomething command is added",
      body: "Congratulations! You have responded to doSomething command",
    };

    const cardJson = AdaptiveCards.declare(doSomethingCard).render(cardData);
    return MessageFactory.attachment(CardFactory.adaptiveCard(cardJson));
  }
}

module.exports = {
  DoSomethingCommandHandler,
};
```

```typescript
/** TypeScript **/
import { Activity, CardFactory, MessageFactory, TurnContext } from "botbuilder";
import {
  CommandMessage,
  TeamsFxBotCommandHandler,
  TriggerPatterns,
  MessageBuilder,
} from "@microsoft/teamsfx";
import doSomethingCard from "./adaptiveCards/doSomethingCommandResponse.json";
import { AdaptiveCards } from "@microsoft/adaptivecards-tools";
import { CardData } from "./cardModels";

export class DoSomethingCommandHandler implements TeamsFxBotCommandHandler {
  triggerPatterns: TriggerPatterns = "doSomething";

  async handleCommandReceived(
    context: TurnContext,
    message: CommandMessage
  ): Promise<string | Partial<Activity>> {
    // verify the command arguments which are received from the client if needed.
    console.log(`App received message: ${message.text}`);

    const cardData: CardData = {
      title: "doSomething command is added",
      body: "Congratulations! You have responded to doSomething command",
    };

    const cardJson = AdaptiveCards.declare(doSomethingCard).render(cardData);
    return MessageFactory.attachment(CardFactory.adaptiveCard(cardJson));
  }
}
```

#### CSharp Command Handler
The TeamsFx .NET SDK provides an interface `ITeamsCommandHandler` for command handler. To handle when an command is triggered from Teams conversation message. Create a new file `Commands/DoSomethingCommandHandler.cs`:

```csharp
using MyCommandApp.Models;
using AdaptiveCards.Templating;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.TeamsFx.Conversation;
using Newtonsoft.Json;

namespace MyCommandApp.Commands
{
    public class DoSomethingCommandHandler : ITeamsCommandHandler
    {
        private readonly ILogger<HelloWorldCommandHandler> _logger;
        private readonly string _adaptiveCardFilePath = Path.Combine(".", "Resources", "DoSomethingCommandResponse.json");

        public IEnumerable<ITriggerPattern> TriggerPatterns => new List<ITriggerPattern>
        {
            new RegExpTrigger("doSomething")
        };

        public HelloWorldCommandHandler(ILogger<HelloWorldCommandHandler> logger)
        {
            _logger = logger;
        }

        public async Task<ICommandResponse> HandleCommandAsync(ITurnContext turnContext, CommandMessage message, CancellationToken cancellationToken = default)
        {
            _logger?.LogInformation($"App received message: {message.Text}");

            // Read adaptive card template
            var cardTemplate = await File.ReadAllTextAsync(_adaptiveCardFilePath, cancellationToken);

            // Render adaptive card content
            var cardContent = new AdaptiveCardTemplate(cardTemplate).Expand
            (
                new HelloWorldModel
                {
                    title: "doSomething command is added",
                    body: "Congratulations! You have responded to doSomething command",
                }
            );

            // Build attachment
            var activity = MessageFactory.Attachment
            (
                new Attachment
                {
                    ContentType = "application/vnd.microsoft.card.adaptive",
                    Content = JsonConvert.DeserializeObject(cardContent),
                }
            );

            // send response
            return new ActivityCommandResponse(activity);
        }
    }
}
```

### Step 4: Register the new command
Each new command needs to be configured in the `ConversationBot`, which powers the conversational flow of the command bot template. 

```javascript
/** JavaScript / TypeScript **/
/** Update ConversationBot  in src/internal/initialize.js(ts) **/
const commandApp = new ConversationBot({
  //...
  command: {
    enabled: true,
    commands: [ 
      new HelloWorldCommandHandler(),
      new DoSomethingCommandHandler()], // newly added command handler
  },
});
```

```csharp
/** .NET **/
/** Update ConversationBot in Program.cs **/
builder.Services.AddSingleton<HelloWorldCommandHandler>();
builder.Services.AddSingleton<DoSomethingCommandHandler>(); // Add doSomething command handler to serrvice container
builder.Services.AddSingleton(sp =>
{
    var options = new ConversationOptions()
    {
        Adapter = sp.GetService<CloudAdapter>(),
        Command = new CommandOptions()
        {
            Commands = new List<ITeamsCommandHandler>
            { 
                sp.GetService<HelloWorldCommandHandler>(),
                sp.GetService<DoSomethingCommandHandler>(),  // Register doSomething command handler to ConversationBot
            }
        }
    };

    return new ConversationBot(options);
});
```

Now, you are all done with the code development of adding a new command and response into your bot app. You can just press `F5` to local debug with the command-response bot, or use provision and deploy command to deploy the change to Azure.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Customize the trigger pattern

The default pattern to trigger a command is through a defined keyword. But often times you would want to collect and process additional information retrieved after the trigger keyword. In addition to keyword match, you could also define your trigger pattern with [regular expressions](https://regex101.com/) and match against `message.text` with more controls.

When using regular expressions, any capture group can be found in `message.matches`. Below is an example that uses regular expression to capture strings after `reboot`, for example if user inputs `reboot myMachine`, `message.matches[1]` will capture `myMachine`:

```javascript
class HelloWorldCommandHandler {
  triggerPatterns = /^reboot (.*?)$/i; //"reboot myDevMachine";
  async handleCommandReceived(context, message) {
    console.log(`Bot received message: ${message.text}`);
    const machineName = message.matches[1];
    console.log(machineName);
    // Render your adaptive card for reply message
    const cardData = {
      title: "Your Hello World Bot is Running",
      body: "Congratulations! Your hello world bot is running. Click the button below to trigger an action.",
    };
    const cardJson = AdaptiveCards.declare(helloWorldCard).render(cardData);
    return MessageFactory.attachment(CardFactory.adaptiveCard(cardJson));
  }
}
```

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

### How to build command response using adaptive card with dynamic content?
Adaptive card provides [Template Language](https://docs.microsoft.com/adaptive-cards/templating/) to allow users to render dynamic content with the same layout (the template). For example, use the adaptive card to render a list of items (todo items, assigned bugs, etc) that could varies according to different user. 

1. Add your adaptive card template JSON file under `bot/adaptiveCards` folder
1. Import the card template to you code file where your command handler exists (e.g. `myCommandHandler.ts`)
1. Model your card data
1. Use `MessageBuilder.attachAdaptiveCard` to render the template with dynamic card data

You can also add new cards if appropriate for your application. Please follow this [sample](https://aka.ms/teamsfx-adaptive-card-sample) to see how to build different types of adaptive cards with a list or a table of dynamic contents using `ColumnSet` and `FactSet`.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Access Microsoft Graph

If you are responding to a command that needs access to Microsoft Graph, you can leverage single sign on to leverage the logged-in Teams user token to access their Microsoft Graph data. Read more about how Teams Toolkit can help you [add SSO](https://aka.ms/teamsfx-add-sso) to your application.

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Connect to existing API

If you want to invoke external APIs in your code but do not have the appropriate SDK, the "Teams: Connect to an API" command in Teams Toolkit VS Code extension or "teamsfx add api-connection" command in TeamsFx CLI would be helpful to bootstrap code to call target APIs. For more information, you can visit [Connect existing API document](https://aka.ms/teamsfx-connect-api).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

## Frequently Asked Questions

### How to extend my command and response bot to support notification?

1. Go to `bot\src\internal\initialize.ts(js)`, update your `conversationBot` initialization to enable notification feature:

    ![enable-notification](https://user-images.githubusercontent.com/10163840/165462039-12bd4f61-3fc2-4fc8-8910-6a4b1e138626.png)

2. Follow [this instruction](https://aka.ms/teamsfx-notification#notify) to send notification to the bot installation target (channel/group chat/personal chat). To quickly add a sample notification triggered by a HTTP request, you can add the following sample code in `bot\src\index.ts(js)`:

    ```typescript
    server.post("/api/notification", async (req, res) => {
      for (const target of await commandBot.notification.installations()) {
        await target.sendMessage("This is a sample notification message");
      }
    
      res.json({});
    });

3. Uninstall your previous bot installation from Teams, and re-run local debug to test your bot notification. Then you can send a notification to the bot installation targets (channel/group chat/personal chat) by using a HTTP POST request with target URL `https://localhost:3978/api/notification`.

To explore more details of the notification feature (e.g. send notification with adaptive card, add more triggers), you can further refer to [the notification document](https://aka.ms/teamsfx-notification).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>

### How to extend my command bot to support adaptive card actions

The Adaptive Card action handler feature enables the app to respond to adaptive card actions that triggered by end users to complete a sequential workflow. When user gets an Adaptive Card, it can provide one or more buttons in the card to ask for user's input, do something like calling some APIs, and then send another adaptive card in conversation to response to the card action.

To add adaptive card actions to command bot, you can follow the steps [here](https://aka.ms/teamsfx-card-action-response#add-more-card-actions).

<p align="right"><a href="#How-to-create-a-command-response-bot">back to top</a></p>