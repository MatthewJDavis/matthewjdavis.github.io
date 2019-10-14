---
title: Create VM images in Azure with Packer
excerpt: How to use Packer with Azure to create custom machine images.
categories:
- packer
tags:
- azure
- packer
published: false
---
August 2019

# Overview

Packer is a tool created by Hashicorp that allows you to create custom images (VMs or containers) for a wide variety of platforms including AWS, Vagrant, Virtual Box and Azure (the focus of this post). 

I have been creating custom images for AWS (AMI) for a while and went through the same process for Azure to see the difference and work out how to get it configured, below is the notes I used to successfully build Ubuntu images for Azure using Packer and a basic packer build script to get started with.

[Packer Azure Resource Manager documents] are well written and provide lots of details on the options available.

This was run using Ubuntu 18.04 LTS, with PowerShell Core 6.2.3 and AZ CLI 2.0.74.

## Set up

### Authentication

You can authenticate to Azure for building with packer 2 ways:

1. Interactive authentication
2. A service principal

**Interactive**

The Packer Azure builder will prompt you to authenticate via a web browser if you have the following 3 variables set:

1. subscription_id
2. managed_image_name
3. resource_group_name

![auth to azure interactive](/images/packer-azure/auth-to-azure.png)

When you run packer build, a code and web address will be shown for you to authenticate and once successful and token will be issued and you'll be able to run the build.

**Azure Service Principal**

An Azure Service Principal should be created for automation purposes. Details on how to create the service principal can be found in the [azure documentation].

As this is my own account for testing, I gave the service principal contributor role access to my subscription via the Azure CLI but for a production account, a customised RBAC should be used so the service principal just has the right amount of access required to run the builds.

```bash
az ad sp create-for-rbac -n "Packer-Matt-Visual-Studio-Subscription-Only" --role contributor --scopes /subscriptions/xxxxxxxx-xxxxxxx-xxxxx-xxxxx-xxxxxxx
```

[create for rbac docs]

### Resource Group

An Azure resource group is required before the packer build runs to store the images that are created.

A resource group can be created in the portal, by the az cli or PowerShell as shown below.

```powershell
$packerRGName = 'packerImageBuilds'
$tags = @{'Environment' = 'Dev'; 'Description' = 'Resource group where images built by packer are stored' }
$resourceGroup = New-AzResourceGroup -Name $Name -Location $location -Tag $tags -Verbose

# cli
az group create --location northeurope --name packerImageBuilds --tags 'Environment=Dev' 'Description=Images produced by Packer builds'
```

### Variables

Values to be passed to the packer build template on validate or build are set as environment variables.

```bash
# set env variables for interactive
export ARM_SUBSCRIPTION_ID=xxxxx-xxxx-xxxx-xxx-xxx
export MANAGED_IMAGE_NAME=ubuntu-18-04-lts-pwsh-v1
export RESOURCE_GROUP_NAME=packerImageBuilds

# set environment variables for service principal

export ARM_CLIENT_ID=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_CLIENT_SECRET=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_SUBSCRIPTION_ID=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_TENANT_ID=xxxxx-xxxx-xxxx-xxx-xxx
export MANAGED_IMAGE_NAME=ubuntu-18-04-lts-pwsh-v1
export RESOURCE_GROUP_NAME=packerImageBuilds
```

Below is an example of how the environment variables are processed in the packer template. Environment variables are passed to the variables set at the top of the packer template and then they become 'user' variables and passed to the builders.

```json
  "variables": {
    "subscription_id": "{{env `ARM_SUBSCRIPTION_ID`}}",
    "managed_image_name": "{{env `MANAGED_IMAGE_NAME`}}",
    "resource_group_name": "{{env `RESOURCE_GROUP_NAME`}}"
  },
  "builders": [
  {
    "type": "azure-arm",
    "subscription_id": "{{user `subscription_id`}}",
    "managed_image_resource_group_name": "{{user `resource_group_name` }}",
    "managed_image_name": "{{user `managed_image_name`}}",
  ........ continued
```

## Packer File

## Azure Shared image repo

## Summary


[Packer Azure Resource Manager documents]: https://www.packer.io/docs/builders/azure.html
[Azure CLI]: https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest
[azure documentation]: https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-2.7.0
[create for rbac docs]: https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac