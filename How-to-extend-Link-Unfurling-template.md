## How to use Zero Install Link Unfurling in Teams

Zero Install Link Unfurling requires link unfurling app to be published. You need an admin account to publish an app into your org.

Login your admin account in Teams. Go to `Manage your apps` -> `Upload an app`. Click `Upload an app to your org's app catalog` to upload your app's zip file.

![upload](https://github.com/OfficeDev/TeamsFx/assets/11220663/895924f4-89e6-44cf-a831-77d282c91d4c)

Switch to another user account. Without installing this app, paste the link "https://www.botframework.com" into chat box, and you should see the adaptive card like below.

![zeroInstall](https://github.com/OfficeDev/TeamsFx/assets/11220663/8b89433c-b73f-486c-b52f-21c449f7347f)

## How to add link unfurling cache in Teams

This template removes cache by default to provide convenience for debug. To add cache, ***REMOVE*** following JSON part from adaptive card in `linkUnfurlingApp.ts` (`linkUnfurlingApp.js`):

```ts
suggestedActions: {
          actions: [
            {
              title: "default",
              type: "setCachePolicy",
              value: '{"type":"no-cache"}',
            },
          ],
        }
```

After removing this, the link unfurling result will be cached in Teams for 30 minutes. Please refer to [link unfurling document](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/how-to/link-unfurling?tabs=desktop%2Cjson%2Cadvantages#remove-link-unfurling-cache) for more details.

## How to customize Zero Install Link Unfurling's adaptive cards

The supported types for Zero Install Link Unfurling are "result" and "auth" and this template uses "result" as default. By changing it to "auth", the adaptive card will be:

![zeroInstallAuth](https://github.com/OfficeDev/TeamsFx/assets/11220663/620b8dd0-8003-4fda-9865-ac94d212972e)

For card with type "auth", the Teams client strips away any action buttons from the card, and adds a sign in action button. Please refer to [zero install link unfurling document](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/how-to/link-unfurling?tabs=desktop%2Cjson%2Climitations#zero-install-for-link-unfurling) for more details.
## How to add stage view

Stage View is a full screen UI component that you can invoke to surface your web content. You can turn URLs into a tab using an Adaptive Card and Chat Services. Follow the instructions below to add stage view in your link unfurling app.

- [Step 1: Update `staticTabs` in manifest.json](#step-1-update-statictabs-in-manifestjson)
- [Step 2: Update source code](#step-2-update-source-code)
- [Step 3: Set `BOT_DOMAIN` and `TEAMS_APP_ID` in environment variables](#step-3-set-bot_domain-and-teams_app_id-in-environment-variables)
- [Step 4: Update adaptive card](#step-4-update-adaptive-card)

### Step 1: Update `staticTabs` in `manifest.json`

In `appPackage/manifest.json`, update `staticTabs` section.

```json
    "staticTabs": [
        {
            "entityId": "stageViewTask",
            "name": "Stage View",
            "contentUrl": "https://${{BOT_DOMAIN}}/tab",
            "websiteUrl": "https://${{BOT_DOMAIN}}/tab",
            "searchUrl": "https://${{BOT_DOMAIN}}/tab",
            "scopes": [
                "personal"
            ],
            "context": [
                "personalTab",
                "channelTab"
            ]
        }
    ],
```

### Step 2: Update source code
In `src/index.ts` (`src/index.js`), add following code.

```ts
server.get("/tab", async (req, res) => {
  const body = `<!DOCTYPE html>
  <html lang="en">
  
  <div class="text-center">
    <h1 class="display-4">Tab in stage View</h1>
  </div>
  
  </html>`;
  res.writeHead(200, {
    'Content-Length': Buffer.byteLength(body),
    'Content-Type': 'text/html'
  });
  res.write(body);
  res.end();
});
```

### Step 3: Set `BOT_DOMAIN` and `TEAMS_APP_ID` in environment variables

For local debug:

Update action `file/createOrUpdateEnvironmentFile` in `teamsapp.local.yml`, add `TEAMS_APP_ID` and `BOT_DOMAIN` to env.

```yaml
  - uses: file/createOrUpdateEnvironmentFile # Generate runtime environment variables
    with:
      target: ./.localConfigs
      envs:
        BOT_ID: ${{BOT_ID}}
        BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
        TEAMS_APP_ID: ${{TEAMS_APP_ID}}
        BOT_DOMAIN: ${{BOT_DOMAIN}}
```

For remote:

Update `infra/azure.parameters.json`. Add following to `parameters`:

```json
    "teamsAppId":{
      "value": "${{TEAMS_APP_ID}}"
    }
```

Add following to `infra/azure.bicep`:

```bicep
param teamsAppId string 

resource webAppSettings 'Microsoft.Web/sites/config@2022-09-01' = {
  parent: webApp
  name: 'appsettings'
  properties: {
    BOT_DOMAIN: webApp.properties.defaultHostName
    BOT_ID: botAadAppClientId
    BOT_PASSWORD: botAadAppClientSecret
    RUNNING_ON_AZURE: '1'
    TEAMS_APP_ID: teamsAppId
    WEBSITE_NODE_DEFAULT_VERSION: '~18'
    WEBSITE_RUN_FROM_PACKAGE: '1'
  }
}
```

### Step 4: Update adaptive card

In `src/adaptiveCards/helloWorldCard.json`, update `actions` to be following.

```json
"actions": [
        {
            "type": "Action.OpenUrl",
            "title": "View Via Deep Link",
            "url": "https://teams.microsoft.com/l/stage/${appId}/0?context=%7B%22contentUrl%22%3A%22https%3A%2F%2F${url}%2Ftab%22%2C%22websiteUrl%22%3A%22https%3A%2F%2F${url}%2Ftab%22%2C%22name%22%3A%22DemoStageView%22%7D"
        }
      ],
```

Run `npm install @microsoft/adaptivecards-tools`. This package helps render placeholders such as `${url}` in adaptive card to be real values.

In `src/linkUnfurlingApp.ts` (`src/linkUnfurlingApp.js`), update variable `attachment` to be following.

```ts
    const data = { url: process.env.BOT_DOMAIN, appId: process.env.TEAMS_APP_ID };

    const renderedCard = AdaptiveCards.declare(card).render(data);

    const attachment = { ...CardFactory.adaptiveCard(renderedCard), preview: previewCard };

```

In Teams, the adaptive card will be like:

![stageView](https://github.com/OfficeDev/TeamsFx/assets/25220706/db467fc6-8e2d-4197-8c89-171e5f29331e)

Opening stage view from Adaptive card via deep link:

![viaDeepLink](https://github.com/OfficeDev/TeamsFx/assets/11220663/3664bcde-eb66-4b00-882b-9e9e44006ec6)

In Outlook, the adaptive card will be like:

![stageView](https://github.com/OfficeDev/TeamsFx/assets/25220706/9ea42c24-e06a-4c48-b8dc-e8a884d8d883)

Opening stage view from Adaptive card via deep link:

![viaDeepLink](https://github.com/OfficeDev/TeamsFx/assets/25220706/b02ae0c8-6ef3-4e79-aabf-09e6b621d567)

Please refer to [Stage view document](https://learn.microsoft.com/microsoftteams/platform/tabs/tabs-link-unfurling) for more details.

## How to add task module (Teams)

Once your link is unfurled into an Adaptive Card and sent in conversation, you can use Task modules to create modal pop-up experiences in your Teams application. Follow the instructions below to add task module in your link unfurling app.

- [Step 1: Update adaptive card](#step-1-update-adaptive-card)
- [Step 2: Add `handleTeamsTaskModuleFetch` function in handler](#step-2-add-handleteamstaskmodulefetch-function-in-handler)
- [Step 3: Add `handleTeamsTaskModuleSubmit` function in handler](#step-3-add-handleteamstaskmodulesubmit-function-in-handler)

#### Step 1: Update adaptive card

In `src/adaptiveCards/helloWorldCard.json`, update `actions` to be following.

```json
    "actions": [
        {
            "type": "Action.Submit",
            "title": "Task module",
            "data": {
                "msteams": {
                    "type": "task/fetch",
                    "data": "task module"
                }
            }
        }
      ],
```

### Step 2: Add `handleTeamsTaskModuleFetch` function in handler

For typescript template, in `src/linkUnfurlingApp.ts`, add following method to `LinkUnfurlingApp` class.

```ts
  public async handleTeamsTaskModuleFetch(context: TurnContext, taskModuleRequest: TaskModuleRequest): Promise<TaskModuleResponse> {
    return {
      task: {
        type: "continue",
        value: {
          title: "Task Module Fetch",
          height: 200,
          width: 400,
          card: CardFactory.adaptiveCard({
            version: '1.0.0',
            type: 'AdaptiveCard',
            body: [
              {
                type: 'TextBlock',
                text: 'Enter Text Here'
              },
              {
                type: 'Input.Text',
                id: 'usertext',
                placeholder: 'add some text and submit',
                IsMultiline: true
              }
            ],
            actions: [
              {
                type: 'Action.Submit',
                title: 'Submit'
              }
            ]
          })
        },
      },
    };
  }
```
For javascript template, in `src/linkUnfurlingApp.js`, add following method to `LinkUnfurlingApp` class.
```js
  handleTeamsTaskModuleFetch(context, taskModuleRequest) {
    return {
      task: {
        type: "continue",
        value: {
          title: "Task Module Fetch",
          height: 200,
          width: 400,
          card: CardFactory.adaptiveCard({
            version: '1.0.0',
            type: 'AdaptiveCard',
            body: [
              {
                type: 'TextBlock',
                text: 'Enter Text Here'
              },
              {
                type: 'Input.Text',
                id: 'usertext',
                placeholder: 'add some text and submit',
                IsMultiline: true
              }
            ],
            actions: [
              {
                type: 'Action.Submit',
                title: 'Submit'
              }
            ]
          })
        },
      },
    };
  }
```
### Step 3: Add `handleTeamsTaskModuleSubmit` function in handler

For typescript template, in `src/linkUnfurlingApp.ts`, add following method to `LinkUnfurlingApp` class.
```ts
  public async handleTeamsTaskModuleSubmit(context: TurnContext, taskModuleRequest: TaskModuleRequest): Promise<TaskModuleResponse> {
    return {
      task: {
        type: 'message',
        value: 'Thanks!'
      }
    };
  }
```
For javascript template, in `src/linkUnfurlingApp.js`, add following method to `LinkUnfurlingApp` class.
```js
  handleTeamsTaskModuleSubmit(context, taskModuleRequest) {
    return {
      task: {
        type: 'message',
        value: 'Thanks!'
      }
    };
  }
```
In Teams, the adaptive card will be like:

![taskModule](https://github.com/OfficeDev/TeamsFx/assets/11220663/eec7cf9b-d134-4f76-badf-c3fe312e8655)

Click "Task module" button:

![taskModuleFetch](https://github.com/OfficeDev/TeamsFx/assets/11220663/c6a0c95f-8708-4b2d-81a5-6b74aa9803e5)

Click "Submit" button:

![taskModuleSubmit](https://github.com/OfficeDev/TeamsFx/assets/11220663/935be38d-f17e-422c-bd5d-e4cfdc9b172a)

Please refer to [Task module document](https://learn.microsoft.com/microsoftteams/platform/task-modules-and-cards/task-modules/task-modules-bots?tabs=nodejs) for more details.

## How to add adaptive card action (Teams)

Adaptive Card actions allow users to interact with your card by clicking a button or selecting a choice. Follow the instructions below to add adaptive card action in your link unfurling app.

- [Step 1: Update `bots` section in manifest](#step-1-update-bots-section-in-manifest)
- [Step 2: Update adaptive card](#step-2-update-adaptive-card)
- [Step 3: Add `onAdaptiveCardInvoke` function in handler](#step-3-add-onadaptivecardinvoke-function-in-handler)

### Step 1: Update `bots` section in manifest

The card action requires bot capability. In `appPackage/manifest.json`, update `bots` section to be following.

```json
    "bots": [
        {
            "botId": "${{BOT_ID}}",
            "scopes": [
                "team",
                "personal",
                "groupchat"
            ],
            "supportsFiles": false,
            "isNotificationOnly": false
        }
    ]
```

### Step 2: Update adaptive card

In `src/adaptiveCards/helloWorldCard.json`, update `actions` to be following.

```json
    "actions": [
        {
            "type": "Action.Execute",
            "title": "card action",
            "verb": "cardAction",
            "id": "cardAction"
        }
    ],
```

### Step 3: Add `onAdaptiveCardInvoke` function in handler

For typescript template, in `src/linkUnfurlingApp.ts`, add following method to `LinkUnfurlingApp` class.

```ts
  public async onAdaptiveCardInvoke(context: TurnContext, invokeValue: AdaptiveCardInvokeValue): Promise<AdaptiveCardInvokeResponse> {
    const card = {
      "type": "AdaptiveCard",
      "body": [
        {
          "type": "TextBlock",
          "text": "Your response was sent to the app",
          "size": "Medium",
          "weight": "Bolder",
          "wrap": true
        },
      ],
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.4"
    };
    const res = { statusCode: 200, type: "application/vnd.microsoft.card.adaptive", value: card };
    return res;
  }
```
For javascript template, in `src/linkUnfurlingApp.js`, add following method to `LinkUnfurlingApp` class.
```js
  onAdaptiveCardInvoke(context, invokeValue) {
    const card = {
      "type": "AdaptiveCard",
      "body": [
        {
          "type": "TextBlock",
          "text": "Your response was sent to the app",
          "size": "Medium",
          "weight": "Bolder",
          "wrap": true
        },
      ],
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.4"
    };
    const res = { statusCode: 200, type: "application/vnd.microsoft.card.adaptive", value: card };
    return res;
  }
```
In Teams, the adaptive card will be like:

![cardAction](https://github.com/OfficeDev/TeamsFx/assets/11220663/5019a59f-4707-42d8-8bc3-9fdf2981b8fa)

Click `card action` button, the adaptive card will be updated to be following:

![cardActionClick](https://github.com/OfficeDev/TeamsFx/assets/11220663/b141dfe0-2586-47af-9364-e1a5c889f3a8)

Please refer to [Universal actions document](https://learn.microsoft.com/microsoftteams/platform/task-modules-and-cards/cards/universal-actions-for-adaptive-cards/overview) for more details.

## How to Extend this template with Notification, Command and Workflow bot.
The Notification, Command and Workflow Bot are scenario templates provided by Teams Toolkit. These templates have similar structure. This guide takes Notification Bot as an example.

### Step 1: Create a Notification Bot template using Teams Toolkit.
Select the Teams Toolkit icon on the left in the VS Code toolbar. Choose "Create a New App"->"Bot"->"Chat Notification Message". Wait for the download complete.

### Step 2: Update source code.
Copy all methods from `src/linkUnfurlingApp.ts` class to Notification Bot's empty `TeamsActivityHandler` in `src/teamsBot.ts`.

### Step 3: Update manifest.
Copy `composeExtension` section in your `appPackage/manifest.json`
to Notification bot's `appPackage/manifest.json`.

Now your Notification bot project has both notification and link unfurling function.

![notification link unfurling](https://github.com/OfficeDev/TeamsFx/assets/25220706/17a5742c-0233-474e-bc2e-78c3ff7e9553)