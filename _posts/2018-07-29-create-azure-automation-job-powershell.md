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

I've just completed a task to restart a service when Splunk detects that is not running on a Windows server. We don't have any on premises Splunk servers, so to restart the service, I created an Azure Automation PowerShell run book that runs on a hybrid worker which is connected to our internal network
