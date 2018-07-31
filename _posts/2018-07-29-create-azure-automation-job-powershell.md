---
title: Create Azure Automation Job triggered by a webhook with PowerShell
author: Matthew Davis
date: 2018-07-29
excerpt: Create an Azure automation job that is triggered by a POST to a webhook with PowerShell and output data from the JSON payload
categories: 
    - azure
tags:
    - azure
    - azure automation
    - powershell runbook
    - webhook
published: true
---

I've just completed a task to restart a Windows service when [splunk] detects that is not running on a Windows server. Once the service is detected as not running, splunk sends an https POST with a JSON payload to an Azure Automation runbook webhook. This post shows how to get the data from a JSON payload and is the platform of how I built the script to start the service from the data sent from splunk.

As it's been a while since I have used Azure automation and runbooks, and after being inspired by recently watching this [channel 9 video] to complete the task with PowerShell (something I had done previously but not for ages), I've decided to write it up here.

Install the AzureRM PowerShell module if it is not already on the system

```powershell
Install-Module -Name AzureRM
```

Connect to Azure and select the correct subscription (if you have more than one)

```powershell
Add-AzureRMAccount

Get-AzureRMSubscription

Select-AzureRMSubscription -SubscriptionID [yoursubscriptionID]
```

## The Runbook

The runbook takes one parameter, which is an object with the name WebhookData. This object has the following properties from the POST request:

1. WebhookName = $WebhookData.WebhookName
2. RequestHeader = $WebhookData.RequestHeader
3. RequestBody = $ $WebhookData.RequestBody

(From the [Microsoft docs on Automation])

The if block on line 12 is needed for testing if you were to use the Azure portal's test pane (shown later). A real webhook received will contain data in the $Webhook.RequestBody, however this isn't the case from the test pane so this logic is required for testing data passed from there.

The data is converted from JSON to a PowerShell object and the properties in the RequestBody property is saved to another variable for output.

The service name and host name are outputted from the script.

<script src="https://gist.github.com/MatthewJDavis/4598eef65dfb370fb0e1d2306fe03d4d.js"></script>

## Creating the resources

For the purposes of this post, I'm going to create a brand new Resource Group and Automation Account. I've broken the script up so I can include screenshots but this should all be ran in the same PowerShell session where the variables are declared and the authentication is done with Azure.

```powershell
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

```powershell
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

```powershell
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

### Important

 The URI returned from the following command contains the security token. It is displayed only once and can't be retrieved at a later time. In production you would save this URI in a safe location such as an encrypted password manager. For this demo I am storing it in the webhookOutput variable but it would be lost as soon as the session is closed or the variables are cleared. Make a note of it if you need it in the future otherwise you will have to recreate the webhook.

```powershell
# Create the webhook with a 5 year expiry date
# Force to accept the warning that the WebhookURI will only be available once
# This information is saved in the webhookOutput variable for this demo but should be stored somewhere secure in production because it includes the security token in the URI and is only available when the webhook is created - it can't be retrieved after
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

```powershell

# Send JSON Payload to the webhook with a Service name and host name
$json = '{
  "WebhookName": "service-host-webhook",
  "RequestBody": "{\"Service\": \"xbgm\",\"Host\": \"vm01\" }"
}'

Invoke-WebRequest -Uri $webhookOutput.WebhookURI -UseBasicParsing -Method Post -Body $json
```

![JSON data saved to a variable](/images/azure-webhook/json-var.png)

![Invoke the web request to send the data to the webhook](/images/azure-webhook/invoke-webrequest.png)

```powershell
# Get the output of the job. Once the job status is completed, we need the job id (saved in the variable job, then we can get the output
Get-AzureRmAutomationJob -RunbookName $runbookName -ResourceGroupName $rgName -AutomationAccountName $autoAcctName
$job = Get-AzureRmAutomationJob -RunbookName $runbookName -ResourceGroupName $rgName -AutomationAccountName $autoAcctName

Get-AzureRmAutomationJobOutput -Id $job.JobId -ResourceGroupName $rgName -AutomationAccountName $autoAcctName
```

![Get automation job](/images/azure-webhook/get-automationjob.png)

![Get output from the automation job](/images/azure-webhook/get-automationjoboutput.png)

## Testing from the portal

You can do all of this manually from the portal if you like (the portal interacts with the same AzureRM APIs as PowerShell does behind the scenes).

I'm not going to go through setting up the resource group, automation account, uploading the runbook and creating the webhook (if you upload the runbook, make sure you publish it otherwise you won't be able to create the webhook) but will show how to run the test.

Select your runbook in the Azure automation account and click **Edit**.

![Test panel Azure automation](/images/azure-webhook/edit-runbook.png)

From the top menu, click **Test Pane**.

![Test panel Azure automation](/images/azure-webhook/test-pane-button.png)

In the test panel, enter the JSON data

```json
{
  "WebhookName": "service-host-webhook",
  "RequestBody": "{\"Service\": \"xbgm\",\"Host\": \"vm01\" }"
}
```

Click **Start**

![Test panel Azure automation](/images/azure-webhook/test-panel.png)

The request will be queued up and the results will be displayed in the test pane

![Test result](/images/azure-webhook/test-result.png)

You will see the Write-Output from within the if block that is true because there is not a value in $WebhookData.Request property.

You can also access this data via PowerShell by running the Get-AzureRmAutomationJob and Get-AzureRmAutomationJobOutput as above.

All that is left to do is clean up.

```powershell
# Tidy up resources
Remove-AzureRmResourceGroup -Name $rgName -Force
```

[splunk]: https://www.splunk.com/
[channel 9 video]: https://channel9.msdn.com/Shows/DevOps-Lab/Azure-Automation-Runbooks-with-PowerShell
[Microsoft Docs on automation]: https://docs.microsoft.com/en-us/azure/automation/automation-webhooks