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
published: false
---

May 2019

# Overview

The [Azure AD connect] service is used to syncronise on premises Active Directory objects to Azure Active Directory. There are a number of alerts that come with the sync service all ready built in (connect health is currently available in [P1 and P2 plans] only), however it will only alert if there has been no sync for over 24 hours. I contacted Azure support to see if this could be amended but that is not possible at present and was given the work around to use the ``` Get-MsolCompanyInformation ``` to see the last sync time. I implemented the workaround as an Azure automation runbook that posts to slack when the sync has not completed within the last 2 hours.
Below is the code to achieve this all deployed via PowerShell core using the PowerShell [AZ module]. This example uses a free automation account but be wary of any potential charges.

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

## MSOnline module

The MSOnline need to be imported into the Automation account to make it available to Runbooks. We can do this by the PowerShell code below (there is an option in the portal too).

```powershell
# Create the module uri
$Module = 'MSOnline'
$uri = (Find-Module $Module).RepositorySourceLocation + 'package/' + $module

# Import the module
New-AzAutomationModule -Name $Module -ContentLinkUri $uri -ResourceGroupName $Name -AutomationAccountName $Name
```

## Create an Azure automation credential

To run the MSOnline commands requires 'Global Admin' so the credential entered here needs to be in that role. See encrypted creds in Azure automation. - Need to test

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

Update the date to check by removing the .AddHours() method which will cause the alert to fire (in this demo just output to the console).

Click the Test Pane
Click Start
Wait for job to complete.

output.png

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

## Summary

Azure AD connect is an extremely important service and as you move more applications and features to Azure AD, it is vital that this is running and updating objects. The 24 hour period seems too long in my opinion for no sync to happen and 2 hours seems about right (the sync runs every 30 mins which is the [minimum] currently allowed by Azure AD)

[Azure AD Connect]: https://docs.microsoft.com/en-us/azure/active-directory/hybrid/whatis-azure-ad-connect
[P1 & P2]: https://azure.microsoft.com/en-ca/pricing/details/active-directory/
[AZ module]: https://docs.microsoft.com/en-us/powershell/azure/new-azureps-module-az?view=azps-2.0.0
[docs]: https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-2.0.0
[bug]: https://github.com/Azure/azure-powershell/issues/8600
[Azure portal]: https://portal.azure.com
[minimum]: https://docs.microsoft.com/en-us/azure/active-directory/hybrid/how-to-connect-sync-feature-scheduler#scheduler-configuration