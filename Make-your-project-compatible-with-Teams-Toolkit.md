Teams Toolkit enables developers to scaffold app projects that run in Teams, Outlook, and the Microsoft 365 app and includes several templates to get you started. In addition, compatible projects benefit from integrated tunneling for debugging bot capabilities, composable automation tasks for each step of the developer journey and launching the app to different organizations (tenants). In this article, you learn how to make your own application or existing Teams app projects compatible with Teams Toolkit.

## Understand the Teams Toolkit architecture
All Teams Toolkit templates have the same file structure, based on Teams Toolkit conventions. THe following hierarchy shows the directory structure you'll build in this tutorial:

```txt
├── .vscode                    [ Tasks for launching and debugging using VS Code ]
├── appPackage                 [ Teams app files: manifest.json, outline.png, and color.png ]
├── env                        [ Environment files used by Teams Toolkit ]
├── infra                      [ Creates and configures Azure resources ]
├── src                        [ Source code for your app ]
├── teamsapp.yaml              [ Defines automation tasks for hosted environments ]
└── teamsapp.local.yaml        [ Defines automation tasks for running from your machine or localhost]
```

Learn more about:
- [Teams Toolkit automation tasks in teamsapp.yml](https://github.com/OfficeDev/TeamsFx/wiki/Available-actions-in-Teams-Toolkit)
- [Teams Toolkit's teamsapp.yml schema](https://aka.ms/teams-toolkit/1.0.0/yaml.schema.json)

## Get the empty starter template
We've put all of the files necessary to use every feature of Teams Toolkit on GitHub so that it's easier to add them to your projects. Throughout this tutorial there are indications of where you may need to customize the files and which are optional. 

1. Download or clone the starter template from [GitHub](https://github.com/therealjohn/teams-toolkit-starter-template). This is the `template directory`. 

## Modify your project directory and copy template files
1. Navigate to the directory that has your app project in it. This is your `project directory`. 
1. Move your app source code either:
    - To the root of your project directory, or
    - In a subdirectory named `src`, or any other suitable name. This is your `source directory`.

### Add support to launch and debug your app
To launch your app in different hubs like Teams, Outlook, and the Microsoft 365 app, Teams Toolkit uses `launch.json` settings for VS Code. To orchestrate that launch, run dependency checks, setup tunneling, and other related tasks, there is a `tasks.json` file too. 

1. Copy the `.vscode` directory from the `template directory` to the root of your `project directory`.
2. Modify the `Start application` task in `tasks.json` to run the appropriate script from your `package.json`. By default, it is `npm run dev:teamsfx`. More info about `dev:teamsfx` is in the [using env variables in code](#using-env-variables-in-code) section below.

### Add support for Teams Toolkit automation tasks
Teams Toolkit includes features that help automate repetitive or tedious setup with composable tasks, or actions, tuned for operations in Teams Platform and Azure. The `template directory` contains an example `teamsapp.local.yml` and `teamsapp.yml` file where these tasks are expressed and understood by Teams Toolkit. 

The `teamsapp.local.yml` file is used to differentiate any automation and setup you need to develop on your own machine like using a different App ID, Bot ID, etc. This is your `local environment`. Note that some necessary resources on Teams Platform are never truly "local", like a Bot Framework or Bot Service registration. Resources like this are still in the cloud, but you have the flexibility to use different configurations between environments with Teams Toolkit.

The `teamsapp.yml` file is used to configure automation for other environments, typically hosted remotely like in Azure. These are your `remote environments`. i.e. `dev`, `test`, `qa`, `acceptance`, `production`

1. Copy the `teamsapp.local.yml` and `teamsapp.yml` files from the `template directory` to the root of your `project directory`.

> If these files are not in the root of the folder you open in VS Code or call `teamsfx-cli` commands from, you will get an error that the directory or project is not recognized by Teams Toolkit. 

The automation tasks rely on several other features included with Teams Toolkit like environment files and variable expansion. For example, in the `teamsapp.local.yml` 

### Add support for Teams Toolkit environments
Teams Toolkit helps you run your app in different environments so that you can develop and test your app in different ways. To reduce the need for duplicating app manifests and manually coordinating the setup, add support to your project using environment files and variable expansion in your manifest.

1. Copy the `env` directory from the `template directory` to the root of your `project directory`. 

Teams Toolkit environments groups of configuration using .env files and variables. If you already have existing resources like a Teams App ID, Azure AD App ID, etc. then make sure to review [using your existing Teams app resources]().

### Add support for hosting in Azure
Teams Toolkit includes (optional) features to help create (provision) Azure resources to host your apps infrastructure and deploy the code to those resources. This is useful for small teams with an Azure subscription where all of that DevOps can be done from VS Code or the CLI. For larger teams or to meet other requirements, the CLI could be used with these features in combination with CI/CD. 

The `template directory` includes smart defaults for Teams apps using Bicep that help you create the right resources in Azure to host the app. The templates ensure idempotence of the resources. 

These steps are optional. Follow them if you want to use Teams Toolkit to help host your app in Azure.

1. Copy the `infra` directory from the `template directory` to the root of your `project directory`. 
1. For apps with tabs, copy the `.storageignore` and `aad.manifest.json` files from the `template directory` to the root of your `project directory`.
1. For apps with a bot, copy the `.appserviceignore` and `web.config` files from the `template directory` to the root of your `project directory`.

Now your project directory supports Teams Toolkit, but you'll need to customize these files to better match any existing resources or differences from the Teams Toolkit conventions.

## Update the app manifest
Teams Toolkit supports variable expansion in the app manifest which means you can use a single `manifest.json` file across many environments. The YAML files have tasks that automate setup and output any generated values to the env files in `./env` by default. You can reference those env variables using the `${{VARIABLE_NAME}}` syntax in your `manifest.json`. For example, here is the task to automate creating a new App ID and saving it's value to the `TEAMS_APP_ID` variable which is then referenced in the `manifest.json`.

teamsapp.local.yml:
```yaml
  - uses: teamsApp/create
    with:
      name: MyAppName-${{TEAMSFX_ENV}}
    writeToEnvironmentFile: 
      teamsAppId: TEAMS_APP_ID
```

manifest.json:
```json
...
"id": "${{TEAMS_APP_ID}}",
...
```

By default, this action is in the `provision` step which will run when you press F5 or Start Debugging in VS Code. Alternately, you can run this using the CLI with `teamsfx provision --env local`. The variable will be replaced with the value from the `./env/.env.local` file.

Where applicable, you can continue updating the manifest to use the matching environment variables. i.e.
- `BOT_ID`
- `TAB_ENDPOINT`
- etc.

## Using existing resources like an App ID
If you have existing resources like a Teams App ID, Azure AD Client ID, etc. then you can update the corresponding environments `.env` file with those values. This makes sure that the variable expansion still works, but that the automation tasks don't re-create new resources. You can also remove automation tasks from `teamsapp.yml` or `teamsapp.local.yml` for the operations you don't need like like `teamsApp/create`. 

> Note: It can be helpful for future developers who join the project to create their own resources for local development. For that reasons, we recommend leaving the defaults in `teamsapp.local.yml` so that the right resources are created to run the app.

## Using env variables in code
Some projects rely on reading env variables in code using techniques like `dotenv` or `process.env`. By default, the env variables used by Teams Toolkit are not given to the shell, or runtime of your app. You can automate this using the `file/createOrUpdateEnvironmentFile` action in the YAML files. For example, `teamsapp.local.yml` contains this action in the `deploy` step:

```yaml
  - uses: file/createOrUpdateEnvironmentFile
    with:
      target: ./.localConfigs
      envs:
        BOT_ID: ${{BOT_ID}}
        BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
```

When `deploy` is run during F5 or Start Debugging in VS Code, `BOT_ID` and `SECRET_BOT_PASSWORD` are written to a `./.localConfigs` file. Teams Toolkit project templates use the `env-cmd` package to provide those values to `node` in the `package.json` file like this:

```json
"scripts": {
    "dev:teamsfx": "env-cmd --silent -f .localConfigs npm run dev",
    ...
},
```

Alternatively, you could use `dotenv` by changing the `file/createOrUpdateEnvironmentFile` to write to a `./.env` file. The call `require('dotenv').config()` in your code. 

## Update your .gitignore file
If you're using Git, it's recommended to add some of these files to your .gitignore file. 

1. View the `.gitignore` in the `template directory` and copy the content.
1. Edit your .gitignore and paste the content.