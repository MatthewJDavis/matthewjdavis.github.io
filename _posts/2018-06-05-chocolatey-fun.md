---
title: Chocolatey fun, Windows package management
author: Matthew Davis
date: 2018-06-05
excerpt: How to find AD users in a specific OU created in the last 31 days
categories: 
    - chocolatey
tags:
    - chocolatey
published: false
---

# Chocolatey goodness

I seem to be back on the Windows track at work and at home, after a spell away learning AWS and doing lots of work with Ubuntu linux. I've been ticking over with PowerShell but definitely plateaued, mainly learning and using AWS cmdlets but not really scripting and it's funny how some of the knowledge gained over the last few years has slipped to the back of the brain. So it's time to get back into PowerShell, Windows and taking a good look at Chocolatey package manager (after using apt-get in Ubuntu which is great and working with Ansible to invoke apt-get, I've used Chocolatey a bit in the past but want to learn deeper about the project and technology and who knows... might even bring back some of those rooted away PowerShell skills!).

Chocolatey what is it?!

Chocolatey is a package manager for Windows, it makes automating software installation, configuration and uninstallation easier by wrapping the underlying installation technologies in an easy to use command line syntax.

Once Chocolatey in installed and providing someone has packaged the software, it's as simple as typing the following to install it:

```powershell
chocolatey install sysinternals
```

That's it, Chocolatey will go to the chocolatey.org repository, download and install the software. Sysinternals are standalone executables and Chocolatey uses a 'shim' so the executables are available from the commandline (Chocolatey has a location on the Path, the chocolatey\bin directory) but it works with software that is installed via msi, setup.exe, inno, msu etc.

## Useful commands

Chocolatey has a number of commands and the help system is great, just like PowerShell's. Use /? to find options on the command line parameters.

The chocolatey command can be abbreviated to choco to save typing!

### help
```powershell
# Show default options and switches
chocolatey /?
# Show install options - abbreviated to choco
choco install /?
# Show list options
chocolatey list /?
# Show uninstall options
choco uninstall /?
```
## Installing

```powershell
chocolatey install package name

```


Package Repos

Checking packages / security

Using with Ansible / other configuration management tools