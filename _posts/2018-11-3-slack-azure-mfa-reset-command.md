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

# Reset Azure MFA with Slack - the need.

Currently only users in the [Global Admin] role can reset the Azure Multi Factor Authenticate (MFA) details of users which is used as two factor authentication for Azure, D365 and Office 365. There has been an [Azure feedback request] (I've been monitoring this for a while now and have voted) with Mircrosoft to open this up to other roles for over 3 years with no movement, it would make sense that the Password/ Helpdesk Administrator role would be able to reset the MFA. This was a big pain point for us recently, sometimes getting up to 5 requests a day for resets from the helpdesk because we couldn't give them Global Admin rights to our Azure tenant!

Having recently moved to Slack, I decided to have a look to see if it would be possible for the helpdesk to use Slack to reset Azure MFA so they no longer had to escalate it to our team.

What I came up with was a [Slack slash command] that send the UPN of the user to reset their MFA to an Azure runbook that then ran a PowerShell command to reset MFA using a service account (the service account has to be a global admin).

Notes

1. This uses the [MSOnline] (MSOL) PowerShell module which is version 1 of the Azure Active Directory modules and there is a newer module that Microsoft encourage you to use. However, the V2 module (AzureAD) does not have the ability to reset Azure MFA for some reason and I have found the features of V2 lacking compared to V1.

2. A service account is required with Global Admin priviladges. This is a security trade off so limit the users that have access to the credentials of this account. When Microsoft finally implement a less privileged role to reset MFA, then the service account can be removed from Global Admins to the less privileged role.

## Azure Automation Runbook

I'll start with a run through of the runbook that makes all of this happen, [Link to full script on Github.]

It has one parameter that takes the input of the webhook data.

A hashtable is used to define users who are allowed to actually perform the reset. I could not find a way to lock the Slack command down to certain users, so any user can run the command, however only users who has their SlackID in this hash table will be allowed to actually execute the Reset-MsolStrongAuthenticationMethodByUpn PowerShell cmdlet.

```powershell
param
(
  [object] $WebhookData
)

#Hashtable of authorised users to run the command - add users and slack ID to this hashtable to allow them to reset Azure MFA
$authorisedUsers = @{
  'helpdesk.user1' = 'USlackID1'
  'helpdesk.user2' = 'USlackID2'
  'helpdesk.user3' = 'USlackID3'
  'helpdesk.user4' = 'USlackID4'
}
```

The runbook connects to AzureAD with the Connect-MSOLService (MSOL [module should be installed] in the Azure automation account) Cmdlet using the credentials stored in Azure automation of a service account that is a global admin (had a long strong random password and only a few users have access to password).

```powershell
Import-Module -name MSOnline
$creds = Get-AutomationPSCredential -Name 'Azure-AD-MFA-Reset'
Connect-MsolService -Credential $creds
```

Slack sends the data separated with the ampersand symbol (&), so the split method is used to save the separated data in a new variable.

```powershell
$slackData = $WebhookData.RequestBody.Split('&')
```

Next, the UPN, SlackID and return URL (where you send the JSON payload responses) is extracted from the data sent by Slack. The data is sent in [percent encoding], so needs to be transformed back into ASCII.

```powershell
 # Get the UPN supplied from Slack
  $upnData = $slackData | Where-Object {$_ -like 'text*'}
  $upnSplit = $upnData.Split('=')
  $upnToReset = $upnSplit[1].Replace('%40', '@')
  Write-Output "UPNToReset = $upnToReset"

  # Get the user who requested the reset
  $userData = $slackData | Where-Object {$_ -like 'user_id*'}
  $userSplit = $userData.Split('=')
  $userWhoSentRequest = $userSplit[1]
  Write-Output "User who sent request = $userWhoSentRequest"

  # Response URL
  $uri = $slackData | Where-Object {$_ -like 'response_url*'}
  $uriSplit = $uri.Split('=')
  $responseUri = $uriSplit[1]
  $responseUri = $responseUri.Replace('%3A', ':')
  $responseUri = $responseUri.Replace('%2F', '/')
  Write-Output "Response URI = $responseUri"
```

Now we have the data, we need to check that a UPN has been sent. I'm doing a basic check for the @ and . symbols are sent. Also form the JSON payload to send back to Slack and send it to the response URL which will post a message back to the user.

```powershell
  # Check that a full UPN has been sent
  if (-Not ($upnToReset.Contains('@') -and $upnToReset.Contains('.'))) {
    Write-Output 'Please provide a full UPN i.e. firstname.surname@domain.com'
    # No @ or . found in provided UPN so send message and break
    $json = "
      {
          'response_type': 'ephemeral',
          'text': 'Reset-Azure-MFA The provided UPN was not in the UPN format of firstname.surename@domain. UPN sent was: $upnToReset'
      }
      "
    Invoke-WebRequest -UseBasicParsing -Uri $responseUri -Method Post -Body $json
    break
  }
```

Now that we know the UPN is in the correct format, check that the user who sent the Slack slash command is in the authorised users hashtable, check there is a user with the UPN supplied, if so reset the MFA setting and then check that the setting has been cleared and send response to Slack. If the user is not in the authorised users hashtable, the UPN can not be found or there was a problem removing the MFA settings, send the appropriate response to Slack.

```powershell
  # Check user who sent request is allowed to reset MFA
  if ($authorisedUsers.ContainsValue($userWhoSentRequest)) {
    if (Get-MsolUser -SearchString $upnToReset) {

      # Found user, reset MFA      
      Reset-MsolStrongAuthenticationMethodByUpn -UserPrincipalName $upnToReset

      # Check the MFA settings have been cleared for the user
      if ((Get-MsolUser -SearchString $upnToReset| Select-Object -ExpandProperty  StrongAuthenticationMethods | Measure-Object).Count -eq 0) {
        $json = "
          {
              'response_type': 'ephemeral',
              'text': 'Reset-Azure-MFA has reset Azure MFA for: $upnToReset'
          }
          "
      }
      else {
        # Problem with clearing MFA settings
        $json = "
          {
              'response_type': 'ephemeral',
              'text': 'Reset-Azure-MFA There was a problem with the reset of Azure MFA for: $upnToReset. Please try again.'
          }
          "
      }
      Invoke-WebRequest -UseBasicParsing -Uri $responseUri -Method Post -Body $json
    }
    else {
      # User not found via the supplied UPN
      $json = "
        {
            'response_type': 'ephemeral',
            'text': 'Reset-Azure-MFA could not find the UPN for: $upnToReset in Azure. Please check the UPN supplied'
        }
        "
      Invoke-WebRequest -UseBasicParsing -Uri $responseUri -Method Post -Body $json
    } 
  }
  else {
    $json = "
      {
        'response_type': 'ephemeral',
        'text': '$userWhoSentRequest Slack User ID is not authorised to reset Azure MFA. Please contact ask to have your Slack ID added.'
      }
      "
    Invoke-WebRequest -UseBasicParsing -Uri $responseUri -Method Post -Body $json
  }    
```

## Azure Automation Job

We need the following to run the runbook via a webhook sent by Slack
- Azure Automation Account
- Service account added to Global Admin role, with credentials added to Azure automation 
- MSOL Module installed
- Upload the runbook and set to run via a Webhook

### Automation Account

If you don't already have one, set up a [new Azure automation account] either in the portal or via PowerShell [New-AzureRmAutomationAccount]. You get 500 minutes of job run time free per month a present.

### Service Account

Create a service account and give it the Global Admin role in Azure (if you don't already have one). This may need to be created in your on premises directory and then synced to Azure AD or directly in Azure AD depending on how your directory is configured / how you prefer to manager your users.

Save the service account's credentials in the Azure Automation account so the runbook will be able to access them to authenticate with Azure AD.

![Azure automation credentials](/images/slack-azure-mfa-reset/service-account-creds.png)

### Install MSOL Module

In the automation account, select *Modules* then click on Browse Gallery under the *More* menu

![Azure automation credentials](/images/slack-azure-mfa-reset/browse-module.png)

Search for MSOnline and click on *Import* and then *OK*. The import process will take a couple of minutes and now the PowerShell Cmdlets will be available to the automation account runbooks.

![Azure automation credentials](/images/slack-azure-mfa-reset/msol-module.png)

### Upload Runbook and set to run via a webhook

We need to create an Azure Automation Runbook that is triggered via a webhook. I've written a [post here] on how to do it and the official guide is [here from Microsoft].

Quick overview

Save the runbook somewhere local and use either PowerShell or the portal to import it.

Click on the runbook and select the webhook (if the webhook option is not displaying, make sure that the runbook is published)

![Azure automation credentials](/images/slack-azure-mfa-reset/runbook-webook.png)

Make a note of the webhook URL somewhere secure. This is only shown once and also contains the security token to run the webhook so needs to be stored securely and will be required to use in the request URL of the Slack Slash Command.

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
[MSOL]: https://docs.microsoft.com/en-us/powershell/module/msonline/?view=azureadps-1.0
[module should be installed]: https://docs.microsoft.com/en-us/azure/automation/automation-runbook-gallery#to-import-a-module-from-the-automation-module-gallery-with-the-azure-portal
[percent encoding]: https://en.wikipedia.org/wiki/Percent-encoding
[post here]: https://matthewdavis111.com/azure/create-azure-automation-job-powershell/
[here from Microsoft]: https://docs.microsoft.com/en-us/azure/automation/automation-creating-importing-runbook
[Link to full script on Github]: https://github.com/MatthewJDavis/Azure/blob/master/Automation/runbooks/slack/Reset-Azure-MFA.ps1
[Slack slash command]: https://api.slack.com/slash-commands
[new Azure automation account]: https://docs.microsoft.com/en-us/azure/automation/automation-create-standalone-account
[New-AzureRmAutomationAccount]: https://docs.microsoft.com/en-us/powershell/module/azurerm.automation/new-azurermautomationaccount?view=azurermps-6.11.0