Azure Key Vault is a secure storage solution to manage secrets, keys and certificates. It can be used to centralize application secrets, securely store secrets and keys, monitor access and use as well as simplify administration of application secrets. 

## Azure Key Vault provision and configuration

Teams Toolkit orchestrates cloud service provision and configuration with an infrastructure as code approach using a Domain Specific Language called [Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview?tabs=bicep).

Follow these steps to provision a new Azure Key Vault service with Teams Toolkit:

1. [Step 1: Create a new bicep file](#step-1-create-a-new-bicep-file)
1. [Step 2: Update existing bicep file](#step-2-update-existing-bicep-file)
1. [Step 3: Execute provision command](#step-3-execute-provision-command)

### Step 1: Create a new bicep file

Create a bicep file called `keyVault.bicep` under `infra` folder with below content for provisioning Aszure Key Vault service.

```bicep
param keyVaultName string
param secretName string
@secure()
param secret string
param identityObjectId string

var tenantId = subscription().tenantId


resource keyVault 'Microsoft.KeyVault/vaults@2019-09-01' = {
  name: keyVaultName
  location: resourceGroup().location
  properties: {
    tenantId: tenantId
    accessPolicies: []
    sku: {
      name: 'standard'
      family: 'A'
    }
  }
}

resource keyVaultAccessPolicy 'Microsoft.KeyVault/vaults/accessPolicies@2019-09-01' = {
  name: '${keyVaultName}/add'
  properties: {
    accessPolicies: [
      {
        tenantId: tenantId
        objectId: identityObjectId
        permissions: {
          secrets: [
            'get'
          ]
        }
      }
    ]
  }
  dependsOn: [
    keyVault
  ]
}

resource secretKv 'Microsoft.KeyVault/vaults/secrets@2019-09-01' = {
  parent: keyVault
  name: secretName
  properties: {
    value: secret
  }
}
```


### Step 2: Update existing bicep file

Update existing `azure.bicep` file under `infra` folder.

1. Add below content for provisioning user-assigned managed identity and Azure Key Vault, and update `<The secret to be stored in Key Vault>`:

    ```bicep
    var keyVaultName = resourceBaseName
    var secretName = 'secret'
    var secretReference = '@Microsoft.KeyVault(VaultName=${keyVaultName};SecretName=${secretName})'

    resource managedIdentity 'Microsoft.ManagedIdentity/userAssignedIdentities@2018-11-30' = {
    name: resourceBaseName
    location: resourceGroup().location
    }

    module keyVaultProvision './keyVault.bicep' = {
    name: 'keyVaultProvision'
    params: {
        keyVaultName: keyVaultName
        secretName: secretName
        secret: <The secret to be stored in Key Vault>
        identityObjectId: managedIdentity.properties.principalId
    }
    }
    ```

1. Update the existing resource for accessing Azure Key Vault.
    
    E.g. If it is a Bot or Function project hosted on Azure Web App, you need to update the bicep content of `webApp`:
    
    1. Add below content under `resource webApp`:

        ```bicep
        identity: {
            type: 'UserAssigned'
            userAssignedIdentities: {
            '${managedIdentity.id}': {}
            }
        }
        dependsOn: [ keyVaultProvision ]
        ```

    1. Add below content under `properties` of `resource webApp`:

        ```bicep
        keyVaultReferenceIdentity: managedIdentity.id
        ```

1. Update the secret value to Key Vault secret reference. E.g. If it is a Bot project, update for the value of `BOT_PASSWORD` under `appSettings` of `resource webApp`:

    ```bicep
    {
        name: 'BOT_PASSWORD'
        value: secretReference
    }
    ```

### Step 3: Execute provision command

Follow this [document](https://learn.microsoft.com/microsoftteams/platform/toolkit/provision?pivots=visual-studio-code) to provision cloud resources.