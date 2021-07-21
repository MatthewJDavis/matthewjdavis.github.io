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

## Set up Terraform

Local state

## Create application with permissions to deploy applications

Service principal.

```bash
az ad sp create-for-rbac --skip-assignment --name 'test-terraform-ad'
```

### Required permissions

[Assign] the [application administrator] role to the service principal previously created *test-terrafrom-ad*.

You can use an MS Graph API call to set the role or via the portal as per screen shot below.

![App admin role in portal](/images/terraform-azure-ad-app/app-admin-role.png)

### Note about permissions

The current azuread Terraform provider uses the legacy Azure Active Directory Graph to interact with the API. You can use the Microsoft Graph API which is in [Beta support] by setting a flag in the provider block however you can't set the display name currently this way which is why I used a AzureAD role for the service principal.

## Create the application and deploy

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