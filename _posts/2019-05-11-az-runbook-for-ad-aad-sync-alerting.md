---
title: Azure Runbook to check last Active Directory to Azure AD sync time
author: Matthew Davis
date: 2019-05-11
toc: true
excerpt: Create an Azure Automation Runbook to check the last sync time between Active Directory and Azure AD.
categories:
- azure
tags:
- runbook
- azure
- powershell
published: true
---

May 2019

# Overview

The [Azure AD connect] service is used to syncronise on premises Active Directory objects to Azure Active Directory. There are a number of alerts that come with the sync service all ready built in (connect health is currently available in [P1 and P2 plans] only), however it will only alert if there has been no sync for over 24 hours. I contacted Azure support to see if this could be amended but that is not possible at present and was given the work around to use the ``` Get-MsolCompanyInformation ``` to see the last sync time. I implemented the workaround as an Azure automation runbook that posts to slack when the sync has not completed within the last 2 hours.
Below is the code to achieve this all deployed via PowerShell core using the PowerShell [AZ module]. This example uses a free automation account which at the time of writing gets you 500 free minutes every month, see the [pricing page] for further details as you may be charged.

```powershell
PSVersion                      6.1.3
PSEdition                      Core
OS                             Microsoft Windows 10
Az                             2.0.0
```

## Install Azure Cmdlet

See install [docs]

```powershell
Install-Module -Name Az -AllowClobber
```

## Connect to Azure

```powershell
# Will be prompted to sign in via browser
Connect-AzAccount

# Use set or select context if you need to change subscriptions.
Set-AzContext -Name 'VSP' -Subscription 'Visual Studio Professional'
Select-AzContext 'VSP'
```

## Resource Group and Automation account

First thing needed is a Resource Group and Azure automation Account (skip this part if you already have one but make sure you get the resource group and automation account names).
To make things easier, I am using the variable 'Name' for both the automation account and resource group's name.

```powershell
$Location = 'canadacentral'
$Name = 'MattDemo'
$Tags = @{'Project' = 'MattDemo'}

New-AzResourceGroup -Name $Name -location $Location -tag $Tags | Out-Null

New-AzAutomationAccount -ResourceGroupName $Name -Name $Name -Location $Location -Plan Free -Tags $Tags
```

![New resource group and account](/images/az-runbook-ad-sync/new-acct.png)

## MSOnline module

The MSOnline needs to be imported into the Automation account to make it available to the Runbook to use. We can do this by the PowerShell code below (there is an option in the portal too).

```powershell
# Create the module uri
$Module = 'MSOnline'
$uri = (Find-Module $Module).RepositorySourceLocation + 'package/' + $module

# Import the module
New-AzAutomationModule -Name $Module -ContentLinkUri $uri -ResourceGroupName $Name -AutomationAccountName $Name
```

![Import module to Azure Automation](/images/az-runbook-ad-sync/import-module.png)

## Create an Azure automation credential

The Global Admin role is required to run the MSOnline Cmdlets so the credential entered here needs to be in that role. The credentials are stored [encrypted] in the Azure automation account.

```powershell
New-AzAutomationCredential -Name  'AzureADConnectSyncAccount' -ResourceGroupName $Name -AutomationAccountName $Name -Value (Get-Credential)
```

## The runbook

<script src="https://gist.github.com/MatthewJDavis/ac373ee8c56446696981228aeb2d6c7f.js"></script>

Copy and save this runbook somewhere local like 'C:\Temp\Test-LastAzureADSyncTime.ps1'

```powershell
$uri = 'https://gist.githubusercontent.com/MatthewJDavis/ac373ee8c56446696981228aeb2d6c7f/raw/bae82a17f6140359171336b0f39b52336d5cbc05/Test-LastAzureADSyncTimeBasic.ps1'
Invoke-WebRequest -Uri $uri -OutFile 'C:\Temp\Test-LastAzureADSyncTime.ps1'
```

## Import and Publish the runbook

Next step is to import the runbook to the automation account and publish it at the same time.

