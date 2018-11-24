---
title: Updating TeamCity build step parameters using PowerShell
author: Matthew Davis
date: 2018-11-20
excerpt: 
categories:
    - powershell
tags:
    - powershell
    - teamcity
published: false
---

# Overview

This post will show how to set and pass variable in PowerShell TeamCity build steps. It uses retrieving Star Wars data from the [swapi Star Wars API] as an example of getting data with PowerShell, setting a variable and using it in the preceding build steps.

## TeamCity Parameters

Creating parameters is a great way for creating flexible build steps and making them more reusable. You can also create empty parameters and update them to pass values between individual build steps.

There are 3 types of parameter in TeamCity

1. Configuration
2. Environment
3. System

For this I am using the configuration parameters. I have tested and found these work well and you don't need to prefix them like the environment and system variables.

The parameters are:

1. film - empty - string
2. filmUrl - https://swapi.co/api/films/1/ - string
3. peopleUrl - empty - string
4. pilotName - empty - string
5. starShipName - empty - string
6. starShipUrl - empty - string

![Parameter settings in TeamCity](/images/tc-build-step-params/params.png)

[More info on TeamCity build parameters]

## Setting the parameters

Now the parameters are set up, they can be updated in the scripts or source with a [service messages]. The documents show the following for setting parameters:

```code
##teamcity[setParameter name='ddd' value='fff']
```

However, in PowerShell # is a comment so you must enclose the statement in double quotes.

```powershell
"##teamcity[setParameter name='ddd' value='fff']"
```

## Updating the parameters with PowerShell

## Summary

[swapi Star Wars API]: (https://swapi.co/)
[More info on TeamCity build parameters]: (https://confluence.jetbrains.com/display/TCD18/Configuring+Build+Parameters
)
[service message]: (
https://confluence.jetbrains.com/display/TCD18/Build+Script+Interaction+with+TeamCity#BuildScriptInteractionwithTeamCity-AddingorChangingaBuildParameter)