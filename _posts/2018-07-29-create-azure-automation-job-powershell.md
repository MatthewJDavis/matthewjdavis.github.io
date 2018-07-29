---
title: Create Azure Automation Job triggered by a webhook with PowerShell
author: Matthew Davis
date: 2018-07-29
excerpt: How to create an Azure automation job that is triggered by a webhook with PowerShell
categories: 
    - azure
tags:
    - azure
    - azure automation
    - powershell runbook
published: false
---

I've just completed a task to restart a service when Splunk detects that is not running on a Windows server. When Splunk detects that the service is not running, it sends data to an Azure automation runbook I created via a JSON payload.

As it's been a while since I have used Azure automation and runbooks, and after being inspired by recently watching this channel 9 video to complete the task using PowerShell (something I had done previously but not for ages) I've decided to write it up here.

If not already installed, install the AzureRM PowerShell module

```PowerShell
Install-Module -Name AzureRM
```

Connect to Azure and select the correct subscription (if you have more than one)

```PowerShell
Add-AzureRMAccount


Get-AzureRMSubscription

Select-AzureRMSubscription -SubscriptionID
```

For the purposes of this post, I'm going to create a brand new Resource Group and Automation Account.

```PowerShell
# Creates resource group, automation account, imports a local PowerShell runbook and creates a webhook

$rgName = 'demo-automation-acct'
$rgLocation = 'northeurope'
$tags = @{'Project'='Demo';'Environment'='Test'}
$autoAcctName = 'demo-automation-account'
$runbookName = 'test-input-output'
$runbookDesc = 'Demo of receiving data from a webhook'
$runbookPath = 'C:\git\Azure\Automation\runbooks\webhook-demo\New-WebhookDisplayData.ps1'
$webhookName = 'demo-webhook'

# Create the resource group
$rg = New-AzureRMResourceGroup -Name $rgName -Location $rgLocation -Tags $tags

# Create the automation account, splat the params
$autoAcctParams = @{
  'Name' = $autoAcctName;
  'ResourceGroupName' = $rgName;
  'Plan' = 'Free';
  'Location' = $rgLocation;
  'Tags' = $tags 
}

New-AzureRMAutomationAccount @autoAcctParams

# Import the local runbook
$runbookParams = @{
  'Path' = $runbookPath;
  'Name' = $runbookName;
  'ResourceGroupName' = $rgName;
  'AutomationAccountName' = $autoAcctName;
  'Description' = $runbookDesc;
  'Tags'  = $tags;
  'Type' = 'PowerShell';
  'Published' = $true
}

Import-AzureRmAutomationRunbook @runbookParams


# Create the webhook with a 5 year expiry date
$hookParams = @{
  'AutomationAccountName' = $autoAcctName;
  'ResourceGroupName'     = $rgName;
  'RunbookName'           = $runbookName;
  'Name'                  = $webhookName;
  'IsEnabled'             = $true;
  'ExpiryTime'            = (get-date).AddYears(5)
}

$webhookOutput = New-AzureRmAutomationWebhook @hookParams

# Send JSON Payload to the webhook with a Service name and host name
$json = '{
  "WebhookName": "service-host-webhook",
  "RequestBody": "{\"Service\": \"xbgm\",\"Host\": \"vm01\" }"
}'

Invoke-WebRequest -Uri $webhookOutput.WebhookURI -UseBasicParsing -Method Post -Body $json

# Get the output of the job. First we need the job id (saved in the variable job, then we can get the output
$job = Get-AzureRmAutomationJob -RunbookName $runbookName -ResourceGroupName $rgName -AutomationAccountName $autoAcctName

Get-AzureRmAutomationJobOutput -Id $job.JobId -ResourceGroupName $rgName -AutomationAccountName $autoAcctName

# Tidy up resources
Remove-AzureRmResourceGroup -Name $rgName -Force
```