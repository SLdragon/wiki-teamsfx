# Configure Bot capability within your Teams app

## Introduction

A bot, chatbot, or conversational bot is an app that responds to simple commands sent in chat and replies in meaningful ways. Examples of bots in everyday use include: bots that notify about build failures, bots that provide information about the weather or bus schedules, or provide travel information. A bot interaction can be a quick question and answer, or it can be a complex conversation. Being a cloud application, a bot can provide valuable and secure access to cloud services and corporate resources. Learn more from [Build bots for Teams
](https://learn.microsoft.com/microsoftteams/platform/bots/what-are-bots).

## Prerequisites

To configure bot as additional capability, please make sure:

- You have a Teams application and its manifest.
- You have a Microsoft 365 account to test the application.

For adding bot to a tab Teams app, please go to: 
[Add bot to a tab Teams app](#add-bot-to-a-tab-teams-app).

For adding bot to a message extension Teams app, please go to: [Add bot to a message extension Teams app](#add-bot-to-a-message-extension-teams-app).

## Add bot to a tab Teams app:

Following are the steps to add Bot capability to a tab app:
1. [Create a bot Teams app using Teams Toolkit](#create-a-bot-app-using-teams-toolkit).
1. [Update manifest file](#configure-bot-capability-in-teams-application-manifest).
1. [Bring bot code to your project](#bring-bot-code-to-your-project).
1. [Setup local debug environment](#setup-local-debug-environment).
1. [Move the application to Azure](#move-the-application-to-azure).

### Create a bot app using Teams Toolkit

Please check the guide [Create a bot app with Teams Toolkit](https://learn.microsoft.com/microsoftteams/platform/toolkit/create-new-project?pivots=visual-studio-code)

### Configure bot capability in Teams application manifest

1. You can configure bot in `appPackage/manifest.json`. You can also refer to [bot schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#bots) if you want to customize.

    Example: 
    ```json
        "bots": [
            {
                "botId": "${{BOT_ID}}",
                "scopes": [
                    "personal",
                    "team",
                    "groupchat"
                ],
                "supportsFiles": false,
                "isNotificationOnly": false,
                "commandLists": [
                    {
                        "scopes": [
                            "personal",
                            "team",
                            "groupchat"
                        ],
                        "commands": [
                            {
                                "title": "welcome",
                                "description": "Resend welcome card of this Bot"
                            },
                            {
                                "title": "learn",
                                "description": "Learn about Adaptive Card and Bot Command"
                            }
                        ]
                    }
                ]
            }
        ]
    ```


### Bring bot code to your project

1. Bring your own bot app code into your project. If you don't have one, you can use the bot app project previously created and copy the source code to into your current project. We suggest you to copy them into a `bot/` folder. Your folder structure will be like:
    ```
    |-- .vscode/
    |-- appPackage/
    |-- env/
    |-- infra/
    |--public/
    |-- bot/           <!--bot source code-->
    |   |-- adaptiveCards/
    |   |-- index.ts
    |   |-- config.ts
    |   |-- teamsBot.ts
    |   |-- package.json
    |   |-- tsconfig.json
    |   |-- web.config
    |   |-- .webappignore
    |-- src/            <!--your current source code-->
    |   |-- app.ts
    |   |-- static/
    |   |-- views/
    |-- package.json
    |-- tsconfig.json
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```
    We suggest you to re-organize the folder structure and create a root package.json as following. You can create a root package.json using command "npm init -y".
     ```
    |-- .vscode/
    |-- appPackage/
    |-- env/
    |-- infra/
    |-- bot/            <!--bot source code-->
    |   |-- adaptiveCards/
    |   |-- index.ts
    |   |-- config.ts
    |   |-- teamsBot.ts
    |   |-- package.json
    |   |-- tsconfig.json
    |   |-- web.config
    |   |-- .webappignore
    |-- tab/           <!--move your current source code to a new sub folder-->
    |   |-- src/
    |   |   |-- app.ts
    |   |   |-- static/
    |   |   |-- views/
    |   |-- package.json
    |   |-- tsconfig.json
    |-- package.json <!--root package.json-->
    |-- teamsapp.local.yml
    |-- teamsapp.yml
    ```
1. Add following to your root package.json:
    ```json
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "install:bot": "cd bot && npm install",
      "install:tab": "cd tab && npm install",
      "install": "concurrently \"npm run install:bot\" \"npm run install:tab\"",
      "dev:bot": "cd bot && npm run dev",
      "start:tab": "cd tab && npm run start",
      "build:tab": "cd tab && npm run build",
      "build:bot": "cd bot && npm run build",
      "build": "concurrently \"npm run build:tab\" \"npm run build:bot\""
    },
    "dependencies": {
        "concurrently": "^7.6.0"
    },
    ```
    Notice: If you are working on a javascript project, the bot doesn't need build. Please remove "build:bot" script and change "build" script to "npm run build:tab". 

### Setup local debug environment

1. Modify `.vscode/task.json`. Add 3 new tasks: `Start local tunnel`, `Start bot`, `Start frontend`. Add `Start bot` and `Start frontend` to task `Start application`'s `dependOn`. Config `Start bot` and `Start frondend`'s `cwd` option since we already move tab and bot's code to `tab/` and `bot/` folder separately. Add `Start local tunnel` to task `Start Teams App Locally`'s `dependOn`.
    ```
    "tasks":[
            {
                // Start the local tunnel service to forward public URL to local port and inspect traffic.
                // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
                "label": "Start local tunnel",
                "type": "teamsfx",
                "command": "debug-start-local-tunnel",
                "args": {
                    "type": "dev-tunnel",
                    "ports": [
                        {
                            "portNumber": 3978,
                            "protocol": "http",
                            "access": "public",
                            "writeToEnvironmentFile": {
                                "endpoint": "BOT_ENDPOINT", // output tunnel endpoint as BOT_ENDPOINT
                                "domain": "BOT_DOMAIN" // output tunnel domain as BOT_DOMAIN
                            }
                        }
                    ],
                    "env": "local"
                },
                "isBackground": true,
                "problemMatcher": "$teamsfx-local-tunnel-watch"
            },
            {
                "label": "Start bot",
                "type": "shell",
                "command": "npm run dev:teamsfx",
                "isBackground": true,
                "options": {
                    "cwd": "${workspaceFolder}/bot"
                },
                "problemMatcher": {
                    "pattern": [
                        {
                            "regexp": "^.*$",
                            "file": 0,
                            "location": 1,
                            "message": 2
                        }
                    ],
                    "background": {
                        "activeOnStart": true,
                        "beginsPattern": "[nodemon] starting",
                        "endsPattern": "restify listening to|Bot/ME service listening at|[nodemon] app crashed"
                    }
                }
            },
            {
               "label": "Start frontend",
               "type": "shell",
               "command": "npm run dev:teamsfx",
               "isBackground": true,
               "options": {
                    "cwd": "${workspaceFolder}/tab"
                },
                "problemMatcher": {
                    "pattern": {
                        "regexp": "^.*$",
                        "file": 0,
                        "location": 1,
                        "message": 2
                    },
                    "background": {
                        "activeOnStart": true,
                        "beginsPattern": ".*",
                        "endsPattern": "listening to|Compiled|Failed|compiled|failed"
                    }
                }
            },
             {
                 "label": "Start application",
                 "dependsOn": [
                     "Start bot",
                     "Start frontend"
                 ]
             },
             {
                 "label": "Start Teams App Locally",
                 "dependsOn": [
                     "Validate prerequisites",
                     "Start local tunnel",
                     "Provision",
                     "Deploy",
                     "Start application"
                 ],
                 "dependsOrder": "sequence"
             },
    ]
    ```

1. Update `teamsapp.local.yml`. Add action `botAadApp/create` and `botFramework/create` under provision. Then update `file/createOrUpdateEnvironmentFile` action under deploy:
    ```yml
    provision:
      - uses: botAadApp/create
        with:
          # The Azure Active Directory application's display name
          name: bot-${{TEAMSFX_ENV}}
        writeToEnvironmentFile:
          # The Azure Active Directory application's client id created for bot.
          botId: BOT_ID
          # The Azure Active Directory application's client secret created for bot.
          botPassword: SECRET_BOT_PASSWORD 

      # Create or update the bot registration on dev.botframework.com
      - uses: botFramework/create
        with:
          botId: ${{BOT_ID}}
          name: bot
          messagingEndpoint: ${{BOT_ENDPOINT}}/api/messages
          description: ""
          channels:
            - name: msteams
    deploy:
      - uses: file/createOrUpdateEnvironmentFile # Generate runtime environment variables
        with:
          target: ./tab/.localConfigs
          envs:
            BROWSER: none
            HTTPS: true
            PORT: 53000
            SSL_CRT_FILE: ${{SSL_CRT_FILE}}
            SSL_KEY_FILE: ${{SSL_KEY_FILE}}

      - uses: file/createOrUpdateEnvironmentFile # Generate runtime environment variables
        with:
          target: ./bot/.localConfigs
          envs:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
    ```
    Here is an [sample project](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab) for reference.

1. Open the `Run and Debug Activity Panel` and select `Debug (Edge)` or `Debug (Chrome)`. Press F5 to preview your Teams app locally.

### Move the application to Azure

1.  Manually merge the content in `infra/` and `teamsapp.yml` folder with yours. 
Here is an [sample project](https://github.com/OfficeDev/TeamsFx-Samples/tree/v3/hello-world-bot-with-tab) for reference.
1. Run `Teams: Provision` command in Visual Studio Code to apply the bicep to Azure.

1. Run `Teams: Deploy` command in Visual Studio Code to deploy your app code to Azure.

1. Open the `Run and Debug Activity Panel` and select `Launch Remote (Edge)` or `Launch Remote (Chrome)`. Press F5 to preview your Teams app.

## Add bot to a message extension Teams app:

Since bot and message extension are both implemented on top of the Bot support architecture within Teams, adding bot to a message extension Teams app is simpler than adding to a tab Teams app.

Following are the steps to add bot capability to a message extension app:
1. [Create a bot Teams app using Teams Toolkit](#create-a-bot-app-using-teams-toolkit-1).
1. [Update manifest file](#configure-bot-capability-in-teams-application-manifest-1).
1. [Bring bot code to your project](#bring-bot-code-to-your-project-1).

### Create a bot app using Teams Toolkit

Please check the guide [Create a bot app with Teams Toolkit](https://learn.microsoft.com/microsoftteams/platform/toolkit/create-new-project?pivots=visual-studio-code)

### Configure bot capability in Teams application manifest

1. You can configure bot in `appPackage/manifest.json`. You can also refer to [bot schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#bots) if you want to customize.

    Example:  
    ```json
        "bots": [
            {
                "botId": "${{BOT_ID}}",
                "scopes": [
                    "personal",
                    "team",
                    "groupchat"
                ],
                "supportsFiles": false,
                "isNotificationOnly": false,
                "commandLists": [
                    {
                        "scopes": [
                            "personal",
                            "team",
                            "groupchat"
                        ],
                        "commands": [
                            {
                                "title": "welcome",
                                "description": "Resend welcome card of this Bot"
                            },
                            {
                                "title": "learn",
                                "description": "Learn about Adaptive Card and Bot Command"
                            }
                        ]
                    }
                ]
            }
        ]
    ```

### Bring bot code to your project

1. If you are adding bot to a message extension Teams app, then you should already have a class that extends `TeamsActivityHandler`. Bring your own bot code, or copy code from your previously created bot app to your own class. Below is an example if you copy code from Teams Toolkit created bot app:

    ```ts
      public class YourHandler extends TeamsActivityHandler{

        // bot code
        constructor(){
          super();
          this.likeCountObj = { likeCount: 0 };
          this.onMessage(async (context, next) => {});
          this.onMembersAdded(async (context, next) => {});
        }
        async onAdaptiveCardInvoke(context: TurnContext,
    invokeValue: AdaptiveCardInvokeValue):  Promise<AdaptiveCardInvokeResponse> {};

        /**
         * your own message extension code
        */
      }
    ```

## What’s next

There are other commonly suggested next steps, for example:

- [Set up CI/CD pipelines](#How-to-add-CICD)
