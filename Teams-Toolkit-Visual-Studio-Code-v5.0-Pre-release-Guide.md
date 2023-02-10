Teams Toolkit enables developers to build Teams applications that bring their existing LOB / ISV / SaaS application into Teams with Teams-native integration constructs such as messaging notifications, messaging interactions, and tab support.

At the same time, Teams Toolkit helps developers simplify the cloud infrastructure and deployment processes that are required to run their application. Developers can leverage existing cloud resources, leverage existing cloud infrastructure source code, or use Teams Toolkit’s opinionated templates to provision and deploy their application.

# Core concepts

Teams Toolkit consists of several tools that come together to enable you to build Teams applications:
* Teams Toolkit CLI / VS Code / VS “engine” – the “engine” enables you to build, provision, deploy, publish, and run applications 
* Samples and templates - Samples and templates help developers start quickly with simple scaffolds
* Accelerators - These are code generators that generate full end to end applications based on developer customization

## Teams Toolkit project

A Teams Toolkit project drives the Teams Toolkit engine – that is, what the Teams Toolkit engine does in each of the stages is defined by the project.

A Teams Toolkit project contains the following files:

<html>
<body>
<!--StartFragment-->

File | Required? | Description
-- | -- | --
teamsapp.yml | Yes | This is the main Teams Toolkit project file. The project file defines two primary things:  Properties and configuration Stage definitions. See teamsapp.yml below for more details.
teamsapp.local.yml | Yes (for local deployments) | This overrides `teamsapp.yml` with actions that enable local execution and debugging. See teamsapp.local.yml below for more details.
env/ | Depends on the contents of the yml file | Name / value pairs are stored in environment files and used by `teamsapp.yml` to customize the provisioning and deployment rules. See environments below for more details.
.vscode/tasks.json and .vscode/launch.json | Yes for Visual Studio Code debugging, No otherwise | Used to configure the debugging process for Visual Studio Code. See Customize debugging process and infrastructure below for more details.

<!--EndFragment-->
</body>
</html>

### Project files

Teams Toolkit looks for `teamsapp.yml` in the current directory when it runs. This file describes your application configuration and defines the set of actions to run in each lifecycle stages. It also looks for an teamsapp.local.yml file if you are in a local environment. This overrides the stages with actions that support local development.

Teams Toolkit generated projects include teamsapp.yml and teamsapp.local.yml files by default, you can customize them to fit your needs.

### Lifecycles

An app developed with Teams Toolkit goes through a few lifecycle stages as developers bring their application from source to production. Teams Toolkit lifecycles are defined in teamsapp.yml and teamsapp.local.yml files and can be customized to your liking.

<html>
<body>
<!--StartFragment-->

Stage | Description
-- | --
Provision | Prepares the current `environment`. This can be creating cloud resources or installing local tools.
Deploy | Build and deploy the application to the environment infrastructure. Cloud resources don't have to be created by Teams Toolkit; they can also be pre-created by your DevOps teams.
Publish | Publish the application to Teams

<!--EndFragment-->
</body>
</html>

## Actions

Actions are individual blocks of processes used in yml files to describe what happens in each app stage. Teams Toolkit provides a set of pre-defined actions that your project can integrate with. You can also write your own script actions to customize Teams Toolkit apps.

