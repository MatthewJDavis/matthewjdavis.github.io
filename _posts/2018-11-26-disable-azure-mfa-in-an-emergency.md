---
title: Disable Azure MFA in an Emergency
author: Matthew Davis
date: 2018-11-26
excerpt: 
categories:
    - powershell
tags:
    - powershell
    - azure
    - azure powershell
    - azure automation
    - mfa
published: false
---

# Overview

The recent MFA outage for Azure that meant those who had setup MFA on their accounts could not authenticate for the entire day got me thinking of a solution around this if this were to happen again. We use on prem MFA so were not affected but I know how frustrating it is when something like this blocks your ability to be productive.

Here's an "emergency break glass" solution that can be implemented if this were to happen again, a way to disable Azure MFA for Admin accounts (or it could be one or two accounts) then the decision could be made to disable MFA for other users so they could login to Azure and Office 365.


## Summary