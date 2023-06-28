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
Create a new app with Teams Toolkit with SPFx tab capability 
> You can use SPFx solution name as app name to create the new app.

## Update the app manifest
- Delete the created SPFx solution under `./src` folder.
- Removing and update `staticTabs` and `configurableTabs` fields for both manifest.json and manifest.local.json under `./appPackage` directory.

  For each of the web parts in your existing SPFx solution, use `id` and `preconfiguredEntries[0].title.default` in web part manifest as `componentId` and `webpartName` to construct the Teams manifest:

**manifest.json**

Add following item to `staticTabs` array for each of the web parts:

```
        {
            "entityId": "`{{componentId}}`",
            "name": "`{{webpartName}}`",
            "contentUrl": "https://{teamSiteDomain}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest=/_layouts/15/teamshostedapp.aspx%3Fteams%26personal%26componentId=`{{componentId}}`%26forceLocale={locale}", 
            "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
            "scopes": [
                "personal"
            ]
        }
```

Add following item to `configurableTabs` array for one of the web parts:

```
        {
            "configurationUrl": "https://{teamSiteDomain}{teamSitePath}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest={teamSitePath}/_layouts/15/teamshostedapp.aspx%3FopenPropertyPane=true%26teams%26componentId=`{{componentId}}`%26forceLocale={locale}", 
            "canUpdateConfiguration": true,
            "scopes": [
                "team"
            ]
        }
```

**manifest.local.json**

Add following item to `staticTabs` array for each of the web parts:
```
        {
            "entityId": "`{{componentId}}`",
            "name": "`{{webpartName}}`",
            "contentUrl": "https://{teamSiteDomain}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest={teamSitePath}/_layouts/15/TeamsWorkBench.aspx%3FcomponentId=`{{componentId}}`%26teams%26personal%26forceLocale={locale}%26loadSPFX%3Dtrue%26debugManifestsFile%3Dhttps%3A%2F%2Flocalhost%3A4321%2Ftemp%2Fmanifests.js", 
            "websiteUrl": "https://products.office.com/en-us/sharepoint/collaboration",
            "scopes": [
                "personal"
            ]
        }
```

Add following item to `configurableTabs` array for one of the web parts:

```
        {
            "configurationUrl": "https://{teamSiteDomain}{teamSitePath}/_layouts/15/TeamsLogon.aspx?SPFX=true&dest={teamSitePath}/_layouts/15/TeamsWorkBench.aspx%3FcomponentId=`{{componentId}}`%26openPropertyPane=true%26teams%26forceLocale={locale}%26loadSPFX%3Dtrue%26debugManifestsFile%3Dhttps%3A%2F%2Flocalhost%3A4321%2Ftemp%2Fmanifests.js", 
            "canUpdateConfiguration": true,
            "scopes": [
                "team"
            ]
        }
```

 - If you already have an existing Teams manifest, you can use your existing one and replace the default ones. But make sure to update the following fields so that your manifest can be compatible with Teams Toolkit:

`$schema` and `manifestVersion`: Use the version specified in default template since we already enabled MetaOS support for SPFx in debug configurations.

`id`: Use `"id": "${{TEAMS_APP_ID}}"` and write existing id to `env.dev` file.

`icons`: Use the value in default template:
```
"icons": {
        "color": "color.png",
        "outline": "outline.png"
    }
```

`staticTabs`: Use value as mentioned above.

`configurableTabs`: Use value as mentioned above.

## Update the app package icons

If color.png and outline.png already exists under `./teams` folder in your SPFx solution, copy and paste to `./appPackage` folder in generated TeamsFx project:

![image](https://github.com/OfficeDev/TeamsFx/assets/73154171/13c5908e-97d9-44d3-b06b-7310b28beb1b)

## Update env file

Teams Toolkit helps you run your app in different environments so that you can develop and test your app in different ways. To reduce the need for duplicating app manifests and manually coordinating the setup, add support to your project using environment files and variable expansion in your manifest.

If you already have an existing Teams manifest in your SPFx solution, set the Teams app id (`id` in Teams manifest file) to `TEAMS_APP_ID` environment variable in `env.dev` file.
