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

Local state

## Create the application and deploy

Create two files main.tf and a terraform.tfvars where the variable values are declared.

Main file.

<script src="https://gist.github.com/MatthewJDavis/226f178381f09f1dc87bfcc8fb3e28f0.js"></script>

Variables file.

<script src="https://gist.github.com/MatthewJDavis/69bd18c079b2f7026f637e6674fac03c.js"></script>

## Update the application

## Destroy the application

## Azure storage for remote state

##

[Assign]: https://docs.microsoft.com/en-us/azure/active-directory/roles/manage-roles-portal
[Application Administrator]: https://docs.microsoft.com/en-us/azure/active-directory/roles/permissions-reference#application-administrator
[Beta support]: https://registry.terraform.io/providers/hashicorp/azuread/latest/docs/guides/microsoft-graph#beta-support-for-microsoft-graph-in-v150
[AAD portal]: https://aad.portal.azure.com