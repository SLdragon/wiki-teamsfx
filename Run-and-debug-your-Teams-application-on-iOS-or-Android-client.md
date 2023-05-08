# Run and debug your Teams application on iOS or Android client.
## Steps for testing a tab app on mobile client

1. Run the Teams tab app locally
   - For VSCode:
     - Update `.vscode/task.json`, add `Start local tunnel` task to make the tab app accessible on the mobile client. 
        ```json
        {
            "version": "2.0.0",
            "tasks": [
                {
                    "label": "Start Teams App Locally",
                    "dependsOn": [
                        "Validate prerequisites",
                        "Start local tunnel",
                        "Provision",
                        "Deploy",
                        "Start application"
                    ],
                    "dependsOrder": "sequence"
                },
                {
                    "label": "Start local tunnel",
                    "type": "teamsfx",
                    "command": "debug-start-local-tunnel",
                    "args": {
                        "type": "dev-tunnel",
                        "ports": [
                            {
                                "portNumber": 53000,
                                "protocol": "https",
                                "access": "private",
                                "writeToEnvironmentFile": {
                                    "endpoint": "TAB_ENDPOINT",
                                    "domain": "TAB_DOMAIN"
                                }
                            }
                        ],
                        "env": "local"
                    },
                    "isBackground": true,
                    "problemMatcher": "$teamsfx-local-tunnel-watch"
                }
            ]
        }
        ```
        > **Note:** When the access of a tab app is set to `private`, it can only be previewed on the mobile client. However, if you want to preview the app on the mobile client and debug it on web clients, you need to set the access level to `public`. It's worth noting that public access raises safety concerns since the tab app can be visited by anyone who knows the app's URL. 
      - Update `teamsapp.local.yml`, **remove** the following action to avoid setting the `TAB_DPMAIN` and `TAB_ENDPOINT` in the env file. 
         ```yaml
         - uses: script
           with:
             run:
               echo "::set-teamsfx-env TAB_DOMAIN=localhost:53000";
               echo "::set-teamsfx-env TAB_ENDPOINT=https://localhost:53000"; 
         ```

      - Update `teamsapp.local.yml`, add the configuration `WDS_SOCKET_PORT=0` to activate hot reloading while debugging React after utilizing the tunnel service. 
         ```yaml
         - uses: file/createOrUpdateEnvironmentFile 
           with: 
             target: ./.localSettings 
             envs: 
               BROWSER: none 
               HTTPS: true 
               PORT: 53000 
               SSL_CRT_FILE: ${{SSL_CRT_FILE}} 
               SSL_KEY_FILE: ${{SSL_KEY_FILE}} 
               REACT_APP_CLIENT_ID: ${{AAD_APP_CLIENT_ID}} 
               REACT_APP_START_LOGIN_PAGE_URL: ${{TAB_ENDPOINT}}/auth-start.html 
               WDS_SOCKET_PORT: 0 
         ```
      - Update `appPackage/manifest.json`, set `showLoadingIndicator` to be `false`.
        ```json
        {
            "showLoadingIndicator": false
        }
        ```
      - Use the `Run and Debug Activity Panel` in Visual Studio Code and click the `Debug in Teams` green arrow button. 
    - For CLI: 
       - Install [ngrok](https://ngrok.com/). 
       - Sign up a ngrok account in https://dashboard.ngrok.com/signup. Copy your personal ngrok authtoken from https://dashboard.ngrok.com/get-started/your-authtoken. 
       - Start your local tunnel service by running the command `ngrok http https://localhost:53000 --authtoken=<your-personal-ngrok-authtoken> `. 
       - In the `env/.env.local` file, fill in the values for `TAB_DOMAIN` and `TAB_ENDPOINT` with your ngrok URL. 
           ```
           TAB_DOMAIN=sample-ngrok-id.ngrok.io 
           TAB_ENDPOINT=https://sample-ngrok-id.ngrok.io 
           ```
      - Update `appPackage/manifest.json`, set `showLoadingIndicator` to be `false`.
        ```json
        {
            "showLoadingIndicator": false
        }
        ```
       - Executing the command `teamsfx provision --env local` in your project directory. 
       - Executing the command `teamsfx deploy --env local` in your project directory. 
       - Executing the command `teamsfx preview --env local` in your project directory. 
1. Open the sideloading URL and install the app in the Teams website as usual. 
   ![image](https://user-images.githubusercontent.com/49138419/236614484-41aef6c4-2a4e-454f-888b-1b2e8d8b547a.png)
   > **Note:** When the dev tunnel access is set to `private`, the tab app cannot be displayed within an iframe on the web client. It is because its login page is hosted on "login.microsoftonline.com", which has the `X-FRAME-Options` header set to DENY. If you want to preview the app on the mobile client and debug it on web clients, you need to set the access level to `public`. It's worth noting that public access raises safety concerns since the tab app can be visited by anyone who knows the app's URL. 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614517-ab553360-ea54-48ee-be13-a314e8d7cc34.png)
1. Open Teams on your mobile device and click "More" to find the previewing app. 
   
   ![image](https://user-images.githubusercontent.com/49138419/236614567-b87c9d6d-6367-468a-b5fa-a9a10442f5f1.png)

   > **Note:** If a user has previously debugged the app, it is advisable for them to clear the cache on their mobile device to ensure immediate app synchronization. After clearing the cache, it may take some time for the app to sync. 
   > 
   > On iOS devices, the Teams app data can be cleared by navigating to Settings -> Teams -> Clear App Data. 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614707-8384e9a8-4675-47d0-a065-57377866a33c.png)
   > 
   > For android devices, the Teams app data can be cleared by navigating to Teams->Settings->Data and storage->Clear app data->Clear data 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614768-096f81b8-fc94-4f25-837d-cf97c588a50b.png)

