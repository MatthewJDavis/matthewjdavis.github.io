-- -
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
-- -
May 2019

# Overview

The Azure AD connect service is used to syncronise on premises Active Directory object to Azure Active Directory. There are a number of alerts that come with the sync service all ready built in (depending on the plan you use), however it will only alert if there has been no sync for over 24 hours. I contact Azure support to see if this could be amended but that is not possible at present and was given the work around to use the ``` Get-MsolCompanyInformation ``` to see the last sync time. In the last two weeks we have had issues of the sync not working (one was because of a clean up of AD had gone over the export deletion threshold and the next occasion was because someone had left the wizard open) I implemented the workaround as an Azure automation runbook that posts to slack when the sync has not completed within the last 2 hours.
Below is the code to achieve this all deployed via PowerShell core (can be used in a CI/CD pipeline in TeamCity or Azure Devops etc).

## Install Azure cmdlet

See install [docs]

```powershell
Install-Module -Name Az -AllowClobber
```

## Connect to Azure

```powershell
Connect-AzAccount

Set-AzContext -Name 'VSP' -Subscription 'Visual Studio Professional'

Select-AzContext 'VSP'
```

## Resource Group and Automation account

First thing needed is a Resource Group and Azure automation Account (skip this part if you already have one but make sure you get the resourcegroup and automation account names).
To make things easier, I am using the variable 'Name' for both the automation account and resource group's name.

```powershell
$Location = 'canadacentral'
$Name = 'MattDemo'
$Tags = @{'Project' = 'MattDemo'}

New-AzResourceGroup -Name $Name -location $Location -tag $Tags

New-AzAutomationAccount -ResourceGroupName $Name -Name $Name -Location $Location -Plan Free -Tags $Tags
```

## MSOnline module

The MSOnline need to be imported into the Automation account to make it available to Runbooks. This can be done via the portal or by the PowerShell code below.

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

Import and Publish the runbook

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

# Start the runbook

```powershell
$job = Start-AzAutomationRunbook -Name $RBName -ResourceGroupName $Name -AutomationAccountName $Name

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name | Get-AzAutomationJobOutput

# Full job output - broken cmdlet see below

Get-AzAutomationJob -JobId $job.JobId -ResourceGroupName $Name -AutomationAccountName $Name | Get-AzAutomationJobOutputRecord

Get-AzAutomationJobOutputRecord : The input object cannot be bound because it did not contain the information required to bind all mandatory parameters:  Id
```

## Full job output

Currently there is a bug with the ```Get-AzAutomationJobOutputRecord``` Cmdlet that looks to have a fix merged in, however I updated the Az module which gave me version 2.0.0 of the AZ module and version 1.2.1 for the Az.Automation module and it was still not working.

## Summary

[docs]: https://docs.microsoft.com/en-us/powershell/azure/install-az-ps?view=azps-2.0.0
[bug]: https://github.com/Azure/azure-powershell/issues/8600