---
title: Create an Azure DevOps build dashboard
excerpt: Use the Universal Dashboard PowerShell module to display build data from Azure DevOps.
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

I have recently watched the [PowerShell DevOps Playbook] course on Pluralsight and was inspired to look at the [Universal Dashboard] PowerShell module again after the section in the course that used it to create a dashboard for [AppVeyor] builds.

Universal Dashboard is a webframework that allows you to easily create web frontends with PowerShell that can be used as a web interface for PowerShell code input and output. I decided to make a similar dashboard as shown the Pluralsight course, using [Azure DevOps] instead. I have a few different project in Azure DevOps with various builds that I can get information about from the [Azure DevOps REST API] and display it in the dashboard.

## Outcome

I'll start with a screen shot of what the end result and the full code and break it down afterwards.

![Dashboard overview](/images/build-dashboard/dashboard1.png)

<script src="https://gist.github.com/MatthewJDavis/58a866c1b36a3b729675569bb7d6f42c.js"></script>

## Set Up

This was tested on Windows and Linux using PowerShell version 7 and on Windows using Windows PowerShell version 5.1.
UniversalDashboard Version: 2.9.0

Azure DevOps [Authentication] methods can be viewed here. To keep things simple, I've created a [Personal Access Token] for my user with Read Access to builds.

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

Universal Dashboard can be installed from the PowerShell gallery with:

``` Install-Module -Name UniversalDashboard.Community -RequiredVersion 2.9.0 ```.

 This installs the free community edition, there is also a very reasonably priced premium licence which adds more great features including authentication.

## Running the dashboard

### Add the personal access token to the environment

The personal access token is needed to include in the headers for authentication and authorisation. To keep the token out of the code and PowerShell history, it is input via the ``` Read-Host ``` Cmdlet to an environment variable. An alternative would be to assign the variable value directly but this would be available in the history which is not be desirable. It's still better than having it in plain text in the script and this could be replaced with the up and coming [PowerShell secrets module] or by getting the value from a secure secrets management platform such as [Hashicorp Vault] or [Azure Key Vault].

Save the code to a file locally and from that directory dot source it so the functions are available in your PowerShell session.

```powershell
$env:pat = Read-Host
'yourPersonalAccessToken'

$uri = 'https://gist.githubusercontent.com/MatthewJDavis/58a866c1b36a3b729675569bb7d6f42c/raw/4f018146ed16307973b8af2a0998f8fbe66041e2/dashboard.ps1'

Invoke-WebRequest -uri $uri -OutFile .\dashboard.ps1
. .\dashboard.ps1

Start-BuildDashboard -OrgName 'yourOrgName'
```

The dashboard should now be running on 'http://localhost:10002/' (you can specify a different port via the Port parameter).

![Dashboard overview](/images/build-dashboard/running.png)

## Code and logic run through

First thing needed is setting up the variables to be able to query the Azure DevOps api.
The PAT token is convert to base64 and included in the headers. The variables are then made available to the Universal Dashboard endpoints via the ``` New-UDEndpointInitialization ``` Cmdlet.

### Project and Caching Build data

Constantly calling the api is can slow the application down so a Scheduled endpoint is used along with a ```$cache ``` variable to update the build data every 5 minutes.

The project data is only populated when the dashboard is started or restarted. The reason being is that the New-UDSelect element list does not update with out a restart.

Once I have a list of projects, this list is iterated over to create a list of builds for each project, saving the properties I want to display in a pscustomobject and adding to a list.

The final part of the $BuildDataRefresh endpoint is to sync the grid. This will update the grid with the new values in the build list (if there are any) when the schedule is run and the cache variable is updated.

### Creating the Dashboard

The first element created is the drop down select element that is populated with the project name and project id. The project name is displayed and the id is used to look up values in the build list to get the relevant build data.
The onchange property sets a session variable with the project id (this is used to display data in the grid and card elements) and syncs the cards and grid elements.

The 3 display cards are created to show the status of the build (last build success, failure), number of builds and success rate in percent.
A div is used so that the cards can be updated via the select endpoint change with the corresponding build data for the project selected. The cards background also changes colour depending on the status of the build.

![failed build status with red background](/images/build-dashboard/failed.png)

![partial success build with blue background](/images/build-dashboard/partial-success.png)

Finally the dashboard is created and passed to the start command.

### Updating Project

At present the ``` Select-UDElement ``` element does not refresh.  To update project list, the dashboard should be stopped then started with ``` Stop-UniversalDashboard 'AzureDevOpsBuildDashboard' ```. It can then be restarted with ``` Start-BuildDashbaord ```.

## Summary

Universal Dashboard is a great module and can quickly be used as a frontend for PowerShell scripts to display useful data. This example could be adapted to interact and make changes to Azure DevOps projects (with an updated PAT with more scope permissions) and could also integrate with other endpoints of the Azure DevOps API.

[PowerShell DevOps Playbook]: https://app.pluralsight.com/library/courses/powershell-devops-playbook/table-of-contents
[AppVeyor]: https://www.appveyor.com/
[Universal Dashboard]: https://universaldashboard.io/
[Authentication]: https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops
[Personal Access Token]: https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page
[Azure DevOps]: https://dev.azure.com/
[Azure DevOps REST API]: https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-5.1
[PowerShell secrets module]: https://devblogs.microsoft.com/powershell/secret-management-preview-2-release/
[Hashicorp Vault]: https://www.vaultproject.io/
[Azure Key Vault]: https://azure.microsoft.com/en-us/services/key-vault/