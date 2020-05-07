---
title: Create an Azure DevOps build dashboard
excerpt: 
date: 2020-05-04
toc: false
classes: wide
categories:
- powershell
tags:
- azure devops
- universal dashboard
published: false
---
May 2020

# Overview

I have recently watched the [PowerShell DevOps Playbook] course on Pluralsight and was inspired to play around with the Universal Dashboard PowerShell module again after the section in the course that used it to create a dashboard for [AppVeyor] builds.

[Universal Dashboard] is a webframework that allows you to easily create web frontends with PowerShell that can then be used with existing scripts and code. After going through the docs to refresh myself with it (I had looked at it a while ago now), I decided it would be a good project to make a similar dashboard as in the Pluralsight course, using Azure DevOps instead. I have a few builds in Azure DevOps and wanted to see how it has come along and using this as a project would be a good refresher on Universal Dashboard and Azure DevOps.

## Outcome

I'll start with a screen shot of what the final out come is and the code.

![Dashboard overview](/images/build-dashboard/dashboard1.png)

code

## Set Up

This was tested on Windows and Linux using PowerShell version 7.

Azure DevOps Authentication and Authorisation methods can be viewed here. To keep things simple, I've created a Personal Access Token for my user with Read Access to builds. The personal access token is then created as an environment variable and read from that so it is never entered directly into any code.

### Azure DevOps Personal Access Token (Pat)

1. Login to Azure DevOps
2. Click on your profile picture
3. Select Personal Access Tokens
![Dashboard overview](/images/build-dashboard/pat.png)
4. Click 'New'
5. Enter the name, choose organisation (or all organisations) select expiry date
6. Under 'Scope' scroll to 'Builds' and select 'Read'
![Dashboard overview](/images/build-dashboard/create-new.png)
7. Copy the token somewhere safe such as a password manager and close.

### Install Universal Dashboard

This can simply be installed from the PowerShell gallery with ``` Install-Module UniversalDashboard -AcceptLicense ```. This installs the free community edition, there is also a very reasonably priced enterprise addition which adds more great features.

## Running the dashboard

Add the personal access token to the environment. 
Note: The command will be in the PowerShell history and also the PSReadLine history so care should be taken to remove this if needs be. It's still better than having it in plain text in the script and this could be replaced with the up and coming PowerShell secrets module or by getting the value from a secure secrets management platform such as Hashicorp Vault or Azure Key Vault.

```powershell
$env:pat = 'yourPersonalAccessToken'
```

Paste the full code from above into a PowerShell session.

Run:

```powershell
Start-BuildDashboard
```

The dashboard should now be running on 'http://localhost:10002/'

![Dashboard overview](/images/build-dashboard/running.png)

## Code and logic run through




[PowerShell DevOps Playbook]: https://app.pluralsight.com/library/courses/powershell-devops-playbook/table-of-contents
[AppVeyor]: https://www.appveyor.com/
[Universal Dashboard]: https://universaldashboard.io/
[Microsoft documentation]: https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page