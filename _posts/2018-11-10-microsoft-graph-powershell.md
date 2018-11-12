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

This post is in introduction on how to query the Microsoft Graph with PowerShell. It shows how to set up an Azure AD application and set permissions, get an autorisation token and query the graph for user data. I've been interested in learning about the Microsoft graph for sometime put was put off by the lack of documentation on how to do it with PowerShell. Recently at work, it has become apparent that we could make good use of some of the data that is exposed by the graph so thought I'd take a second look. 

I found the [Identity, Application, and Network Services on Microsoft Azure] course on Pluralsight and watched the Microsoft Graph sections of the 'Integrating app with Azure AD' module which was really helpful in understanding how to authenticate to the graph to get an access token and was able to test using Postman and then convert that to PowerShell.

## Azure AD application

First we need to create an Azure AD application. This will be used by the client (PowerShell) to authenticate with and get an access token. It is also used to set the relevant permissions to the directory.

Here's the PowerShell to create an application. The client secret will be used by PowerShell to get an access token that can then be used to query the graph.

```powershell

$AppName = 'GraphDemo'
$ReplyUrl = 'http://localhost'
$IdUri = 'http://GraphDemo'
$ClientSecret = '09j32amgio*madsfmkaf'

$SecureStringPassword = ConvertTo-SecureString -String $ClientSecret -AsPlainText -Force
$app = New-AzureRmADApplication -DisplayName $AppName -ReplyUrls $ReplyUrl -Password $SecureStringPassword -IdentifierUris $IdUri

$app.ApplicationId.Guid

# User.ReadAll for application permissions

```

After the application is created, the permissions need to be granted. I could not find a way to do this via PowerShell so had to resort going back to the portal and doing manually. You need to be an administrator to set application level permissions for the whole tenant or request that an administrator does it for you.

Login to the main Azure portal and navigate to Azure Active Directory or go directly to the Azure AD portal: https://aad.portal.azure.com/

1. Click Azure Active Directory
2. Click App registrations (preview)
3. Click All Applications
4. Search (if needed) and then click on the created applications

![Azure AD application registration](/images/ms-graph-powershell/app-reg.png)

1. Click API permissions
2. Click Add a permission

![API permission blade](/images/ms-graph-powershell/api-permissions.png)

1. Click on Microsoft Graph

![API Microsoft Graph tile](/images/ms-graph-powershell/api-ms-graph.png)

1. Click on Application Permissions

![API application permissions tile](/images/ms-graph-powershell/app-permissions.png)

1. Select User
2. Check User.Read.All 
3. Click Add Permission

![API User Read All permissions button](/images/ms-graph-powershell/user-read-all.png)

See [permissions reference]

1. Click Grant admin consent for...
2. Click Yes

![API admin consent button](/images/ms-graph-powershell/admin-consent.png)

Now you wil see that admin consent has been granted:

![API admin consent granted](/images/ms-graph-powershell/admin-consent-granted.png)

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

[Identity, Application, and Network Services on Microsoft Azure]: https://www.pluralsight.com/courses/microsoft-azure-identity-application-network-services

[permissions reference]: https://developer.microsoft.com/en-us/graph/docs/concepts/permissions_reference