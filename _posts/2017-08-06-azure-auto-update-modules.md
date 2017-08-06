---
title: Azure Automation PowerShell runbook failing to login to AzureRM
author: Matthew Davis
date: 2017-08-06
excerpt: My Azure Automation PowerShell runbook started failing with the error Run Login-AzureRmAccount to login.
categories: 
    - azure
tags:
    - azure
    - azure automation
    - powershell runbook
---

While creating a PowerShell runbook to automate the shutting down of tagged Virtual Machines, I ran into the following error:

```powershell
Get-AzureRmResourceGroup : Run Login-AzureRmAccount to login.
At test:7 char:7
+ 
    + CategoryInfo          : InvalidOperation: (:) [Get-AzureRmResourceGroup], PSInvalidOperationException
    + FullyQualifiedErrorId : InvalidOperation,Microsoft.Azure.Commands.Resources.GetAzureResourceGroupCommand
```

![login error when running PowerShell runbook](/images/azure-auto-module-update/login-error.png)

PowerShell runbooks that had completed successfully also produced the same error despite not being updated.

After checking the certificate being used by the service principal hadn't expired and the service principal looked OK, I tested the tutorial runbook which also failed.

I came across this post in a [Microsoft Forum] which fixed the problem:

**The modules in my Automation account were not updated**. 

![out of date modules](/images/azure-auto-module-update/azure-module-before-update.png)

To update the modules, go to the automation account and select **Modules** from the left hand menu.

![azure automation menu](/images/azure-auto-module-update/azure-auto-module.png)

The **Update Azure Modules** button is along the top. This action takes a couple of minutes to complete.

![update azure modules button](/images/azure-auto-module-update/update-azure-modules.png)

I ran the update and now my PowerShell runbooks authenticated to AzureRM ok again and were running as expected.

![updated modules](/images/azure-auto-module-update/azure-auto-module-update.png)

## Re-link schedules

To use the latest modules, the runbooks have to be un-linked and re-linked to a schedule:

> Azure modules have been updated; for runbooks that use these modules and have a linked schedule you will need to unlink and re-link the schedule so that the updated modules will be used by the runbook.

Runbooks can be un-linked and linked via Azure PowerShell, below is an example.

### Save the runbook schedule as a variable

```powershell
$schedule = Get-AzureRmAutomationScheduledRunbook -AutomationAccountName autoAcctName -ResourceGroupName rgName -name rbName
```

**Note** The parameters are not returned with this command (they are visible in the portal), the [issue] is raised on Github.

### Unregister the scheduled runbook

```powershell
 Unregister-AzureRmAutomationScheduledRunbook -JobScheduleId $schedule.JobScheduleId -AutomationAccountName $schedule.AutomationAccountName -ResourceGroupName $schedule.ResourceGroupName -Force
```

### Register the runbook

**Create the parameters as required**

```powershell
$params = @{'powerOffTime' = '23:00'}
```

```powershell
Register-AzureRmAutomationScheduledRunbook -RunbookName $schedule.RunbookName -ScheduleName $schedule.ScheduleName -ResourceGroupName $schedule.ResourceGroupName -AutomationAccountName $schedule.AutomationAccountName -Parameters $params
```

The runbook and schedule is now linked again and the updated modules should now work. Hopefully the Get-AzureRmAutomationScheduledRunbook issue will be resolved so the updating of the modules and un-linking, linking of the runbooks can be automated.


[Microsoft Forum]: https://social.msdn.microsoft.com/Forums/en-US/c38e01df-dac8-4095-9658-7b1d981fe8e6/azure-automation-error-run-loginazurermaccount-to-login?forum=azureautomation

[issue]: https://github.com/Azure/azure-powershell/issues/2180