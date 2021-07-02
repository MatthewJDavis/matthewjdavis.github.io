---
title: Get list of AzureAD in pri
excerpt: Use the Microsoft Graph PowerShell module to get a list of users consuming a particular licence.
date: 2021-03-17
toc: false
classes: wide
categories:
- powershell
tags:
- azuread
- powershell
- graph
published: false
---
July 2021

# Overview

I wanted to get a list of users assigned to the Global Administrators and User Administrator roles for an audit report and decided to use the PowerShell Microsoft Graph module to achieve this so the report could be automated to run periodically.

## Getting the information

Installing the graph module is straight forward. Finding the correct command to do what I wanted wasn't! The Graph module is a 'meta module' and installs numerous other modules that contain the Cmdlets so that took a bit of getting used to. I also found the graph module Cmdlets lacking in help details I'm used to in other PowerShell modules.

My first action was to search the graph module for cmdlets that looked like they would return the information I wanted.

First I had a look at all the modules, then narrowed down the search to just the Identity ones.

```powershell
Get-Module Microsoft.Graph* -ListAvailable
Get-Command -Module Microsoft.Graph.Identity* -Verb Get -Noun "*role*"
```

I found a promising Cmdlet: ```Get-MgPrivilegedRole``` but when I ran it, the following error was returned.

```powershell
Get-MgPrivilegedRole_List: {"error":{"code":"TenantEnabledInAadRoleMigration","message":"The current endpoints of AAD roles have been disabled for the tenantfor migration purpose. Please use the new Azure AD RBAC roles. Please refer to https://aka.ms/PIMFeatureUpdateDoc for new PIM features; https://aka.ms/PIMAPIUpdateDoc for API and PowerShell changes because of migration."}}
```

After some more searching of the module, I discovered ```Get-MgDirectoryRole``` and this returned the information I needed.

```powershell
Get-MgDirectoryRole
Get-MgDirectoryRole | Select-Object -Property DisplayName, Description | Sort-Object -Property DisplayName
```

I found that to filter for a specific role, you can do the following:

```powershell
Get-MgDirectoryRole -Filter "DisplayName eq 'Global Administrator'"
Get-MgDirectoryRole -Filter "DisplayName eq 'User Administrator'"
```

This was good and did what I needed but I realised it wasn't returning all of the privileged roles available

```powershell
Get-MgRoleManagementDirectoryRoleDefinition
Get-MgRoleManagementDirectoryRoleDefinition -Filter "DisplayName eq 'Global Administrator'"
```

Now that I can find the roles and their IDs, I created the script to get the users.

```powershell
$memberList = [System.Collections.Generic.List[string]]::new()
$roleId = (Get-MgDirectoryRole -Filter "DisplayName eq 'Global Administrator'").Id
$userList = Get-MgDirectoryRoleMember -DirectoryRoleId $roleId

foreach ($user in $userList) {
    $upn = (Get-MgUser -UserId $user.id).UserPrincipalName
    $memberList.Add($upn)
}
```

Now I had a list of users in the privileged groups that is used in the report.

## Summary

Graph https://docs.microsoft.com/en-us/graph/api/rbacapplication-list-roledefinitions?view=graph-rest-beta&tabs=http#code-try-3