```powershell

$RBName = 'Test-LastAzureADSyncTime'

$params = @{
    'Path'                  = 'C:\Temp\Test-LastAzureADSyncTime.ps1'
    'Description'           = 'Checks last Sync time of AD to Azure AD'
    'Name'                  = $RBName
    'Type'                  = 'PowerShell'
    'Tags'                  = $Tags
    'ResourceGroupName'     = $Name
    'AutomationAccountName' = $Name
    'Published'             = $true
}

Import-AzAutomationRunbook @params
```

## Testing

You can test the runbook via the [Azure portal].

![Edit pane overview](/images/az-runbook-ad-sync/edit.png)

Update the date to check by removing the .AddHours() method which will cause the alert to fire (in this demo just output to the console).

![Change the date in the edit pane](/images/az-runbook-ad-sync/change-date.png)

Click the Test Pane
Click Start
Wait for job to complete.

![Output from running the command in test](/images/az-runbook-ad-sync/output.png)

Change ```(Get-Date)``` back to ```(Get-Date).AddHours(-2)``` and this will now fire when the sync has not ran within 2 hours.

## Add schedule

We are going to add a schedule to trigger the runbook hourly.

```powershell
# Create the schedule
$ScheduleName = 'RunADSyncCheckHourly'

$schParams = @{
    Name                  = $ScheduleName
    HourInterval          = 1
    ResourceGroupName     = $Name
    AutomationAccountName = $name
    StartTime             = (get-date).AddMinutes(10)
}

New-AzAutomationSchedule @schParams

# Link schedule to runbook

$regParams = @{
    RunbookName = $RBName
    ScheduleName = $ScheduleName
    ResourceGroupName = $Name
    AutomationAccountName = $Name
}

Register-AzAutomationScheduledRunbook @regParams
```

![Registering the runbook to the schedule](/images/az-runbook-ad-sync/register.png)

The runbook will now run every hour.

## Start the runbook from PowerShell

You can also start the runbook via PowerShell.

Currently there is a bug with the ```Get-AzAutomationJobOutputRecord``` Cmdlet that looks to have a fix merged in but has not been released to the most recent version which I was using: version 1.2.1 for the Az.Automation module.

```powershell
$job = Start-AzAutomationRunbook -Name $RBName -ResourceGroupName $Name -AutomationAccountName $Name

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name | Get-AzAutomationJobOutput

# Full job output - broken cmdlet see below

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name | Get-AzAutomationJobOutputRecord
```

## Example: Sending an alert

The above code gives the basic overview and structure and below is code that will send a notification to a Slack channel using a basic webhook. See the [slack apps documentation] on creating the app and webhook, you will need to add an encryted automation variable for the Slack webhook URI.

```powershell
# Create the automation variable
New-AzAutomationVariable -Name 'AlertsSlackWebHookUri' -Encrypted $true -value (Read-Host) -ResourceGroupName $Name -AutomationAccountName $Name
```

Complete script to send a basic message to a slack channel with helper function to post to slack.

<script src="https://gist.github.com/MatthewJDavis/d2a52cb13b4ddf53eefd9e680b6327e3.js"></script>

![Output in slack](/images/az-runbook-ad-sync/slack.png)

## Summary

Azure AD connect is an extremely important service and as you move more applications and features to Azure AD, it is vital that this is running and updating objects. The 24 hour period seems too long in my opinion for no sync to happen and 2 hours seems about right (the sync runs every 30 mins which is the [minimum] currently allowed by Azure AD).
I've seen it where the console was left open in a disconnected session on the server which means that there is no sync which we would not of been aware of for over 24 hours as it was going into the weekend and the email alerts don't trigger highlevel alerts.
This is another good example of using Azure runbooks as orchestration for handy PowerShell scripts.

[Azure AD Connect]: https://docs.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect
[P1 and P2 plans]: https://azure.microsoft.com/en-ca/pricing/details/active-directory/
[Pricing Page]: https://azure.microsoft.com/en-ca/pricing/details/automation/
[AZ module]: https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-2.0.0
[docs]: https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-2.0.0
[encrypted]: https://docs.microsoft.com/en-us/azure/automation/shared-resources/credentials
[bug]: https://github.com/Azure/azure-powershell/issues/8600
[Azure portal]: https://portal.azure.com
[Slack apps documentation]: https://get.slack.help/hc/en-us/articles/115005265063-Incoming-WebHooks-for-Slack
[minimum]: https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-sync-feature-scheduler#scheduler-configuration