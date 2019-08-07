---
title: Use PowerShell core on linux to manage Azure AD
author: Matthew Davis
date: 2019-08-06
toc: false
classes: wide
excerpt: Use PowerShell core running on linux to connect to and manage Azure AD.
categories:
    - powershell
tags:
    - azure
    - core
    - powershell
published: false
---
August 2019

# Overview
I have been mainly using PowerShell core for my day to today work for a while now and have been using a lot recently to interact with Azure and Azure AD so will go through some basics of getting it setup to work and useful commands. I have been using Azure for a few years now, getting started with cloud services and the classic deployment model and migrating over to the Azure Resource Manager way. This has also seen the move away from the AzureRM PowerShell cmdlets to now use the AZ cmdlets (a lot less typing and not too bad to migrate old scripts over too).

Install PowerShell core for your distribution following the guidlines. For ubuntu 18.04 I installed following the method to add the Microsoft repository apt-get and install from there. Link to [Ubuntu 18.04 install instructions].

Although this guide is showing it running on Ubuntu, the cmdlets will be the same running PowerShell core on Mac, Windows and even PowerShell Desktop edition running on Windows.

## Install AZ module

```powershell
Find-Module az

Install-Module -Name az -Scope CurrentUser
```

Now we can see the module is installed and as mentioned earlier, it can be used both on PowerShell core and the desktop edition.

```powershell
Get-Module -Name az -ListAvailable
```

## Connecting

```powershell
Connect-AzAccount
```

Now we have to sign in via the browser so a token will be issued to your PowerShell session.

## Context

If you only have one Azure subscription, this will be the context you are using. If you have more than one subscription, you can use Set-AZContext cmdlet to change the subscription.

```powershell
Get-AzSubscription
Set-AzContext
```


## Getting Users Get-AzADUser

The Get-AzADUser command with no parameters specified will return all users, my test Azure AD only has a small number of users but you can use the -First parameter to limit the number of users returned.

```powershell
(Get-AzADUser | Measure-Object).Count
```

### Searching

```powershell
Get-AzADUser -DisplayName demo1

Get-AzADUser -StartsWith demo
```

### New User

```powershell
New-AzADUser -DisplayName $creds.UserName -UserPrincipalName 'demo2@matthewdavis111.com' -MailNickname $creds.UserName -Password $creds.Password -ForceChangePasswordNextLogin:$false
```

## Summary

MSOL has more features and functionality but the AZ module allows management via linux, mac or Windows.

https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model
[Ubuntu 18.04 install instructions]: https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804