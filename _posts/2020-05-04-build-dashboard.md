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
published: true
---
May 2020

# Overview

I recently watched the [PowerShell DevOps Playbook] course on Pluralsight by [Adam Bertram] and was inspired to look at the [Universal Dashboard] PowerShell module again after the section in the course that used it to create a dashboard to display information for [AppVeyor] builds.

Universal Dashboard is a web framework that allows you to easily create web front ends with PowerShell that can be used as an interface for PowerShell code. I decided to make a similar dashboard as shown the Pluralsight course, using [Azure DevOps]. I have a few different project in Azure DevOps with various builds that I can get information about from the [Azure DevOps REST API] and display it in the dashboard - giving a nice overview of all project builds in one easy to use dashboard.

This post will show how to create the dashboard in the gif below by querying the Azure DevOps API and displaying the data in a Universal Dashboard using various components to give an easy way to see builds that have run in each project for the organisation.

## Outcome

![Dashboard overview](/images/build-dashboard/dashboard.gif)

Complete code

<script src="https://gist.github.com/MatthewJDavis/58a866c1b36a3b729675569bb7d6f42c.js"></script>

## Set Up

- Tested on Windows and Linux.
- PowerShell core version 7 and Windows PowerShell version 5.1.
- UniversalDashboard Version: 2.9.0

There are various different Azure DevOps [Authentication] methods. To keep things simple I've created a [Personal Access Token] for my user with [Read Access] to builds in my organisation.

### Azure DevOps Personal Access Token (Pat)

1. Login to Azure DevOps
2. Click on your profile picture
3. Select Personal Access Tokens
![Dashboard overview](/images/build-dashboard/pat.png)
4. Click 'New'
5. Enter the name, choose organisation (or all organisations) select expiry date
6. Under 'Scope' scroll to 'Builds' and select 'Read'
![Dashboard overview](/images/build-dashboard/create-new.png)
7. Copy the token somewhere safe and secure such as a [password manager] and close.

### Install Universal Dashboard

Universal Dashboard can be installed from the PowerShell gallery with:

``` Install-Module -Name UniversalDashboard.Community -RequiredVersion 2.9.0 ```

 This installs the free community edition, there is also a very reasonably priced [premium licence] which adds more great features including authentication for your dashboards.

## Running the dashboard

### Add the personal access token to the environment

The personal access token needs to be included in the headers for authentication and authorisation. To keep the token out of the code and PowerShell history, it is input via the ``` Read-Host ``` Cmdlet to an environment variable. An alternative would be to assign the variable value directly but this would be available in the history which is not desirable. It's still better than having it in plain text in the script and this could be replaced with the up and coming [PowerShell secrets module] or by getting the value from a secure secrets management platform such as [Hashicorp Vault] or [Azure Key Vault].

Save the code to a file locally and from that directory [dot source] it so the functions are available in your PowerShell session.

```powershell
$env:pat = Read-Host
'yourPersonalAccessToken'

# download full code from github gist, dot source the file and run the function to start the dashboard
$uri = 'https://gist.githubusercontent.com/MatthewJDavis/58a866c1b36a3b729675569bb7d6f42c/raw/f6fe18127c3f490e3cfca6d0498e25ec20bef45d/dashboard.ps1'

Invoke-WebRequest -uri $uri -OutFile .\dashboard.ps1
. .\dashboard.ps1

Start-BuildDashboard -OrgName 'yourOrgName'
```

![Starting up the dashboard](/images/build-dashboard/start-dashboard.png)

