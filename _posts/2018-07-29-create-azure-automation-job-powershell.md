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

I've just completed a task to restart a service when [splunk] detects that is not running on a Windows server. When Splunk detects that the service is not running, it sends data to an Azure automation runbook I created via a JSON payload.

As it's been a while since I have used Azure automation and runbooks, and after being inspired by recently watching this [channel 9 video] to complete the task using PowerShell (something I had done previously but not for ages) I've decided to write it up here.

If not already installed, install the AzureRM PowerShell module

```PowerShell
Install-Module -Name AzureRM
```

Connect to Azure and select the correct subscription (if you have more than one)

```PowerShell
Add-AzureRMAccount

Get-AzureRMSubscription

Select-AzureRMSubscription -SubscriptionID [yoursubscriptionID]

```

## The Runbook

<script src="https://gist.github.com/MatthewJDavis/4598eef65dfb370fb0e1d2306fe03d4d.js"></script>

## Creating the resources
For the purposes of this post, I'm going to create a brand new Resource Group and Automation Account.

```PowerShell
# Creates resource group, automation account, imports a local PowerShell runbook and creates a webhook
# Update runbookPath variable to location of where the PowerShell runbook is stored locally

$rgName = 'demo-automation-demo-rg'
$rgLocation = 'northeurope'
$tags = @{'Project'='Demo';'Environment'='Test'}
$autoAcctName = 'demo-automation-account'
$runbookName = 'display-data-from-JSON'
$runbookDesc = 'Demo of receiving data from a webhook'
$runbookPath = 'C:\git\Azure\Automation\runbooks\webhook-demo\New-WebhookDisplayData.ps1'
$webhookName = 'demo-webhook'

# Create the resource group
$rg = New-AzureRMResourceGroup -Name $rgName -Location $rgLocation -Tags $tags

```

```PowerShell
# Create the automation account, splat the params
$autoAcctParams = @{
  'Name' = $autoAcctName;
  'ResourceGroupName' = $rgName;
  'Plan' = 'Free';
  'Location' = $rgLocation;
  'Tags' = $tags 
}

New-AzureRMAutomationAccount @autoAcctParams

```


![Create a new automation account](/images/azure-webhook/new-automation-account.png)

```PowerShell
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

```

![Import the PowerShell runbook](/images/azure-webhook/import-runbook.png)


```PowerShell
# Create the webhook with a 5 year expiry date
# Force to accept the warning that the WebhookURI will only be available once
# This information is saved in the webhookOutput variable for this demo but should be stored somewhere secure in production because it includes the authorisation token in the URI
# View the URI by  $webhookOutput.WebhookURI
$hookParams = @{
  'AutomationAccountName' = $autoAcctName;
  'ResourceGroupName'     = $rgName;
  'RunbookName'           = $runbookName;
  'Name'                  = $webhookName;
  'IsEnabled'             = $true;
  'ExpiryTime'            = (get-date).AddYears(5);
  'Force'                 = $true
}

$webhookOutput = New-AzureRmAutomationWebhook @hookParams

```

![New webhook](/images/azure-webhook/new-webhook.png)

## Testing the runbook

```PowerShell

# Send JSON Payload to the webhook with a Service name and host name
$json = '{
  "WebhookName": "service-host-webhook",
  "RequestBody": "{\"Service\": \"xbgm\",\"Host\": \"vm01\" }"
}'

Invoke-WebRequest -Uri $webhookOutput.WebhookURI -UseBasicParsing -Method Post -Body $json

```

![JSON data saved to a variable](/images/azure-webhook/json-var.png)
![Invoke the web request to send the data to the webhook](/images/azure-webhook/invoke-webrequest.png)

```PowerShell
# Get the output of the job. Once the job status is completed, we need the job id (saved in the variable job, then we can get the output
Get-AzureRmAutomationJob -RunbookName $runbookName -ResourceGroupName $rgName -AutomationAccountName $autoAcctName
$job = Get-AzureRmAutomationJob -RunbookName $runbookName -ResourceGroupName $rgName -AutomationAccountName $autoAcctName

Get-AzureRmAutomationJobOutput -Id $job.JobId -ResourceGroupName $rgName -AutomationAccountName $autoAcctName
```

![Get automation job](/images/azure-webhook/get-automationjob.png)
![Get output from the automation job](/images/azure-webhook/get-automationjoboutput.png)

```PowerShell
# Tidy up resources
Remove-AzureRmResourceGroup -Name $rgName -Force
```
[splunk]: https://www.splunk.com/
[channel 9 video]: https://channel9.msdn.com/Shows/DevOps-Lab/Azure-Automation-Runbooks-with-PowerShell