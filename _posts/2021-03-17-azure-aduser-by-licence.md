---
title: Get list of AzureAD users by licence type
excerpt: Use the Microsoft Graph PowerShell module to get a list of users consuming a particular licence.
date: 2021-03-17
toc: false
classes: wide
categories:
- powershell
tags:
- azuread
- powershell
- graph
published: true
---
March 2021

# Overview

Today I was looking at the [Microsoft Graph PowerShell module] to find out if any users had incorrect licences applied ([dynamic groups] are used but the numbers looked a bit off). I had covered querying the [Microsoft Graph] with PowerShell in a [previous post] but thought this would be a good opportunity to give the official module a go now. Below is a quick summary on how to connect to the MS Graph with the PowerShell module and get all users using a specific licence type.

## Set up and connect

```powershell
Install-Module microsoft.graph
```

Set the module to use the Graph beta endpoint (otherwise licence data is not returned).

```powershell
Select-MgProfile -Name "beta" 
```

Connect to the graph with the required permissions (scopes)

```powershell
Connect-MgGraph -Scopes "User.Read.All"
```

Open the link and authenticate and authorise the PowerShell session to interact with the graph. Note I found this would not work with my Microsoft account sourced user and needed to be done by a user created in my Azure Active Directory.

![Dashboard overview](/images/azure-ad-ms-graph-lic/scopes.png)

## Filtering by licence type

You can find the licence guids in the [official documentation] and these are used to [filter] the user results directly from the graph.

```powershell
$ems = Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq efccb6f7-5641-4e0e-bd10-b4976e1bf68e)" -All # ENTERPRISE MOBILITY + SECURITY E3
$p1 = Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq 078d2b04-f1bd-4111-bbd4-b4b1b354cef4)" -All # AZURE ACTIVE DIRECTORY PREMIUM P1 
$p2 = Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq 84a661c4-e949-4bd2-a560-ed7766fcaf2b)" -All # AZURE ACTIVE DIRECTORY PREMIUM P2
```

The above will only return a maximum of 200 users with the ``All`` property and 999 users with 999 value set in the ```PageSize``` property.

A work around if there are more than 999 users per licence type is to get all users in the tenant, then assign them into variables depending on licence type.

```powershell
$userList = Get-MgUser -All

$ems = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq 'efccb6f7-5641-4e0e-bd10-b4976e1bf68e'} | Select-Object  UserPrincipalName 

$p1 = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq '078d2b04-f1bd-4111-bbd4-b4b1b354cef4'} | Select-Object UserPrincipalName 
 
$ems = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq 'efccb6f7-5641-4e0e-bd10-b4976e1bf68e'} | Select-Object  UserPrincipalName 

$p2 = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq '84a661c4-e949-4bd2-a560-ed7766fcaf2b'} | Select-Object UserPrincipalName 
```

Now you have a list of users you could create reports and discover which users are assigned to multiple licences which may could up adjusted to free up licences.

## Find users with two different licences

```powershell
Compare-Object -ReferenceObject $ems.userprincipalname -DifferenceObject $p1.userprincipalname -IncludeEqual -ExcludeDifferent 
```

[Microsoft Graph]: https://docs.microsoft.com/en-us/graph/overview
[Microsoft Graph PowerShell module]: https://github.com/microsoftgraph/msgraph-sdk-powershell
[dynamic groups]: https://docs.microsoft.com/en-us/azure/active-directory/enterprise-users/groups-dynamic-membership
[filter]: https://docs.microsoft.com/en-us/graph/query-parameters#filter-parameter
[official documentation]:https://docs.microsoft.com/en-us/azure/active-directory/enterprise-users/licensing-service-plan-reference
[previous post]: https://matthewdavis111.com/powershell/microsoft-graph-powershell/