1. If you are accessing the dev tunnel for the first time, you will need to sign in with your M365 account and confirm the anti-phishing page. 
   
   ![image](https://user-images.githubusercontent.com/49138419/236614829-0d9948d2-8d1a-4188-acbf-43b4179d981b.png)
   ![image](https://user-images.githubusercontent.com/49138419/236614843-d96c5de4-f6f4-4656-9da0-841848eedc1c.png)

   > **Note:** The login process should only be required once per device, and confirmation of the anti-phishing page must be completed after every installation of the app. 

1. Show a mobile friendly tab app. 
1. For Android devices, you can use [DevTools](https://learn.microsoft.com/en-us/microsoftteams/platform/tabs/how-to/developer-tools#access-devtools-from-an-android-device) to debug your tab while it is running.

## Steps for debugging a Teams bot app on mobile client

1. Start a Bot App in VSC / CLI 
   Same to the current Teams Toolkit behavior.  
1. Open the sideloading URL and install the app in the Teams website as usual. 
1. Open Teams on your mobile device and click "More" to find the previewing app. 

   ![image](https://user-images.githubusercontent.com/49138419/236614567-b87c9d6d-6367-468a-b5fa-a9a10442f5f1.png)

   > **Note:** If a user has previously debugged the bot app and the Team app manifest file is changes, it is advisable for them to clear the cache on their mobile device to ensure immediate app synchronization. After clearing the cache, it may take some time for the app to sync. 
   > 
   > On iOS devices, the Teams app data can be cleared by navigating to Settings -> Teams -> Clear App Data. 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614707-8384e9a8-4675-47d0-a065-57377866a33c.png)
   > 
   > For android devices, the Teams app data can be cleared by navigating to Teams->Settings->Data and storage->Clear app data->Clear data 
   > 
   > ![image](https://user-images.githubusercontent.com/49138419/236614768-096f81b8-fc94-4f25-837d-cf97c588a50b.png)

1. If you are debugging a bot app with SSO and accessing the dev tunnel for the first time, you will need to confirm the anti-phishing page. 
   
   ![image](https://user-images.githubusercontent.com/49138419/236614843-d96c5de4-f6f4-4656-9da0-841848eedc1c.png)
   > **Note:** The confirmation of the anti-phishing page must be completed after every installation of the app. 
1. Debug the bot app on your mobile.  
   ![image](https://user-images.githubusercontent.com/49138419/236615112-ee793006-bb63-4bd8-a76e-7d977a617d74.png)
   ![image](https://user-images.githubusercontent.com/49138419/236615116-35bc81f6-acf6-490b-b6b4-b270c6e133af.png)


 

 

 