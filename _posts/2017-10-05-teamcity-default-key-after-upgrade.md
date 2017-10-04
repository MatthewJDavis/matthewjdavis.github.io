---
title: Specify Teamcity default keys after upgrade
author: Matthew Davis
date: 2017-10-05
excerpt: After completing a teamcity upgrade from version 9 to 2017, the project wasn't able to connect to Github because of authentication failure.
categories: 
    - teamcity
tags:
    - teamcity
published: false
---

I've recently been testing upgrading [teamcity] server from version 9.1 to the latest version 2017.1. During testing of the upgrade, we noticed that projects that were using the default ssh key to connect to Github were failing authentication failure. When testing the connection in the project settings, specifying the ssh key by name directly in the project worked, so the key was ok, but the default key setting couldn't find the key.

Luckily teamcity gives a good message on the screen on where it looks for the default key settings, in a config file: C:\Windows\System32\config\systemprofile\.ssh\config There was no config file on the previous installation, after some investigation a ssh config file is required.

The keys that are uploaded for teamcity can be found in the teamcity data directory under: config\projects\_Root\pluginData\ssh_keys\
For example, if your data directory is installed on a drive mapped to **t** and the directory is called "Teamcity-Data", you will find your uploaded keys:

T:\Teamcity-Data\config\projects\_Root\pluginData\ssh_keys\

Here's the process on how to reference where the private keys are (stored in the teamcity data drive) which should get the default keys working for all the projects that reference it.

1. Create the .ssh directory

```powershell
New-Item -Path C:\Windows\System32\config\systemprofile\.ssh -ItemType Directory
```

2. Create the config file

```powershell
New-Item -Path C:\Windows\System32\config\systemprofile\.ssh\config -ItemType File
```

3. Add content for where the key is located

```powershell
$content = "Host *`r`nIdentityFile T:\Teamcity-Data\config\projects\_Root\pluginData\ssh_keys\github_rsa"
Set-Content -Path C:\Windows\System32\config\systemprofile\.ssh\config -Value $content -Force
```

The upgrade was performed on a Windows 2016 server.


[teamcity]: https://www.jetbrains.com/teamcity/