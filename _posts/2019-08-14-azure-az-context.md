---
title: Manage multiple Azure accounts with PowerShell using AzContext cmdlets
author: Matthew Davis
date: 2019-08-14
toc: false
classes: wide
excerpt: Using the AzContext cmdlets to manage multiple Azure environments and subscriptions
categories:
    - powershell
tags:
    - azure
    - powershell
published: false
---
August 2019

# Overview



## Connect-AzAccount

```powershell
Get-AzContext -ListAvailable
```

Connect personal account

```powershell
Connect-AzAccount
```



Connect work account

```powershell
Connect-AzAccount
```

## Renaming the contexts

When importing the context, they have long names with spaces in them. Typing the first letter(s) of the name and pressing the tab key will auto complete the name, but I like to rename the subscriptions.

```powershell
Rename-AzContext -SourceName 'Pay-As-You-Go (xxxxxxxx-xxxx-xxxxxx-xxxx-xxxxxxxxxx) xxx.xxx@xxx' -TargetName 'PAYG'

Rename-AzContext -SourceName 'Visual Studio Professional (xxxxxxxx-xxxx-xxxxxx-xxxx-xxxxxxxxxx) xxx.xxx@xxx' -TargetName 'VSP'
```

Now we can change context easily.

```powershell
Select-AzContext 'VSP'
Select-AzContext 'PAYG'
```

## Saving the context settings

Now the two contexts have been set up, they can be saved to be imported later on.

```powershell
Save-AzContext -Path /home/matt/Documents/azure/azure-context.json
```

## Importing the context settings

The saved contexts can be imported easily enough

```powershell
Import-AzContext /home/matt/Documents/azure/azure-context.json
```

## Adding more subscriptions

My personal account only has one subscription, but my work account has multiple subscriptions so I need to import the ones that I will be working with.

First we can get the subscription details.

Make sure you are using the context of the Azure account that has the other subscriptions you want to import.

```powershell
Get-AzSubscription
```

To add a new context for a new subscription

```powershell
 Set-AzContext -Subscription 'xxxxx-xxxx-xxxx-xxxx-xxxx' -Name 'DevSub'
```

To add all subscriptions to your Azure context you can run the following.

```powershell
Get-AzSubscription | Set-AzContext
```

### Iterating over subscriptions

Now we have all the subscriptions we are working with set up as context, you can work with resources across all of the context using a loop.

```powershell
# save list of contexts
$ctxList = Get-AzContext -ListAvailable

# output resource group name and location for each context
foreach($ctx in $ctxList){
  Select-AzContext -Name $ctx.Name | Out-Null
  Write-Output "$($ctx.Name) resource groups:"
  Get-AzResourceGroup | Select-Object ResourceGroupName, Location
}

```

## Summary

Working across multiple Azure accounts and subscriptions is made easier with the AzContext cmdlet, saving the context with an easier to type name helps (especially in automation), however using tab to auto complete when working interactively makes it less painful.
