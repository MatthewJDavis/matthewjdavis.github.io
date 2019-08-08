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

I have been mainly using PowerShell core for my day to today work for a while now and have been using a lot recently to interact with Azure and Azure AD so will go through some basics of getting it setup to work and useful commands. I have been using Azure for a few years now, getting started with cloud services and the classic deployment model (the old Windows Azure Service Management API) and migrating over to the Azure Resource Manager API. This has also seen the move away from the AzureRM PowerShell cmdlets to now use the cross platform AZ cmdlets (a lot less typing and not too bad to migrate old scripts over too).

Install PowerShell core for your distribution following the guidlines. For ubuntu 18.04 I installed following the method to add the Microsoft repository apt-get and installed from there. Link to [Ubuntu 18.04 install instructions].

Although this guide is showing it running on Ubuntu, the AZ cmdlets will be the same running PowerShell core on Mac, Windows and even PowerShell Desktop edition running on Windows.

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
Set-AzContext -Subscription 'subscriptionID'
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

Creating a new user is straight forward, I use the Get-Credential cmdlet to store a PSCredential object in the 'creds' variable then pass the username and password properties to the New-AzADUser cmdlet. 

The MailNickname is required and cannot contain the '@' sign (you currently can not view this attribute with the az cmdlets at time of writing). This is the mail alias and used if you are using [Exchange online].

```powershell
$creds = Get-Credential

New-AzADUser -DisplayName $creds.UserName -UserPrincipalName 'demo2@matthewdavis111.com' -MailNickname $creds.UserName -Password $creds.Password -ForceChangePasswordNextLogin:$false
```

## Groups

### New Group

Similar to the New-AzADUser, the New-AzADGroup cmdlet is straight forward to use and again you need the MailNickName attribute.

```powershell
New-AzADGroup -DisplayName a-demo-group -MailNickname a-demo-group -Description 'Group to hold demo users'
```

### Adding and removing Group Members

Adding and removing is straight forward, you can add multiple members in one cmdlet as shown below. One thing to note is that the Add-AzADGroupMember cmdlet will error if the group already contains a user and at present, will not add any of the users.

```powershell

# Add a single user to a group
Add-AzADGroupMember -TargetGroupDisplayName 'a-demo-group' -MemberUserPrincipalName 'demo1@matthewdavis111.com'

Get-AzADGroupMember -GroupDisplayName 'a-demo-group'

# Remove the user
Remove-AzADGroupMember -GroupDisplayName 'a-demo-group' -MemberUserPrincipalName 'demo1@matthewdavis111.com'

# Add multiple users to a group
$userList = Get-AzADUser -StartsWith 'demo'
Add-AzADGroupMember -TargetGroupDisplayName 'a-demo-group' -MemberUserPrincipalName $userList.UserPrincipalName
Get-AzADGroupMember -GroupDisplayName 'a-demo-group'

# Remove all users from a group
$RemoveUserList =  Get-AzADGroupMember -GroupDisplayName 'a-demo-group'
Remove-AzADGroupMember -GroupDisplayName 'a-demo-group' -MemberUserPrincipalName $RemoveUserList.UserPrincipalName
Get-AzADGroupMember -GroupDisplayName 'a-demo-group'
```

### Delete the group

Finally, we can tidy up by deleting the group.

Remove-AzADGroup -DisplayName 'a-demo-group' -PassThru -Force

## Summary

Using the AZ module in PowerShell core is a handy way to do some basic Azure AD user and group management and can easily enough be incorporated into automation. The MS Online (MSOL) module has a lot more features and gives access to lots more properties and I do need to use it from time to time but the AZ module has become my daily driver and allows management via linux, mac or Windows.

https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model
[Ubuntu 18.04 install instructions]: https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804

[Exchange online]: https://support.microsoft.com/en-ca/help/2824766/alias-or-mailnickname-are-changed-for-a-synced-user