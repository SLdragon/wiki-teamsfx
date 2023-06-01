# Introduction
In Azure App Service, you can run your apps directly from a deployment ZIP package file. This article shows how to enable/disable this functionality in your app.

# Advantages and Limitations
When you enable this functionality in your Azure App Service, the files in the package are not copied to the wwwroot directory. Instead, the ZIP package itself gets mounted directly as the read-only wwwroot directory. There are several benefits to running directly from a package:

* Eliminates file lock conflicts between deployment and runtime.
* Ensures only full-deployed apps are running at any time.
* Can be deployed to a production app (with restart).
* Improves the performance of Azure Resource Manager deployments.
* May reduce cold-start times, particularly for JavaScript functions with large npm package trees.

There are also some limitations to running from a package:
* The wwwroot folder becomes read-only and you'll receive an error when writing files to this directory. Files are also read-only in the Azure portal.
* The maximum size for a deployment package file is currently 1 GB.
* Can't use local cache or local DB when running from a deployment package.
* Can't use remote build

You can find more details in the following links:

https://learn.microsoft.com/en-us/azure/app-service/deploy-run-package

https://github.com/projectkudu/kudu/wiki/WEBSITE_RUN_FROM_PACKAGE-and-WEBSITE_CONTENTAZUREFILECONNECTIONSTRING-Best-Practices

# How to enable run from package
1. Make sure that the version of Node.js on your machine is the same as the Node.js version on App Service.
1. Check your .webappignore file at the project root, make sure all the dependencies that running required are not excluded by this file.
1. Enable `WEBSITE_RUN_FROM_PACKAGE` app setting in your Azure App Service.
   * Add WEBSITE_RUN_FROM_PACKAGE flag to your bicep file.
   * Follow [the documentation](https://learn.microsoft.com/en-us/azure/app-service/deploy-run-package#enable-running-from-package) do it manually.
   * Go to Azure Portal. Navigate to your App Service - Setting - Configuration - Applications settings, + New application setting, Name: WEBSITE_RUN_FROM_PACKAGE, Value: 1, Click OK.
1. **Make sure there is no `SCM_SCRIPT_GENERATOR_ARGS` setting in your Azure App Service**. If your project is upgraded from the old version, this setting may be set to `--node`. Remove this setting before deploying your project.
1. Put the `web.config` file to the root of the project if your project is using nodeJS. You can find the template file from [here](https://github.com/projectkudu/kudu/blob/master/Kudu.Core/Scripts/iisnode.config.template) and example from [here](https://github.com/Azure-Samples/nodejs-docs-hello-world/blob/master/web.config). If your project is built by Dotnet, skip this step.

# How to disable run from package
1. Remove `WEBSITE_RUN_FROM_PACKAGE` app setting from your Azure App Service.
1. If your project is built by nodeJS and you already have a `web.config` file in your project root, you can keep this file and everything is fine now. If you want to delete the `web.config` file, remember to [enable the build process](https://learn.microsoft.com/en-us/azure/app-service/deploy-zip?tabs=cli#enable-build-automation-for-zip-deploy) or set `SCM_SCRIPT_GENERATOR_ARGS=--node` (This option will also need you to upload all the dependencies to the Azure App Service).
1. Adjust your .webappignore file at the project root, make sure all source code is not excluded by this file. This file excludes *.ts files by default. If you enabled the build process during deploy, you should upload all the source code to the App Service. If you use `SCM_SCRIPT_GENERATOR_ARGS=--node` option or keep the `web.config` at the project root, skip this step.

# Q&A
## Deployment failed, and a `FAILD TO INITIALIZED RUN FROM PACKAGE.txt` file was in my wwwroot folder.
Make sure you have removed the `COMMAND` and `SCM_SCRIPT_GENERATOR_ARGS` settings in your Azure App Service.

## You do not have permission to view this directory or page
Please refer to [this page](https://learn.microsoft.com/en-us/azure/app-service/configure-language-nodejs?pivots=platform-windows#you-do-not-have-permission-to-view-this-directory-or-page) to find solve.