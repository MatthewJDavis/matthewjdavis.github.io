---
title: Create and remove Azure Container Instances from a TeamCity build
author: Matthew Davis
date: 2018-11-03
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

Using PowerShell Desktop Version 5.1 on Windows 10 1803.

## Service Principal

From https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-authenticate-service-principal-powershell

```powershell
# Create an Azure Service principal with a cert for authentication
$cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=tcContainerTestCert" -KeySpec KeyExchange
$keyValue = [System.Convert]::ToBase64String($cert.GetRawCertData())

$servicePrincipal = New-AzureRMADServicePrincipal -DisplayName 'azure-container-instances-teamcity-testing' -CertValue $keyValue -EndDate $cert.NotAfter -StartDate $cert.NotBefore
Start-Sleep 20
```

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
