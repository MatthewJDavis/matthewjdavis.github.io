---
title: Use Terraform to deploy an Azure AD application
excerpt: Use Terraform to deploy an Azure AD application and set MS Graph permissions and retrieve the secret.
date: 2021-07-20
toc: false
classes: wide
categories:
- terraform
tags:
- azuread
- terraform
published: true
---
July 2021

# Overview

This post will cover how to deploy an Azure AD application using Terraform and assign it permissions to the Microsoft Graph with User.Read scope. I'm using the latest version of Terraform (currently 1.0.3), latest version of the [AzureAD provider] (currently 1.0.6) and running on Ubuntu 18.04. Although this was carried out on a Ubuntu machine, it should be very similar for Windows or Mac.

The current AzureAD Terraform provider (1.0.6) uses the [legacy Azure Active Directory Graph] to interact with the API by default. Version 2 of the provider will be updated to use the [Microsoft Graph] API. For this post I've decided to use the Microsoft Graph which is currently in [Beta support] for this provider version but is not advised for production use.

This isn't a beginners guide to Terraform or AzureAD, more of a post of how I have deployed to AzureAD with Terraform so a basic understanding of both would be beneficial.

## Create application with permissions to deploy applications

The first step is we need to authenticate to be able to perform actions such as create, update and remove resources.

The easiest way to create a service principal is the Azure CLI. The [Azure CLI can be installed locally] or you can use it from the [Azure cloud shell] from the browser.

```bash
az login
az ad sp create-for-rbac --skip-assignment --name 'test-terraform-ad'
```

After running the above command, appID and password are displayed, keep a note of these somewhere secure such as a password manager.

Alternatively there is the [guide on creating a service principal in the portal], however the Azure CLI is much quicker and easier.

Now a service principal will be created in Azure AD. You can view this by going to the [AzureAD portal], selecting the **Enterprise applications** blade, changing the **Application type** to **All Applications** then searching tor the test-terraform-ad service principal.

![Service principal in portal](/images/terraform-azure-ad-app/sp-in-portal.png)

Note: Because we [authenticate interactively] with ```az login```, Terraform would be authenticated in that session but I want to demonstrate how to do this with Service Principal credentials so they can be used in a CI/CD pipeline in the future.

### Required permissions

[Assign] the [application administrator] role to the service principal previously created *test-terrafrom-ad*.

You can use the [Microsoft Graph] API to set the role or via the portal as per screen shot below.

Go to **Azure Active Directory** then the **Roles and administrators** blade. Click on the **Application administrator** role, then the **+ Add assignments** button and then **Select Members** link. Search for *test-terrafrom-ad* and confirm.

![App admin role in portal](/images/terraform-azure-ad-app/app-admin-role.png)

## Set up Terraform

Create a new directory locally, and change directory to it.

```bash
mkdir azure-tf-demo
cd azure-tf-demo
touch main.tf
touch terraform.tfvars
```

Copy the contents of the following files.

Main file (main.tf).

<script src="https://gist.github.com/MatthewJDavis/226f178381f09f1dc87bfcc8fb3e28f0.js"></script>

Variable values file (terraform.tfvars).

<script src="https://gist.github.com/MatthewJDavis/69bd18c079b2f7026f637e6674fac03c.js"></script>

### Setting the credentials

Set the credentials of the Service Principal in authentication environment variables:

```bash
export ARM_SUBSCRIPTION_ID='abcde'
export ARM_TENANT_ID="abcde"
export ARM_CLIENT_ID="abcde"
 export ARM_CLIENT_SECRET="abcde"
```

To initialize Terraform run the following:

```bash
terraform init
```

This will store the [state] locally on your machine in the same directory.

![output of terraform init](/images/terraform-azure-ad-app/terraform-init.png)

Validate everything is good by running:

```bash
terraform validate
```

The secret line is intentionally indented to keep it out of the bash history.

## Plan the deployment

Now it's time to run a plan to see what is going to be created. The plan is save to a file locally and the will be used to deploy the resources.

```bash
terraform plan -out deployment.tfplan
```

## Deploying the application

If all looks good, it can be deployed using the output from the above plan command.

```bash
terraform apply -auto-approve deployment.tfplan
```

In the AzureAD portal under **Azure Active Directory** blade **App registrations** blade and **All applications** search for the created app.

![app in portal](/images/terraform-azure-ad-app/app-in-portal.png)

Select the app then the **API permissions** blade to see the *User.Read* scope granted to the app.

![user read api permissions granted](/images/terraform-azure-ad-app/api-permissions.png)

## The application password

Previously with the legacy Azure API you could specify the application secret however with the Microsoft Graph API, the secret is generated. To find the generated value, look in the ```terraform.tfstate``` file. Under the "type": "azuread_service_principal_password" you'll see "attributes" and then "value" which has the password. This is why it is very important to protect the Terraform state file (it can contain [sensitive information]), especially when working with a remote backend, it should be encrypted at rest and limited access applied to the backend.

## Update the application

Now the deployment has been made, the plan is stale and needs updating.

```bash
terraform plan -out deployment.tfplan
```

Running an apply command again will result in nothing being changed.

Change the name of the value in the tfvariables.tf file to something else and run:

```bash
terraform plan -out deployment.tfplan
```

A change will be displayed.

Run the apply command to update the application.

```bash
terraform apply -auto-approve deployment.tfplan
```

Refresh the **Overview** page and you'll see the new Application name.

![update name displaying in portal](/images/terraform-azure-ad-app/change.png)

## Destroy the application

Clean up the application and service principal using the destroy command.

```bash
terraform destroy
```

Because the auto-approve option is not specified, a prompt will ask to confirm.

The resources managed by Terraform will be deleted.

To remove the service principal created earlier run the Azure CLI:

```bash
az ad sp delete --id id-of-sp-created-earlier
```

Now all the resources for this post will be removed.

[Terraform]: https://www.terraform.io/downloads.html
[guide on creating a service principal in the portal]: https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-create-service-principal-portal#register-an-application-with-azure-ad-and-create-a-service-principal
[Azure CLI can be installed locally]: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
[Azure cloud shell]: https://docs.microsoft.com/en-us/azure/cloud-shell/overview
[Assign]: https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal
[Application Administrator]: https://docs.microsoft.com/en-us/azure/active-directory/roles/permissions-reference#application-administrator
[Beta support]: https://registry.terraform.io/providers/hashicorp/azuread/latest/docs/guides/microsoft-graph#beta-support-for-microsoft-graph-in-v150
[AzureAD portal]: https://aad.portal.azure.com
[Microsoft Graph]: https://docs.microsoft.com/en-us/graph/overview
[AzureAD provider]: https://registry.terraform.io/providers/hashicorp/azuread/latest/docs
[legacy Azure Active Directory Graph]:https://docs.microsoft.com/en-us/graph/migrate-azure-ad-graph-planning-checklist
[authenticate interactively]: https://registry.terraform.io/providers/hashicorp/azuread/latest/docs/guides/azure_cli
[state]: https://www.terraform.io/docs/language/state/index.html
[sensitive information]: https://www.terraform.io/docs/language/state/sensitive-data.html