---
title: Get list of AzureAD User Principal Names of Global Admins in Azure AD.
excerpt: Use the Microsoft Graph PowerShell module to get a list of User Principal Names of Global Admins Azure AD role.
date: 2021-07-03
toc: false
classes: wide
categories:
- powershell
tags:
- azuread
- powershell
- graph
published: true
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

![get-command output](/images/azure-ad-ms-graph-roles/get-command.png)

I found a promising Cmdlet: ```Get-MgPrivilegedRole``` but when I ran it, the following error was returned.

```powershell
Get-MgPrivilegedRole_List: {"error":{"code":"TenantEnabledInAadRoleMigration","message":"The current endpoints of AAD roles have been disabled for the tenantfor migration purpose. Please use the new Azure AD RBAC roles. Please refer to https://aka.ms/PIMFeatureUpdateDoc for new PIM features; https://aka.ms/PIMAPIUpdateDoc for API and PowerShell changes because of migration."}}
```

![error output](/images/azure-ad-ms-graph-roles/error.png)

After some more searching of the module, I discovered ```Get-MgDirectoryRole``` and this returned the information I needed.

```powershell
Get-MgDirectoryRole
Get-MgDirectoryRole | Select-Object -Property DisplayName, Description | Sort-Object -Property DisplayName
```

![get-mgdirectoryrole output](/images/azure-ad-ms-graph-roles/get-mgdirectoryrole.png)

I found that to filter for a specific role, you can do the following:

```powershell
Get-MgDirectoryRole -Filter "DisplayName eq 'Global Administrator'"
Get-MgDirectoryRole -Filter "DisplayName eq 'User Administrator'"
```

![get-mgdirectoryrole filter output](/images/azure-ad-ms-graph-roles/get-mgdirectoryrolefilter.png)

Now that I can find the roles and their IDs, I created the script to get the user User Principal Names that belonged to the Global Administrators group I needed to report on.

```powershell
$memberList = [System.Collections.Generic.List[string]]::new()
$roleId = (Get-MgDirectoryRole -Filter "DisplayName eq 'Global Administrator'").Id
$userList = Get-MgDirectoryRoleMember -DirectoryRoleId $roleId

foreach ($user in $userList) {
    $upn = (Get-MgUser -UserId $user.id).UserPrincipalName
    $memberList.Add($upn)
}
```

### Update on getting all the roles

After raising an [issue on GitHub], it turns out that ```Get-MgDirectoryRole``` only returns [activated roles]. A role is activated via the AAD portal when a user or group is added to the role. It can also be activated via the [Graph API].

Below is a screenshot after I added a user to the Security Operator role and a group to the Conditional Access Administrator roles.

![Output after adding user and group to roles](/images/azure-ad-ms-graph-roles/updated-roles.png)

### Getting all of the roles

Use the following Cmdlet to get all of the roles in Azure AD including those that have not been activated.

```powershell
Get-MgDirectoryRoleTemplate | Select-Object DisplayName, Description | Sort-Object DisplayName
```

![Output of all roles](/images/azure-ad-ms-graph-roles/role-templates.png)

You can also find a full list of roles on the [official docs].

## Complete Script

<script src="https://gist.github.com/MatthewJDavis/6b385187cde51b26ac2a03a84d619835.js"></script>

## Summary

The Microsoft Graph PowerShell module is far from finished and is definitely lacking in help and examples at the moment which looks to be being worked on at the moment. This task took some time to figure out but because going [forward this is the module that will be invested in for PowerShell] interaction with the Microsoft Graph it was a good opportunity to use it more. There is still a lot to be done to add functionality. For example currently you can add members to a group via this module but you [can't remove them] currently without a workaround, however I will start using this module more in automation in the future.

[official docs]: https://docs.microsoft.com/en-us/azure/active-directory/roles/permissions-reference
[issue on GitHub]: https://github.com/microsoftgraph/msgraph-sdk-powershell/issues/739
[forward this is the module that will be invested in for PowerShell]: https://techcommunity.microsoft.com/t5/azure-active-directory-identity/automate-and-manage-azure-ad-tasks-at-scale-with-the-microsoft/ba-p/1942489
[can't remove them]: https://github.com/microsoftgraph/msgraph-sdk-powershell/issues/452
[activated roles]: https://docs.microsoft.com/en-us/graph/api/directoryrole-list?view=graph-rest-1.0&tabs=http
[Graph API]: https://docs.microsoft.com/en-us/graph/api/directoryrole-post-directoryroles?view=graph-rest-1.0&tabs=http
