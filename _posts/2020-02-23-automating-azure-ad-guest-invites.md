---
title: Automate sending guest invites to Azure AD with PowerShell and Azure automation
excerpt:
date: 2020-02-23
toc: false
classes: wide
categories:
- azure
tags:
- azure active directory
- powershell
- azure automation
published: false
---
February 2020

# Overview

A recent project involved integrating a new data analytics platform called [ThoughSpot] that is being used for 3rd party company employees access to data and analytics produced by the data team. The [ThoughtSpot] implementation required an access solution that allowed a large number of 3rd party companies and their employees to their data while being managable and scalable from our side, creating accounts in the application itself was a no go due to the sheer number of accounts that needed access and the issue of if an employee from a 3rd party company left, they would still have access to the data unless we were informed (which was highly unlikely to happen the majority of the time).

After investigation and working with support, ThoughSpot is compatible with Azure Active Directory Applications and users can be created and authenticated via SAML, so this was implemented which gave a number of benefits:

* 3rd party companies that had their own Azure AD tenant meant their employees use the same logins they already use
* Users with a company that does not have an Azure AD tenant could still access the system via the One Time Password option (preview feature)
* No user creation or password management on our side. A user leaves their company and access to their Azure AD account or work email (for One Time Password) codes is revoked and they can no longer access the system
* Access to the application was controlled via an Azure AD group
* Able to automate the invitation process

This post will go through the automation of the invite process but also cover the setting up of the One Time Password preview feature and the flow of how that works because it is a neat solution.

## Set up of One Time Passwords

This post uses the preview feature of [One Time Passwords for external guest accounts].

To set this up, you need to have the correct permissions in the Azure AD tenant. Go to the Azure AD portal: https://aad.portal.azure.com

Click on the 'User' blade and select 'User Settings'
Click on 'Manager external collaboration settings' link

![Azure AD user settings](/images/azuread-guest-invite/user-settings.png)

In the settings, change the 'Enable Email One-Time Passcode for guests (Preview)' setting to 'Yes'

![Azure AD external guest settings](/images/azuread-guest-invite/external-settings.png)

## Set up Azure AD Service account

An account is needed to send the invites from. I have covered how to create an Azure AD user with PowerShell core in a [previous post].

```powershell
Connect-AzAccount

$creds = Get-Credential

New-AzADUser -DisplayName $creds.UserName -UserPrincipalName 'welcome@matthewdavis111.com' -MailNickname $creds.UserName -Password $creds.Password -ForceChangePasswordNextLogin:$false
```

Unfortuantely the Az module is lacking functionality for Azure Users.

To set the user so the password never expires, the old MSOL module is required:

```powershell
Set-MsolUser -UserPrincipalName welcome@matthewdavis111.com -PasswordNeverExpires $true
```

The module can also be used so the user has the role to invite guest users:

```powershell
Add-MsolRoleMember -RoleName "Company Administrator" -RoleMemberEmailAddress "welcome@matthewdavis111.com"
```

Or this can be achieved via the portal:
Roles and Administrators
Search for 'Guest Inviter' role
Search for the user and 'add' them.

![adding to the inviter role](/images/azuread-guest-invite/inviter-role.png)

## Azure automation runbook

### Prerequist Module

The script relies on the AzureAD module to run. This needs to be installed in the automation account beforehand.
The easiest way to do this it in the automation account, click on 'Modules', then 'Browse Gallery'. Search for AzureAD and install it.

![Install azuread module](/images/azuread-guest-invite/azuread-module.png)

### Runbook

Below is the basic Azure Automation Runbook. In production, I have it sending messages to slack, with number of users added for the day and errors but here this just writes out to the Azure Automation output window. I have also changed the name of the variables to refer to the app as 'demoApp'.

<script src="https://gist.github.com/MatthewJDavis/7b6b5be967628d7a97d4c4dd239bd732.js"></script>

There are a number of variables that are taken from Azure Automation, including the Azure blob storage uri which includes the SAS token to give the Runbook access and the credentials of a service account in Azure AD that is used to invite external guests.

### Azure AD user group

This runbook also adds the user to the Azure AD group 'DemoApp' which gives them access to the enterprise application. The user that is used to send the invite needs to be an owner of the group so they can add external users to the group when the invite is sent.

###

The csv file uploaded to Azure blob storage looks like:

```csv
username,email
Matt External,demo.email.matt@gmail.com
Mark Smtih, mark.smith@matthewdavis111.com
```

Only one of these email addresses will be invited, the demo gmail one. The email with the internal address will not be invited, this email will not be added to the 'clean email list' as it is an internal one.

### Output

![Azure automation output](/images/azuread-guest-invite/output.png)

The runbook can be set to run via a schedule, webhook, or event such as when the csv is uploaded.

## External guest invitation in action

When run runbook is executed, an invitation is sent to the user's email pictured below.

![Sent invitation](/images/azuread-guest-invite/invitation.png)

![Send login code](/images/azuread-guest-invite/send-code.png)

![Reviewing permissions](/images/azuread-guest-invite/permissions.png)

![Login code](/images/azuread-guest-invite/sent-code.png)

## Summary

[One Time Passwords for external guest accounts]: https://docs.microsoft.com/en-us/azure/active-directory/b2b/one-time-passcode
[ThoughtSpot]: https://www.thoughtspot.com/
[previous post]: https://matthewdavis111.com/powershell/manage-azure-ad-powershell-core/