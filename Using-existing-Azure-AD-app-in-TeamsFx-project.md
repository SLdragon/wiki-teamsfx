This doc is for using existing Azure AD app or Manually Create Azure AD app for TeamsFx project in Visual Studio or Visual Studio Code. Please follow the instruction and make sure all reqirued info is properly set in your TeamsFx project.


# Create an Azure AD app (Optional)

> You may skip this part if you already has an Azure AD app.
1. Go to the [Azure Portal](https://portal.azure.com) and select "Azure Active Directory".

1. Select "App Registrations" and click on "New registration" to create a new Azure AD app:
   * **Name**: The name of your configuration app.
   * **Supported account types**: Select "Account in this organizational directory only"
   * Leave the "Redirect URL" field blank for now.
   * Click on the "Register" button.

1. When the app is registered, you'll be taken to the app's "Overview" page. Copy the **Application (client) ID** and **Object ID**; we will need it later. Verify that the "Supported account types" is set to **My organization only**.

### Create client secret for Azure AD app (Required)

1. Go to app's "Certificates & secrets" page, select "Client Secret" and Click on "New client secret".
   * **Description**: The descirption of your client secret.
   * **Expires**: The expire time of your client secret.
   * Click on the "Add" button.

1. When the client secret is added, press the copy button under the "Value" column to copy the **Client Secret**.


# Create Access As User Scope for Azure AD app (Optional)

> You can skip this part if your M365 account has permission to update this Azure AD app. We will create the scope for you.
1. Go to app's "Expose an API" page, click "Add" button to add a new Application ID URI:
   - If your project is tab app, fill in `api://<your-tab-domain>/<aad-app-client-id>`
   - If your project is bot or message extension app, fill in `api://botid-<your-bot-id>`
   - If your project contains both tab and bot / message extension, fill in `api://<your-tab-domain>/botid-<your-bot-id>`

1. Click on "Add a scope" under "Scopes defined by this API".
   * **Scope name**: Fill in "access_as_user".
   * **Who can consent?**: Choose "Admins and users".
   * **Admin consent display name**: Fill in "Teams can access app’s web APIs".
   * **Admin consent description**: Fill in "Allows Teams to call the app’s web APIs as the current user.".
   * **User consent display name**: Fill in "Teams can access app’s web APIs and make requests on your behalf".
   * **User consent description**: Fill in "Enable Teams to call this app’s web APIs with the same rights that you have".
   * **State**: Choose "Enabled".
   * Click on "Add scope".

1. On the same page, click on "Add a client application" under "Authorized client applications".
   * **Client ID**: Fill in "1fec8e78-bce4-4aaf-ab1b-5451cc387264" which is Client Id for Teams on mobile and client.
   * **Authorized scopes**: Choose the existing "access_as_user" scope.
   * Click on "Add application".

1. Click again on "Add a client application".
   * **Client ID**: Fill in "5e3ce6c0-2b1f-4285-8d4b-75ee78787346" which is Client Id for Teams on web.
   * **Authorized scopes**: Choose the existing "access_as_user" scope.
   * Click on "Add application".

1. Go to app's "Manifest" page, copy the "id" under "oauth2Permissions" as **Access As User Scope ID**.


# Get necessary info from existing Azure AD app

* You may skip this part if you follow the instruction above to create an Azure AD app.

1. Go to the [Azure Portal](https://portal.azure.com) and select "Azure Active Directory".

1.  Select "App Registrations" and find your existing Azure AD app.

1. Go to app's "Overview" page, copy the **Application (client) ID** and **Object ID**; we will need it later. Verify that the "Supported account types" is set to **My organization only**.

1. Go to app's "Certificates & secrets" page, press the copy button under the "Value" column to copy the **Client Secret**. Note: if you can not copy the secret, please follow the [instruction](#create-client-secret-for-azure-ad-app) to create a new client secret.

1. Go to app's "Expose an API" page. Update the Application ID URI if the format is not as below:
   - If your project is tab app, fill in `api://<your-tab-domain>/<aad-app-client-id>`
   - If your project is bot or message extension app, fill in `api://botid-<your-bot-id>`
   - If your project contains both tab and bot / message extension, fill in `api://<your-tab-domain>/botid-<your-bot-id>`

1. If you have already add "access_as_user" scope under "Scopes defined by this API" and pre-auth the two Teams Client Ids, go to app's "Manifest" page, copy the "id" under "oauth2Permissions" as **Access As User Scope ID**. Otherwise, follow the steps in `Create Access As User Scope for Azure AD app` to create a new one.

# Set necessary info in TeamsFx project

1. Open your TeamsFx project, and open `.fx/configs/config.dev.json`.

1. Set `AAD_APP_CLIENT_SECRET` = **Client Secret** in your system environment variable.

    *Note: You can change the env name `AAD_APP_CLIENT_SECRET` here, and remember to replace  `AAD_APP_CLIENT_SECRET` with your env name in the next step.*

1. Add follow code after existing code.

     ```
     "$schema": "https://aka.ms/teamsfx-env-config-schema",
     "description": "...",
     "manifest": {
       ...
     },
     // Add code below. Note you need to replace the placeholders with the values copied in previous steps.
     "auth": {
       "objectId": **Object ID**,
       "clientId": **Application (client) ID**,
       "clientSecret": {{ $env.AAD_APP_CLIENT_SECRET }},
       (optional) "accessAsUserScopeId": **Access As User Scope ID**
     }
     ```

1. [For Visual Studio Code] Open Teams Toolkit extension and click on "Provision in the cloud". Wait until your project is successfully provisioned.

1. [For Visual Studio] Click "Project" -> "Teams Toolkit" -> "Provision in the cloud". Wait until your project is successfully provisioned.

# Upload Azure AD app manifest to Azure portal

* If Teams Toolkit failed to update Azure AD app, there will be an alert says:

  ```
  Failed in step: Update AAD app. You need to go to Azure Protal and mannually update Azure AD app manifest for the provided Azure AD app.
  ```

   Please follow the instruction to update permission if you see the above message.

1. Open `templates/appPackage/aad.template.json`

1. Get content of Azure AD app manifest
   - For Visual Studio Code:

      1. Click on "preview" as shown below:

         ![image](https://user-images.githubusercontent.com/11220663/172798183-98becfab-fb82-4518-b569-684b65c72563.png)


      1. Select your env, and you manifest can be found under `build/appPackage/manifest.${env}.json`.

      1. Copy the content in the manifest file.

   - For Viusal Studio: 
      1. Copy the content and follow the structure below to replace all the values in the pattern `{{file.component.value}}` in the manifest file:
         - {{state.component.value}}: you can get the value by finding `value` under `component` in `.fx/states/state.{env}.json`
         - {{config.component.value}}: you can get the value by finding `value` under `component` in `.fx/configs/config.{env}.json`

1. Go to the [Azure Portal](https://portal.azure.com) and select "Azure Active Directory".

1.  Select "App Registrations" and find your existing Azure AD app.

1. Go to app's "Manifest" page, paste the manifest content into the editor and Click `Save` to save the changes.