---
title: Installing the AzureRM Managed Service Identity PowerShell Module
author: Matthew Davis
date: 2018-11-16
excerpt: Updating PowerShellGet module to enable the installation of the prerelease module for AzureRM
categories: 
    - powershell
tags:
    - powershell
    - powershellget
    - azurerm
published: true
---
November 2018

# Overview

This post will details how to install the [prerelease AzureRM Managed Service Identity PowerShell module] from the PowerShell gallery which also involved me updating the [PowerShellGet] module (currently version 2.0.2) to get the Find-Module and Get-Module Cmdlets AllowPreRelease switch (introduced in version 1.6.0).

I wanted to investigate using [Azure Container Instances] in a [TeamCity] or [Jenkins] job. I've been working mainly in AWS and have been using [Instance Profiles] to apply to [EC2 Instances] giving them access to resources such as S3 buckets without the need of storing credentials anywhere on the machines and wanted to see how this worked in Azure.

After a bit of searching around, I had found out about [Managed identities for Azure resources] and applying System Assigned or User Assigned identities to an Azure VM. This looked like what I wanted to do so thought I do it in PowerShell.

![VM Identity blade via the portal](/images/install-managed-service-identity-module/vm-identity.png)

I fired up my console and authenticated to AzureRM.

```powershell
Add-AzureRmAccount
```

I then tried to find AzureRmUserAssignedIdentity Cmdlets, no luck. I checked for the module and it wasn't on my machine. I knew I had the latest AzureRM module because I had updated it a week ago.

I searched the PowerShell Gallery for it with the Find-Module Cmdlet, still nothing so I decided to have a look on the [PowerShell Gallery website].

I did find the module with the [AzureRM.ManagedServiceIdentity] Cmdlets via the search on the PowerShell gallery website and noticed it was a Prerelease version with the -AllowPreRelease switch specified for Install-Module. I've not seen this before (or come across prereleased modules before) and checked my install module and find module commands to find the Prerelease switch was missing. Prerelase modules were announced in December 2017 on the [PowerShell Team Blog]

![Find-Module with no prerelease switch](/images/install-managed-service-identity-module/module-search.png)

I checked to see if I had the latest version of PowerShellGet on my system as I don't remember ever updating it and I hadn't. I was using Version 1.0.0.1 and the latest on the gallery was 2.0.2 so thought I would update to it.

![Showing the latest version of PowerShellget from the PowerShell gallery](/images/install-managed-service-identity-module/latest-psget-module.png)

Looking through the release notes, I could see the feature was added in version 1.6.0.

![Getting the release notes for the module](/images/install-managed-service-identity-module/release-notes.png)

```powershell
$module = Find-Module powershellget
$module.ReleaseNotes > releasenotes.txt
code .\releasenotes.txt
```

![Release notes shown in vscode](/images/install-managed-service-identity-module/release-notes1.png)

I ran the Update-Module -Name powershell get but received the following error:

```powershell
Update-Module : Module 'powershellget' was not installed by using Install-Module, so it cannot be updated.
At line:1 char:1
+ Update-Module powershellget
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (powershellget:String) [Write-Error], WriteErrorException
    + FullyQualifiedErrorId : ModuleNotInstalledUsingInstallModuleCmdlet,Update-Module
```

So I installed with the Install-Module Cmdlet:

```powershell
Install-Module -Name PowerShellGet -Force
```

I then got the following warning:

```powershell
WARNING: The version '1.2.2' of module 'PackageManagement' is currently in use. Retry the operation after
closing the applications.
```

After exiting the console (refreshenv didn't work), I could now see the latest module and imported it with the -RequiredVersion parameters (see [this post] on the order PowerShell imports modules with different versions on the system).

```powershell
Import-Module -Name PowerShellGet -RequiredVersion 2.0.2
```

![Importing version 2.0.2 of PowerShellGet with Import-Module cmdlet](/images/install-managed-service-identity-module/ps-get-2-0-2.png)

Now I could see all the versions available to me on the PowerShell gallery with the following command:

```powershell
Find-Module -Name AzureRM.ManagedServiceIdentity -AllowPrerelease -AllVersions
```

![Finding all the versions of the module from the gallery](/images/install-managed-service-identity-module/find-module-prerelease.png)

And installed the latest version with:

```powershell
Install-Module -Name AzureRM.ManagedServiceIdentity -AllowPrerelease
```

And could see the three cmdlets with the Get-Command:

```powershell
Get-Command -Module AzureRM.ManagedServiceIdentity
```

![Output of Get-Command](/images/install-managed-service-identity-module/get-command.png)

## Summary

That's it, I found out about the preview modules on the PowerShell gallery, updating my PowerShellGet module so I was able to find the latest ones, how to get the release notes with the Find-Module command releasenotes property and upgrading my PowerShellGet module so I could use the preview module. Now back to the original task of looking at applying the Managed Identity to the Azure VM and seeing how it works!

[PowerShellGet]: https://www.powershellgallery.com/packages/PowerShellGet/2.0.2
[Azure Container Instances]: https://azure.microsoft.com/en-ca/services/container-instances/
[Instance Profiles]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html
[EC2 instances]: https://aws.amazon.com/ec2/
[TeamCity]: https://www.jetbrains.com/teamcity/
[Jenkins]: https://jenkins.io/
[Managed identities for Azure resources]: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[PowerShell Team Blog]: https://blogs.msdn.microsoft.com/powershell/2017/12/05/prerelease-versioning-added-to-powershellget-and-powershell-gallery/
[PowerShell Gallery website]: https://www.powershellgallery.com
[AzureRM.ManagedServiceIdentity]: https://docs.microsoft.com/en-us/powershell/module/azurerm.managedserviceidentity/?view=azurermps-6.12.0
[prerelease AzureRM Managed Service Identity PowerShell module]: https://www.powershellgallery.com/packages/AzureRM.ManagedServiceIdentity/1.0.5-preview
[this post]: https://info.sapien.com/index.php/scripting/scripting-modules/which-version-does-import-module-import