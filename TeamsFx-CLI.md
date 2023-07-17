> The content is under construction and is subject to rapid changes.

# teamsfx preview
> Before running teamsfx preview, teamsfx provision and teamsfx deploy should be run first.

Preview the current application from local or remote.

## Parameters for teamsfx preview
|Parameter|Requirement|Description|
|--|--|--|
|`--env`|No|Select an existing env for the project. The default value is `local`.|
|`--run-command`|No|The command to start local service. Work for `local` environment only. If not specified, teamsfx will use the auto detected one from project type (`npm run dev:teamsfx` or `dotnet run` or `func start`). If empty, teamsfx will skip starting local service.|
|`--running-pattern`|No|The ready signal output that service is launched. Work for `local` environment only. If not specified, teamsfx will use the default common pattern (`started\|successfully\|finished\|crashed\|failed`). If empty, teamsfx treats process start as ready signal.|
|`--open-only`|No|Work for `local` environment only. If true, directly open web client without launching local service.|
|`--m365-host`|No|Preview the application in Teams, Outlook or the Microsoft 365 app. Options are `teams`, `outlook` and `office`. The default value is `teams`.|
|`--browser`|No|The browser to open Teams web client. The options are `chrome`, `edge` and `default` such as system default browser and the value is `default`.|
|`--browser-arg`|No|Argument to pass to the browser, requires --browser, can be used multiple times, for example, `--browser-args="--guest"`|
|`--exec-path`| No | The paths that will be added to the system environment variable PATH when the command is executed, defaults to `${folder}/devTools/func`. |

## Scenarios for teamsfx preview
The following list provides the common scenarios for`teamsfx preview:
- Local Preview Tab App
  - Executing the commands in your project directory.
    ```shell
    teamsfx provision --env local
    teamsfx deploy --env local
    teamsfx preview --env local
    ```

- Local Preview Bot App / Message Extension App
  - Install [dev tunnel cli](https://aka.ms/teamsfx-install-dev-tunnel).
  - Login with your M365 Account using the command `devtunnel user login`.
  - Start your local tunnel service by running the command `devtunnel host -p 3978 --protocol http --allow-anonymous`.
  - In the `env/.env.local` file, fill in the values for `BOT_DOMAIN` and `BOT_ENDPOINT` with your dev tunnel URL.
    ```
    BOT_DOMAIN=sample-id-3978.devtunnels.ms
    BOT_ENDPOINT=https://sample-id-3978.devtunnels.ms/
    ```
  - Executing the commands in your project directory.
    ```shell
    teamsfx provision --env local
    teamsfx deploy --env local
    ```
  - If you are previewing a Azure Functions hosted notification bot, please execute the following command in your project directory.
    ```shell
    npm run prepare-storage:teamsfx
    ```
  - Executing the command in your project directory.
    ```shell
    teamsfx preview --env local
    ```

- Remote Preview
```shell
teamsfx provision --env dev
teamsfx deploy --env dev
teamsfx preview --env dev
```
