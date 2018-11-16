---
title: Authenticate and query the Microsoft Graph with PowerShell
author: Matthew Davis
date: 2018-11-16
excerpt: Use PowerShell to authenticate with and query the Microsoft Graph
categories: 
    - powershell
tags:
    - powershell
    - azure
    - graph
published: false
---

I wanted to do a post on using Azure Contaner Instances in a TeamCity job. I've been working mainly in AWS and have been setting Instance Policies to apply to EC2 Instances giving them access to resources such as S3 buckets without the need of storing credentials anywhere on the machines and wanted to see how this worked in Azure.

After a bit of searching around, I had found you can apply System Assigned or User Assigned identities to an Azure VM, cool I thought, let's do this in PowerShell.


![VM Identity blade via the portal](/images/install-managed-service-identity-module/vm-identity.png)

I fired up my console and authenticated to AzureRM.

I then tried to find AzureRmUserAssignedIdentity Cmdlets, no luck. I checked for the module and it wasn't on my machine. I knew I had the latest AzureRM module because I had updated it a week ago.

I searched the PowerShell Gallery for it with the Find-Module Cmdlet, still nothing so I decided to have a look on the PowerShell Gallery website.

I did find the [AzureRM.ManagedServiceIdentity] module via the search on the website and noticed it was a Prerelease version with the -AllowPreRelease switch specified for Install-Module. I've not seen this before (or come across prereleased modules before) and checking my install module and find module commands, the Prerelease switch was missing.

![Find-Module with no prerelease switch](/images/install-managed-service-identity-module/find-module.png)

I thought I'd check to see if I had the latest version of PowerShellGet on my system as I don't remember ever updating it... and I hadn't. I was using Version 1.0.0.1 and the latest on the gallery was 2.0.2 so thought I'd install it.

Looking through the release notes, I could see the feature was added in version 1.6.0.

![Getting the release notes for the module](/images/install-managed-service-identity-module/release-notes.png)

```powershell
$module = Find-Module powershellget
$module.ReleaseNotes > releasenotes.txt
code .\releasenotes.txt
```

I ran the Update-Module -Name powershell get but received the following error:

```powershell
Update-Module : Module 'powershellget' was not installed by using Install-Module, so it cannot be updated.
At line:1 char:1
+ Update-Module powershellget
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (powershellget:String) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : ModuleNotInstalledUsingInstallModuleCmdlet,Update-Module
```

So had to install it with 

```powershell
Install-Module -Name PowerShellGet -Force
```

I then got the following warning: 

```powershell
WARNING: The version '1.2.2' of module 'PackageManagement' is currently in use. Retry the operation after
closing the applications.
```

After exiting the console (a refreshenv didn't work), I could now see the latest module and imported it (PowerShell imports the latest version by default).

![Importing version 2.0.2 of PowerShellGet with Import-Module cmdlet](/images/install-managed-service-identity-module/ps-get-2-0-2.png)




![Release notes shown in VSCode](/images/install-managed-service-identity-module/release-notes2.png)




![Showing the latest version of PowerShellget from the PowerShell gallery](/images/install-managed-service-identity-module/latest-psget-module.png)

Now I could see all the versions available to me on the PowerShell gallery with the following command:

```powershell
Find-Module -Name AzureRM.ManagedServiceIdentity -AllowPrerelease -AllVersions
```

![Finding all the versions of the module from the gallery](/images/install-managed-service-identity-module/find-module-prerelease.png)


And installed the latest version with:

```powershell
Install-Module -Name AzureRM.ManagedServiceIdentity -AllowPrerelease
```

And could see the three cmdlets with the old Get-Command:

```powershell
Get-Command -Module AzureRM.ManagedServiceIdentity
```

![Output of Get-Command](/images/install-managed-service-identity-module/get-command.png)

## Summary

That's it, I found out about the preview modules on the PowerShell gallery, updating my PowerShellGet module so I was able to find the latest ones, how to get the release notes with the Find-Module command releasenotes property and upgrading my PowerShellGet module so I could use the preview module. Now back to the original task of looking at applying the Identity to the Azure VM and seeing how it works!

https://docs.microsoft.com/en-us/powershell/module/azurerm.managedserviceidentity/?view=azurermps-6.12.0

[AzureRM.ManagedServiceIdentity]: https://www.powershellgallery.com/packages/AzureRM.ManagedServiceIdentity/1.0.5-preview