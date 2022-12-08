Teams Toolkit helps you to access existing APIs for building Teams applications. These APIs are developed by your organization or third-party. 

# Requirement
Make sure that your project contains backend service, such as Azure Function or Azure bot service.

# Steps to add Api Connction
The following steps help you to add API connection using Visual Studio Code:
It is recommended to use an alias to distinguish multiple different api connections, the alias will be used in both sample code and env variables.
1. Add a reference to the `@microsoft/teamsfx` package to `package.json`.
1. Give a unique alias for API connection and modify your project with reference to the sample code.
1. Refer to the sample code to add application settings for your API in `./teamsfx/.env.local` that configures local debugging.
---
## Sample Code and Application Settings
The following sample code and settings assume the alias for API connection is `kudos`. Please replace the alias in your code.
<details>
<summary><b>Basic Auth
</b></summary>

Sample code for Basic Auth
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");
// Load application configuration
const teamsFx = new teamsfxSdk.TeamsFx();
// Initialize a new axios instance to call kudos
const authProvider = new teamsfxSdk.BasicAuthProvider(
  process.env.TEAMSFX_API_KUDOS_USERNAME,
  process.env.TEAMSFX_API_KUDOS_PASSWORD
);
const kudosClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_KUDOS_ENDPOINT,
  authProvider
);
module.exports.kudosClient = kudosClient;
```
Add application settings for your API to `teamsfx/.dev.local`
```
TEAMSFX_API_KUDOS_ENDPOINT=
TEAMSFX_API_KUDOS_USERNAME=
TEAMSFX_API_KUDOS_PASSWORD=
```
</details>
<details>
<summary><b>Certification
</b></summary>

Sample code for Certification
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// Load application configuration
const teamsFx = new teamsfxSdk.TeamsFx();
// Initialize a new axios instance to call kudos
const authProvider = new teamsfxSdk.CertificateAuthProvider(
  // TODO: 
  // 1. Add code to read your certificate and private key.
  // 2. Replace "<your-cert>" and "<your-private-key>" with your actual certificate and private key values
  // If you have a .pfx certificate, you can use the `createPfxCertOption` function to initialize your certificate
  teamsfxSdk.createPemCertOption("<your-cert>", "<your-private-key>")
);
const kudosClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_KUDOS_ENDPOINT,
  authProvider
);
module.exports.kudosClient = kudosClient;
```
Add application settings for your API to `teamsfx/.dev.local`
```
TEAMSFX_API_KUDOS_ENDPOINT=
```
</details>
<details>
<summary><b>Azure Active Directory
</b></summary>

There are 2 scenarios here, please choose one of them. 
- Scenario 1 is reusing the project AAD app, make sure your project contains an existing AAD app.
- Scenario 2 is using an existing AAD App.

```javascript
const teamsfxSdk = require("@microsoft/teamsfx");
// There are 2 scenarios here, please choose one of them. This sample uses the client credential flow to acquire a token for your API.
// Scenario 1. reuse the project AAD app.
const teamsFx = new teamsfxSdk.TeamsFx(teamsfxSdk.IdentityType.App, {
  authorityHost: process.env.AAD_APP_OAUTH_AUTHORITY_HOST,
  tenantId: process.env.AAD_APP_TENANT_ID,
  clientId: process.env.AAD_APP_CLIENT_ID,
  clientSecret: process.env.SECRET_AAD_APP_CLIENT_SECRET,
});
// Scenario 2. use an existing AAD App.
const teamsFx = new teamsfxSdk.TeamsFx(teamsfxSdk.IdentityType.App, {
  // You can replace the default authorityHost URL
  authorityHost: "https://login.microsoftonline.com",
  tenantId: process.env.TEAMSFX_API_KUDOS_TENANT_ID,
  clientId: process.env.TEAMSFX_API_KUDOS_CLIENT_ID,
  // This references the client secret that you must add in the file `.env.teamsfx.local`.
  clientSecret: process.env.TEAMSFX_API_KUDOS_CLIENT_SECRET,
});
// Initialize a new axios instance to call kudos
const appCredential = teamsFx.getCredential();
const authProvider = new teamsfxSdk.BearerTokenAuthProvider(
  // TODO: Replace '<your-api-scope>' with your required API scope
  async () => (await appCredential.getToken("<your-api-scope>")).token
);
const kudosClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_KUDOS_ENDPOINT,
  authProvider
);
module.exports.kudosClient = kudosClient;
```
Add application settings for your API to `teamsfx/.dev.local`
```
// Must have
TEAMSFX_API_KUDOS_ENDPOINT=
// Scenario 2
TEAMSFX_API_KUDOS_TENANT_ID=
TEAMSFX_API_KUDOS_CLIENT_ID=
TEAMSFX_API_KUDOS_CLIENT_SECRET=
```
</details>
<details>
<summary><b>API Key
</b></summary>

Sample code for API Key
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// Load application configuration
const teamsFx = new teamsfxSdk.TeamsFx();
// Initialize a new axios instance to call kudos, store API key in request header.
const authProvider = new teamsfxSdk.ApiKeyProvider(
  "{API-KEY-name}",
  process.env.TEAMSFX_API_KUDOS_API_KEY,
  teamsfxSdk.ApiKeyLocation.Header
);
// or store API key in request params.
const authProvider = new teamsfxSdk.ApiKeyProvider(
  "{API-KEY-name}",
  process.env.TEAMSFX_API_KUDOS_API_KEY,
  teamsfxSdk.ApiKeyLocation.QueryParams
);
const kudosClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_KUDOS_ENDPOINT,
  authProvider
);
module.exports.kudosClient = kudosClient;
```
Add application settings for your API to `teamsfx/.dev.local`
```
TEAMSFX_API_KUDOS_ENDPOINT=
TEAMSFX_API_KUDOS_API_KEY=
```
</details>
<details>
<summary><b>Custom Auth Implementation
</b></summary>

Sample code for Custom Auth Implementation
```javascript
const teamsfxSdk = require("@microsoft/teamsfx");

// A custom authProvider implements the `AuthProvider` interface.
// This sample authProvider implementation will set a custom property in the request header
class CustomAuthProvider {
  customProperty;
  customValue;

  constructor(customProperty, customValue) {
    this.customProperty = customProperty;
    this.customValue = customValue;
  }

  // Replace the sample code with your own logic.
  AddAuthenticationInfo = async (config) => {
    if (!config.headers) {
      config.headers = {};
    }
    config.headers[this.customProperty] = this.customValue;
    return config;
  };
}

// Load application configuration
const teamsFx = new teamsfxSdk.TeamsFx();

const authProvider = new CustomAuthProvider(
  // You can also add configuration to the file `.env.teamsfx.local` and use `process.env.{setting_name}` to read the configuration. For example:
  //  process.env.TEAMSFX_API_KUDOS_CUSTOM_PROPERTY,
  //  process.env.TEAMSFX_API_KUDOS_CUSTOM_VALUE
  "customPropery",
  "customValue"
);
// Initialize a new axios instance to call kudos
const kudosClient = teamsfxSdk.createApiClient(
  process.env.TEAMSFX_API_KUDOS_ENDPOINT,
  authProvider
);
module.exports.kudosClient = kudosClient;
```
Add application settings for your API to `teamsfx/.dev.local`
```
TEAMSFX_API_KUDOS_ENDPOINT=
```
</details>