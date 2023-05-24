Azure SQL Database is an always-up-to-date, fully managed relational database service built for the cloud. You can easily build applications with Azure SQL Database and continue to use the tools, languages, and resources you're familiar with.

# Steps to create Azure SQL Database

Teams Toolkit orchestrates cloud service provision and configuration with an infrastructure as code approach using a Domain Specific Language called [Bicep](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview?tabs=bicep).

You can follow these steps to add Azure SQL Database to your app with bicep:
1. Step 1: [Add Azure SQL Database declaration to your bicep file](#step-1-add-azure-sql-database-declaration-to-your-bicep-file)
2. Step 2: [Add parameters for Azure SQL Database bicep snippet](#step-2-add-parameters-for-azure-sql-database-bicep-snippet)
3. Step 3: [Connect your computing resource to Azure SQL Database](#step-3-connect-your-computing-resource-to-azure-sql-database)
4. Step 4: [Add code to connect to Azure SQL Database](#step-4-add-code-to-connect-to-azure-sql-database)
4. Step 5: [Update your cloud infrastructure](#step-5-update-your-cloud-infrastructure)

## Step 1: Add Azure SQL Database declaration to your bicep file

After you created a project using Teams Toolkit, the bicep file is usually located at `infra/azure.bicep`. Open this file and append following content to it. If your project is not created using Teams Toolkit, append the content to your own bicep file.

```
@description('The name of the SQL logical server.')
param serverName string = uniqueString('sql', resourceGroup().id)

@description('The name of the SQL Database.')
param sqlDBName string = 'SampleDB'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('The administrator username of the SQL logical server.')
param administratorLogin string

@description('The administrator password of the SQL logical server.')
@secure()
param administratorLoginPassword string

resource sqlServer 'Microsoft.Sql/servers@2021-08-01-preview' = {
  name: serverName
  location: location
  properties: {
    administratorLogin: administratorLogin
    administratorLoginPassword: administratorLoginPassword
  }
}

resource sqlDB 'Microsoft.Sql/servers/databases@2021-08-01-preview' = {
  parent: sqlServer
  name: sqlDBName
  location: location
  sku: {
    name: 'Standard'
    tier: 'Standard'
  }
}

// Allow Azure services connect to the SQL Server
resource sqlFirewallRules 'Microsoft.Sql/servers/firewallRules@2021-08-01-preview' = {
  parent: sqlServer
  name: 'AllowAzure'
  properties: {
    endIpAddress: '0.0.0.0'
    startIpAddress: '0.0.0.0'
  }
}

```

> Note: above content generates a server name based on your resource group name and sets database name to `SampleDB` by default. If you want to change the names, you can set additional parameters `serverName` and `sqlDBName` in step 2.

<p align="right"><a href="#steps-to-create-azure-sql-database">back to top</a></p>

## Step 2: Add parameters for Azure SQL Database bicep snippet

We need to add some required parameters for the bicep snippet in step 1.
1. Add following parameter to bicep's parameter file and set the value of `administratorLogin` to desired login name. For projects created using Teams Toolkit, the parameter file usually located at `infra/azure.parameters.json`.
   ```
    "administratorLogin": {
      "value": ""
    },
    "administratorLoginPassword": {
      "value": "${{SQL_ADMIN_PASSWORD}}"
    }
   ```
2. Add following content to `env/.env.{env_name}.user` and set the value of `SQL_ADMIN_PASSWORD`
   ```
   SQL_ADMIN_PASSWORD=
   ```
> Note: ${{ENV_NAME}} is a special placeholder supported by Teams Toolkit, which references the value of an environment variable. You can replace the real values in `azure.parameters.json` with this placeholder and set the environment variable values in `env/.env.{env_name}`, `env/.env.{env_name}.user` or set to your machine's environment variable directly. The folder of .env files is controlled by `environmentFolderPath` property in `teamsapp.yml`. The folder name is `env` by default but may be customized after project creation.

<p align="right"><a href="#steps-to-create-azure-sql-database">back to top</a></p>

## Step 3: Connect your computing resource to Azure SQL Database

There are 2 ways to connect to your Azure SQL Database in Azure: using username/password and using Azure Managed Identity.

### Connect using username/password

To connect to Azure SQL Database using the traditional username/password way, you can compose the connection string based on your programming language and library, then configure the connection string to your computing resource. For example, you can configure your connection string to Azure App Service using bicep as below:
```
resource webApp 'Microsoft.Web/sites@2021-02-01' = {
  kind: 'app'
  location: location
  name: webAppName
  properties: {
    serverFarmId: serverfarm.id
    siteConfig: {
      appSettings: [
        // other app settings...
        {
          name: 'SQL_SERVER_ENDPOINT'
          value: sqlServer.properties.fullyQualifiedDomainName
        }
        {
          name: 'SQL_DATABASE_NAME'
          value: sqlDBName
        }
        {
          name: 'SQL_USERNAME'
          value: administratorLogin // this is only used for demonstration purpose. DO NOT use admin credential to connect your SQL databases
        }
        {
          name: 'SQL_PASSWORD'
          value: administratorLoginPassword // this is only used for demonstration purpose. DO NOT use admin credential to connect your SQL databases
        }
      ]
    // other site configs...
    }
  }
}
```

### Connect using Azure Managed Identity

Managed identities provide an automatically managed identity in Azure Active Directory for applications to use when connecting to resources that support Azure Active Directory (Azure AD) authentication.

You can refer this document to understand how to connect to Azure SQL Database using Managed Identity: https://learn.microsoft.com/en-us/azure/app-service/tutorial-connect-msi-azure-database

<p align="right"><a href="#steps-to-create-azure-sql-database">back to top</a></p>

## Step 4: Add code to connect to Azure SQL Database

After you included Azure SQL Database related app settings in bicep file, you can follow this tutorial to connect your app to Azure SQL Database: https://learn.microsoft.com/en-us/azure/azure-sql/database/connect-query-nodejs?view=azuresql&tabs=windows. This tutorial is for nodejs, you can use `process.env.SQL_SERVER_ENDPOINT`, `process.env.SQL_DATABASE_NAME`, `process.env.SQL_USERNAME` and `process.env.SQL_PASSWORD` to reference the app settings configured to your Azure App Service in step 3.

If you are using other programming language, you can find tutorials for your programming language in the website's table of content.

<p align="right"><a href="#steps-to-create-azure-sql-database">back to top</a></p>

## Step 5: Update your cloud infrastructure and deploy your app

After you updated bicep file for your project, you need to run `Teams: Provision` command in VS Code extension to apply your changes to bicep file.

After you updated your source code, you need to run `Teams: Deploy` command in VS Code extension to deploy your code to cloud.

<p align="right"><a href="#steps-to-create-azure-sql-database">back to top</a></p>


