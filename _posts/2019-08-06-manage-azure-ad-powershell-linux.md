---
title: Use PowerShell core on linux to manage Azure AD Users and Groups
author: Matthew Davis
date: 2019-08-06
toc: false
classes: wide
excerpt: Going over the basics of using the AZ PowerShell module to manage users and groups in Azure Active Directory
categories:
    - powershell
tags:
    - azure
    - aad
    - core
    - powershell
published: false
---
August 2019

# Overview

I have been mainly using [PowerShell Core] for my day to today work for a while now and have been using a lot recently to interact with Azure and Azure AD so will go through some details of getting it set-up to work and useful commands.

I have been using Azure for a few years now, getting started with cloud services and the classic deployment model (the old Windows Azure Service Management API - [now deprecated]) and [migrating] over to the [Azure Resource Manager API]. This has also seen the move away from the AzureRM PowerShell cmdlets to now use the cross platform [AZ cmdlets] (a lot less typing and not too bad to [migrate old scripts] over too).

This guide has been written running PowerShell Core on Ubuntu linux, but as long as you are running PowerShell Core on any of the supported platforms, you'll be able to follow along.

## Install PowerShell core

[Install PowerShell Core] for your platform or linux distribution. For ubuntu 18.04 I installed following the method to add the Microsoft repository apt-get and installed from there. Link to [Ubuntu 18.04 install instructions].

Although this guide is showing it running on Ubuntu, the AZ cmdlets will be the same running PowerShell core on Mac, Windows and even PowerShell Desktop edition running on Windows.

![PowerShell core running on Ubuntu 18.04](/images/powershell-core-azure/ps-info.png)


## Install AZ module

First we need to install the AZ module previously mentioned. This can be done using PowerShell, the Find-Module cmdlet searches the [PowerShell gallery] (providing you have not changed the default provider) and the Install-Module cmdlet installs it under the currentuser (you will need to sudo if on linux to install the module for all users).

```powershell
Find-Module az

Install-Module -Name az -Scope CurrentUser
```

![installing the az module](/images/powershell-core-azure/install-module.png)

Now we can see the module is installed and as mentioned earlier, it can be used both on PowerShell core and the desktop edition.

```powershell
Get-Module -Name az -ListAvailable
```

![module details](/images/powershell-core-azure/module-details.png)

## Connecting

Note: You will need to sign in with a user who has permissions to manage users and groups in Azure AD (The [User administrator role] would be good for least privilege access)

Now we have to sign in via a web browser for a token to be issued for your PowerShell session.

```powershell
Connect-AzAccount
```

![connecting to azure](/images/powershell-core-azure/connect.png)

Once you have signed in with the correct credentials, you will see the message below and can close the browser tab.

![successful authentication](/images/powershell-core-azure/authed.png)

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

[PowerShell Core]: https://github.com/PowerShell/PowerShell
[now deprecated]: https://azure.microsoft.com/en-ca/updates/deprecating-service-management-apis-support-for-azure-app-service/
[migrating]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model
[Install PowerShell Core]: https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-6
[Ubuntu 18.04 install instructions]: https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804
[Azure Resource Manager API]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview
[migrate old scripts]: https://docs.microsoft.com/en-us/powershell/azure/migrate-from-azurerm-to-az?view=azps-2.5.0
[AZ cmdlets]: https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-2.5.0
[PowerShell gallery]: https://www.powershellgallery.com/
[user administrator role]: https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles#user-administrator
[Exchange online]: https://support.microsoft.com/en-ca/help/2824766/alias-or-mailnickname-are-changed-for-a-synced-user