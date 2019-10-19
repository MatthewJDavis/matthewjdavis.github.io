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
October 2019

# Overview

Packer is a tool created by Hashicorp that allows you to create custom images (VMs or containers) for a wide variety of platforms including AWS, Vagrant, Virtual Box and Azure (the focus of this post). 

I have been creating custom images for AWS (AMI) for a while and went through the same process for Azure to see the difference and work out how to get it configured, below is the notes I used to successfully build Ubuntu images for Azure using Packer and a basic packer build script to get started with.

[Packer Azure Resource Manager documents] are well written and provide lots of details on the options available.

This was run using Ubuntu 18.04 LTS, with PowerShell Core 6.2.3, Azure CLI version 2.0.74, Packer 1.4.4 and on Windows 10 with PowerShell Core 6.2.3 and Packer 1.4.4.

## Set up

### Authentication

You can authenticate to Azure for building with packer 2 ways:

1. Interactive authentication
2. A service principal

The method packer uses depends on the environment variables that are set in the shell when the Packer build command is run.

**Interactive**

The Packer Azure builder will prompt you to authenticate via a web browser if you have the following 3 environment variables set:

1. subscription_id
2. managed_image_name
3. resource_group_name

To set the environment variables in bash:

```bash
# set env variables for interactive logon
export ARM_SUBSCRIPTION_ID=xxxxx-xxxx-xxxx-xxx-xxx
export MANAGED_IMAGE_NAME=ubuntu-18-04-lts-nginx
export RESOURCE_GROUP_NAME=packerImageBuilds
```

To set the environment variables in PowerShell:

```powershell
$env:ARM_SUBSCRIPTION_ID='xxxxx-xxxx-xxxx-xxx-xxx'
$env:MANAGED_IMAGE_NAME='ubuntu-18-04-lts-nginx'
$env:RESOURCE_GROUP_NAME='packerImageBuilds'
```

![powershell environment vars added](/images/packer-azure/ps-env-vars.png)

When you run packer build, a code and web address will be shown for you to authenticate and once successful and token will be issued and you'll be able to run the build.

Running on linux:

![auth to azure interactive](/images/packer-azure/auth-to-azure.png)

Running on Windows:

![auth to azure interactive](/images/packer-azure/build-windows.png)

**Azure Service Principal**

An Azure Service Principal should be created for automation purposes. Details on how to create the service principal can be found in the [azure documentation].

As this is my own account for testing, I gave the service principal contributor role access to my subscription via the Azure CLI but for a production account, a customised RBAC should be used so the service principal just has the right amount of access required to run the builds. AN even better way would be to use a separate Azure subscription for Packer builds and the images can be uploaded shared with using an [Azure shared image library] 

```bash
az ad sp create-for-rbac -n "Packer-Matt-Visual-Studio-Subscription-Only" --role contributor --scopes /subscriptions/xxxxxxxx-xxxxxxx-xxxxx-xxxxx-xxxxxxx
```

[create for rbac docs]

To set the environment variables in bash:

```bash
# set environment variables for service principal

export ARM_CLIENT_ID=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_CLIENT_SECRET=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_SUBSCRIPTION_ID=xxxxx-xxxx-xxxx-xxx-xxx
export ARM_TENANT_ID=xxxxx-xxxx-xxxx-xxx-xxx
export MANAGED_IMAGE_NAME=ubuntu-18-04-lts-nginx
export RESOURCE_GROUP_NAME=packerImageBuilds
```

To set the environment variables in PowerShell:

```powershell
$env:ARM_CLIENT_ID='xxxxx-xxxx-xxxx-xxx-xxx'
$env:ARM_CLIENT_SECRET='xxxxx-xxxx-xxxx-xxx-xxx'
$env:ARM_SUBSCRIPTION_ID='xxxxx-xxxx-xxxx-xxx-xxx'
$env:ARM_TENANT_ID='xxxxx-xxxx-xxxx-xxx-xxx'
$env:MANAGED_IMAGE_NAME='ubuntu-18-04-lts-nginx'
$env:RESOURCE_GROUP_NAME='packerImageBuilds'
```

