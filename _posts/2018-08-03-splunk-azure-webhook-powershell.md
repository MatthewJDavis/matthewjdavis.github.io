---
title: Use Splunk data sent in a webhook to start a Windows Service with PowerShell and Azure Automation
author: Matthew Davis
date: 2018-08-03
excerpt: Splunk can be set to alert when a critical service is not running on a server. Data can be sent via a JSON payload to a webhook set up on Azure automation.
categories: 
    - azure
tags:
    - azure
    - azure automation
    - splunk
    - powershell runbook
published: false
---
August 2018

# Overview

I've just completed a task to restart a service when Splunk detects that is not running on a Windows server. We don't have any on premises Splunk servers, so to restart the service, I created an Azure Automation PowerShell run book that runs on a hybrid worker which is connected to our internal network
