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

## SAS Token

To authenticate to the blob storage container, a 2 year SAS token is created. After creation this is set as an environment variable for Terraform to use. This should also be securely saved somewhere such as a password manager and or a secrets manager solution such as Azure Key Vault or Hashicorp Vault for use in a CI/CD pipeline.

```bash
accountKey = $(az storage account keys list --resource-group $rg --account-name $sa --query '[0].value' -o tsv)
end=`date -u -d "2 years" '+%Y-%m-%dT%H:%MZ'`
sas=`az storage container generate-sas -n $container --account-key $accountKey --account-name $sa --https-only --permissions dlrw --expiry $end -o tsv

 export ARM_SAS_TOKEN=$sas
```

## Terraform Azure backend configuration

Now we can access the blob storage via the SAS token, the Terraform configuration should be updated with the details of where to store the state. This is done in the 'backend' section.

```terraform
terraform {
  required_providers {
    azuread = {
      version = "=1.6.0"
    }
  }
  backend "azurerm" {
    key = "msgraphapp.tfstate"
    resource_group_name   = "terraform-state-demo"
    storage_account_name  = "tfstate13069"
    container_name        = "tfstate"
  }
}
```

If you want to keep the details for the storage account out of the main file, you can use a [partial configuration]. For example you could create a file called backend.conf (which you can also add to the .gitignore file) and have the following values.

```terraform
resource_group_name   = "terraform-state-demo"
storage_account_name  = "tfstate13069"
container_name        = "tfstate"
```

The start of the main.tf would look like the following (I like to keep the name of the state file in main.tf):

```terraform
terraform {
  required_providers {
    azuread = {
      version = "=1.5.0"
    }
  }
  backend "azurerm" {
    key = "msmsgraphapp.tfstate"
  }
}
```

Then you can run the initialisation to specify the backend file:

```bash
terraform init -backend-config=backend.conf
```

This solution would allow you to keep some or all of the configuration of of source control if desired.

## Terraform

### Init
Running the initialisation command will create the state file in the blob container.

```bash
terraform init
```

Here you can see the blob created with the name we specified (msmsgraphapp.tfstate) and the lease state is available.

![blob storage showing state file](/images/terraform-azure-backend/state-blob.png)

 If you download the state file and take a look you'll see a state file with no details.

![state file with barebones outline](/images/terraform-azure-backend/blank-state.png)

### Plan and apply

Running a plan and apply will update the state with the details of the application - including the client secret. That is why it is important to protect the access to the blob storage and only allow those who would need to access.

```bash
terraform plan -out deployment.tfplan
```

```bash
terraform apply -auto-approve deployment.tfplan
```

![state file with details after apply](/images/terraform-azure-backend/state-after-apply.png)

If the apply takes along time, you can go to the blob details and you'll see the lease status as Locked and the Lease state as unavailable which will prevent anyone else being able to update the infrastructure until you have finished and released the lock.

## Clean up

```bash
terraform destroy
```

```bash

```


[backend section]: https://www.terraform.io/docs/language/state/backends.html
[partial configuration]: https://www.terraform.io/docs/language/settings/backends/configuration.html#partial-configuration