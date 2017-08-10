---
title: Stop Azure VMs with Azure Automation PowerShell runbook
author: Matthew Davis
date: 2017-08-08
excerpt: Create a PowerShell run book to stop VMs with a certain tag value to automatically stop Azure VMs
categories: 
    - azure
tags:
    - azure
    - azure automation
    - powershell runbook
    - virtual machines

---

Leaving your Virtual Machines (VMs) running when they are not needed is an unnecessary cost. With [DevTest Labs] auto-shutdown feature you can schedule your development VMs to shutdown on a schedule and this feature is available now to individual VMs but controlling shutdown schedules with [Tags] and Azure automation allows for a solution that can scale to many VMs easily.

I've also had it when I've just shut down the VM from Windows, but there are still charges because the VM is holding resources and not deallocated from the Azure fabric.

The new [Azure Mobile App] allows you to remotely shutdown your VMs if you forget while you're on the move, handy if you remember you left one running and it's not on an auto-shutdown schedule.

This post will show you how to
- Tag VMs
- Create an Azure PowerShell workflow to shutdown VMs
- Create Automation variables
- Create an Automation schedule
- Link a PowerShell workflow to a schedule

You'll need:
- An [Azure subscription]
- An [Azure Automation Account] set up
- [Azure PowerShell]

## Tagging the VM

The script will look for Virtual Machines that have the powerOffTime tag set to a specific value (this example uses 23:00).
You can tag the VMs via the portal or with PowerShell as shown below.

Tags are Key Value pairs that are attached to resources and enable you to group them via the tag values, for instance you could tag all resources by department and run reports on all resources used by a particular department.

### Authenticate to AzureRM and select the subscription

```powershell
Add-AzureRmAccount
```

```powershell
Select-AzureRmSubscription -SubscriptionName subName
```

### Save the tags in a hash table
See the next section if your VM has existing tags and you just want to add one.

Create a hashtable to store the tag key value pairs, the below hastable will create the following tags, so change the keys and values to what you like, the important one is the powerOffTime tag:
- Env:Demo
- powerOffTime:23:00 (this is the one the script will look for when it is run)
- createdBy:Matt
- project:Domain Services
- role:Domain Services Management VM

```powershell
$tags = @{"Env"="Demo";"powerOffTime"='23:00';'createdBy'='Matt';'project'='Domain Services';'role'='Domain Services Management VM'}
```

### Save the VM in a variable
To add the tag to a VM (or it could be a set of VMs), we need to save the VM object into a variable. Replace the name and resourcgroup to get your VM>

```powershell
$vm = get-azurermvm -Name vmName -ResourceGroupName resourcegroupname
```

### Update the VM with the tags
Now we pass the VM object which is stored in the $vm variable to the Update-AzureRmVM command along with the tags hashtable. This will update the VM with the tags.

```powershell
$vm | update-azurermvm -Tags $tags
```
Once it's updated, check the tags by running:

```powershell
Get-AzureRmVm -Name vmname -ResourceGroupName resourcegroupname | Select-Object -ExpandProperty Tags
```

## Add tag to VM with existing tags
This will preserve the existing tags on the VM and just add an extra tag.
Save the VM object into a variable

```powershell
$vm = get-azurermvm -Name vmName -ResourceGroupName resourcegroupname
```

Using the add method of the Dictionary type (this is the type of object the tags are), we can add the power off tag to the vm varable with the following code.

```powershell
$vm.Tags.Add('powerOffTime','23:00')
```

Finally we pipe the VM object to the update-azurermvm cmdlet to add the tag.

```powershell
$vm | update-azurermvm -Tags $tags
```

Once it's updated, check the tags by running:

```powershell
Get-AzureRmVm -Name vmname -ResourceGroupName resourcegroupname | Select-Object -ExpandProperty Tags
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


```powershell
$pathToRunbook = 'C:\Downloads\Stop-MDAzureRMVMByTag.ps1'
$description = 'stop VMs by powerOffTime tag value'
$runbookType = 'PowerShellWorkflow'
$automationAccountName = 'demo-auto-acct'
$resourceGroupName = 'demo-auto-rg'

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
We need to create an Azure Automation variable to hold the subscription name. The script below will create the following:

- Variable named: 'subscriptionName
- Value: demo

Update the $subName variable with your subscription name. Update the automation account name and resource group name variables. This value does not need to be encrypted so it is set to the false value.

```powershell
$varName = 'subscriptionName'
$subName = 'demo'
$encrypted = $false
$automationAccountName = 'demo-auto-acct'
$resourceGroupName = 'demo-auto-rg'

$autoVarParams = @{
  'Name' = $varName;
  'Value' = $subName;
  'Encrypted' = $encrypted;
  'AutomationAccountName' = $automationAccountName;
  'ResourceGroupName' = $resourceGroupName
}

New-AzureRmAutomationVariable @autoVarParams
```

## Create a schedule
The runbook is uploaded and you should now be able to run it and it will shutdown any VMs that have the tag powerOffTime with the value 23:00. Now we want to run this at 23:00 daily with a schedule.
The script below will create a schedule. The timezone is taken from the machine you are running the script on and the start time is set to 23:00 of the day you run the script.
It's run daily so uses the day interval parameter set to 1.

```powershell
$timeZone = (Get-TimeZone).id
$startTime = get-date "23:00"
$dayInterval = 1
$scheduleName = 'Daily 2300 Power Off'
$scheduleDescription = 'Shutdown tagged VMs at 23:00'
$automationAccountName = 'demo-auto-acct'
$resourceGroupName = 'demo-auto-rg'

$scheduleParams = @{
  'Name' = $scheduleName;
  'StartTime' = $startTime;
  'DayInterval' = $dayInterval;
  'Description' = $scheduleDescription;
  'TimeZone' = $timeZone;
  'AutomationAccountName' = $automationAccountName;
  'ResourceGroupName' = $resourceGroupName 
}

New-AzureRmAutomationSchedule @scheduleParams
```

## Link the runbook and schedule
Now we link the created runbook with the created schedule so it will run daily at 23:00.

```powershell
$runbookName = 'Stop-MDAzureRMVMByTag'
$powerOffTime = '23:00'
$automationAccountName = 'demo-auto-acct'
$resourceGroupName = 'demo-auto-rg'
$powerOffTime = '23:00'

$runSchdParams = @{
  RunbookName = $runbookName;
  ScheduleName = $scheduleName;
  AutomationAccountName = $automationAccountName;
  ResourceGroupName = $resourceGroupName;
  'Parameters' = @{'PowerOffTime' = $powerOffTime}
}


Register-AzureRMAutomationScheduledRunbook @runSchdParams
```

[DevTest Labs]: https://docs.microsoft.com/en-gb/azure/devtest-lab/devtest-lab-set-lab-policy
[Tags]: https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-using-tags
[Azure Mobile App]: https://azure.microsoft.com/en-gb/features/azure-portal/mobile-app/
[Azure Subscription]: https://azure.microsoft.com/en-gb/free/
[Azure Automation Account]: https://docs.microsoft.com/en-us/azure/automation/automation-create-standalone-account
[Azure PowerShell]: https://docs.microsoft.com/en-us/powershell/azure/overview