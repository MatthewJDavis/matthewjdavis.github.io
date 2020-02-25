---
title: Automate sending guest invites to Azure AD with PowerShell and Azure automation
excerpt: Solution using PowerShell and Azure automation to send invites to guest users leveraging Azure AD B2B capabilities.
date: 2020-02-24
toc: false
classes: wide
categories:
- azure
tags:
- azure active directory
- powershell
- azure automation
published: true
---
February 2020

# Overview

A recent project I worked on involved integrating a new data analytics platform called [ThoughtSpot] that is being used for 3rd party company employees access to data and analytics produced by the data team. The [ThoughtSpot] implementation required an access solution that allowed a large number outside users access to their data while being manageable and scalable from our side, creating accounts in the application itself was a no go due to the sheer number of accounts that needed access and the issue of if an employee from a 3rd party company left, they would still have access to the data unless we were informed (which past experience has shown, was highly unlikely to happen the majority of the time).

After investigation and working with support, ThoughSpot is compatible with [Azure Active Directory Applications] and users can be created and authenticated via SAML, so this was the chosen implementation which gave a number of benefits:

* 3rd party companies that had their own Azure AD tenant or other supported Identity Provider means their employees use the same logins as they do to access their own company resources
* Users with a company that does not have an Azure AD tenant could still access the system via the One Time Password option (currently a [preview feature])
* No user creation or password management on our side. A user leaves their company and access to their Azure AD account or work email (for One Time Password) codes is revoked and they can no longer access the system
* Access to the application was controlled via an Azure AD group
* Able to automate the invitation process

This post will go through the automation of the invite process but also cover the setting up of the One Time Password preview feature and the flow of how that works because it is a neat solution.

## Set up of One Time Passwords

To set up One Time Passwords (you need to have the correct administrator permissions in the Azure AD tenant), go to the Azure AD portal: https://aad.portal.azure.com

Click on the 'User' blade and select 'User Settings'
Click on 'Manager external collaboration settings' link

![Azure AD user settings](/images/azuread-guest-invite/user-settings.png)

In the settings, change the 'Enable Email One-Time Passcode for guests (Preview)' setting to 'Yes'

![Azure AD external guest settings](/images/azuread-guest-invite/external-settings.png)

## Set up Azure AD User as a service account

An account is needed to send the invites from. I have covered how to create an Azure AD user with PowerShell core in a [previous post].

```powershell
Connect-AzAccount

$creds = Get-Credential

New-AzADUser -DisplayName $creds.UserName -UserPrincipalName 'welcome@matthewdavis111.com' -MailNickname $creds.UserName -Password $creds.Password -ForceChangePasswordNextLogin:$false
```

Unfortunately the Az module is lacking functionality for Azure Users.

To set the user so the password never expires, the older [MSOL] module (Windows PowerShell only) is required. MSOnline module in [PowerShell Gallery].

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

### Azure AD user group

This runbook also adds the user to the Azure AD group 'DemoApp' which gives them access to the enterprise application. The user that is used to send the invite needs to be an owner of the group so they can add external users to the group when the invite is sent.

![Added user as gropu owner](/images/azuread-guest-invite/group-owner.png)

## Azure automation runbook

### Prerequist Module

The script relies on the AzureAD module to run. This needs to be installed in the automation account beforehand.
The easiest way to do this it in the automation account, click on 'Modules', then 'Browse Gallery'. Search for AzureAD and install it.

![Install azuread module](/images/azuread-guest-invite/azuread-module.png)

### Runbook

Below is the basic Azure Automation Runbook. In production, I have it sending messages to slack, with number of users added for the day and errors but here this just writes out to the Azure Automation output window. I have also changed the name of the variables to refer to the app as 'demoApp'.

<script src="https://gist.github.com/MatthewJDavis/7b6b5be967628d7a97d4c4dd239bd732.js"></script>

There are a number of variables that are taken from Azure Automation, including the Azure blob storage URI which includes the [SAS token] to give the Runbook access and the credentials of a service account in Azure AD that is used to invite external guests.

### CSV file

The csv file uploaded to Azure blob storage looks like:

```csv
username,email
Matt External,demo.email.matt@gmail.com
Mark Smtih, mark.smith@matthewdavis111.com
```

Only one of these email addresses will be invited, the demo gmail one. The email with the internal address will not be invited, this email will not be added to the 'clean email list' as it is an internal one (something that occurred in production, data team were sending the CSV list which included internal users so this had to be accounted for).

Note: I am only using a gmail address in this as an example. For a production instance I would advise against this and use only email addresses that are controlled by the company so when the user leaves, they are no longer able to access the One Time Passwords that are sent.

### Output

![Azure automation output](/images/azuread-guest-invite/output.png)

The runbook can be set to run via a [schedule], [webhook], or use [event grid] to trigger the runbook when the csv file is uploaded.

## Diagram of solution

![Diagram overview of solution](/images/azuread-guest-invite/diagram.png)

## External guest invitation in action

When run runbook is executed, an invitation is sent to the user's email pictured below.

![Sent invitation](/images/azuread-guest-invite/invitation.png)

If the user's company has an Azure Active Directory tenant, they will sign into that and then be redirected to the page with the Application on it.
If they don't, then a login code valid for 30 mins is sent as pictured below.

![Send login code](/images/azuread-guest-invite/send-code.png)

![Login code](/images/azuread-guest-invite/sent-code.png)

On first login, Azure requires the user to accept some permissions.

![Reviewing permissions](/images/azuread-guest-invite/permissions.png)

The user can now access the application from the portal linked in the email.

![DemoApp now available in the portal for external user](/images/azuread-guest-invite/external-app.png)


## Summary

The [Azure AD B2B] functionality of allowing guest users to access resources is a really nice tool. It takes away the headache of having to manage those third party user accounts and gives the end user a more seamless experience, using a login they are already familiar with. The One Time Password feature is very good too, not everyone uses Azure AD or support Identity Provider, so being able to integrate and service for those companies with the same solution makes life easier.

Automating the whole process with Azure Automation and PowerShell made this project really satisfying to work on. There is still work to do on removing external guest accounts that are no longer used, but it should not be too hard to script out removal of the guest users that have not logged in for x amount of days and if they access is still required in the ThoughtSpot system, they will receive an invite the next time the process runs.

[preview feature]: https://docs.microsoft.com/en-us/azure/active-directory/b2b/one-time-passcode
[ThoughtSpot]: https://www.thoughtspot.com/
[previous post]: https://matthewdavis111.com/powershell/manage-azure-ad-powershell-core/
[schedule]: https://docs.microsoft.com/en-us/azure/automation/shared-resources/schedules
[webhook]: https://docs.microsoft.com/en-us/azure/automation/automation-webhooks
[event grid]: https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-event-overview
[Azure Active Directory Applications]: https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/what-is-application-management
[MSOL]: https://docs.microsoft.com/en-us/powershell/module/msonline/?view=azureadps-1.0
[PowerShell Gallery]: https://www.powershellgallery.com/packages/MSOnline/1.1.183.57
[SAS Token]: https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview
[Azure AD B2B]: https://docs.microsoft.com/en-us/azure/active-directory/b2b/what-is-b2b