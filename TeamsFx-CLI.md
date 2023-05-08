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

- Local Preview Bot App
  - To make the bot endpoint public, you should use a tunnel service solution. For example, you can use ngrok by the following steps:
      - Install [ngrok](https://ngrok.com/download)
      - Start your local tunnel service by running the command `ngrok http 3978`.
      > **Note:**
      > If you encounter the ngrok page below, please follow the steps to solve this issue.
      > <img width="596" alt="ngrok-authtoken-page" src="https://user-images.githubusercontent.com/49138419/230855631-e2228a47-402b-4b15-b8b0-5a2323050157.png">
      > 1. Sign up an ngrok account in https://dashboard.ngrok.com/signup.
      >    Copy your personal ngrok authtoken from https://dashboard.ngrok.com/get-started/your-authtoken.
      > 1. Start your local tunnel service by running the command `ngrok http 3978 --authtoken=<your-personal-ngrok-authtoken>`.
  - In the `env/.env.local` file, fill in the values for `BOT_DOMAIN` and `BOT_ENDPOINT` with your ngrok URL.
    ```
    BOT_DOMAIN=sample-id.ngrok.io
    BOT_ENDPOINT=http://sample-id.ngrok.io
    ```
  - Executing the commands in your project directory.
    ```shell
    teamsfx provision --env local
    teamsfx deploy --env local
    teamsfx preview --env local
    ```

- Remote Preview
```shell
teamsfx provision --env dev
teamsfx deploy --env dev
teamsfx preview --env dev
```
