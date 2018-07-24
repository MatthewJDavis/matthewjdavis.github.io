---
title: Windows 10 Search stopped working
author: Matthew Davis
date: 2018-07-23
excerpt: On my main VM Windows 10 search stopped working along with input to the Edge browser
categories: 
    - Windows
tags:
    - windows 10
    - PowerShell
published: false
---

About a month ago I realised that the search functionality on my main Windows 10 VM had stopped working. I could not search with the Windows key, search in Windows explorer for files or folders and could not search in any of the settings search boxes. It wasn't a big deal as I mainly used it for PowerShell and VSCode and got by fine for the last month like that but decided to fix it today. I tried a number of fixes including killing the Cortana process, disabling and enabling Cortana, reinstalling the Windows store packages via PowerShell, running sfc /scannow and DSIM

I tried the first 6 suggestions from this article but none worked.

I then decided to see if Edge was working (this is probably the first time I had opened Edge on the VM), I couldn't type in the address bar or the search bar in the main msn webpage ... interesting.

While talking with a friend about the problem, he found the following post with the marked answer that worked for me: https://answers.microsoft.com/en-us/windows/forum/windows_10-win_cortana/cant-type-in-windows-10-search-bar/7dce8411-8671-4d3e-90d1-b9bdc0aa4734

This worked straight away however the fix did not persist after reboot.

Adding the following via PowerShell to your registry will run the command everytime the computer is booted.

```PowerShell
New-ItemProperty  -Name ctfmon -Value  C:\Windows\system32\ctfmon.exe -Path HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\run
```