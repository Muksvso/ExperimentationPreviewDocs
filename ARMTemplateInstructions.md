---
title: ARM template to create Experimentation setup 
description: How to use CLI and ARM template to create Experimentation setup
author: muksvso    
ms.author: mubatra
ms.service: azure-app-configuration
ms.topic: how-to 
ms.date: 22/03/2024
ms.custom: template-how-to
---

# Use ARM template for Experimentation resources

This article will help you set up the Experimentation pipeline depicted in this [Figure](./how-to-setup-experimentation.md#how-does-experimentation-work-in-azure-app-configuration). You will be using Azure CLI and ARM template for the Azure resources setup, and then use the Azure portal flow to create Feature flags and experiments.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/dotnet). Owner or Contributor and UserAdminAccess roles in the Azure subscription. This subscription should be whitelisted by Microsoft Product team to run Experimentation.

- You will need [Azure CLI](/cli/azure/install-azure-cli) installed. Sign in to Azure using the `az login` command in Azure CLI. This command will prompt your web browser to launch and load an Azure sign-in page. If the browser fails to open, use device code flow with `az login --use-device-code`. For more sign in options, go to [sign in with the Azure CLI](/cli/azure/authenticate-azure-cli).

---

## Enterprise app for Data access policy

First step is to either download or clone this repo. Then we will create and configure the Entra ID's Enterprise App setup. This Entra ID app is used for accessing the Data plane of the Split SaaS from App Configuration and Split Experimentation Workspace. ARM template for experimentation requires the App ID generated in this step.

- To create the Enterprise app registration for data access from Split
  
	```azurecli-interactive
	        az ad app create --display-name <appRegistrationName>
	```
- Output will show objectid in the oauth2Permissions. Copy the value for the object id and run the following command with the copied object id.
  
	```azurecli-interactive
		az ad sp create --id <objectid>
	```
- Check for the created application registration in **Entra Id -> App Registrations** on Azure portal. Go to **Manifest** blade, update the 'isEnabled' on line 35 to false, save the manifest. Copy the following values from the manifest,  `id` at line2, `appId` at line 7 and `name` at line 26, keep these values for use in next steps.

	![Manifest](./Images/ManifestMenu.png)

-  Go to the downloaded contents of [ARM template](./ARMTemplate/). Replace the entire manifest in portal with the content copied from the downloaded manifest.json. Make following edits in the manifest with the corresponding values copied in previous step :
    1. "id" on line 2
    1. "appId" on line 7
    1. Add the "appId" on line 43 resulting in `api://$appId`.
    1. "name" of the app registration on line 55.
        Save the manifest. You should see a manifest updated message.

- Copy the following snippet on line 81, in the `preAuthorizedApplications`, and click save. You should get a manifest updated message.
	```json
	{
		"appId": "d3e90440-4ec9-4e8b-878b-c89e889e9fbc",
		"permissionIds": [
			"b77a34fd-bafc-45e0-8907-a6098d55c56d"
		]
	},
	{
		"appId": "1e2401ea-428f-4575-9bbf-b301f7e1eb67",
		"permissionIds": [
			"b77a34fd-bafc-45e0-8907-a6098d55c56d"
		]
	},
	{
		"appId": "73b67c52-525b-4470-9c5c-1e02c60b8a05",
		"permissionIds": [
			"b77a34fd-bafc-45e0-8907-a6098d55c56d"
		]
	}
	```
- Click on **Overview** blade and copy the value for `Application (client) ID` in Essentials, save the value for later use. Open the linked Enterprise application in the `Managed application in local directory` in Essentials.

	![Overview](./Images/Overview_EApp.png)

-  In the Enterprise Application, you can now assign the users and/or groups to the app-roles in the application.

   	![Users and groups](./Images/UsersinEapp.png)

## ARM template to setup the Experimentation resources

- Create a resource group for Experimentation, in the subscription enabled for Preview. Set the subscription as default subscription for the CLI.
  
	```azurecli-interactive
		az group create --location <location> --name <resource_group_name>
	```

- Check the downloaded files from the [ARM Template(./ARMTemplate)] folder, it has two more jsons, a parameters file for the ARM template and the template itself. Open ExperimentationPreviewParams.json, edit the value for "SEWEntraApplicationId" with the `application id` saved in the App registration step. Update the "RGname" parameter to have the name of the resource group you created in previous step. Run the deployment command on cli.

	```azurecli-interactive
	        az deployment group create --resource-group <resource_group_name> --template-file <templatefile> --parameters <paramsfile>
	```
Successful deployment of this template will give you the complete Experimentation pipeline. For running an experiment in the App configuration store, follow the **How to set up experimentation** from [step 3](./how-to-setup-experimentation.md#step-3-create-a-variant-feature-flag-and-enable-telemetry) onwards.  

## Next steps

[Instruction Set for Experimentation](./how-to-setup-experimentation.md#step-3-create-a-variant-feature-flag-and-enable-telemetry)
