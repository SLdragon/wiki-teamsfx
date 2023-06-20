Teams Toolkit enables developers to scaffold app projects that run in Teams, Outlook, and the Microsoft 365 app and includes several templates to get you started. In addition, compatible projects benefit from integrated tunneling for debugging bot capabilities, composable automation tasks for each step of the developer journey and launching the app to different organizations (tenants). In this article, you learn how to make your existing SPFx solution compatible with Teams Toolkit.

## Understand the Teams Toolkit architecture
All SPFx project templates in Teams Toolkit have the same file structure, based on Teams Toolkit conventions. The following hierarchy shows the directory structure you'll build in this tutorial:

```txt
├── .vscode                    [ Tasks for launching and debugging using VS Code ]
├── appPackage                 [ Teams app files: manifest.json, manifest.local.json, outline.png, and color.png ]
├── env                        [ Environment files used by Teams Toolkit ]
├── src                        [ Source code of your SPFx solution ]
├── teamsapp.yml              [ Defines automation tasks for hosted environments ]
└── teamsapp.local.yml        [ Defines automation tasks for running from your machine or localhost]
```

Learn more about:
- [Teams Toolkit automation tasks in teamsapp.yml](https://github.com/OfficeDev/TeamsFx/wiki/Available-actions-in-Teams-Toolkit)
- [Teams Toolkit's teamsapp.yml schema](https://aka.ms/teams-toolkit/1.0.0/yaml.schema.json)

## Set up
Create a new app with Teams Toolkit with SPFx tab capability (You can use SPFx solution name as app name to create the new app).

## Update the app manifest
- Delete the SPFx solution under `./src` folder.
- Update `staticTabs` and `configurableTabs` fields for both manifest.json and manifest.local.json under `./appPackage` directory:
For each of the web parts in your SPFx solution, use `manifest["id"]` as 

Move your existing Teams manifest into the new project, replacing the default ones. Teams Toolkit scaffolded projects save these files in the appPackage folder. If you choose to move these files to a different location, make sure you update [actions](https://aka.ms/teamsfx-actions) in teamsapp.local.yml that reference them.

Update your manifest file. You can use ${{MY_VARIABLE_NAME}} as placeholders in manifest.json, Teams Toolkit will replace them with their corresponding value at runtime. For a bot project, you usually need to use placeholders for id, name.short, and bots[].botId. Make sure you save the values into an environment file.

### Add support for Teams Toolkit environments
Teams Toolkit helps you run your app in different environments so that you can develop and test your app in different ways. To reduce the need for duplicating app manifests and manually coordinating the setup, add support to your project using environment files and variable expansion in your manifest.

1. Copy the `env` directory from the `template directory` to the root of your `project directory`. 

Teams Toolkit environments groups of configuration using .env files and variables. If you already have existing resources like a Teams App ID, Azure AD App ID, etc. then make sure to review [using your existing Teams app resources]().


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