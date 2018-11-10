---
title: Authenticate and query the Microsoft Graph with PowerShell
author: Matthew Davis
date: 2018-11-10
excerpt: Use PowerShell to authenticate with and query the Microsoft Graph
categories: 
    - powershell
tags:
    - powershell
    - azure
    - graph
published: false
---

# PowerShell and the Microsoft Graph

This post is in introduction on how to query the Microsoft Graph with PowerShell. 
Intro about the graph

## Azure AD application

```powershell

$AppName = 'GraphDemo2'
$ReplyUrl = 'http://localhost'
$IdUri = 'http://GraphDemo2'
$ClientSecret = '09j32amgi£*madsfmkaf'

$SecureStringPassword = ConvertTo-SecureString -String $ClientSecret -AsPlainText -Force
$app = New-AzureRmADApplication -DisplayName $AppName -ReplyUrls $ReplyUrl -Password $SecureStringPassword -IdentifierUris $IdUri

$app.ApplicationId.Guid

# User.ReadAll for application permissions

```

## PowerShell script

```powershell


$tenantID =  (Get-AzureRmTenant).Id
$ApplicationClientID = $app.ApplicationId.Guid
$graphUrl = 'https://graph.microsoft.com'
$tokenEndpoint = "https://login.microsoftonline.com/$tenantID/oauth2/token"

$tokenHeaders = @{
  "Content-Type" = "application/x-www-form-urlencoded";
}

$tokenBody = @{
  "grant_type"    = "client_credentials";
  "client_id"     = "$ApplicationClientID";
  "client_secret" = "$ClientSecret";
  "resource"      = "$graphUrl";
}

# Post request to get the access token so we can query the Microsoft Graph (valid for 1 hour)
$response = Invoke-RestMethod -Method Post -Uri $tokenEndpoint -Headers $tokenHeaders -Body $tokenBody

# Create the headers to send with the access token obtained from the above post
$getHeaders = @{
  "Content-Type" = "application/json"
  "Authorization" = "Bearer $($response.access_token)"
}

# Create the URL to access all the users and send the query to the URL along with authorization headers
$queryUrl = $graphUrl + "/v1.0/users"
$users = Invoke-RestMethod -Method Get -Uri $queryUrl -Headers $getHeaders

# Output the displaynames
Write-Output $users.value.displayName

```