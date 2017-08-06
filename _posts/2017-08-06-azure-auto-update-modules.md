---
title: Azure Automation PowerShell runbook failing to login to AzureRM
author: Matthew Davis
date: 2017-08-06
excerpt: My Azure Automation PowerShell runbook started failing with the error Run Login-AzureRmAccount to login.
categories: 
    - azure
tags:
    - azure
    - active automation
    - powershell runbook
---

While creating a PowerShell runbook to automate the shutting down of tagged Virtual Machines, I ran into the following error:

'''powershell
Get-AzureRmResourceGroup : Run Login-AzureRmAccount to login.
At test:7 char:7
+ 
    + CategoryInfo          : InvalidOperation: (:) [Get-AzureRmResourceGroup], PSInvalidOperationException
    + FullyQualifiedErrorId : InvalidOperation,Microsoft.Azure.Commands.Resources.GetAzureResourceGroupCommand
```

![login error when running PowerShell runbook](/images/azure-auto-module-update/login-error.png)

PowerShell runbooks that had previously run successfully also produced the same error when they ran despite not being updated.

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

[Microsoft Forum]: https://social.msdn.microsoft.com/Forums/en-US/c38e01df-dac8-4095-9658-7b1d981fe8e6/azure-automation-error-run-loginazurermaccount-to-login?forum=azureautomation