---
title:  sysinternals PowerShell script
author: Matthew Davis
date: 2017-02-05
categories: 
    - powershell
tags:
    - powerShell
    - sysinternals
---
March 2017

# Overview

[sysinternals][sysinternals] are a great set of tools for the people administrating and troubleshootning Windows machines. It also includes [ZoomIt][ZoomIt] which is a really handy tool for presentations.

The tools can be run directly from the live site.

You can add the [rss feed][rss feed] to your favourite reader or Outlook to get the latest news on updates and the script below can be run to grab the latest versions of the tool and save them to a file location.

Here is a small PowerShell script that I use to download the latest sysinternals tools. I set this script to run once a month via a scheduled task to make sure I have the latest versions.

```powershell
$uri = 'https://live.sysinternals.com/'
$sysToolsPage = Invoke-WebRequest -Uri $uri
$sysIntPath = 'C:\sysinternals'

if (-not (Test-Path -Path $sysIntPath)){
    New-Item -Path $sysIntPath -ItemType Directory
}

Set-Location -Path $sysIntPath

$sysTools = $sysToolsPage.Links.innerHTML | Where-Object -FilterScript {$_ -like "*.exe" -or $_ -like "*.chm"} 


foreach ($sysTool in $sysTools){
    Invoke-WebRequest -Uri "$uri/$sysTool" -OutFile $sysTool
}
```

[sysinternals]: https://technet.microsoft.com/en-gb/sysinternals/bb545021.aspx
[rss feed]: https://blogs.technet.microsoft.com/sysinternals/feed/
[ZoomIt]: https://technet.microsoft.com/en-us/sysinternals/zoomit.aspx
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
