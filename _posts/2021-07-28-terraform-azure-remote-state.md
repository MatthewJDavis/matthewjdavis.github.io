---
title: Use Azure storage for Terraform remote state.
excerpt: Save Terraform state files on Azure blob storage.
date: 2021-07-28
toc: false
classes: wide
categories:
- terraform
tags:
- azuread
- terraform
- azure storage
published: false
---
July 2021

# Overview

In this post I will cover setting up Terraform and Azure storage to save state files for Terraform. State files allow Terraform to track the current resources provisioned and can calculate the changes that updates to the Terraform file will make to your infrastructure.

Azure blob storage is an object store similar to AWS S3. The advantages of storing state in an Azure blob storage container is you get locking on the blob by default - this means that two people can't update the infrastructure at the same time.

In my previous post, I detailed the steps required to provision an Azure AD application via Terraform and this post will build on that.

There is a charge for using storage accounts, see the documentation for details.

## Azure Storage account

Azure CLI way

```bash
az login

rg=terraform-state-demo
sa=tfstate$RANDOM
container=tfstate

# Resource group
az group create --name $rg --location eastus2 --tags 'Project=Terraform' 'Env=Demo'

# account
az storage account create --resource-group $rg --name $sa --sku Standard_LRS --encryption-services blob

# container
az storage container create --name $container --account-name $sa
```

This creates a new storage account in the resource group and a container that is set to private access (you do not want your Terraform state files public access because they can contain secrets).


## Terraform Azure backend configuration

## 