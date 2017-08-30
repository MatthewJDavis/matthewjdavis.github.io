---
title: Deploy AWS Cloudformation stack with PowerShell
author: Matthew Davis
date: 2017-08-31
excerpt: Deploy a Cloudformation template for AWS with PowerShell and the AWS module
categories: 
    - aws
tags:
    - aws
    - cloudformation
    - powershell
published: false
---

Following from my last post about testing the cloudformation, I'm going to write up how to use PowerShell to deploy resources to AWS with a Cloudformation stack.
I use yaml templates but the process is the same for templates in JSON.

The AWS PowerShell module is required and can be installed via the [PowerShell gallery] for both the desktop and core versions of PowerShell.
The below commands will find the module on the PSGallery and pipe the result to Install-Module to install (you can change the scope to the current user if you need to - the default will install for all users of the machine)

```powershell
# For Desktop PowerShell:
Find-Module -Name AWSPowerShell | Install-Module

# For Core PowerShell:
Find-Module -Name AWSPowerShell.NetCore | Install-Module
```


[PowerShell galler]: https://www.powershellgallery.com/api/v2/