### Resource Group for Packer images

An Azure resource group is required before the packer build runs to store the images that are created.

A resource group can be created in the portal, by the az cli or PowerShell as shown below.

```powershell
$packerRGName = 'packerImageBuilds'
$tags = @{'Environment' = 'Dev'; 'Description' = 'Resource group where images built by packer are stored' }
$resourceGroup = New-AzResourceGroup -Name $Name -Location $location -Tag $tags -Verbose
```

```bash
# cli
az group create --location northeurope --name packerImageBuilds --tags 'Environment=Dev' 'Description=Images produced by Packer builds'
```

### Variables in the Packer file

Below is an example of how the environment variables are passed to the packer template. Environment variables are passed to the variables set at the top of the packer template and then they become 'user' variables and passed to the builders.

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

Below is a complete packer file called azure-ubuntu-nginx-packer.json that will create a custom Ubuntu 18.04 LTS image with nginx installed for Azure.

<script src="https://gist.github.com/MatthewJDavis/840cce73e920f73628b2b88373ce8e21.js"></script>

```bash
packer build azure-ubuntu-nginx-packer.json
```

After running a successful build, an image is created in the resource group set in the RESOURCE_GROUP_NAME environment variable.

![image uploaded to the resource group](/images/packer-azure/image-rg.png)

Clicking on the image shows the details of the image and gives you the option to create a VM from that image.

![details of image upload](/images/packer-azure/image-details.png)

Below is a small VM that was created directly from the image that allows port 80 through the security group so the default nginx welcome page is displayed on the public IP address.

![vm created from the image](/images/packer-azure/nginx-portal.png)

The nginx welcome page

![nginx welcome page](/images/packer-azure/welcome-page.png)

Every time this image is used to create a VM, nginx will be running so can be used as the image to create VMs in a load balancer etc.

## Azure Shared image repo

As mentioned earlier, a good way to segregate Packer builds is to do them in their own Azure subscription and then us an Azure share image repository to share the built images with certain subscriptions and even different tenants.

First, create an Azure image repository, this can be done via the portal, Azure CLI or PowerShell (example below)

```powershell
# create shared image gallery

$Name = 'packerImageGallery'
$location = 'northeurope'
$tags = @{'Environment' = 'Dev'; 'Description' = 'Resources for packer image builds' }

$resourceGroup = New-AzResourceGroup -Name $Name -Location $location -Tag $tags -Verbose
$gallery = New-AzGallery -GalleryName $Name -ResourceGroupName $Name -Location $location -Description 'Shared Image Gallery for packer builds' -Tag $tags -Verbose
```

![create a new shared image gallery in azure](/images/packer-azure/new-shared-image-gallery.png)

After the operation is completed, you'll be able to see the shared image gallery in the specified resource group.

![create a new shared image gallery in azure](/images/packer-azure/azure-image-gallery.png)

Next an image definition is required, this will be used by packer to upload the image to the gallery.

```powershell
$Name = 'packerImageGallery'
$location = 'northeurope'
$imageName = 'ubuntu-18-04-lts-nginx'

$params = @{
  GalleryName       = $Name
  ResourceGroupName = $Name
  Location          = $location
  Name              = $imageName
  OsState           = 'generalized'
  OsType            = 'Linux'
  Publisher         = 'matt'
  Offer             = 'Webserver'
  Sku               = $imageName
  
}

$galleryImage = New-AzGalleryImageDefinition @params
```

## Summary

[Packer Azure Resource Manager documents]: https://www.packer.io/docs/builders/azure.html
[Azure CLI]: https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest
[azure documentation]: https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azps-2.7.0
[create for rbac docs]: https://docs.microsoft.com/en-us/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac
[Azure shared image library]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/shared-image-galleries