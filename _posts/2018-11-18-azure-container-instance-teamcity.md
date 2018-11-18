---
title: Create and remove Azure Container Instances from a TeamCity build
author: Matthew Davis
date: 2018-11-18
excerpt: 
categories:
    - powershell
tags:
    - powershell
    - azure
    - teamcity
    - aci
    - azure container instances
published: false
---

# Overview

Create a container in Azure Container Instance Groups from a TeamCity build job for testing.

The requirement came up recently to be able to test certain steps of a build against an application running in a linux container. The build agents used run on EC2 and couldn't do nested virtualisation so the original plan of using docker on the agent to run a linux container to test against didn't work due to running linux containers on Windows requires hyper-v to be installed.
https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/linux-containers
I looked around at some other solutions and watched the channel9 video on Azure container instances and thought they could fit the need.
https://channel9.msdn.com/Shows/Tuesdays-With-Corey/More-info-on-Azure-Container-Instances

After checking the PowerShell docs and giving it a go, it is simple to get a container up and running and remove it:

```powershell
# Create Nginx Container with date time for unique DNS name

$date = Get-Date -Format yyyyMMddHHMMss
$ContainerGroupName = "demoContainer"
$DnsName = "demoContainer-$date"
$OsType = 'Linux'
$Port = '80'
$ContainerImage = 'nginx'

$containerGroupParams = @{
  'ResourceGroupName' = $ResourceGroup;
  'Name'              = $ContainerGroupName;
  'Image'             = $ContainerImage;
  'DnsNameLabel'      = $DnsName;
  'OsType'            = $OsType;
  'Port'              = $Port
}

New-AzureRmContainerGroup @containerGroupParams

Remove-AzureRmContainerGroup -ResourceGroupName $ResourceGroup -Name $ContainerGroupName
```

With it being that easy, I came up with the plan to use this as part of a TeamCity build job which would involve:

1. Authenticating to Azure
2. Creating the container
3. Running the tests against the container
4. Removing the container
5. Removing the Azure session

Because the build agents are running in AWS, I had to create a service principal to connect to Azure and run the PowerShell commands. This post will cover:

1. Creating a Service Principal with a Certificate
2. Giving the service principal just enough permissions to a resource group via a custom Azure role to create and remove containers
3. Setting up TeamCity build steps and parameters to connect to Azure, create, test against and remove then container the remove the Azure session
4. The PowerShell scripts to run in item 3.

The examples in this post was carried out using PowerShell Desktop Version 5.1 on Windows 10 1803.

## Service Principal

First I create the Azure Service Principal following the documentation. I wanted to use a certificate for the Service Principal authentication because I had not done this before.
Here's the code to create the Service Principal, it creates a certificate locally that is used for the Service Principal to connect. This certificate should be copied (included the private key) to where you want the service principal to be able to login (i.e. the build agents).

From https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell

```powershell
# Create an Azure Service principal with a cert for authentication
$certStoreLoc = 'cert:\CurrentUser\My'
$certSubject = 'CN=teamCityAzureContainerSP'
$servicePrincipalName = 'azure-container-instances-teamcity-testing'

# Create local self-signed cert - use cert authority in production
$cert = New-SelfSignedCertificate -CertStoreLocation $certStoreLoc -Subject $certSubject -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

Connect-AzureRMAccount

# Create the service principal with the certificate just created
$servicePrincipal = New-AzureRMADServicePrincipal -DisplayName $servicePrincipalName -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
Start-Sleep 20
```

[pic of cert output]

[pic of sp output]

This creates the AzureAD application and Service Principal the will be used by the TeamCity build step to authenticate to Azure.


## Create a custom role

Update subscription-id-here with Azure SubscriptionID ```Get-AzureRmSubscription```

```json
{
  "Name": "Container Instance Container Group Manager",
  "Id": null,
  "IsCustom": true,
  "Description": "Allow full access to the Azure Container Instance container group resources",
  "Actions": [
    "Microsoft.ContainerInstance/containerGroups/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/subscription-id-here"
  ]
}
```

```powershell
New-AzureRmRoleDefinition -InputFile
```

### How to find the namespace for roles

https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles-powershell

https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations#microsoftcontainerinstance

```powershell
Get-AzureRmProviderOperation | Where-Object {$_.ProviderNamespace -like "*container*"} | Select-Object -Property ProviderNamespace -Unique
```

## Assign Service Principal

```powershell
# Give service principal contribute at resource group level
New-AzureRmRoleAssignment -ObjectId $servicePrincipal.ApplicationId -RoleDefinitionName 'Container Instance Container Group Manager' -ResourceGroupName 'test-iam'
New-AzureRmRoleAssignment -ObjectId $servicePrincipal.ApplicationId -RoleDefinitionName 'reader' -ResourceGroupName 'test-iam'
```

## Test Script

## TeamCity Build Job

http://ralbu.com/teamcity-build-parameters-for-powershell