---
title: Finding installed software with PowerShell on Windows
author: Matthew Davis
date: 2019-05-23
toc: false
excerpt: A couple of methods on how to find installed software with PowerShell
categories:
    - powershell
tags:
    - chocolatey
    - ciminstance
published: false
---
May 2019

# Overview

I've been using Pester to verify software that is installed on TeamCity build agents that are being created with Pacer and a number of PowerShell scripts (this is before being migrated to building the agents with Ansible). Getting a list of the installed software has taken a number of different approaches so decided to write them up here. I also recently watched a session from the PowerShell summit 2019 on [youtube] that brings up the issue that using Win32_Product makes the msi installers run consistency checks and potentially run repairs. Even the [Microsoft docs] give the ```Get-WMIObbject -Class Win32_Product``` as an example to list installed software.

## The registry way

```powershell
# Find installed software via registry
Get-ChildItem -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\ | Get-ItemProperty |
Select-Object DisplayName, UninstallString | Format-List

Get-ChildItem -Path HKLM:\SOFTWARE\Wow6432node\Microsoft\Windows\CurrentVersion\Uninstall | Get-ItemProperty |
Select-Object DisplayName, UninstallString | Format-List
```

## Chocolatey

```powershell
if($env:Path -like '*chocolatey*') {
    choco list -lo
}
```

## Avoid Win32_Product

Using Get-CimInstance (or the old method of Get-WMIObject) using the Win32_Product, not only takes longer to query but also will cause the msi installers to fire and will reconfigure or run consistency checks that could have undesired consequences to the software installed on the machine.

```powershell
Get-CimInstance -ClassName 'Win32_product'
```

```powershell
Get-WinEvent -FilterHashtable  @{Logname='Application';Id=1035} -MaxEvents 20
```

## DotNet frameworks

```powershell
# https://docs.microsoft.com/en-us/dotnet/framework/migration-guide/how-to-determine-which-versions-are-installed
# .Net Version 4.5 and above
(Get-ChildItem 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full\' |
    Get-ItemProperty -Name Version).version

# .Net versions below 4.5
```

After installing .Net 4.8

```powershell
 choco install netfx-4.8-devpack --version 4.8.0.0-rtw2 --pre -y
```



[Microsoft docs]: https://docs.microsoft.com/en-us/powershell/scripting/samples/working-with-software-installations?view=powershell-6
[youtube]: https://youtu.be/fAfxDjg1Y_M?t=
https://mcpmag.com/articles/2017/07/27/gathering-installed-software-using-powershell.aspx