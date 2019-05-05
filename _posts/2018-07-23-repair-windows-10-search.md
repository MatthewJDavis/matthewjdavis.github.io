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
    - registry
    - search
published: true
---
July 2018

# Overview

About a month ago I realised that the search functionality on my main Windows 10 VM had stopped working. I could not search from the Windows menu, search in Windows explorer for files or folders and could not search in any of the settings search boxes. It wasn't a big deal as I mainly used it for PowerShell and VSCode and got by fine for the last month but decided to fix it today. I tried a number of fixes (including the good old turn it off an on) that I saw on various Microsoft forum answers including killing the Cortana process, disabling and enabling Cortana, running Windows trouble shooter for search, reindexing and running sfc /scannow. None however worked.

I then decided to see if Edge was working (this is probably the first time I had opened Edge on the VM), I couldn't type in the address bar or the search bar in the main web page that opens in edge... interesting.

While talking with a friend about the problem, he found the following [post in the windows 10 forum] with the marked answer that worked for me: 

Run the ctfmon.exe found:

**C:\Windows\system32\ctfmon.exe**

This worked straight away, I could now search from the Windows menu, search in edge and Windows explorer.

However the fix did not persist after reboot, which was slightly annoying.

You can however set registry keys ([details in this msdn article)]) to run every time you login.

Adding the following via PowerShell to your registry will run the command every time you log into the computer. You don't need to run PowerShell as an admin on this occasion as you are adding to the current user registry.

<script src="https://gist.github.com/MatthewJDavis/e46333ba3eab570c3b585cc8ab38e3f6.js"></script>

And there you have it. All was good in the search world of my Windows 10 VM. This may not fix your search problems, as I've seen many out there caused by different errors, but this one fixed mine so it's worth trying and if it does work, then add it to your registry for the next time you login.


[post in the windows 10 forum]:https://answers.microsoft.com/en-us/windows/forum/windows_10-win_cortana/cant-type-in-windows-10-search-bar/7dce8411-8671-4d3e-90d1-b9bdc0aa4734
[details in this msdn article]:https://docs.microsoft.com/en-gb/windows/desktop/SetupApi/run-and-runonce-registry-keys