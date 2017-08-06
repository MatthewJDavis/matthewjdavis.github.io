---
title: Azure Automation PowerShell runbook failing to login to AzureRM
author: Matthew Davis
date: 2017-08-06
excerpt: AzureRM Automation PowerShell runbook started failing, Run Login-AzureRmAccount to login.
categories: 
    - azure
tags:
    - azure
    - active automation
    - powershell runbook
published: false
---

While creating a PowerShell runbook to automating the shutting down of tagged Virtual Machines, I ran into the following error:

{% highlight powershell %}
Get-AzureRmResourceGroup : Run Login-AzureRmAccount to login.
At test:7 char:7
+ 
    + CategoryInfo          : InvalidOperation: (:) [Get-AzureRmResourceGroup], PSInvalidOperationException
    + FullyQualifiedErrorId : InvalidOperation,Microsoft.Azure.Commands.Resources.GetAzureResourceGroupCommand

{% endhighlight %}

PowerShell runbooks that had previously run successfully also produced the same error when they ran despite not being updated.

After checking that the certificate the service principal was using hadn't expired and the service principal looked OK and testing with the demo runbook which also failed, I came across this post in a [Microsoft Forum] which fixed the problem:

**The modules in my Automation account were not updated**. 

I ran the update and now my PowerShell runbooks authenticated to AzureRM ok again and were running as expected.



[Microsoft Forum]: https://social.msdn.microsoft.com/Forums/en-US/c38e01df-dac8-4095-9658-7b1d981fe8e6/azure-automation-error-run-loginazurermaccount-to-login?forum=azureautomation