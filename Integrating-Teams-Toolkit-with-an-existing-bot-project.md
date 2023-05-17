Teams Toolkit can integrate with your existing Teams app project or infrastructure to enable a better development experience.

# Prerequisites

* Teams Toolkit for VSCode v5
* (or) Teams Toolkit CLI v2

<b>Make sure you have git enabled or some form of back up with your existing project in case errors happen.</b>

# Set up

* Create a new app with Teams Toolkit with bot capability.
* Your new project has the following project structure.

![image](https://user-images.githubusercontent.com/103554011/233884601-85748bc3-8b13-4600-81be-f754e44db148.png)

* We mainly need to work with `teamsapp.yml` and `teamsapp.local.yml`. <b>[Learn more about Teams Toolkit yml files](https://aka.ms/teamsfx-v5.0-guide)</b>

## Integrating the debugging process

### Update Teams app manifest

* Move your existing Teams manifest, color icon, and outline icon files into the new project, replacing the default ones. Teams Toolkit scaffolded projects save these files in the `appPackage` folder. If you choose to move these files to a different location, make sure you update [actions](https://aka.ms/teamsfx-actions) in `teamsapp.local.yml` that reference them.

* Update your manifest file. You can use `${{MY_VARIABLE_NAME}}` as placeholders in `manifest.json`, Teams Toolkit will replace them with their corresponding value at runtime. For a bot project, you usually need to use placeholders for `id`, `name.short`, and `bots[].botId`. Make sure you save the values into an environment file.

![image](https://user-images.githubusercontent.com/103554011/233888989-0631a8a0-da2b-4d95-9335-649f339682bb.png)

![image](https://user-images.githubusercontent.com/103554011/233889036-0d5fe024-f501-480b-aa46-84b4caef23e4.png)

![image](https://user-images.githubusercontent.com/103554011/233889071-0c3f493d-1e80-45f5-a47f-de75e0cfd3ee.png)

### Update environment variables

* Teams Toolkit scaffolded projects come with 2 predefined environments, local and dev. We will use `local` for the debug process. More about [environments](https://aka.ms/teamsfx-v5.0-guide#environments)

* Update `env/.env.local` with your existing Teams app and bot configurations.

![image](https://user-images.githubusercontent.com/103554011/233890805-89341c60-ee48-48c5-b7d4-8f78c30f92fa.png)

* Update `env/.env.local.user` with your bot password.

![image](https://user-images.githubusercontent.com/103554011/233890893-2c95bfb7-c8d2-472c-a597-a45bddf27724.png)

* Here are some useful links to help you find your existing Teams app and bot configurations:
  * [Use your existing Teams app ID](https://aka.ms/teamsfx-v5.0-guide#use-your-existing-teams-app-id)
  * [Customize Azure subscription ID and resource group](https://aka.ms/teamsfx-v5.0-guide#customize-azure-subscription-id-and-resource-group)
  * [Using existing Azure Active Directory app ID for bot](https://aka.ms/teamsfx-v5.0-guide#using-existing-azure-active-directory-app-id-for-bot)
  * [Using existing Azure Bot Service Messaging Endpoint](https://aka.ms/teamsfx-v5.0-guide#using-existing-azure-bot-service-messaging-endpoint)

### Integrate source code

* Move the source code from your existing project into the new Teams Toolkit project. This includes all the code for your bot to run, any adaptive cards and other necessary config files.
* Update your source code to reference environment variables, such as bot credentials, configurations, etc. Your can reference environment variables using `process.env`
* Update `package.json` with the dependencies used by your existing project.

### Replace Bot Framework with Azure Bot Service in the debugging process

* Teams Toolkit scaffolded projects use Bot Framework in the debugging process, however many existing projects and samples use Azure Bot Service. We need to update the new Teams Toolkit project to use Azure Bot Service for debugging.
* Update the provision stage in `teamsapp.local.yml`. Replace the [`botFramework/create` action](https://aka.ms/teamsfx-actions#botframeworkcreate) with the [`arm/deploy` action](https://aka.ms/teamsfx-actions#armdeploy)
* `arm/deploy` reference Azure bicep files. By default, your Teams Toolkit scaffolded bot project generates these bicep files in the `infra/` folder. You may need to modify the default bicep files to fit your project. Note, the default bicep files are referenced by `teamsapp.yml` for provisioning Azure resources. You can use environment variables to separate configurations, or create new copies of the bicep files and reference them in `teamsapp.local.yml`.
* Update `env/.env.local` with environment variables referenced in `teamsapp.local.yml`.

Now you can run and debug your app with Teams Toolkit.

[Learn more about Teams Toolkit's debugging process.](https://aka.ms/teamsfx-v5.0-guide#debug-f5-in-visual-studio-code)

## Integrating the remote provision, deploy, and publish process

You can customize Teams Toolkit's remote provision, deploy and publish process with [`teamsapp.yml`](https://aka.ms/teamsfx-v5.0-guide#yml-definition).

### Update `teamsapp.yml`

* Update the provision, deploy, and publish lifecycles with actions that suits your existing application.
* Note: you can use [`script` action](https://aka.ms/teamsfx-actions#script) to run any arbitrary commands.

### Update environment variables

* Update the `.env.{envName}` and `.env.{envName}.user` files with environment variable values referenced by `teamsapp.yml`, `appPackage/manifest.json`, Azure bicep files, and your code.
* Teams Toolkit scaffolded project use the `dev` environment for remote by default. You can create new environments. [Learn more about Teams Toolkit environments](https://aka.ms/teamsfx-v5.0-guide#environments).
