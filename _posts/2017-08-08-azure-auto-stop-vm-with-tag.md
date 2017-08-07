---
title: Stop Azure VMs with Azure Automation PowerShell runbook
author: Matthew Davis
date: 2017-08-06
excerpt: Create a PowerShell run book to stop VMs with a certain tag value to automatically stop Azure VMs
categories: 
    - azure
tags:
    - azure
    - azure automation
    - powershell runbook
    - virtual machines
published: false
---

# Automating Virtual Machine shut down with Azure Automation
Leaving your Virtual Machines (VMs) running when they are not needed is an unnecessary cost. With DevTest Labs auto-shutdown feature you can schedule your development VMs to shutdown on a schedule and this feature is available now to individual VMs but controlling shutdown schedules with Tags and Azure automation allows for a solution that can scale to many VMs easily.
I've also had it when I've just shut down the VM from Windows, but there are still charges because the VM is holding resources and not deallocated from the Azure fabric.

The new **Azure Mobile App88 allows you to remotely shutdown your VMs if you forget while you're on the move, handy if you remember you left one running and it's not on an auto-shutdown schedule.

This post will show you how to
- Tag VMs
- Create an Azure PowerShell workflow to shutdown VMs
- Create Automation variables
- Create an Automation schedule
- Link a PowerShell workflow to a schedule

## Tagging the VM

The script will look for Virtual Machines that have the powerOffTime tag set to a specific value (this example uses 23:00).
You can tag the VMs via the portal or with PowerShell as shown below.
Tags are Key Value pairs that are attached to resources and enable you to group them via the tag values, for instance you could tag all resources by department and run reports on all resources used by a particular department.

### Authenticate to AzureRM and select the subscription

```PowerShell
Add-AzureRmAccount
```

```PowerShell
Select-AzureRmSubscription -SubscriptionName subName
```

### Save the tags in a hash table
Create a hashtable to store the tag key value pairs, the below hastable will create the following tags, so change the keys and values to what you like, the important one is the powerOffTime tag:
- Env:Demo
- powerOffTime:23:00 (this is the one the script will look for when it is run)
- createdBy:Matt
- project:Domain Services
- role:Domain Services Management VM

```PowerShell
$tags = @{"Env"="Demo";"powerOffTime"='23:00';'createdBy'='Matt';'project'='Domain Services';'role'='Domain Services Management VM'}
```

### Save the VM in a variable
To add the tag to a VM (or it could be a set of VMs), we need to save the VM object into a variable. Replace the name and resourcgroup to get your VM>

```PowerShell
$vm = get-azurermvm -Name ds-manag-vm -ResourceGroupName DOMAIN-SERVICES-RG
```


### Update the VM with the tags
Now we pass the VM object which is stored in the $vm variable to the Update-AzureRmVM command along with the tags hashtable. This will update the VM with the tags.

```PowerShell
$vm | update-azurermvm -Tags $tags
```

## Create and upload an Azure PowerShell runbook
This workflow has the powerOffTime parameter set to a default time of 23:00, you can pass other time values to the script so that it is reusable if you wanted to shutdown vms on different schedules. 
The script also gets an Azure automation variable with the name of the subscription to target so you can control which subscriptions the script is run against, for instance you may not want it run against your production subscription etc.

After authenticating to Azure RM and selecting the subscription, the script stores VMs that have been tagged with the "powerOffTime" tag and has a matching value of the time that was passed through (in this case 23:00) in the $vms variable.  
The foreach loop, cycles through each value in the $vms variable outputting the name (this will be viable in the logs) and sending the shutdown command to deallocate the VM and stop the charges.

Save the following script:

<script src="https://gist.github.com/MatthewJDavis/f982af8f4cf9a4632b447a71d356a9e5.js"></script>

### Publish the runbook
Update the variables as needed (the path where the runbook was saved, automation account and resource group name and description if you like).
The parameters are splatted into a hashtable to make for easier reading and then passed to the Import-AzureRmAutomationRunbook cmdlet. 


```PowerShell
$pathToRunbook = 'C:\Downloads\Stop-MDAzureRMVMByTag.ps1'
$description = 'stop VMs by powerOffTime tag value'
$runbookType = 'PowerShellWorkflow'
$automationAccountName = 'matt-auto-acct'
$resourceGroupName = 'automation'

$runbookParams = @{
  'Path' = $pathToRunbook;
  'Description' = $description;
  'Type' = $runbookType
  'AutomationAccountName' = $automationAccountName;
  'ResourceGroupName' = $resourceGroupName;
  'Published' = $true 
}

Import-AzureRmAutomationRunbook @runbookParams
```

## Creating the subscription variable 
New-AzureRmAutomationVariable -Name 'subscriptionName' -Value 'demo' -Encrypted $false -AutomationAccountName matt-auto-acct -ResourceGroupName automation

