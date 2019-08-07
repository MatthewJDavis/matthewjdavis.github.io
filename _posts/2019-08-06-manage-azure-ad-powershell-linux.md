---
title: Use PowerShell core on linux to manage Azure AD
author: Matthew Davis
date: 2019-08-06
toc: false
classes: wide
excerpt: Use PowerShell core running on linux to connect to and manage Azure AD.
categories:
    - powershell
tags:
    - azure
    - core
    - powershell
published: false
---
August 2019

# Overview
I have been mainly using PowerShell core for my day to today work for a while now and have been using a lot recently to interact with Azure and Azure AD so will go through some basics of getting it setup to work and useful commands. I have been using Azure for a few years now, getting started with cloud services and the classic deployment model and migrating over to the Azure Resource Manager way. This has also seen the move away from the AzureRM PowerShell cmdlets to now use the AZ cmdlets (a lot less typing and not too bad to migrate old scripts over too).

Install PowerShell core for your distribution following the guidlines. For ubuntu 18.04 I installed following the method to add the Microsoft repository apt-get and install from there. Link to [Ubuntu 18.04 install instructions].


## Summary


https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-manager-deployment-model
[Ubuntu 18.04 install instructions]: https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804