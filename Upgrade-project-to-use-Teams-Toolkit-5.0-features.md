Teams Toolkit continuously evolves to offer more powerful features for developers. Teams Toolkit 5.0 provides following new features to help you better build your existing Teams app project using Teams Toolkit:
* Use an existing Teams app ID
* Use an existing Azure AD app registration ID
* Use a different tunneling solution or customize the defaults
* Use existing infrastructure, resource groups, and more when provisioning
* Add custom steps to debugging, provisioning, deploying, publishing, etc.

These new features will require an update to your existing project structure. Teams Toolkit can automatically upgrade your project with your consent.

> Please note that the new features will be enabled by default in the future release of Teams Toolkit. We thrive to make Teams Toolkit compatible with existing projects but long-term backwards compatibility is not 100% guaranteed, thus we strongly recommend updating your project configurations to continue using the latest Teams Toolkit.

## FAQ about the upgrade
### Q: Will the upgrade impact my source code?

A: No. The upgrade only changes the files used by Teams Toolkit. You can refer [File Structure Change](#file-structure-change) to learn more about the possible changes.

### Q: Can I go back to previous project?

A: Yes. The upgrade will backup changed files in your project. You can refer [How to roll back](#how-to-roll-back) to roll back the changes.

### Q: Can I postpone the upgrade?

A: If you do not want to upgrade temporary, you can use VS Code Teams Toolkit 4.X / TeamsFx CLI 1.X / Visual Studio 2022 17.4 to develop your projects.

## How to Upgrade

Teams Toolkit can automatically upgrade your project with your consent. To trigger upgrade, you can:
1. Trigger any Teams Toolkit command, Teams Toolkit will pop up dialog to confirm upgrade.
    > You need to trigger the expected command again after upgrade.
2. Open VS Code command palette (Shift + Command + P (Mac) / Ctrl + Shift + P (Windows/Linux)) and run `Teams: Upgrade project` command.
3. Click Teams Toolkit sidebar in VS Code, and click `Upgrade project` button.

> We recommend using git to track changes during upgrade.

## File Structure Change

Teams Toolkit performs the following changes to your project during upgrade.

### File changes

1. Created `teamsapp.yml` and `teamsapp.local.yml` in your project's root folder
    > These file contains lifecycle configuration for your project
2. Moved environment files in `.fx` to `.env.{env}` in `env` folder
    > The environment files in old project contains `state.{env}.json`, `config.{env}.json`
3. If your project contains file `.fx/states/{env}.userdata`, the content will be moved to `.env.{env}.user` in `env` folder
4. Moved `templates/appPackage` to `appPackage` and updated placeholders in it per latest tooling's requirement.
    > This step also renamed `appPackage/manifest.template.json` to `appPackage/manifest.json`
    > The tooling now uses `${{ENV_VAR_NAME}}` to reference environment variables and all old placeholders have been renamed to environment variable
5. If your project contains `templates/appPackage/aad.template.json`, moved it to `aad.manifest.json` and updated placeholders in it per latest tooling's requirement.
    > The tooling now uses `${{ENV_VAR_NAME}}` to reference environment variables and all old placeholders have been renamed to environment variable
6. If your project contains file `.vscode/tasks.json` and `.vscode/launch.json`, they will be updated.
7. Updated `.gitignore` to ignore new environment files.
8. Removed `.fx` folder
    > Content under this folder is no longer used.

> All existing files will be moved into `.backup` folder for your reference. You can safely delete the `.backup` folder after you reviewed the changes.

## How to roll back

If you still want to restore your project configuration after the upgrade is successful and continue to use old version Teams Toolkit, you can:
1. Copy every folder / file under `.backup` folder to your project root path
    > You can safely overwrite any existing files if you didn't change the files after upgrade.
2. Remove the new folders / files mentioned in [File Changes](#file-changes) section:
    * teamsapp.yml and teamsapp.local.yml
    * aad.manifest.json
    * env folder
    * appPackage folder

## Known issues

### STATE__FX_RESOURCE_FRONTEND_HOSTING__ENDPOINT missing error in some projects
If your project only contains a bot, an error might occur saying `STATE__FX_RESOURCE_FRONTEND_HOSTING__ENDPOINT` is missing when executing commands. Replace this placeholder with a valid URL in `appPackage/manifest.json` to fix it.
> This happens in projects created using very old Teams Toolkit. The old tooling provides a default example URL for your project if this placeholder does not exist. In latest tooling, we want to make everything more transparent so requires you to provide your URL here.

### InvalidParameter: Following parameter is missing or invalid for aadApp/create action: name
You may be trying to upgrade a project created by Teams Toolkit for Visual Studio Code v3.x / Teams Toolkit CLI v0.x / Teams Toolkit for Visual Studio v17.3. Please install Teams Toolkit for Visual Studio Code v4.x / Teams Toolkit CLI v1.x / Teams Toolkit for Visual Studio v17.4 and run upgrade first.

### SimpleAuthEndpoint in configuration is invalid
If your tab app is created with Teams Toolkit 3.2.0 or earlier version, you may see error `simpleAuthEndpoint in configuration is invalid` when remote debugging your app.

To fix this: Open `teamsapp.yml`, find the npm build command in deploy lifecycle and add an additional env `REACT_APP_TEAMSFX_ENDPOINT` as below:
```
  - uses: cli/runNpmCommand
    env:
      REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
      REACT_APP_START_LOGIN_PAGE_URL: ${{PROVISIONOUTPUT__FRONTENDHOSTINGOUTPUT__ENDPOINT}}/auth-start.html
      REACT_APP_TEAMSFX_ENDPOINT: ${{PROVISIONOUTPUT__SIMPLEAUTHOUTPUT__ENDPOINT}}
    with:
      workingDirectory: tabs
      args: run build --if-present
```

Since simple auth is deprecated, refer to [wiki](https://github.com/OfficeDev/TeamsFx/wiki/How-Authentication-Works-in-TeamsFx) for best practices and iterate your application to use the latest TeamsFx SDK.

### teamsApp/validateAppPackage failed error
If your project failed to provision or publish by `teamsApp/validateAppPackage` action after upgrade but had provisioned successfully before, please use the [validation](https://dev.teams.microsoft.com/validation) to check your appPackage zip file and fix the error.

### Start local service error
If you've encountered the error message "Teams Toolkit now uses Dev Tunnel as default tunnel solution. For manual updates, see https://aka.ms/teamsfx-tasks/local-tunnel.", you'll need to update your `Start local tunnel` task to continue using Teams Toolkit. Please follow the instructions in this [document](#start-tunnel-service) to update the task.

## Feature changes that impact your development flow

There're some changes to existing features you should be aware of:

### Environment management

1. All the user environment files `.env.{env_name}.user` will be gitignored by default. You need to sync the environment variables in `.env.{env_name}.user` files under `env` folder by yourselves to other machines to operate corresponding environments.
2. When creating new environments(call `Creating New Environment` in command palette in VS Code), you need to fill customized fields in the new `.env.{env_name}` file. Usually you need to provide values for all environment values with `CONFIG__` prefix. 
3. When creating new secret environments, you need to fill customized fields in the new `.env.{env_name}.user` file. Usually you need to provide values for all environment values with `CONFIG__` prefix.
4. When creating new environments, you need to manually create `templates/azure/azure.parameters.{env_name}.json` as Azure provision parameters and fill the parameter values accordingly.
5. Some additional steps are required if you added SQL or APIM to your project. Refer [Provision SQL databases](#provision-sql-databases) and [Provision APIM service](#provision-apim-service) section to learn more.

### Launch your app

1. When launching your app for a remote environment in VS Code, Teams Toolkit will no longer ask you to select an environment. You need to manually change environment name for `${dev:teamsAppId}` in `.vscode/launch.json` to launch your app for a certain environment.

### Provision SQL databases

If you didn't use Teams Toolkit to add SQL databases to your project, these changes won't impact your project.

1. When you provision a new environment, you need to provide values for `STATE__FX_RESOURCE_AZURE_SQL__ADMIN` and `SECRET_FX_RESOURCE_AZURE_SQL__ADMINPASSWORD` in `.env.{env_name}.user` which are required inputs for creating SQL databases.
    > If you're provisioning an existing environment, you don't need this step.
2. You need to grant permission to user assigned identity manually after provisioning a new environment. Here're the steps:
   1. Go to `env/.env.{envName}` and find SQL Server resource id. The resource id usually saved in `PROVISIONOUTPUT__AZURESQLOUTPUT__SQLRESOURCEID` environment variable and has this pattern: `/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.Sql/servers/{sql_server_name}`. Record the resource group name and SQL Server name, they will be used later.
   2. Go to `env/.env.{envName}` and find SQL Server database name. The database name usually saved in `PROVISIONOUTPUT__AZURESQLOUTPUT__DATABASENAME` environment variable.
   3. Go to `env/.env.{envName}` and find User Assigned Identity resource id. The resource id usually saved in `PROVISIONOUTPUT__IDENTITYOUTPUT__IDENTITYRESOURCEID` environment variable and has this pattern `/subscriptions/{subscription_id}/resourceGroups/{resource_group_name}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{user_assigned_identity_name}`. Record the User Assigned Identity name, it will be used later.
   4. Configure AAD admin in SQL Database. You can follow [set AAD admin](https://docs.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-configure?tabs=azure-powershell#provision-azure-ad-admin-sql-database) to set AAD admin for the SQL Server found in step i. Usually you can use the **account logged-in Azure** as AAD admin.
   5. Go to your SQL Database found in step ii, login using the AAD admin
![image](https://user-images.githubusercontent.com/16605901/217272776-36aeb5ba-ceb7-4922-ae22-c9553f3bb501.png)

   6. Create contained database users. Replace `{user_assigned_identity_name}` with the value from step iii in following Transact-SQL and execute:
      ```
      CREATE USER [{user_assigned_identity_name}] FROM EXTERNAL PROVIDER;
      go
      sp_addrolemember  'db_datareader',  '{user_assigned_identity_name}';
      go
      sp_addrolemember  'db_datawriter',  '{user_assigned_identity_name}';
      go
      ```

   7. If there are multiple databased in your project, add User Assigned Identity to all of them.

### Provision APIM service

If you didn't use Teams Toolkit to add API Management service to your project, these changes won't impact your project.

1. When you provision an environment, you need to provide values for `APIM__PUBLISHEREMAIL` and `APIM__PUBLISHERNAME` in `.env.{env_name}` which are required inputs for creating or updating APIM services.
2. You need to manually create an Azure Active Directory app for APIM service when provisioning a new environment. This [document](https://aka.ms/teamsfx-add-azure-apim) includes tutorials about how to create an Azure Active Directory app for APIM service.
3. Teams Toolkit no longer support deploy API spec to APIM any more.
Teams Toolkit will reuse your provisioned resource when upgrading (except Bot Service), when you wish to add a new environment after project upgrading, please remember to change resource name in `azure.parameters.{your_env_name}.json` to avoid name conflicts. 

### Start tunnel service
The Teams Toolkit now uses Dev Tunnel as its default tunnel solution. If you have customized the Start local tunnel task in your `.vscode/tasks.json` file, you will need to manually update it. Here are the steps to follow:
1. Refer to this [document](https://aka.ms/teamsfx-tasks/local-tunnel) to learn how to use Teams Toolkit dev tunnel service.
2. If you prefer to use ngrok instead, you can still do so by following this [document](https://aka.ms/teamsfx-tasks/customize-tunnel-service
) to set up.
By updating your local tunnel service, you can continue to use Teams Toolkit seamlessly and customize it according to your preferences.


## Upgrade your projects manually


This section helps you understand how to upgrade the old project template (created by Teams Toolkit 4.X) to latest structure. If you have customized the project after creation, you can make additional changes during the upgrade steps for your customized part.

### Overview

In general, the upgrade process contains 2 steps:

1. Create a new project using latest Teams Toolkit and copy your source code to it
   > Refer [Step 1: create a new project and copy your source code to it](#step-1-create-a-new-project-and-copy-your-source-code-to-it) for detailed steps

2. Transform the placeholders used by old Teams Toolkit to new format
   > Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) for detailed steps

### Step 1: create a new project and copy your source code to it

Based on the Teams capabilities in your project, choose the guide that suitable to your project:
* If your project only contains 1 capability (tab, bot or message extension), follow the steps in [Project with single Teams capability that hosted on Azure](#project-with-single-teams-capability-that-hosted-on-azure)
* If your project contains multiple capabilities (tab, bot, message extension) or contains a backend api, follow the steps in [Project with multiple Teams capability that hosted on Azure](#project-with-multiple-teams-capability-that-hosted-on-azure)
* If your project is a SPFx tab, follow the steps in [SPFx tab project](#spfx-tab-project)

#### Project with single Teams capability that hosted on Azure

The steps in this section is prepared for project that contains only 1 capability (tab, bot or message extension).

1. Trigger "Teams: Create a new app" command using Teams Toolkit 5.0.0 to create a new project with same capability or same scenario as your current project.

2. Remove source code in the new project and copy your current project's source code to the new project

    1. If your project is a tab app, copy everything under `tab` folder to the new project's root folder

    2. If your project is a bot app or message extension app, copy everything under `bot` folder to the new project's root folder

3. Find project id under `.fx/configs/projectSettings.json`, put the value to the new project's `teamsapp.yml`'s `projectId` property

4. Copy your ARM template to the new project

    1. Remove everything under the new project's `infra` folder. 

    2. Copy everything under `templates/azure` in your current project to the new project's `infra` folder. 

    3. Rename `main.bicep` to `azure.bicep`.

    4. Copy `.fx/configs/azure.parameters.{env}.json` to the new project's `infra` folder.

    5. Update the new project's `teamsapp.yml` file, change `parameters: ./infra/azure.parameters.json` to `parameters: ./infra/azure.parameters.${{TEAMSFX_ENV}}.json`

5. Copy your Teams manifest to the new project

    1. Remove everything under the new project's `appPackage` folder

    2. Copy everything except `aad.template.json` (if have) under your existing project's `templates/appPackage` folder to `appPackage` in your new project

    3. Rename `appPackage/manifest.template.json` to `appPackage/manifest.json`.

6. If your existing project contains `templates/appPackage/aad.template.json`, copy the content of this file to `aad.manifest.json` (under root of the project folder) in your new project.

7. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `aad.manifest.json` (if have) and `infra/azure.parameters.{env}.json`.

#### Project with multiple Teams capability that hosted on Azure

The steps in this section is prepared for project that contains multiple 1 capabilities (tab, bot or message extension), or contains a backend api.

1. Download sample project based on your project's capabilities
    | Capabilities | Sample project location |
    | --- | --- |
    | tab, api, SSO | [sample project](https://github.com/OfficeDev/TeamsFx/tree/chyuan/add-manual-upgrad-sample-project/docs/vscode-extension/5.0-multi-capability-sample/tab-and-api-with-sso) |
    | tab, bot / message extension  | [sample project](https://github.com/OfficeDev/TeamsFx/tree/chyuan/add-manual-upgrad-sample-project/docs/vscode-extension/5.0-multi-capability-sample/tab-and-bot) |
    | tab, bot / message extension, SSO | [sample project](https://github.com/OfficeDev/TeamsFx/tree/chyuan/add-manual-upgrad-sample-project/docs/vscode-extension/5.0-multi-capability-sample/tab-and-bot-with-sso) |
    | tab, api, bot / message extension, SSO | [sample project](https://github.com/OfficeDev/TeamsFx/tree/chyuan/add-manual-upgrad-sample-project/docs/vscode-extension/5.0-multi-capability-sample/tab-api-and-bot-with-sso) |

2. Copy your project's source code to the new project
    1. If your project contains a tab, copy everything under `tab` folder to the new project's `tab` folder
    2. If your project contains api, copy everything under `api` folder to the new project's `api` folder
    3. If your project contains a bot or/and message extension, copy everything under `bot` folder to the new project's `bot` folder

3. Find project id under `.fx/configs/projectSettings.json`, put the value to the new project's `teamsapp.yml`'s `projectId` property

4. Copy your ARM template to the new project

    1. Remove everything under the new project's `infra` folder. 

    2. Copy everything under `templates/azure` in your current project to the new project's `infra` folder. 

    3. Rename `main.bicep` to `azure.bicep`.

    4. Copy `.fx/configs/azure.parameters.{env}.json` to the new project's `infra` folder.

5. Copy your Teams manifest to the new project

    1. Remove everything under the new project's `appPackage` folder

    2. Copy everything except `aad.template.json` (if have) under your existing project's `templates/appPackage` folder to `appPackage` in your new project
    
    3. Rename `appPackage/manifest.template.json` to `appPackage/manifest.json`.

6. If your existing project contains `templates/appPackage/aad.template.json`, copy the content of this file to `aad.manifest.json` (under root of the project folder) in your new project.

7. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `aad.manifest.json` (if have) and `infra/azure.parameters.{env}.json`.

#### SPFx tab project

The steps in this section is prepared SPFx tab project.

1. Trigger "Teams: Create a new app" command using Teams Toolkit 5.0.0 to create a new SPFx tab project

2. Copy everything under your project's `SPFx` folder to the new project's root folder

3. Find project id under `.fx/configs/projectSettings.json`, put the value to the new project's `teamsapp.yml`'s `projectId` property

4. Copy your Teams manifest to the new project

    1. Copy your existing project's `templates/appPackage/resources` folder to `appPackage` in your new project

    2. Record the values of `contentUrl` and `configurationUrl` in the new project's `appPackage/manifest.json` file and `appPackage/manifest.local.json` file

    3. Copy content of your existing project's `template/appPackage/manifest.template.json` file to the new project's `appPackage/manifest.json` and `appPackage/manifest.local.json`

    4. Copy the value of `contentUrl` and `configurationUrl` recorded in step 2 back to `appPackage/manifest.json` and `appPackage/manifest.local.json`

5. Refer [Step 2: transform the placeholders to new format](#step-2-transform-the-placeholders-to-new-format) to update placeholders in `appPackage/manifest.json`, `appPackage/manifest.local.json`.


### Step 2: transform the placeholders to new format

The latest Teams Toolkit can resolve environment variables in your manifest / parameter files, which can be easily integrated with different engineering system. This requires you to update the placeholders in your existing manifest / parameter files. 

#### Overview
For every manifest or parameter file, you need to do following things. This is just an overview, you can find more details in [Steps to update the old placeholders](#steps-to-update-the-old-placeholders). If you're not familiar with 

1. Add new environment variables to `env/.env.{env}` files. Each environment variable represents an old placeholder in your project.
    > If `.fx/configs/config.{env}.json` or `.fx/states/state.{env}.json` contains value for the old placeholder, you also need to copy the value to corresponding environment variable in `env/.env.{env}` files.

    > If `.fx/states/{env}.userdata` contains value for the old placeholder, you also need to copy the value to corresponding environment variable in `env/.env.{env}.user` files.
2. Replace the old placeholders `{{xxx}}` and `{{{xxx}}}` in your manifest / parameter files with new format `${{xxx}}`. The name used in the new placeholder should be same with the environment variable name you added in above step.

#### Steps to update the old placeholders

1. Replace `{{config.xxx}}` placeholders in `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json` files.
    1. Copy following sample snippet to `env/.env.{env}`.
        ```
        COLOR_ICON=resources/color.png
        OUTLINE_ICON=resources/outline.png
        DESCRIPTION_SHORT=Short description of myapp
        DESCRIPTION_FULL=Full description of myapp
        APP_NAME_SHORT=myapp
        APP_NAME_FULL=Full name for myapp
        ```
        > You need to change the values to the actual one for your project. The values can be found in `.fx/configs/config.{env}.json`. You can also change the environment variable names as you want.
        > Your project may not use all the placeholders in the sample, you can delete the unused one.

    2. Open `manifest.json`, `aad.manifest.json` and `azure.parameters.{env}.json`, replace every old placeholder in below table to the new one.
        | Old placeholder | New placeholder |
        | --- | --- |
        | {{config.manifest.icons.color}} | ${{COLOR_ICON}} |
        | {{config.manifest.icons.outline}} | ${{OUTLINE_ICON}} |
        | {{config.manifest.appName.short}} | ${{DESCRIPTION_SHORT}} |
        | {{config.manifest.appName.full}} | ${{DESCRIPTION_FULL}} |
        | {{config.manifest.description.short}} | ${{APP_NAME_SHORT}} |
        | {{config.manifest.description.full}} | ${{APP_NAME_FULL}} |
        
        > If your project uses additional `{{config.xxx}}` placeholders, you can refer the sample to add additional environment variables.

2. Replace `{{{state.fx-resource-aad-app-for-teams.applicationIdUris}}}` placeholder in `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json` files.
    * If you do not use any SSO functionality and your project does not require an AAD application, you can remove following redundant things in your project
        1. Remove `aad.manifest.json` file
        2. Remove `aadApp/create` and `aadApp/update` actions in `teamsapp.yml` and `teamsapp.local.yml`
        3. Remove `webApplicationInfo` property in `appPackage/manifest.json`
    * If your project only enables SSO for tab, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/${{AAD_APP_CLIENT_ID}}`
    * If your project only enables SSO for bot or message extension, convert it to `api://botid-${{BOT_ID}}`
    * If your project enables SSO for both and bot / message extension, convert it to `api://{{state.fx-resource-frontend-hosting.domain}}/botid-${{BOT_ID}}`

    > The `{{state.fx-resource-frontend-hosting.domain}}` with be replaced again in later step, don't worry on having this placeholder temporary

3. Replace following placeholders to the new one in `aad.manifest.json` file.
    | Old placeholder | New placeholder |
    | --- | --- |
    | {{state.fx-resource-aad-app-for-teams.frontendEndpoint}} | {{state.fx-resource-frontend-hosting.endpoint}} |
    | {{state.fx-resource-aad-app-for-teams.botEndpoint}} | {{state.fx-resource-bot.siteEndpoint}} |

    > The new placeholder will be replaced again in later step, don't worry on having this placeholder temporary

4. Replace `{{state.xxx}}` placeholders in `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json` files.
    1. Copy following sample snippet to `env/.env.{env}` and update your manifest / parameter files to reference the environment variables in the samples.
        ```
        TEAMS_APP_TENANT_ID=00000000-0000-0000-0000-000000000000 # use ${{TEAMS_APP_TENANT_ID}} to replace the old placeholder {{state.fx-resource-appstudio.tenantId}}
        TEAMS_APP_ID=00000000-0000-0000-0000-000000000000 # use ${{TEAMS_APP_ID}} to replace the old placeholder {{state.fx-resource-appstudio.teamsAppId}}
        AAD_APP_CLIENT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_CLIENT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.clientId}} or {{state.aad-app.clientId}}
        SECRET_AAD_APP_CLIENT_SECRET=your-aad-app-secret # use ${{SECRET_AAD_APP_CLIENT_SECRET}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.clientSecret}} or {{state.aad-app.clientSecret}}
        AAD_APP_OBJECT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_OBJECT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.objectId}} or {{state.aad-app.objectId}}
        AAD_APP_ACCESS_AS_USER_PERMISSION_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauth2PermissionScopeId}} or {{state.aad-app.oauth2PermissionScopeId}}
        AAD_APP_TENANT_ID=00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_TENANT_ID}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.tenantId}} or {{state.aad-app.tenantId}}
        AAD_APP_OAUTH_AUTHORITY_HOST=https://login.microsoftonline.com # use ${{AAD_APP_OAUTH_AUTHORITY_HOST}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauthHost}} or {{state.aad-app.oauthHost}}
        AAD_APP_OAUTH_AUTHORITY=https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000 # use ${{AAD_APP_OAUTH_AUTHORITY}} to replace the old placeholder {{state.fx-resource-aad-app-for-teams.oauthAuthority}} or {{state.aad-app.oauthAuthority}}
        BOT_ID=00000000-0000-0000-0000-000000000000 # use ${{BOT_ID}} to replace the old placeholder {{state.fx-resource-bot.botId}} or {{state.teams-bot.botId}}
        SECRET_BOT_PASSWORD=your-aad-app-secret # use ${{SECRET_BOT_PASSWORD}} to replace the old placeholder {{state.fx-resource-bot.botPassword}} or {{state.teams-bot.botPassword}}
        ```
        > You need to change the values to the actual one for your project. The values can be found in `.fx/states/state.{env}.json`. If there is no `state.{env}.json` file or `state.{env}.json` does not contain a property for the placeholder, leave the environment variable value empty.

        > Value of `SECRET_AAD_APP_CLIENT_SECRET` and `SECRET_BOT_PASSWORD` can be found in `.fx/states/{env}.userdata`. You can copy the encrypted value to new place directly.

        > You should not change the environment variable names in above sample.

        > Your project may not use all the placeholders in the sample, you can delete the unused one.

    2. Open `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json`, replace every old placeholder in below table to the new one.
        | Old placeholder | New placeholder |
        | --- | --- |
        | {{state.fx-resource-appstudio.tenantId}} | ${{TEAMS_APP_TENANT_ID}} |
        | {{state.fx-resource-appstudio.teamsAppId}} | ${{TEAMS_APP_ID}} |
        | {{state.fx-resource-aad-app-for-teams.clientId}} | ${{AAD_APP_CLIENT_ID}} |
        | {{state.aad-app.clientId}} | ${{AAD_APP_CLIENT_ID}} |
        | {{state.fx-resource-aad-app-for-teams.clientSecret}} | ${{SECRET_AAD_APP_CLIENT_SECRET}} |
        | {{state.aad-app.clientSecret}} | ${{SECRET_AAD_APP_CLIENT_SECRET}} |
        | {{state.fx-resource-aad-app-for-teams.objectId}} | ${{AAD_APP_OBJECT_ID}} |
        | {{state.aad-app.objectId}} | ${{AAD_APP_OBJECT_ID}} |
        | {{state.fx-resource-aad-app-for-teams.oauth2PermissionScopeId}} | ${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}} |
        | {{state.aad-app.oauth2PermissionScopeId}} | ${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}} |
        | {{state.fx-resource-aad-app-for-teams.tenantId}} | ${{AAD_APP_TENANT_ID}} |
        | {{state.aad-app.tenantId}} | ${{AAD_APP_TENANT_ID}} |
        | {{state.fx-resource-aad-app-for-teams.oauthHost}} | ${{AAD_APP_OAUTH_AUTHORITY_HOST}} |
        | {{state.aad-app.oauthHost}} | ${{AAD_APP_OAUTH_AUTHORITY_HOST}} |
        | {{state.fx-resource-aad-app-for-teams.oauthAuthority}} | ${{AAD_APP_OAUTH_AUTHORITY}} |
        | {{state.aad-app.oauthAuthority}} | ${{AAD_APP_OAUTH_AUTHORITY}} |
        | {{state.fx-resource-bot.botId}} | ${{BOT_ID}} |
        | {{state.teams-bot.botId}} | ${{BOT_ID}} |
        | {{state.fx-resource-bot.botPassword}} | ${{SECRET_BOT_PASSWORD}} |
        | {{state.teams-bot.botPassword}} | ${{SECRET_BOT_PASSWORD}} |

5. Replace remaining `{{state.xxx}}` placeholders in `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json` files. There placeholders are generated during ARM deployment.
    1. Understand the rule to convert ARM deployment to environment variable names. The rule is: PROVISIONOUTPUT__`{output name in infra/provision.bicep}`__`{output property name in infra/provision.bicep}`.
        For example, if you have following output in `infra/provision.bicep`:
        ```
        output azureStorageTabOutput object = {
            teamsFxPluginId: 'teams-tab'
            domain: azureStorageTabProvision.outputs.domain
            endpoint: azureStorageTabProvision.outputs.endpoint
            indexPath: azureStorageTabProvision.outputs.indexPath
            storageResourceId: azureStorageTabProvision.outputs.storageResourceId
        }
        ```
        Teams Toolkit will transform them to following environment variables based on the rule:
        ```
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__TEAMSFXPLUGINID=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__DOMAIN=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__INDEXPATH=
        PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID=
        ```
        
    2. Understand how to map `{{state.xxx.xxx}}` to ARM deployment output. The middle part in `{{state.xxx.xxx}}` represents the `teamsFxPluginId` in `infra/provision.bicep`. For example, `{{state.fx-resource-frontend-hosting.endpoint}}` comes from output with `teamsFxPluginId=fx-resource-frontend-hosting`. Follow this rule to find outputs in `infra/provision.bicep` for old placeholders and figure out their new environment variable names. Add the new environment variables to `env/.env.{env}` and replace the old placeholders to reference the new environment variables in `appPackage/manifest.json`, `aad.manifest.json` and `infra/azure.parameters.{env}.json` files.

        If you don't find plugin ID in `infra/provision.bicep`, map the plugin ID from `{{state.xxx.xxx}}` to the new one based on following table
        | old plugin ID | new plugin ID |
        | --- | --- |
        | fx-resource-frontend-hosting | teams-tab |
        | fx-resource-function | teams-api |
        | fx-resource-bot | teams-bot |
        | fx-resource-aad-app-for-teams | aad-app |

        For example, the steps to replace placeholder `{{{state.fx-resource-frontend-hosting.endpoint}}}` are:
           1. The plugin id for this placeholder is `fx-resource-frontend-hosting`.
           2. Refer above table, find bicep output `azureStorageTabOutput` that contains plugin id `fx-resource-frontend-hosting` or `teams-tab`.
           3. The environment variable name should be `PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT`
           4. Replace `{{{state.fx-resource-frontend-hosting.endpoint}}}` with `{{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT}}`

    3. Find `file/updateEnv` in `teamsapp.local.yml`, add the new environment variables you figured out in previous step, and provide expected values for them when you run your app locally. For example, `PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT` is expected to be `https://localhost:53000` when you run your tab app locally:
        ```
        - uses: file/updateEnv
          with:
          envs:
            PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__ENDPOINT: https://localhost:53000 # the value is just an example
            # other environment variables ...
        ```
    
    4. Find `Start local tunnel` in `.vscode/tasks.json`, update the output to output `endpoint` and `domain` to the new environment variables for bot. For example:
        ``` json
        {
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
                            "endpoint": "PROVISIONOUTPUT__AZUREWEBAPPBOTOUTPUT__SITEENDPOINT", // you need to update the environment variable name here
                            "domain": "PROVISIONOUTPUT__AZUREWEBAPPBOTOUTPUT__DOMAIN" // you need to update the environment variable name here
                        }
                    }
                ],
                "env": "local"
            },
            "isBackground": true,
            "problemMatcher": "$teamsfx-local-tunnel-watch"
        }
        ```
        > This step is only required when your project contains bot or message extension.

    5. Open `teamsapp.yml`, for any placeholder not added by you, replace them based on their meaning. For example, `azureStorage/deploy` requires an Azure Storage resource id, so you need to change the placeholder to `${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID}}`:
        ``` yml
        - uses: azureStorage/deploy
          with:
            distributionPath: ./build
            resourceId: ${{TAB_AZURE_STORAGE_RESOURCE_ID}} # you need to change ${{TAB_AZURE_STORAGE_RESOURCE_ID}} to ${{PROVISIONOUTPUT__AZURESTORAGETABOUTPUT__STORAGERESOURCEID}}
        ```

After above steps, you should be able to develop your Teams app with Teams Toolkit 5.0. Please read [Feature changes that impact your development flow](#feature-changes-that-impact-your-development-flow) to understand how development flow changes in Teams Toolkit 5.0.

## Troubleshooting
### UpgradeAppPackageNotExist
The project to be upgraded has no appPackage folder under templates which is required for Upgrade. You can refer to [upgrade your projects manually](https://github.com/OfficeDev/TeamsFx/wiki/Upgrade-project-to-use-Teams-Toolkit-5.0-features#upgrade-your-projects-manually) to upgrade your project manually.

### UpgradeManifestTemplateNotExist
The project to be upgraded has no manifest.template.json file under templates/appPackage which is required for Upgrade. You can refer to [upgrade your projects manually](https://github.com/OfficeDev/TeamsFx/wiki/Upgrade-project-to-use-Teams-Toolkit-5.0-features#upgrade-your-projects-manually) to upgrade your project manually.

### UpgradeAadManifestTemplateNotExist
The project to be upgraded has no aad.template.json file under templates/appPackage which is required for your project. You may be trying to upgrade a project created by Teams Toolkit <= v3.8.0. Please install Teams Toolkit v4.x and run upgrade first, or refer to [upgrade your projects manually](https://github.com/OfficeDev/TeamsFx/wiki/Upgrade-project-to-use-Teams-Toolkit-5.0-features#upgrade-your-projects-manually) to upgrade your project manually.