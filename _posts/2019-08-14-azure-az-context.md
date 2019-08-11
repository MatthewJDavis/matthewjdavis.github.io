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

## Adding more subscriptions

My personal account only has one subscription, but my work account has multiple subscriptions so I need to import the ones that I will be working with.

First we can get the subscription details.

Make sure you are using the context of the Azure account that has the other subscriptions you want to import.

```powershell

```



## Saving the context settings

## Importing the context settings

## Summary