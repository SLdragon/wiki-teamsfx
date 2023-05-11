# Using Teams Toolkit for App Development in Codespaces

## Introduction
[GitHub Codespaces](https://docs.github.com/en/codespaces/overview) is a cloud-based development environment for your GitHub projects that allows developers to develop and collaborate on code from anywhere, using a web browser or Visual Studio Code. With GitHub Codespaces, Teams Toolkit provides new "install-less" getting started experience for Teams app development, so you can quickly get started with a fully configured dev environment without spending time on local environment setup.

## Prerequisites
Before getting started building Teams app with Codespaces, please make sure: 
1. You have a GitHub account which will be used to create a Codespaces instance.
2. You have a M365 Account with sideloading permissions. You may find the setup instructions in the [Prepare your Microsoft 365 tenant](https://docs.microsoft.com/en-us/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant). If you don’t already have a Microsoft 365 account, you can register for a free one through the [Microsoft 365 Developer Program](https://developer.microsoft.com/en-us/microsoft-365/dev-program).

## In this tutorial, you will learn
* [Quickly getting started Teams app with Codespaces](#getting-started-from-codespaces-enabled-sample)
* [How to enable codespaces for TeamsFx Tab](#enable-codespaces-for-tab-app)
* [How to enable codespaces for TeamsFx Bot / ME](#enable-codespaces-for-bot--message-extension)

## Getting started from codespaces-enabled sample
| Sample name | Entry point |
|-----|-----|
| Hello World Tab | [![Open hello-world tab in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=v3&repo=348288141&machine=standardLinux32gb&location=WestUs2&devcontainer_path=.devcontainer%2Fhello-world-tab-codespaces%2Fdevcontainer.json&resume=1) |
| Notification Bot| [![Open notification app in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=v3&repo=348288141&machine=basicLinux32gb&location=WestUs2&devcontainer_path=.devcontainer%2Fnotification-codespaces%2Fdevcontainer.json&resume=1) |
| NPM Search Message Extension | [![Open notification app in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://github.com/codespaces/new?hide_repo_select=true&ref=v3&repo=348288141&machine=basicLinux32gb&location=WestUs2&devcontainer_path=.devcontainer%2Fnpm-search-message-extension-codespaces%2Fdevcontainer.json) |

Step-by-step guide to getting started with Codespaces:

### 1. Creating your codespaces

Click the entry point button / link from documentation to enter the creation page, and then click `Create codespace` to create a codespace for a specific sample:

<img src="https://user-images.githubusercontent.com/10163840/224228193-d1d3bf5c-13d5-4b84-b922-93aac12d198e.png" width="70%" height="70%">

> Note: you can customize the creation options (e.g. region, machine type) according to your needs.

### 2. Running the sample app in codespaces
Once your codespace is created, the TeamsFx-Samples repository will be automatically cloned into it. The sample specified in the `devcontainer.json` will be launched in Codespaces browser editor. And then select `Preview your Teams app (F5)` from Teams Toolkit or simply press `F5` to run and preview your application:

<img src="https://user-images.githubusercontent.com/10163840/224229999-e032325e-24b1-4b4f-8f98-5f19462c7961.png" width="70%" height="70%">

### 3. Preview App
When Teams launches in the browser, select the `Add` button in the dialog to install your app to Teams.

   > **Note**: You may need to **allow pop-ups** so that Codespace can open a new browser to sideload the app to Teams:
   >
   > ![image](https://user-images.githubusercontent.com/10163840/225506097-18d04d70-ea4c-4a10-bde4-9d38654a2e72.png)

## Enable Codespaces for Tab App
You can enable codespaces configuration for your TeamsFx Tab project on GitHub by following the steps below:

### 1. Add a dev container configuration

To set up your repository to use a custom dev container for building apps with Teams Toolkit, you'll need to create a `devcontainer.json` file and place it in the `.devcontainer` folder located in the root directory of your project. You can start from using the following sample `devcontainers.json`:

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/javascript-node
{
  "name": "tab-codespaces",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:16",
  "forwardPorts": [
    53000
  ],
  "portsAttributes": {
    "53000": {
      "label": "app",
      "protocol": "https"
    }
  },
  "remoteUser": "node",
  "customizations": {
    "vscode": {
      "settings": {
        "extensions.autoUpdate": false,
        "extensions.autoCheckUpdates": false
      },
      "extensions": [
        "TeamsDevApp.ms-teams-vscode-extension@prerelease"
      ]
    }
  },
  // Install GitHub CLI
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {
      "version": "latest"
    }
  },
  // Install teamsfx CLI
  "postCreateCommand": "npm install -g @microsoft/teamsfx-cli"
}
```

### 2. Update `.vscode/tasks.json`
Add the following debug tasks in `.vscode/tasks.json`:
* `Configure port visibility` task: set the port visibility to public using the GitHub CLI.
* `Open Teams Web Client` task: launch Teams web client to sideload your tab app.
* `Start Teams App in Codespaces` task: the main task to start the app in Codespaces.

```json
{
    "version": "2.0.0",
    "tasks": [
        ...
        {
            "label": "Configure port visibility",
            "type": "shell",
            "command": "gh codespace ports visibility 53000:public -c $CODESPACE_NAME"
        },
        {
            // Launch Teams web client.
            // See https://aka.ms/teamsfx-tasks/launch-web-client to know the details and how to customize the args.
            "label": "Open Teams Web Client",
            "type": "teamsfx",
            "command": "launch-web-client",
            "args": {
              "env": "local",
              "manifestPath": "${workspaceFolder}/appPackage/manifest.json"
            }
        },
        {
            "label": "Start Teams App in Codespaces",
            "dependsOn": [
                "Validate prerequisites",
                "Configure port visibility",
                "Provision",
                "Deploy",
                "Start application",
                "Open Teams Web Client"
            ],
            "dependsOrder": "sequence"
        },
        ...
    ]
}
```

### 3. Update `.vscode/launch.json`
Add launch configuration `.vscode/launch.json` to preview your app in Codespaces:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Preview (Codespaces)",
            "type": "node",
            "request": "launch",
            "preLaunchTask": "Start Teams App in Codespaces",
            "presentation": {
                "group": "all",
                "order": 0
            },
            "internalConsoleOptions": "neverOpen"
        },
        ...
    ]
    ...
}
```

### 4. Update `teamsapp.local.yml`

In contrast to a locally running app, an app running in Codespaces will have its endpoint forwarded to a publicly accessible Codespaces URL. So we need to update the `TAB_DOMAIN` and `TAB_ENDPOINT` with codespaces URL instead of using `localhost:53000`:

```yml
...
TAB_DOMAIN: ${{CODESPACE_NAME}}-53000.${{GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}}
TAB_ENDPOINT`: https://${{CODESPACE_NAME}}-53000.${{GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}}
```

### 5. Commit changes to GitHub repository
Finally, you need to commit all the above code changes to your project repository so that developers can create codespaces for your Teams project.

## Enable Codespaces for Bot / Message Extension
You can enable codespaces configuration for your TeamsFx bot/ME project on GitHub by following the steps below:

### 1. Add a dev container configuration
To set up your repository to use a custom dev container for building apps with Teams Toolkit, you'll need to create a `devcontainer.json` file and place it in the `.devcontainer` folder located in the root directory of your project. You can start from using the following sample `devcontainers.json`:

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/javascript-node
{
  "name": "bot-codespaces",
  "image": "mcr.microsoft.com/devcontainers/typescript-node:16",

  // Default workspace folder
  "workspaceFolder": "/workspaces/TeamsFx-Samples/notification-codespaces/",

  // Use 'forwardPorts' to make a list of ports inside the container available locally.
  "forwardPorts": [
    3978
  ],
  "portsAttributes": {
    "3978": {
      "label": "app"
    }
  },
  "remoteUser": "node",
  "customizations": {
    "vscode": {
      "settings": {
        "extensions.autoUpdate": false,
        "extensions.autoCheckUpdates": false
      },
      "extensions": [
        "TeamsDevApp.ms-teams-vscode-extension@prerelease"
      ]
    }
  },
  // Install GitHub CLI
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {
      "version": "latest"
    }
  },
  // Install teamsfx CLI
  "postCreateCommand": "npm install -g @microsoft/teamsfx-cli"
}
```

### 2. Update `.vscode/tasks.json`
Add the following debug tasks in `.vscode/tasks.json`:
* `Configure port visibility` task: set the port visibility to public using the GitHub CLI.
* `Open Teams Web Client` task: launch Teams web client to sideload your tab app.
* `Start Teams App in Codespaces` task: the main task to start the app in Codespaces.

```json
{
    "version": "2.0.0",
    "tasks": [
        ...
        {
            "label": "Start Teams App in Codespaces",
            "dependsOn": [
                "Validate prerequisites",
                "Configure port visibility",
                "Provision",
                "Deploy",
                "Start application",
                "Open Teams Web Client"
            ],
            "dependsOrder": "sequence"
        },
        {
            "label": "Configure port visibility",
            "type": "shell",
            "command": "gh codespace ports visibility 3978:public -c $CODESPACE_NAME"
        },
        {
            // Launch Teams web client.
            // See https://aka.ms/teamsfx-tasks/launch-web-client to know the details and how to customize the args.
            "label": "Open Teams Web Client",
            "type": "teamsfx",
            "command": "launch-web-client",
            "args": {
              "env": "local",
              "manifestPath": "${workspaceFolder}/appPackage/manifest.json"
            }
        }
        ...
    ]
}
```

### 3. Update `.vscode/launch.json`
Add the following `Debug (Codespaces)` launch configuration in `.vscode/launch.json` to debug your app in Codespaces:
```json
{
    "version": "0.2.0",
    ...
    "compounds": [
        {
            "name": "Debug (Codespaces)",
            "configurations": [
                "Attach to Bot"
            ],
            "preLaunchTask": "Start Teams App in Codespaces",
            "presentation": {
                "group": "all",
                "order": 1
            },
            "stopAll": true
        }
    ]
}
```

### 4. Update `teamsapp.local.yml`

In contrast to a locally running app, an app running in Codespaces will have its endpoint forwarded to a publicly accessible Codespaces URL. So we need to update the `messagingEndpoint` with codespaces URL instead of using `https://localhost:3978/api/message`:

```yml
...
messagingEndpoint: https://${{CODESPACE_NAME}}-3978.${{GITHUB_CODESPACES_PORT_FORWARDING_DOMAIN}}/api/messages
```

### 5. Commit changes to GitHub repository
Finally, you need to commit all the above code changes to your project repository so that developers can create codespaces for your Teams project.

## What's Next
TBD.