The dashboard should now be running on [http://localhost:10002/] (you can specify a different port via the Port parameter).

![Dashboard overview](/images/build-dashboard/running.png)

## Code and logic run through

Variables are set with the values to query the Azure DevOps API.

The PAT token is convert to base64 and included in the headers. The variables are then made available to the Universal Dashboard endpoints via the ``` New-UDEndpointInitialization ``` Cmdlet.

```powershell
$PAToken = $env:PAT
$uri = "https://dev.azure.com/$OrgName"
$Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($PAToken)")) }
$DashboardName = 'AzureDevOpsBuildDashboard'
$Init = New-UDEndpointInitialization -Variable @('OrgName', 'PAToken', 'uri', 'Headers')
$BuildRefresh = New-UDEndpointSchedule -Every 5 -Minute
```

### Error handling Project data

If you query the Azure DevOps API and something goes wrong, depending on what went wrong you can get a different result or object returned. The general exception thrown contains a lot of text (an entire web page is returned) so the below section uses a try catch block to catch common errors I encountered. There is a final catch that is a catch all that will display the whole text but this is better than the dashboard starting up and having no projects populated.

```powershell
try {
    $projectList = Invoke-RestMethod -Uri $projectUri -Method Get -Headers $Headers
} catch [System.Net.WebException] {
    if ($_.exception -like "*could not be resolved*") {
        throw "Check Network connection. Error: $($_.exception.message)"
    } elseif ($_.exception.response.statuscode -eq 'NotFound') {
        throw "Check OrgName $orgName is correct. Status code received: $($_.exception.response.statuscode)"
    } elseif ($_.exception.response.statuscode -eq 'Unauthorized') {
        throw "Check OrgName $orgName is correct,  Personal Access Token is correct, has read permissions to builds and has not expired. Status code received: $($_.exception.response.statuscode)"
    } else {
        throw $_
    }
} catch {
    throw "$($_.Exception)"
}
```

Even after the try catch, things can still go wrong so the final check is that we receive a pscustomobject and that there is at least one project.

Note: The project data is only populated when the dashboard is started or restarted. The reason being the New-UDSelect element list does not update unless the dashboard is stopped then started again. That is why an endpoint or $cache variable is not used for it.

```powershell
if ($projectList.gettype().Name -ne 'PSCustomObject') {
    throw "Did not get correct response from Azure DevOps API. Require 'PSCustomObject' but got $($projectList.gettype().Name) type. Check OrgName, Personal Access Token value, permissions and expiration"
} elseif ($projectList.count -lt 1) {
    throw "No projects found in org $OrgName"
} else {
    $projectListSorted = $projectList.value | Sort-Object -Property name
}
```

Wrong PAT added
![incorrect response](/images/build-dashboard/incorrect-response.png)

Organisation doesn't exist
![project does not exist](/images/build-dashboard/not-exist.png)

Organisation doesn't exist (new [PowerShell 7 error handling] concise view)
![New error handling in PS7](/images/build-dashboard/error-handling-ps-7.png)

### Caching Build data

Constantly calling the API can slow the application down so a Scheduled endpoint is used along with a ``` $cache ``` variable to update the build data every 5 minutes.

Once there is  a list of projects, this list is iterated over to create a list of builds for each project, saving the properties to display in a [pscustomobject] and adding to the build list.

The build number and commit properties are added to the object as links which will take the user to the respective pages if clicked on. The commit id is shortened to the first 6 characters.

The final part of the $BuildDataRefresh endpoint is to sync the grid. This will update the grid with the new values in the build list (if there are any) when the schedule is run and the cache variable is updated.

Click on the commit id takes you to the code.
![Click on commit id](/images/build-dashboard/click-commit.gif)

```powershell
$buildDataRefresh = New-UDEndpoint -Schedule $BuildRefresh -Endpoint {
    $Cache:dataList = [System.Collections.Generic.List[pscustomobject]]::new()
    foreach ($project in $projectListSorted) {
        $BuildURI = "$uri/$($project.id)/_apis/build/builds?api-version=5.1"
        $buildList = Invoke-RestMethod -Uri $BuildURI -Headers $Headers
        foreach ($build in $buildList.value) {
            $Cache:dataList.Add(
                [pscustomobject]@{
                    'ProjectId'   = $project.id
                    'BuildNumber' = (New-UDLink -Text $($build.buildNumber) -Url $($build._links.Web.href))
                    'StartTime'   = $build.StartTime
                    'FinishTime'  = $build.FinishTime
                    'Result'      = $build.result
                    'Commit'      = (New-UDLink -Text $($build.sourceVersion.Substring(0, 6)) -Url $($build._links.sourceVersionDisplayUri.href))
                }
            )
        }
    }
    Sync-UDElement -Id 'grid'
}
```

### Creating the Dashboard

The first element created is the drop down select element that is populated with the project name and project id. The project name is displayed and the id is used to look up values in the build list to display the relevant build data.
The onchange property sets a session variable with the project id (this is used to display data in the grid and card elements) and syncs the cards and grid elements.

```powershell
$projectSelect = New-UDSelect -Label "Project" -Id 'projectSelect' -Option {
    $SelectionList = [System.Collections.Generic.List[pscustomobject]]::new()
    $default = [pscustomobject]@{
        'Name'  = 'Select Project'
        'Value' = 'default'
    }
    $SelectionList.Add($default)
    foreach ($project in $projectListSorted) {
        $SelectionList.Add(
            [pscustomobject]@{
                'Name'  = $project.name
                'Value' = "$($project.id)"
            }
        )
    }
    foreach ($item in $SelectionList) {
        New-UDSelectOption -Name $item.Name -Value $($item.Value)
    }
} -OnChange {
    $Session:Projectid = $eventData
    Sync-UDElement -Id 'grid'
    Sync-UDElement -id 'Div1'
} # end UDSelect
```

The 3 display cards are created to show the status of the build (last build success, failure), number of builds and success rate in percent.
A div is used so that the cards can be updated via the select endpoint change with the corresponding build data for the project selected. The background of the card also changes colour depending on the status of the last build.

Build with last status is failed
![failed build status with red background](/images/build-dashboard/failed.png)

Build with last status of partial success
![partial success build with blue background](/images/build-dashboard/partial-success.png)

Build that was failing but latest now passes
![partial success build with blue background](/images/build-dashboard/now-passing.png)

```powershell
$card = New-UDElement -Tag div -Id "Div1" -Endpoint {
    if ($null -eq $Session:Projectid) {
        # No project id yet so nothing to display - prevent divide by 0 errors for percentage
        $latestResult = 'none'
        $successRate = '0'
    }
    $latestResult = ($Cache:dataList | Where-Object -Property 'ProjectID' -EQ $Session:Projectid | Select-Object -property 'Result' -First 1).Result
    $resultList = ($Cache:dataList | Where-Object -Property 'ProjectID' -EQ $Session:Projectid | Select-Object -property 'Result').Result
    $total = $resultList.count
    if ($total -gt 0) {
        $success = ($resultList | Group-Object | Where-Object -Property Name -eq 'succeeded').Count # get how many builds were successful
        if (-not $null -eq $Session:Projectid) {
            $successRate = "$([math]::round($success / $total * 100, 2))" + '%' # calculate percentage of sucessful build to 2 decimal places
        }
    } else {
        $successRate = '0%'
    }
    New-UDLayout -Columns 3 -Content {
        $backgroundColour = switch ($latestResult) {
            'succeeded' { 'green' }
            'partiallySucceeded' { 'blue' }
            'failed' { 'red' }
            Default { 'white' }
        }
        New-UDCard -Id 'statusCard' -Title 'Current Status' -BackgroundColor $backgroundColour -FontColor 'White' -Text $latestResult
        New-UDCard -Id 'buildCount' -Title 'Build Count' -BackgroundColor $backgroundColour -FontColor 'White' -Text ($Cache:dataList | Where-Object -Property 'ProjectID' -EQ $Session:Projectid | Measure-Object ).Count 
        New-UDCard -Id 'successRate' -Title 'Success Rate' -BackgroundColor $backgroundColour -FontColor 'White' -Text $successRate
    }
} #end UDElement
```

The grid element is created with the cached build data. Custom headers are used for the display and selected properties are passed through to omit the projectid property.

```powershell
$grid = New-UDGrid -Id 'grid' -Title "Build Information" -Headers @('Build Number', 'Result', 'Commit', 'Start Time', 'Finish Time') -Properties @('BuildNumber', 'Result', 'Commit', 'StartTime', 'FinishTime') -Endpoint {
    $Cache:dataList | Where-Object -Property 'Projectid' -EQ $Session:Projectid | Out-UDGridData
}
```

Finally the dashboard is created and passed to the start command.

```powershell
$dashboard = New-UDDashboard -Title "Azure DevOps $OrgName" -Content { $projectSelect, $card, $grid } -EndpointInitialization $Init
Start-UDDashboard -Dashboard $dashboard -Name $DashboardName -Endpoint @($buildDataRefresh) -Port $Port
```

### Updating Project

As previously mentioned, the ``` Select-UDElement ``` element does not refresh. To update project list, the dashboard should be stopped then started with:

```powershell
Stop-UniversalDashboard -Name 'AzureDevOpsBuildDashboard'

Start-BuildDashbaord
```

## Summary

Universal Dashboard is a great module and can quickly be used as a frontend for PowerShell scripts to display useful data. This example could be adapted to interact and make changes to Azure DevOps projects (with an updated PAT with more scope permissions) and could also integrate with other endpoints of the Azure DevOps API.
I enjoyed working on this project over the last couple of weeks and implemented it into an Azure DevOps project. Full code can be found in the [repo on Github].

[PowerShell DevOps Playbook]: https://app.pluralsight.com/library/courses/powershell-devops-playbook/table-of-contents
[Adam Bertram]: https://app.pluralsight.com/profile/author/adam-bertram
[AppVeyor]: https://www.appveyor.com/
[Universal Dashboard]: https://universaldashboard.io/
[Authentication]: https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/authentication-guidance?view=azure-devops
[Personal Access Token]: https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page
[Azure DevOps]: https://dev.azure.com/
[Azure DevOps REST API]: https://docs.microsoft.com/en-us/rest/api/azure/devops/?view=azure-devops-rest-5.1
[PowerShell secrets module]: https://devblogs.microsoft.com/powershell/secret-management-preview-2-release/
[Hashicorp Vault]: https://www.vaultproject.io/
[Azure Key Vault]: https://azure.microsoft.com/en-us/services/key-vault/
[PowerShell 7 error handling]: https://www.petri.com/how-error-handling-works-in-powershell-7
[Repo on Github]: https://github.com/MatthewJDavis/devops-dashboard
[pscustomobject]: https://powershellexplained.com/2016-10-28-powershell-everything-you-wanted-to-know-about-pscustomobject/
[Read Access]: https://docs.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/oauth?view=azure-devops#scopes
[password manager]: https://en.wikipedia.org/wiki/Password_manager
[premium licence]: https://ironmansoftware.com/powershell-universal-dashboard/
[dot source]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scripts?view=powershell-7#script-scope-and-dot-sourcing
[http://localhost:10002/]: http://localhost:10002/