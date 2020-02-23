---
title: Automate sending guest invites to Azure AD with PowerShell and Azure automation
excerpt:
date: 2020-02-23
toc: false
classes: wide
categories:
- azure
tags:
- azure active directory
- powershell
- azure automation
published: false
---
February 2020

# Overview

Sending invites to guests to access an app - saml

## Set up

This post uses the preview feature of [One Time Passwords for external guest accounts].

https://aad.portal.azure.com

![Azure AD user settings](/images/azuread-guest-invite/user-settings.png)
![Azure AD external guest settings](/images/azuread-guest-invite/external-settings.png)

![Reviewing permissions](/images/azuread-guest-invite/permissions.png)

![Send login code](/images/azuread-guest-invite/send-code.png)

![Login code](/images/azuread-guest-invite/sent-code.png)



## The script

### Tests

## Azure automation runbook

<script src="https://gist.github.com/MatthewJDavis/7b6b5be967628d7a97d4c4dd239bd732.js"></script>

## Azure schedule

## Summary

[One Time Passwords for external guest accounts]: https://docs.microsoft.com/en-us/azure/active-directory/b2b/one-time-passcode