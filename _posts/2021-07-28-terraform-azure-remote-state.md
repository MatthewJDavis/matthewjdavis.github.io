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
published: true
---
July 2021

# Overview

In this post I will cover setting up Terraform and [Azure blob storage] to save state files for Terraform. State files allow Terraform to track the current resources provisioned and can calculate the changes that updates to the Terraform file will make to your infrastructure.

Azure blob storage is an [object store] similar to [AWS S3]. The advantages of storing state in an Azure blob storage container is you get [locking] on the blob by default - this means that two people can't update the infrastructure at the same time. You can set up the same for S3 using a [DynamoDB table] to track the locking state but I find the Azure blob storage way much easier to implement.

In my [previous post], I detailed the steps required to provision an Azure AD application via Terraform and this post will build on that.

It requires that the service principal is still available from the [previous post] to deploy the application and the details are set in environment variables as before.

```bash
export ARM_SUBSCRIPTION_ID='abcde'
export ARM_TENANT_ID="abcde"
export ARM_CLIENT_ID="abcde"
 export ARM_CLIENT_SECRET="abcde"
```

There is a charge for using storage accounts (including storage, transfer and read/write operations), see the [Azure Storage Blob pricing] for details.

## Azure Storage account

### Create with the Azure CLI

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
You could also use one of the storage account [access keys], however I would advise to use a SAS token that is ony scoped to that particular storage account and container - just enough privilege!

```bash
accountKey=$(az storage account keys list --resource-group $rg --account-name $sa --query '[0].value' -o tsv)
end=`date -u -d "2 years" '+%Y-%m-%dT%H:%MZ'`
sas=`az storage container generate-sas -n $container --account-key $accountKey --account-name $sa --https-only --permissions dlrw --expiry $end -o tsv`

 export ARM_SAS_TOKEN=$sas

 az logout
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

Here is the updated main.tf file from my [previous post].

<script src="https://gist.github.com/MatthewJDavis/2c76473c4d46a54d7930047dd52575b8.js"></script>

The variables file is the same as before.

<script src="https://gist.github.com/MatthewJDavis/69bd18c079b2f7026f637e6674fac03c.js"></script>

### Init

Running the initialisation command will create the state file in the blob container. If you are still using the state file from the previous post, then the state file will be copied from the local computer storage to Azure blob storage - pretty neat.

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

If the apply takes along time, you can go to the blob details and you'll see the lease status as Locked and the lease state as Leased which will prevent anyone else being able to update the infrastructure until you have finished and released the lock.

![leased state on blob](/images/terraform-azure-backend/leased-state.png)

## Clean up

Delete the resources created by Terraform:

```bash
terraform destroy
```

Delete the state file:

```bash
# Add the variables in again for container, sa and rg if in a new session.
az login
accountKey=$(az storage account keys list --resource-group $rg --account-name $sa --query '[0].value' -o tsv)
az storage blob delete -c $container --account-name $sa --account-key $accountKey -n msgraphapp.tfstate 
```

## Summary

That's it, now the remote state can be shared with others in the team (that have a SAS token) or can be used in a CI/CD pipeline.

[Azure blob storage]: https://azure.microsoft.com/en-ca/services/storage/blobs/
[object store]: https://en.wikipedia.org/wiki/Object_storage
[AWS S3]: https://aws.amazon.com/s3/
[DynamoDB table]:https://www.terraform.io/docs/language/settings/backends/s3.html
[locking]: https://docs.microsoft.com/en-us/rest/api/storageservices/lease-blob
[previous post]: https://matthewdavis111.com/terraform/terraform-azure-ad-app/
[backend section]: https://www.terraform.io/docs/language/state/backends.html
[partial configuration]: https://www.terraform.io/docs/language/settings/backends/configuration.html#partial-configuration
[access key]: https://www.terraform.io/docs/language/settings/backends/azurerm.html#access_key
[Azure Storage Blob pricing]: https://azure.microsoft.com/en-us/pricing/details/storage/blobs/