<b>[Learn more about actions](https://aka.ms/teamsfx-actions)</b>

## Environments

Teams Toolkit environments are principally a collection of cloud resources that are targets for a deployment. For example, the `dev` environment consists of a set of cloud resources that are used for development, and the `prod` environment consists of a set of cloud resources that are used for production.

Teams Toolkit environments are defined in a `.env` file. There are several ways to create a `.env` file:

* Using the provisioning lifecycle stage. Teams Toolkit will generate an environment file for you
* Manually. If you are not using Teams Toolkit to provision your cloud environment, you can create an environment file manually
* Importer (not yet implemented). We plan to provide an importer that allows you to generate an environment by selecting cloud resources in a friendly manner

Environments are optional.

Your project file can hard code cloud resources in their deployment targets. When you do this, you do not need a Teams Toolkit environment.

However, your project file can reference values by name defined in the environment files. When you do this, you can supply different environment files and Teams Toolkit will deploy to the cloud resources defined in the environment file. In this way you can have a single set of deploy rules for an arbitrary number of environments.

### Environment file definition

The .env files follow the naming convention of .env.{environment-name}. For each lifecycle execution you are required to provide an environment name (local, dev, etc) and Teams Toolkit will load the corresponding .env.{environment-name} into the execution process.

You can also define variables in your current shell environment, Teams Toolkit will load these environment variables when running a lifecycle stage. Environment variables defined in the current shell overwrites variables defined in .env files when there is a name conflict.

### Environment file location

By default, Teams Toolkit generated templates configures the project to store `.env` in  the `~/<app>/env` folder. You change this by configuring the project file - set the `environmentFolderPath` field in `teamsapp.yml` appropriately.

### Local environments

There is nothing special from Team Toolkit's perspective about local environments.

However, Teams Toolkit templates and samples all come with an environment called `local`. This environment enables you to run and deploy your Azure components locally. *Note* App registrations and the Teams client itself still runs in the cloud.

The pattern that the templates follow is to use `teamsapp.local.yml` to override the lifecycle stages with specific actions that run locally.

<b>[Learn more about how to customize actions in teamsapp.local.yml](https://aka.ms/teamsfx-actions)</b>

# Reference

## yml definition

Note - earlier we defined several lifecycle stages. In actuality, some lifecycle stages are broken down into sub-stages in the yml definition. Therefore Teams Toolkit yml files have the following lifecycle definitions:

* registerApp: performs app registrations, e.g. registering a Teams app in developer portal
* provision: provisions cloud resources, e.g. provisioining cloud resources with ARM templates
* configureApp: configures the registered apps, e.g. updating a Teams app in developer portal
* deploy: deploys application code to cloud resources, e.g. deploying to Azure Storage
* publish: publishes a Teams app to the Teams app catalog

These map to the lifecycle stages in this way:

* Provision -> runs `registerApp`, `provision`, and `configureApp` in that order
* Deploy -> runs `deploy`
* Publish -> runs `publish`

# Examples

# Customize debugging

## Debug Teams app (F5) in Visual Studio Code

There are three files defining the details F5 steps, `.vscode/launch.json`, `.vscode/tasks.json`, and `teamsapp.local.yml`.

- `launch.json` - this is the entrypoint of VSCode F5. It defines the final launch target (for Teams App it's opening web client in browser) and the `preLaunchTask` to be executed before launching final target.
- `tasks.json` - this defines all tasks to be executed by VSCode. There is one entry task which consists of several small tasks, while some of those tasks reference `teamsapp.local.yml` to represent teams app lifecycle.
- `teamsapp.local.yml` - this defines teams app specific lifecycles. Each lifecycle contains a set of teams app specific actions.

In addition, there is a `.env.local` file (by default under `env/` folder) which includes environment variables referenced/generated by `teamsapp.local.yml`.

Following shows the step-by-step flow after clicking F5, and the place where each step is defined in:
![F5](images/vsc-f5-flow.png)

1. The `preLaunchTask` property in `launch.json` is pointed to a task defined in `tasks.json`. That task is an entrypoint and consists of several other tasks. The task set can be customized in `tasks.json`.
    ``` json
    // launch.json
    {
    ...
      "preLaunchTask": "Start Teams App Locally",
    ...
    }
    ```
    ``` json
    // tasks.json
    {
        "label": "Start Teams App Locally",
        // consists of other tasks
        "dependsOn": [
            "Validate prerequisites",
            "Start local tunnel",
            "Create resources",
            "Install tools and Build project",
            "Start services"
        ],
        "dependsOrder": "sequence"
    }
    ```

2. Task `Validate prerequisites` is to check and auto-resolve prerequisites required by debugging. This task can be customized in `tasks.json`.

3. If your Teams App has capability `bot`, task `Start local tunnel` is to launch a local tunnel service (ngrok) to make your local bot message endpoint public. This task can be customized in `tasks.json`.

4. Task `Create resource` is to execute lifecycle `provision` to prepare Teams App related resources. It's referencing `teamsapp.local.yml` so the steps and actions can be customized in `teamsapp.local.yml`.
    ``` json
    // tasks.json
    {
        "label": "Create resources",
        "type": "teamsfx",
        "command": "provision",
        "args": {
            // reference teamsapp.local.yml
            "template": "${workspaceFolder}/teamsapp.local.yml",
            // set env to local to reference .env.local
            "env": "local"
        }
    }
    ```

5. Task `Install tools and Build project` is to execute lifecycle `deploy` to ensure local project is runnable. It's referencing `teamsapp.local.yml` so the steps and actions can be customized in `teamsapp.local.yml`.
    ``` json
    // tasks.json
    {
        "label": "Install tools and Build project",
        "type": "teamsfx",
        "command": "deploy",
        "args": {
            // reference teamsapp.local.yml
            "template": "${workspaceFolder}/teamsapp.local.yml",
            // set env to local to reference .env.local
            "env": "local"
        }
    }
    ```

6. Task `Start services` is to launch local project. It usually contains one or more tasks to run different commands `npm run xxx` to launch local service directly or launch certain dependency service(s). All these tasks can be customized in `tasks.json`.

7. The `url` property in `launch.json` defines the target URL to be opened in browser. Usually it's in format <u>*...teams.microsoft.com/...**\${local:teamsAppId}**?...**\${account-hint}**...*</u>. The two place holders ***local:teamsAppId*** and ***account-hint*** will be replaced dynamically by Teams Toolkit to navigate to the correct Teams App.

> Note, the place holder *\${**local**:teamsAppId}* or *\${**dev**:teamsAppId}* in `launch.json` means to load Teams App ID from *local* or *dev* environment. If you creates your own environment (e.g., ***test01***), you can change that to any environment (e.g., *\${**test01**:teamsAppId}*).

## Integrate your own local tunneling tool for debug

Teams Toolkit use ngrok for tunneling, but you could customize the tunneling settings or use your own tunneling service by modifying .`vscode/tasks.json`. Visit [teamsfx local tunnel task](https://aka.ms/teamsfx-local-tunnel-task) for more details.

## Use your existing Teams app ID

You can use your existing Teams app ID instead of registering a new one with Teams Toolkit.

You can do this by setting the `TEAMS_APP_ID` environment variable. When `TEAMS_APP_ID` exist, the `teamsApp/create` action won't register a new Teams app, the Teams `manifest.json` file can reference `TEAMS_APP_ID`. <b>[Learn more about Teams Toolkit actions](#teamsappcreate)</b>

If you are developing with a Teams Toolkit scaffolded project, you can set it in `.env` files listed under the env folder.

![image](https://user-images.githubusercontent.com/103554011/217702296-731b046e-0c16-476f-af12-28c9827ae568.png)

Also make sure your `manifest.json` file correctly reference it.

![image](https://user-images.githubusercontent.com/103554011/217994243-71bfed2f-5149-4953-b36e-7395320c1f0a.png)

You can find your Teams app ID in Teams Developer Portal, under your app -> Configure -> Basic information -> App ID

![image](https://user-images.githubusercontent.com/103554011/217705137-53021988-7616-4781-8f2b-824b60e065cc.png)

If you are upgrading a project created by Teams Toolkit v4.x.x, the Teams app ID can be found in `.fx/states/state.{env}.json` under `fx-resource-appstudio.teamsAppId`

![image](https://user-images.githubusercontent.com/103554011/217706120-a2f7fc6a-d6e4-4950-a145-5d7f25b06590.png)

## Use your existing Azure Active Directory app ID

Teams Toolkit creates Azure Active Directory apps for projects with bot or single sign-on tab capabilities by default using a few actions:
* [`botAadApp/create`](https://aka.ms/teamsfx-actions/botaadapp-create)
* [`aadApp/create`](https://aka.ms/teamsfx-actions/aadapp-create)
* [`aadApp/update`](https://aka.ms/teamsfx-actions/aadapp-update)

However, Teams Toolkit supports using existing Azure Active Directory app IDs. <b>[Learn more about Teams Toolkit actions](https://aka.ms/teamsfx-actions)</b>

### Using existing Azure Active Directory app ID for bot

If you are developing with a Teams Toolkit scaffolded project, you can set `BOT_ID` and `SECRET_BOT_PASSWORD` in `.env` files listed under the env folder.

![image](https://user-images.githubusercontent.com/103554011/217710285-2ea82d7a-4985-4b88-94c4-64a075e7b9b0.png)

You can find your app's `BOT_ID` and `SECRET_BOT_PASSWORD` in [Azure Active Directory](https://ms.portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview).
* Select your app under "App registration"
* In the "Overview" page, `BOT_ID` is the value of "Application (client) ID"

![image](https://user-images.githubusercontent.com/103554011/217711207-7cb5e975-e319-4607-87e3-385e38e7e583.png)

* In the "Certificates & secrets" page, you can find the value for `SECRET_BOT_PASSWORD`

![image](https://user-images.githubusercontent.com/103554011/217711875-471a314d-a5ed-47c1-bf2e-07a8ef43e9e8.png)

If you are upgrading a project created by Teams Toolkit v4.x.x, `BOT_ID` and `SECRET_BOT_PASSWORD` can be found in `.fx/states/state.{env}.json` under `fx-resource-bot.botId` and `fx-resource-bot.botPassword`.

![image](https://user-images.githubusercontent.com/103554011/217713535-ac050428-31f8-4ac9-874c-199d41551283.png)

`{{fx-resource-bot.botPassword}}` here references `.fx/states/{env}.userdata`.

<img src='https://user-images.githubusercontent.com/103554011/217713759-627ba6cb-27ce-4af6-a257-625c53632bf2.png' width='300px' /><br />

 Note: if the value starts with `crypto_`, it is encrypted. Make sure to copy `projectId` from `.fx/configs/projectSettings.json` and paste it to the `projectId` field of `teamsapp.yml`. This will ensure Teams Toolkit can decrypt the secret correctly.

<img src='https://user-images.githubusercontent.com/103554011/218045248-a827a42e-5e52-4b22-8c52-33073b6bc086.png' width='300px' />

### Using existing Azure Active Directory app ID for SSO

If you are developing with a Teams Toolkit scaffolded project, you can set the following environment variables in `.env` files listed under the env folder:
* `AAD_APP_CLIENT_ID` - the client id of AAD app
* `AAD_APP_CLIENT_SECRET` - the client secret of AAD app
* `AAD_APP_OBJECT_ID` - the object id of AAD app
* `AAD_APP_TENANT_ID` - the tenant id of AAD app
* `AAD_APP_OAUTH_AUTHORITY_HOST` - the host of OAUTH authority of AAD app
* `AAD_APP_OAUTH_AUTHORITY` - the OAUTH authority of AAD app

You can find the values for these environment variables in [Azure Active Directory]:
* Select your app under "App registration"
* In the "Overview" page:
    * "Application (client) ID" maps to `AAD_APP_CLIENT_ID`
    * "Object ID" maps to `AAD_APP_OBJECT_ID`
    * "Directory (tenant) ID" maps to `AAD_APP_TENANT_ID`

![image](https://user-images.githubusercontent.com/103554011/217727282-b662f681-4f6b-41a9-ac29-6f6b879e215d.png)

* In the "Certificates & secrets" page, you can find the value for `SECRET_BOT_PASSWORD`

![image](https://user-images.githubusercontent.com/103554011/217728890-06a75d9f-cf5a-42f8-95bf-35ed889edc75.png)

### Upgrading a project created by Teams Toolkit v4.x.x, the variable values can be found in `.fx/states/state.{env}.json`

* `fx-resource-aad-app-for-teams.clientId` maps to `AAD_APP_CLIENT_ID`
* `fx-resource-aad-app-for-teams.clientSecret` () maps to `AAD_APP_CLIENT_SECRET`
* `fx-resource-aad-app-for-teams.objectId` maps to `AAD_APP_OBJECT_ID`
* `fx-resource-aad-app-for-teams.tenantId` maps to `AAD_APP_TENANT_ID`
* `fx-resource-aad-app-for-teams.oauthHost` maps to `AAD_APP_OAUTH_AUTHORITY_HOST`
* `fx-resource-aad-app-for-teams.oauthAuthority` maps to `AAD_APP_OAUTH_AUTHORITY`

![image](https://user-images.githubusercontent.com/103554011/217771277-6cc4404e-177c-4e58-9d92-8d90ecdfd33d.png)

## Customize Azure subscription ID and resource group

Teams Toolkit scaffolded projects leverage ARM templates for provisioning. You can use the subscription ID and resource group of your choice by setting `AZURE_SUBSCRIPTION_ID` and `AZURE_RESOURCE_GROUP_NAME` in the `.env` files in the env folder.

<img src='https://user-images.githubusercontent.com/103554011/217989311-8faf1319-f078-4f63-ab35-63d00da4e611.png' width='400px' />

`AZURE_SUBSCRIPTION_ID` and `AZURE_RESOURCE_GROUP_NAME` are referenced by the `arm/deploy` action in `teamsapp.yml`. <b>[Learn more about how to customize actions](https://aka.ms/teamsfx-actions)</b>

<img src='https://user-images.githubusercontent.com/103554011/217990688-4c4ee481-2020-4b44-9488-99d4756823c9.png' width='400px' />