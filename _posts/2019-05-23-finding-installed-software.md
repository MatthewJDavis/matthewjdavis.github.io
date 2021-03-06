---
title: Finding installed software with PowerShell on Windows
author: Matthew Davis
date: 2019-05-23
toc: true
excerpt: How to find installed software on Windows using PowerShell
categories:
    - powershell
tags:
    - chocolatey
    - ciminstance
published: true
---
May 2019

# Overview

Here's a post on a few ways you can find installed software on Windows using PowerShell.

I've been using Pester to verify software that is installed on TeamCity build agents that are being created with Packer and a number of PowerShell scripts. Getting a list of the installed software has taken a number of different approaches with PowerShell but allows to test that the correct software is installed and also produce a text file with the software installed information as a build artifact for later reference.

I also recently watched a session from the PowerShell summit 2019 on [youtube] that brings up the issue that using Win32_Product makes the msi installers run consistency checks and potentially run repairs. Even the [Microsoft docs] give the ```Get-WMIObbject -Class Win32_Product``` as an example to list installed software but don't mention the side effects of using this.

## The registry way

```powershell
# Find installed software via registry
Get-ChildItem -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\ | Get-ItemProperty |
Select-Object DisplayName, DisplayVersion

Get-ChildItem -Path HKLM:\SOFTWARE\Wow6432node\Microsoft\Windows\CurrentVersion\Uninstall | Get-ItemProperty |
Select-Object DisplayName, DisplayVersion | Format-List
```

## Chocolatey

Simple check to see if chocolatey is installed then list all of the packages that have been installed locally.

```powershell
if($env:Path -like '*chocolatey*') {
    choco list -lo
}
```

## Avoid Get-CimInstance and Get-WMIObject

Using Get-CimInstance (or the old method of Get-WMIObject) using the Win32_Product, not only takes longer to query but also will cause the msi installers to fire and will reconfigure or run consistency checks that could have undesired consequences to the software installed on the machine as shown below.

```powershell
Get-CimInstance -ClassName 'Win32_product'
```

```powershell
Get-WinEvent -FilterHashtable  @{Logname='Application';Id=1035} -MaxEvents 20
```

![do not use Get-CimInstance](/images/finding-installed-software/get-ciminstance.png)

## DotNet frameworks

.Net frameworks from version 4 are [backwards compatible], so version 4.8 can run applications created in .Net 4.0 to 4.7.2
We can use the registry method to check the installed versions of .Net 4.

```powershell
# https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
# .Net Version 4.5 and above
(Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full\' |
    Get-ItemProperty -Name Version).version

# .Net versions below 4.5
```

On a fresh install of Windows Server 2019

![dotnet 4.7 install](/images/finding-installed-software/dotnet4-7.png)

After installing .Net 4.8, 4.8 is the version that is found in the registry after a reboot

```powershell
choco install netfx-4.8-devpack --version 4.8.0.0-rtw2 --pre -y

(Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full\' |
    Get-ItemProperty -Name Version).version
```

![dotnet 4.8 install](/images/finding-installed-software/dotnet4-8.png)

### .Net 3.5

The .Net 3.5 version can be again found in the registry.

```powershell
# In the 3.5 registry key

(Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v3.5'  |
    Get-ItemProperty -Name Version).version
```

![dotnet 3.5 install](/images/finding-installed-software/dotnet3-5.png)

### Other bits of software

Some software may not be installed, but can still be run by including the path to the exe file in the environment path.
To identify where to look, you can iterate over the environment path directories for exe files.

```powershell
$pathList = $env:Path -split ';'
$pathList | Foreach-Object {if(gci -Path $_ -Filter *.exe) {Write-Output "$_"}
```

![from path](/images/finding-installed-software/from-path.png)

Obviously there will be a lot of Microsoft executables in there but you can see from the screen shot, I have go, packer, vagrant etc available to run on the machine.

## Using in Pester tests

Lastly an example using some of the methods above in a Pester script to check that certain software versions are installed. This can be run as part of a CI/CD pipeline for creating machines.

```powershell
Describe 'Available Software Checks' {
    context 'Standard installation location installs' {
        $installsList = [collections.generic.list[psobject]]::new()

        $installs64 =  Get-ChildItem -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\' | Get-ItemProperty |
            Select-Object DisplayName, DisplayVersion

        $installs32 =  Get-ChildItem -Path 'HKLM:\SOFTWARE\Wow6432node\Microsoft\Windows\CurrentVersion\Uninstall\' | Get-ItemProperty |
            Select-Object DisplayName, DisplayVersion

        $installs64 | ForEach-Object {$installsList.Add($_)}
        $installs32 | ForEach-Object {$installsList.Add($_)}

        it 'should have Microsoft Monitoring Agent version 8 installed' {
            ($installsList | Where-Object -Property DisplayName -eq 'Microsoft Monitoring Agent').DisplayVersion | Should -BeLike "8.0*"
        }
        it 'should have Google Chrome version 74 installed' {
             ($installsList | Where-Object -Property DisplayName -eq 'Google Chrome').DisplayVersion | Should -BeLike "74*"
        }
    }
    context 'Chocolatey Installs' {
        $chocoPackages = chocolatey list --localonly

        it 'Should have notepad plus plus installed' {
            ($chocoPackages -like 'notepadplusplus*').Count -ge 1 | Should -Be $true
        }
        it 'Should have nssm installed' {
            ($chocoPackages -like 'nssm*').Count -ge 1 | Should -Be $true
        }
    }
    context '.Net Framework installs' {
        # Get .Net framework versions from the registry
        $dotNet3dot5 = (Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v3.5' |
            Get-ItemProperty -Name Version).version

        $dotNet4 = (Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full\' |
            Get-ItemProperty -Name Version).version

        # .Net framework tests
        it 'Should have .Net 3.5 framework installed' {
            $dotNet3dot5 | Should -Belike '3.5.*'
        }
        it 'Should have .Net 4.8 framework installed' {
            $dotNet4 | Should -BeLike '4.8*'
        }
    }
}
```

![from path](/images/finding-installed-software/pester-tests.png)

## Summary

We can use PowerShell to check what's installed on a machine that can be used for reporting or verification with Pester to make sure all the correct software and right versions are installed. One thing to take away is use the registry method for normal installations and the .Net frameworks.

[Article] on differences between registry way and wmi way.

[Microsoft docs]: https://docs.microsoft.com/en-us/powershell/scripting/samples/working-with-software-installations?view=powershell-6
[youtube]: https://youtu.be/fAfxDjg1Y_M?t=
[backwards compatible]: https://github.com/dotnet/docs/blob/master/docs/framework/install/on-windows-10.md
[Article]: https://mcpmag.com/articles/2017/07/27/gathering-installed-software-using-powershell.aspx