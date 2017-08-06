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




 
## Tagging the VM

The script will look for Virtual Machines that have the powerOffTime tag set to a specific value (this example uses 23:00).
You can tag the VMs via the portal or with PowerShell as shown below.

### Save the tags in a hash table
$tags = @{"Env"="Demo";"powerOffTime"='23:00';'createdBy'='Matt';'project'='Domain Services';'role'='Domain Serivces Management VM'}

### Save the VM in a variable
$vm = get-azurermvm -Name ds-manag-vm -ResourceGroupName DOMAIN-SERVICES-RG

### Pass the VM object which is stored in the variable to the Update-AzureRmVM command along with the tags
$vm | update-azurermvm -Tags $tags


## Creating the subscription variable 
New-AzureRmAutomationVariable -Name 'subscriptionName' -Value 'demo' -Encrypted $false -AutomationAccountName matt-auto-acct -ResourceGroupName automation

