---
title: Updating TeamCity build step parameters using PowerShell
author: Matthew Davis
date: 2018-11-24
toc: true
excerpt: How to pass variable between build steps in TeamCity using PowerShell
categories:
    - powershell
tags:
    - powershell
    - teamcity
published: true
---
November 2018

# Overview

This post will show how to set and pass variables in PowerShell TeamCity build steps. It uses retrieving Star Wars data from the [swapi Star Wars API] as an example of getting data with PowerShell, setting a variable in a build step and using it in the preceding build steps.

## TeamCity Parameters

Creating parameters is a great way for creating flexible build steps and making them more reusable. You can also create empty parameters and update them to pass values between individual build steps.

There are 3 types of [parameter in TeamCity]:

1. Configuration
2. Environment
3. System

I am using the configuration parameters throughout. I have tested and found these work well and you don't need to prefix them like the environment and system variables.

The parameters are:

1. film - empty - string
2. filmUrl - https://swapi.co/api/films/1/ - string (used as the first call to the Star Wars API)
3. peopleUrl - empty - string
4. pilotName - empty - string
5. starShipName - empty - string
6. starShipUrl - empty - string

![Parameter settings in TeamCity](/images/tc-build-step-params/params.png)

## Setting the parameters

Now the parameters are set up, they can be updated in the scripts or source with [service messages]. The documents show the following example for setting parameters:

```code
##teamcity[setParameter name='ddd' value='fff']
```

However, in PowerShell # is a comment so you must **enclose the statement in double quotes**.

```powershell
"##teamcity[setParameter name='ddd' value='fff']"
```

## Updating the parameters with PowerShell

This demo breaks down querying the Star Wars api into different build steps just to demonstrate the passing of parameters (you could easily do this in one step!). As noted in the TeamCity documents, the modified values of the parameters will be available in the build steps that follow (if you need to use them in the current build step, assign them to a variable in that script before using that variable to set the system message value).

Here are the general build settings for each step. Where the steps differ is in the source code box which I'll set out separately. I'm using PowerShell Core Version 6.1.1 running on Ubuntu but this will work on PowerShell desktop too.

- Runner type: PowerShell
- Step Name: See heading below
- Execute Step: If all previous steps finish successfully
- PowerShell version: 6.1.1
- Platform: x64
- Edition: Core
- Format stderr output: error
- Working Directory: left blank
- Script: Source code
- Script Source: See code under each build step heading below
- Script Execution Mode: Execute .ps1 from external file
- Options: Checked Add -NoProfile argument

### 1: Get Film

Script Source:

```powershell
<# Query the Star Wars API for the film (set in the build configuration parameters)
Set the film parameter and starship url. Output them in a system message for use in later scripts
#>

$film =  Invoke-RestMethod -Method Get -Uri %filmUrl% -UseBasicParsing
Write-Output $film

$title = $film.title
$starShipUrl = $film.starships[4]

"##teamcity[setParameter name='film' value='$title']"
"##teamcity[setParameter name='starShipUrl' value='$starShipUrl']"
```

![Build configuration for step 1. Get Film](/images/tc-build-step-params/1-get-film.png)

### 2: Get StarShip

```powershell
<#
Query the Star Wars API for the starship information using the url set in the previous build step.
Save the starship name and people Url in the service message to use in later build steps 
#>

$starShip = Invoke-RestMethod -Uri "%starShipUrl%" -UseBasicParsing

Write-Output "url is %starShipUrl%"
Write-Output $starShip
$starShipName = $starShip.name
$peopleUrl = $starShip.pilots[1]

"##teamcity[setParameter name='starShipName' value='$starShipName']"
"##teamcity[setParameter name='peopleUrl' value='$peopleUrl']"
```

![Build configuration for step 2. Get StarShip](/images/tc-build-step-params/2-get-starship.png)

### 3: Get Pilot

```powershell
<#
Query the Star Wars API for the people information using the url set in the previous build step.
Save the pilot name in the service message to use in the last build step.
#>

$pilot = Invoke-RestMethod -Method Get -Uri "%peopleUrl%" -UseBasicParsing

Write-Output $pilot
$pilotName = $pilot.name

"##teamcity[setParameter name='pilotName' value='$pilotName']"
```

![Build configuration for step 3. Get Pilot](/images/tc-build-step-params/3-get-pilot.png)

### 4: Putting it together

```powershell
<#
Output all of the parameters set in the previous build steps using a here string.
Output a sentence using the parameters.
#>
$params = @"

Here are the populated parameters that were set in the build configuration as empty:

film = %film%
starShip = %starShipName%
starShipUrl = %starShipUrl%
peopleUrl = %peopleUrl%
pilotName = %pilotName%

"@

Write-Output -InputObject $params

Write-Output "`nIn the film %film% there is a starship called the %starShipName%. %pilotName% was one of the pilots of the %starShipName%.`n"
```

![Build configuration for step 4. Putting it together](/images/tc-build-step-params/4-output.png)

Here is the output from the build log:

```code
[15:32:11]Step 4/4: Putting it together (PowerShell)
[15:32:11][Step 4/4] PowerShell running in non-virtual agent context
[15:32:11][Step 4/4] PowerShell Executable: /opt/microsoft/powershell/6/pwsh
[15:32:11][Step 4/4] Working directory: /opt/TeamCity/buildAgent/work/74e814b1d9185321
[15:32:11][Step 4/4] Wrapper script: /opt/TeamCity/buildAgent/temp/buildTmp/powershell_gen_15430735315018172525048450495645.sh
[15:32:11][Step 4/4] Command:  /opt/microsoft/powershell/6/pwsh -NoProfile -NonInteractive -File /opt/TeamCity/buildAgent/temp/buildTmp/powershell1234487986819058447.ps1
[15:32:12][Step 4/4] 
[15:32:12][Step 4/4] Here are the populated parameters that were set in the build configuration as empty:
[15:32:12][Step 4/4] 
[15:32:12][Step 4/4] film = A New Hope
[15:32:12][Step 4/4] starShip = Millennium Falcon
[15:32:12][Step 4/4] starShipUrl = https://swapi.co/api/starships/10/
[15:32:12][Step 4/4] peopleUrl = https://swapi.co/api/people/14/
[15:32:12][Step 4/4] pilotName = Han Solo
[15:32:12][Step 4/4] 
[15:32:12][Step 4/4] 
[15:32:12][Step 4/4] In the film A New Hope there is a starship called the Millennium Falcon. Han Solo was one of the pilots of the Millennium Falcon.
[15:32:12][Step 4/4] 
[15:32:12][Step 4/4] Process exited with code 0
```

![Output in build log](/images/tc-build-step-params/output.png)

## Summary

Being able to set and pass parameters between build steps in TeamCity is really useful and makes the builds much more flexible. This example showed how to do so using Source Code entered directly in the script block for PowerShell and there is a slightly different technique if it is PowerShell scripts that need to use the input parameters. You must remember to set the parameters in the build configuration before hand. If you create the parameters in the source code script, TeamCity will automatically populate them for you and will display a warning saying they are not set. Go into the parameters section and save them as empty and it will work!

[swapi Star Wars API]: (https://swapi.co/)
[parameter in TeamCity]: (https://confluence.jetbrains.com/display/TCD18/Configuring+Build+Parameters)
[service messages]: (https://confluence.jetbrains.com/display/TCD18/Build+Script+Interaction+with+TeamCity#BuildScriptInteractionwithTeamCity-AddingorChangingaBuildParameter)