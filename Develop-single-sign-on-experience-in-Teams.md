# Enable single sign-on for tab applications

Microsoft Teams provides a mechanism by which an application can obtain the signed-in Teams user token to access Microsoft Graph (and other APIs). Teams Toolkit facilitates this interaction by abstracting some of the Azure Active Directory flows and integrations behind some simple, high level APIs. This enables you to add single sign-on (SSO) features easily to your Teams application.

Basically you will need take care these configurations:

* In the Azure Active Directory app manifest file, you need to specify URIs such as the URI to identify the Azure AD authentication app and the redirect URI for returning token.
* In the Teams manifest file, add the SSO application to link it with Teams application.
* In the Teams Toolkit configuration files and Infra files, you need to add necessary configurations to make SSO works for your Teams app.
* Add SSO application information in Teams Toolkit configuration files in order to make sure the authentication app can be registered on backend service and started by Teams Toolkit when you debug or preview Teams application.

In this tutorial you will learn:
* [Steps to add SSO to Teams Tab app](#teams-tab-app)
  * [Create Azure Active Directory app manifest](#azure-active-directory-app-manifest)
  * [Update Teams app manifest](#teams-app-manifest)
  * [Update Teams Toolkit configuration files](#teams-toolkit-configuration-files)
  * [Update Source Code](#update-source-code)
* [Steps to add SSO to Teams Bot/Messaging Extension app](#teams-botmessaging-extension-app)
  * [Create Azure Active Directory app manifest](#azure-active-directory-app-manifest-1)
  * [Update Teams app manifest](#teams-app-manifest-1)
  * [Update Teams Toolkit configuration files](#teams-toolkit-configuration-files-1)
  * [Update Infra](#update-infra)
  * [Update Source Code](#update-source-code-1)

## Teams Tab app

### Azure Active Directory app manifest

> Note: You can find detail info about Azure Active Directory app manifest [here](https://learn.microsoft.com/azure/active-directory/develop/reference-app-manifest).

Download Azure Active Directory app manifest template [here](https://aka.ms/teamsfx-aad-manifest-v3-tab) to `./aad.manifest.json`. The Azure Active Directory [manifest](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest) allows you to customize various aspects of your application registration. You can update the manifest as needed.

### Teams app manifest

> Note: You can find detail info about Teams app manifest [here](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema).

You can find your original Teams app manifest template in `./appPackages/manifest.json`, and you need to add `webApplicationInfo` in the manifest.

[`webApplicationInfo`](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema#webapplicationinfo) provides your Azure Active Directory App ID and Microsoft Graph information to help users seamlessly sign into your app.

Example: You can add following object into your Teams app manifest for your Tab project.
```json
"webApplicationInfo": {
  "id": "${{AAD_APP_CLIENT_ID}}",
  "resource": "api://${{TAB_DOMAIN}}/${{AAD_APP_CLIENT_ID}}"
}
```
> Note: You can use `${{ENV_NAME}}` to reference variables in `env/.env.{TEAMSFX_ENV}`.

### Teams Toolkit configuration files

You can find your Teams Toolkit configuration files `./.yml`. Azure Active Directory related changes and configs needs to be added into your configuration files:
- add `aadApp/create` under `provision`:
  * For creating new Azure Active Directory apps used for SSO.
  * You can find more info [here](https://aka.ms/teamsfx-actions/aadapp-create)
- add `aadApp/update` under `provision`
  * For updating your Azure Active Directory app with AAD app manifest in step 1.
  * You can find more info [here](https://aka.ms/teamsfx-actions/aadapp-update)
- for React project, update `cli/runNpmCommand` under `deploy`:
  * For adding following environment variables when local debug:
    * REACT_APP_CLIENT_ID: Azure Active Directory app client id
    * REACT_APP_START_LOGIN_PAGE_URL: Azure Active Directory app client secret
- for React project, update `file/createOrUpdateEnvironmentFile`
  * For adding following environment variables when local debug:
    * REACT_APP_CLIENT_ID: Azure Active Directory app client id
    * REACT_APP_START_LOGIN_PAGE_URL: Login start page for authentication

You can set following values if you are using TeamsFx Tab template.

#### `aad/create`
Add following lines in `provision` in `teamsapp.yml` and `teamsapp.local.yml`:
```yml
- uses: aadApp/create
  with:
    name: "YOUR_AAD_APP_NAME"
    generateClientSecret: true
    signInAudience: "AzureADMyOrg"
  writeToEnvironmentFile:
      clientId: AAD_APP_CLIENT_ID
      clientSecret: SECRET_AAD_APP_CLIENT_SECRET
      objectId: AAD_APP_OBJECT_ID
      tenantId: AAD_APP_TENANT_ID
      authority: AAD_APP_OAUTH_AUTHORITY
      authorityHost: AAD_APP_OAUTH_AUTHORITY_HOST
```
> Note: Replace the value of "name" with your expected Azure Active Directory app name.

#### `aad/update`
Add following lines in `provision` in `teamsapp.yml` and `teamsapp.local.yml`:

```yml
- uses: aadApp/update
  with:
    manifestPath: "./aad.manifest.json"
    outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json
```
> Note: Replace the value of "manifestPath" with the relative path of Azure Active Directory app manifest template (`aad.manifest.json`) if you have modify the path of this file.

> Note: For local you need to place `aad/update` after `file/createOrUpdateEnvironmentFile` action since `aad/update` will consume output of `file/createOrUpdateEnvironmentFile`.

#### `cli/runNpmCommand`
If you are building tab app with React framework, find `cli/runNpmCommand` action with name `build app` in `teamsapp.yml` and add following env:
```yml
env:
  REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
  REACT_APP_START_LOGIN_PAGE_URL: ${{TAB_ENDPOINT}}/auth-start.html
```

#### `file/createOrUpdateEnvironmentFile`
If you are building tab app with React framework, find `file/createOrUpdateEnvironmentFile` action for deploy in `teamsapp.local.yml` and add following env:
```yml
envs:
  ...
  REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
  REACT_APP_START_LOGIN_PAGE_URL: ${{TAB_ENDPOINT}}/auth-start.html
```

### Update Source Code
With all changes above, your environment is ready and can update your code to add SSO to your Teams app.

#### React

Suppose you are building your tab app with React framework. You can find and download sample code for TeamsFx Tab below to `./auth`:
  * for [js](https://github.com/OfficeDev/TeamsFx/tree/main/packages/fx-core/templates/plugins/resource/aad/auth/tab/js)
  * for [ts](https://github.com/OfficeDev/TeamsFx/tree/main/packages/fx-core/templates/plugins/resource/aad/auth/tab/ts)

You can follow the following steps to update your source code:

1. Move `auth-start.html` and `auth-end.html` in `auth/public` folder to `public/`.
  * These two HTML files are used for auth redirects.

2. Move `sso` folder under `auth/` to `src/sso/`.
  * `InitTeamsFx`: This file implements a function that initialize TeamsFx SDK and will open `GetUserProfile` component after SDK is initialized.
  * `GetUserProfile`: This file implements a function that calls Microsoft Graph API to get user info.

3. Import and add `InitTeamsFx` in `Welcome.*`.

You can also find sample for SSO enabled Tab [here](https://github.com/OfficeDev/TeamsFx-Samples/tree/dev/hello-world-tab).

#### Vanilla JavaScript

For non-React tab app, following is a basic sample for getting SSO token with vanilla JavaScript.

    ```js
    function getSSOToken() {
      return new Promise((resolve, reject) => {
        microsoftTeams.authentication
          .getAuthToken()
          .then((token) => {
            resolve(token);
          })
          .catch((error) => {
            alert(error);
            reject("Error getting token: " + error);
          });
      });
    }

    function getBasicUserInfo() {
      getSSOToken().then((ssoToken) => {
        const tokenObj = JSON.parse(window.atob(ssoToken.split(".")[1]));;
        console.log(`username: ${tokenObj.name}`);
        console.log(`user email: ${tokenObj.preferred_username}`);
      });
    }
    ```

## Teams Bot/Messaging Extension app

### Azure Active Directory app manifest
> Note: You can find detail info about Azure Active Directory app manifest [here](https://learn.microsoft.com/azure/active-directory/develop/reference-app-manifest).

Download Azure Active Directory app manifest template [here](https://aka.ms/teamsfx-aad-manifest-v3-bot) to `./aad.manifest.json`. The AAD [manifest](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest) allows you to customize various aspects of your application registration. You can update the manifest as needed.

### Teams app manifest

> Note: You can find detail info about Teams app manifest [here](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema).

You can find your original Teams app manifest template in `./appPackages/manifest.json`.

#### Add `webApplicationInfo`
[`webApplicationInfo`](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema#webapplicationinfo) provides your Azure Active Directory App ID and Microsoft Graph information to help users seamlessly sign into your app.

Example: You can add following object into your Teams app manifest for your Tab project.
```json
"webApplicationInfo": {
  "id": "${{AAD_APP_CLIENT_ID}}",
  "resource": "api://botid-${{BOT_ID}}"
}
```
> Note: You can use `${{ENV_NAME}}` to reference variables in `env/.env.{TEAMSFX_ENV}`.

#### Register command in `commandLists`
[`commandLists`](https://learn.microsoft.com/microsoftteams/platform/resources/schema/manifest-schema#botscommandlists) contains commands that your bot can recommend to users.

You can set following values if you are using TeamsFx Bot template.
```json
{
  "title": "profile",
  "description": "Show user profile using Single Sign On feature"
}
```

#### Valid Domains
[`validDomains`](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema#validdomains) contains valid domains for websites the app expects to load within the Teams client.

You can set following values if you are using TeamsFx Bot template.
```json
"validDomains": [
  "${{BOT_DOMAIN}}"
]
```

### Teams Toolkit configuration files

You can find your Teams Toolkit configuration files `./.yml`. Azure Active Directory related changes and configs needs to be added into your configuration files:

  - add `aadApp/create` under `provision`:
    * For creating new Azure Active Directory apps used for SSO.
    * You can find more info [here](https://aka.ms/teamsfx-actions/aadapp-create)
  - add `aadApp/update` under `provision`
    * For updating your Azure Active Directory app with Azure Active Directory app manifest in step 1.
    * You can find more info [here](https://aka.ms/teamsfx-actions/aadapp-update)
  - update `file/createOrUpdateEnvironmentFile`
    * For adding following environment variables when local debug:
      * M365_CLIENT_ID: Azure Active Directory app client id
      * M365_CLIENT_SECRET: Azure Active Directory app client secret
      * M365_TENANT_ID: Tenant id of Azure Active Directory app
      * INITIATE_LOGIN_ENDPOINT: Login start page for authentication
      * M365_AUTHORITY_HOST: Azure Active Directory app oauth authority host
      * M365_APPLICATION_ID_URI: IdentifierUri for Azure Active Directory app

You can set following values if you are using TeamsFx Tab/Bot template.

#### `aad/create`
Add following lines in `provision` in `teamsapp.yml` and `teamsapp.local.yml`:
```yml
- uses: aadApp/create
  with:
    name: "YOUR_AAD_APP_NAME"
    generateClientSecret: true
    signInAudience: "AzureADMyOrg"
  writeToEnvironmentFile:
    clientId: AAD_APP_CLIENT_ID
    clientSecret: SECRET_AAD_APP_CLIENT_SECRET
    objectId: AAD_APP_OBJECT_ID
    tenantId: AAD_APP_TENANT_ID
    authority: AAD_APP_OAUTH_AUTHORITY
    authorityHost: AAD_APP_OAUTH_AUTHORITY_HOST
```
> Note: Replace the value of "name" with your expected AAD app name.

#### `aad/update`
Add following lines in `provision` in `teamsapp.yml` and `teamsapp.local.yml`:

```yml
- uses: aadApp/update
  with:
    manifestPath: "./aad.manifest.json"
    outputFilePath : ./build/aad.manifest.${{TEAMSFX_ENV}}.json
```
> Note: Replace the value of "manifestPath" with the relative path of AAD app manifest template (`aad.manifest.json`) if you have modify the path of this file.

#### `file/createOrUpdateEnvironmentFile`
Find `file/createOrUpdateEnvironmentFile` action in `teamsapp.local.yml` and add following env:
```yml
envs:
  ...
  M365_CLIENT_ID: ${{AAD_APP_CLIENT_ID}}
  M365_CLIENT_SECRET: ${{SECRET_AAD_APP_CLIENT_SECRET}}
  M365_TENANT_ID: ${{AAD_APP_TENANT_ID}}
  INITIATE_LOGIN_ENDPOINT: ${{BOT_ENDPOINT}}/auth-start.html
  M365_AUTHORITY_HOST: ${{AAD_APP_OAUTH_AUTHORITY_HOST}}
  M365_APPLICATION_ID_URI: api://botid-${{BOT_ID}}
```

### Update Infra

AAD related configs needs to be configured in your remote service. Following example shows the configs on Azure Webapp.
  * M365_CLIENT_ID: Azure Active Directory app client id
  * M365_CLIENT_SECRET: Azure Active Directory app client secret
  * M365_TENANT_ID: Tenant id of Azure Active Directory app
  * INITIATE_LOGIN_ENDPOINT: Login start page for authentication
  * M365_AUTHORITY_HOST: Azure Active Directory app oauth authority host
  * M365_APPLICATION_ID_URI: IdentifierUri for Azure Active Directory app

You can set follow the steps below if you are using TeamsFx Tab/Bot template.

1. Open `infra/azure.parameters.json` and add following lines into `parameters`:
  ```json
  "m365ClientId": {
    "value": "${{AAD_APP_CLIENT_ID}}"
  },
  "m365ClientSecret": {
    "value": "${{SECRET_AAD_APP_CLIENT_SECRET}}"
  },
  "m365TenantId": {
    "value": "${{AAD_APP_TENANT_ID}}"
  },
  "m365OauthAuthorityHost": {
    "value": "${{AAD_APP_OAUTH_AUTHORITY_HOST}}"
  }
  ```

2. Open `infra/azure.bicep` find follow line:
  ```bicep
  param location string = resourceGroup().location
  ```
and add following lines:
  ```bicep
  param m365ClientId string
  param m365TenantId string
  param m365OauthAuthorityHost string
  param m365ApplicationIdUri string = 'api://botid-${botAadAppClientId}'
  @secure()
  param m365ClientSecret string
  ```

3. Add following lines before output
  ```bicep
  resource webAppSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    name: '${webAppName}/appsettings'
    properties: {
      M365_CLIENT_ID: m365ClientId
      M365_CLIENT_SECRET: m365ClientSecret
      INITIATE_LOGIN_ENDPOINT: uri('https://${webApp.properties.defaultHostName}', 'auth-start.html')
      M365_AUTHORITY_HOST: m365OauthAuthorityHost
      M365_TENANT_ID: m365TenantId
      M365_APPLICATION_ID_URI: m365ApplicationIdUri
      BOT_ID: botAadAppClientId
      BOT_PASSWORD: botAadAppClientSecret
      RUNNING_ON_AZURE: '1'
    }
  }
  ```
> Note: If you want add additional configs to your Azure Webapp, please add the configs in the webAppSettings.

> Note: You may also need to specify default node version by adding the following config:

  ```bicep
  WEBSITE_NODE_DEFAULT_VERSION: '14.20.0'
  ```

### Update Source Code
You can find and download sample code for TeamsFx Tab below to `./auth`:
  * for [js](https://github.com/OfficeDev/TeamsFx/tree/main/packages/fx-core/templates/plugins/resource/aad/auth/bot/js)
  * for [ts](https://github.com/OfficeDev/TeamsFx/tree/main/packages/fx-core/templates/plugins/resource/aad/auth/bot/ts)

#### For Bot
1. Move files under `auth/sso` folder to `src`. ProfileSsoCommandHandler class is a sso command handler to get user info with SSO token. You can follow this method and create your own sso command handler.
2. Move `auth/public` folder to `src/public`. This folder contains HTML pages that the bot application hosts. When single sign-on flows are initiated with Azure Active Directory, Azure Active Directory will redirect the user to these pages.
3. Execute the following commands under `./` folder: `npm install isomorphic-fetch --save`
4. (For ts only) Execute the following commands under `./` folder: `npm install copyfiles --save-dev` and replace following line in package.json:
    ```json
    "build": "tsc --build && shx cp -r ./src/adaptiveCards ./lib/src",
    ```
    with:
    ```json
    "build": "tsc --build && shx cp -r ./src/adaptiveCards ./lib/src && copyfiles src/public/*.html lib/",
    ```
  By doing this, the HTML pages used for auth redirect will be copied when building this bot project.

5. In `src/index` file, you need to add following line to import `isomorphic-fetch`:
  ```ts
  require("isomorphic-fetch");
  ```

  and add following lines to add redirect to auth pages:
  ```ts
  server.get(
    "/auth-:name(start|end).html",
    restify.plugins.serveStatic({
        directory: path.join(__dirname, "public"),
    })
  );
  ```

  and update commandApp.requestHandler to make auth works:
  ```ts
  await commandApp.requestHandler(req, res).catch((err) => {
    // Error message including "412" means it is waiting for user's consent, which is a normal process of SSO, sholdn't throw this error.
    if (!err.message.includes("412")) {
      throw err;
    }
  });
  ```

6. Add `ssoConfig` and `ssoCommands` in `ConversationBot` in `src/internal/initialize`:

```ts
import { ProfileSsoCommandHandler } from "../profileSsoCommandHandler";

export const commandBot = new ConversationBot({
  ...
  // To learn more about ssoConfig, please refer teamsfx sdk document: https://docs.microsoft.com/microsoftteams/platform/toolkit/teamsfx-sdk
  ssoConfig: {
    aad :{
      scopes:["User.Read"],
    },
  },
  command: {
    enabled: true,
    commands: [new HelloWorldCommandHandler() ],
    ssoCommands: [new ProfileSsoCommandHandler()],
  },
});
```

#### For Messaging Extension
Implement the Key API `handleMessageExtensionQueryWithSSO` in `TeamsActivityHandler.handleTeamsMessagingExtensionQuery`. Read this [Wiki](https://github.com/OfficeDev/TeamsFx/wiki/SSO-for-Message-Extension) for more detail about the API `handleMessageExtensionQueryWithSSO`

To make this work in your application:
1. Move the `auth/public` folder to `src/public`. This folder contains HTML pages that the bot application hosts. When single sign-on flows are initiated with Azure Active Directory, Azure Active Directory will redirect the user to these pages.
2. Modify your `src/index` to add the appropriate `restify` routes to these pages.

```ts
const path = require("path");

// Listen for incoming requests.
server.post("/api/messages", async (req, res) => {
  await adapter.process(req, res, async (context) => {
    await bot.run(context);
  }).catch((err) => {
    // Error message including "412" means it is waiting for user's consent, which is a normal process of SSO, sholdn't throw this error.
    if(!err.message.includes("412")) {
        throw err;
      }
    })
});

server.get(
  "/auth-:name(start|end).html",
  restify.plugins.serveStatic({
    directory: path.join(__dirname, "public"),
  })
);
```
3. Execute the following commands under `./` folder: `npm install @microsoft/teamsfx`
4. Execute the following commands under `./` folder: `npm install isomorphic-fetch`
5. Implement the key API `handleMessageExtensionQueryWithSSO` in `TeamsActivityHandler.handleTeamsMessagingExtensionQuery`
6. (For ts only) Install `copyfiles` npm packages in your bot project, add or update the `build` script in `src/package.json` as following

```json
"build": "tsc --build && copyfiles ./public/*.html lib/",
```
By doing this, the HTML pages used for auth redirect will be copied when building this bot project.

1. Update `templates/appPackage/aad.template.json` your scopes which used in `handleMessageExtensionQueryWithSSO`.
```json
  "requiredResourceAccess": [
      {
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
              {
                  "id": "User.Read",
                  "type": "Scope"
              }
          ]
      }
  ]
  ```

## Debug your application

You can debug your application by pressing F5.

Teams Toolkit will use the Azure Active Directory manifest file to register a AAD application registered for SSO.

To learn more about Teams Toolkit local debug functionalities, refer to this [document](https://docs.microsoft.com/microsoftteams/platform/toolkit/debug-local).

## Customize Azure Active Directory applications

The Azure Active Directory [manifest](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest) allows you to customize various aspects of your application registration. You can update the manifest as needed.

Follow this [document](https://aka.ms/teamsfx-aad-manifest#how-to-customize-the-aad-manifest-template) if you need to include additional API permissions to access your desired APIs.

Follow this [document](https://aka.ms/teamsfx-aad-manifest#How-to-view-the-AAD-app-on-the-Azure-portal) to view your AAD application in Azure Portal.