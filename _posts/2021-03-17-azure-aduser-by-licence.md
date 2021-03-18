---
title: Get list of AzureAD users by licence type
excerpt: Use the Microsoft Graph PowerShell module to get a list of users consuming a particular licence.
date: 2020-05-04
toc: false
classes: wide
categories:
- powershell
tags:
- azuread
- powershell
published: false
---
March 2021

Intro

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

```powershell
Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq efccb6f7-5641-4e0e-bd10-b4976e1bf68e)" -All # ENTERPRISE MOBILITY + SECURITY E3
Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq 078d2b04-f1bd-4111-bbd4-b4b1b354cef4)" -All # AZURE ACTIVE DIRECTORY PREMIUM P1 
Get-MgUser -Filter "assignedLicenses/any(x:x/skuId eq 84a661c4-e949-4bd2-a560-ed7766fcaf2b)" -All # AZURE ACTIVE DIRECTORY PREMIUM P2
```

The above will only return up to 999 users with the ```PageSize``` property and doesn't work currently with more than that number of users assigned to a licence.

A work around for this is to get all users, then filter them into variables depending on licence type.

```powershell
$userList = Get-MgUser -All

$ems = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq 'efccb6f7-5641-4e0e-bd10-b4976e1bf68e'} | Select-Object  UserPrincipalName 

$p1 = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq '078d2b04-f1bd-4111-bbd4-b4b1b354cef4'} | Select-Object UserPrincipalName 
 
$ems = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq 'efccb6f7-5641-4e0e-bd10-b4976e1bf68e'} | Select-Object  UserPrincipalName 

$p2 = $userList | Where-Object {$_.AssignedLicenses.SkuId -eq '84a661c4-e949-4bd2-a560-ed7766fcaf2b'} | Select-Object UserPrincipalName 
```

Now you have a list of users you could create reports and discover which users are assigned to multiple licences which may could up adjusted to free up licences.

```powershell
Compare-Object -ReferenceObject $ems.userprincipalname -DifferenceObject $p1.userprincipalname -IncludeEqual -ExcludeDifferent 
```
