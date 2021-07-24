---
title: Use Terraform to deploy an Azure AD application
excerpt: 
date: 2021-07-03
toc: false
classes: wide
categories:
- terraform
tags:
- azuread
- terraform
published: false
---
July 2021

# Overview

How to deploy an Azure AD application that has a preconfigured secret using Terraform and assign permissions to the Microsoft Graph with User.Read scope.

## Create application with permissions to deploy applications

First step is we need to authenticate with the Azure API to be able to perform actions such as create, update and remove resources. In this instance a service principal will be created which authenticates non interactive so it can later be used in a CI/CD pipeline

The easiest way to create a service principal is the Azure CLI.

```bash
az login
az ad sp create-for-rbac --skip-assignment --name 'test-terraform-ad'
```

After running the above command, appID and password are displayed, keep a note of these somewhere secure such as a password manager.

Now a service principal will be created in Azure AD. You can view this by going to the [AAD portal], selecting the **Enterprise applications** blade, changing the **Application type** to **All Applications** then searching.

![Service principal in portal](/images/terraform-azure-ad-app/sp-in-portal.png)

### Required permissions

[Assign] the [application administrator] role to the service principal previously created *test-terrafrom-ad*.

You can use an MS Graph API call to set the role or via the portal as per screen shot below.

![App admin role in portal](/images/terraform-azure-ad-app/app-admin-role.png)

### Note about permissions

The current azuread Terraform provider uses the legacy Azure Active Directory Graph to interact with the API. You can use the Microsoft Graph API which is in [Beta support] by setting a flag in the provider block however you can't set the display name currently this way which is why I used a AzureAD role for the service principal.

## Set up Terraform

Create a new directory, and change directory to it. 

I create ```azure-tf-demo```

Create two files main.tf and a terraform.tfvars where the variable values are declared.

Main file.

<script src="https://gist.github.com/MatthewJDavis/226f178381f09f1dc87bfcc8fb3e28f0.js"></script>

Variables file.

<script src="https://gist.github.com/MatthewJDavis/69bd18c079b2f7026f637e6674fac03c.js"></script>

To initialize Terraform run the following from the directory with the main and terraform.tfvars are located.

```bash
terraform init
```

This will store the state locally on your machine in the same directory.

![output of terraform init](/images/terraform-azure-ad-app/terraform-init.png)

Validate everything is good by running:

```bash
terraform validate
```

There will be a warning regarding the attribute *homepage* being deprecated and will change names in version 2.

Now it's time to set the credentials via environment variables. This is appID (ARM_CLIENT_ID below) and password for the ARM_CLIENT_SECRET. A subscription ID and tenant ID is also required.

You can get the subscription and tenant IDs from the Azure CLI:

```bash
az account list
```

Setting the authentication environment variables:

```bash
export ARM_SUBSCRIPTION_ID="abcde"
export ARM_TENANT_ID="abcde"
export ARM_CLIENT_ID="abcde"
 export ARM_CLIENT_SECRET="abcde"
```

The secret line is intentionally indented to keep it out of the bash history.

The last thing to set is the password value for the application. Note: When the Terraform provider is updated to version 2.0 and used the MS Graph API, this will be read only and will be generated so will no longer applies.
The password is set via an environment variable again

```bash
 export TF_VAR_app_password="randomPasswordHere"
```

Now it's time to run a plan.

```bash
terraform plan -out deployment.tfplan
```

If all looks good, it can be deployed using the output from the above plan command.

```bash
terraform apply -auto-approve deployment.tfplan
```

After the deployment has finished- check the portal to see it there.

![app in portal](/images/terraform-azure-ad-app/app-in-portal.png)


## Update the application

## Destroy the application

## Azure storage for remote state

##

[Assign]: https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal
[Application Administrator]: https://docs.microsoft.com/en-us/azure/active-directory/roles/permissions-reference#application-administrator
[Beta support]: https://registry.terraform.io/providers/hashicorp/azuread/latest/docs/guides/microsoft-graph#beta-support-for-microsoft-graph-in-v150
[AAD portal]: https://aad.portal.azure.com