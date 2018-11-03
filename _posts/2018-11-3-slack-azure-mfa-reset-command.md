---
title: Reset Azure MFA settings with a slack command
author: Matthew Davis
date: 2018-11-11
excerpt: How to delegate the resetting of Azure MFA to others via a Slack command without the need for elevating their user account to Global Admin
categories: 
    - powershell
tags:
    - poweshell
    - azure
    - slack
    - mfa
published: false
---

Currently only users in the [Global Admin] role can reset the Azure Multi Factor Authenticate (MFA) details of users which is used as two factor authentication for Azure, D365 and Office 365. There has been an [Azure feedback request] (I've been monitoring this for a while now and have voted) with Mircrosoft to open this up to other roles for over 3 years with no movement, it would make sense that the Password/ Helpdesk Administrator role would be able to reset the MFA. and was a big pain point for us recently, sometimes getting up to 5 requests a day for resets from the helpdesk because we couldn't give them Global Admin rights to our Azure tenant!

Having recently moved to Slack, I decided to have a look to see if it would be possible for the helpdesk to use Slack to reset Azure MFA so they no longer had to escalate it to our team.

What I came up with was a [Slack slash command] that send the UPN of the user to reset their MFA to an Azure runbook that then ran a PowerShell command to reset MFA using a service account (the service account has to be a global admin).

## Azure Automation Runbook

I'll start with the runbook that makes all of this happen.

<script src="https://gist.github.com/MatthewJDavis/25196e589860f557180883050dce7eb9"></script>

It has one parameter that takes the input of the webhook data.

A hashtable is used to define users who are allowed to actually perform the reset. I could not find a way to lock the Slack command down to certain users, so any user can run the command, however only users who has their SlackID in this hash table will be allowed to actually execute the Reset-MsolStrongAuthenticationMethodByUpn PowerShell cmdlet.

The runbook connects to AzureAD with the Connect-MSOLService (MSOL [module should be installed] in the Azure automation account) Cmdlet using the credentials stored in Azure automation of a service account that is a global admin (had a long strong random password and only a few users have access to password).

Slack sends the data separated with the ampersand symbol (&), so the split method is used to save the separated data in a new variable.

Next, the UPN, SlackID and return URL (where you send the JSON payload responses) is extracted from the data sent by Slack. The data is sent in [percent encoding], so needs to be transformed back into ASCII.

We need to create an Azure Automation Runbook that is triggered via a webhook. I've written a [post here] on how to do it and the offical guide is [here from Microsoft].

## Slack Slash Command

Creating the slash command is pretty straightforward after reading through the docs. You have to create a Slack app, then approve it or get it approved for your workspace.

### Slash command

Creating the slash command involves:
Command: reset-azure-mfa (or another name the the users will type to call the command)
Request URL: The https URL of the webhook to trigger the Runbook in Azure Automation (includes the Auth token)
Short Description: Resets Azure MFA for Azure, Office 365, D365 etc
Usage hint: [Enter UPN to reset i.e. firstname.surname@domain.com]

After entering all of that you'll get a preview.

![Slack slash command settings](/images/slack-azure-mfa-reset/slash-command.png)




[Global Admin]: https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles
[Azure feedback request]: https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/10072839-allow-the-user-admin-role-to-enable-disable-mfa-fo
[module should be installed]: https://docs.microsoft.com/en-us/azure/automation/automation-runbook-gallery#to-import-a-module-from-the-automation-module-gallery-with-the-azure-portal
[percent encoding]: https://en.wikipedia.org/wiki/Percent-encoding
[post here]: https://matthewdavis111.com/azure/create-azure-automation-job-powershell/
[here from Microsoft]: https://docs.microsoft.com/en-us/azure/automation/automation-creating-importing-runbook
[Slack slash command]: https://api.slack.com/slash-commands