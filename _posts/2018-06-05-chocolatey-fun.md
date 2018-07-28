---
title: Chocolatey fun, Windows package management
author: Matthew Davis
date: 2018-06-05
excerpt: Using package mangent on Windows with Chocolatey and useful commands.
categories: 
    - chocolatey
tags:
    - chocolatey
published: true
---

# Chocolatey goodness

I seem to be back on the Windows track at work, after a spell away learning AWS and doing lots of work with Ubuntu linux. I've been ticking over with PowerShell but definitely plateaued, mainly learning and using AWS cmdlets but not really scripting and it's funny how some of the knowledge gained over the last few years has slipped to the back of the brain. So it's time to get back into PowerShell, Windows and taking a good look at [Chocolatey package manager] (after using apt-get in Ubuntu which is great and working with Ansible to invoke apt-get, I've used Chocolatey a bit in the past but want to learn deeper about the project and technology and who knows... might even bring back some of those rooted away PowerShell skills!).

Chocolatey what is it?!

Chocolatey is a package manager for Windows, it makes automating software installation, configuration and uninstallation easier by wrapping the underlying installation technologies in an easy to use command line syntax.

**Chocolatey should be run from an admin command prompt, but you can run as a non admin by following the official [guide]**

Once Chocolatey in installed and providing someone has packaged the software, it's as simple as typing the following to install it:

```powershell
chocolatey install sysinternals
```

That's it, Chocolatey will go to the chocolatey.org repository, download and install the software. Sysinternals are standalone executables and Chocolatey uses a 'shim' so the executables are available from the commandline (Chocolatey has a location on the Path, the chocolatey\bin directory) but it works with software that is installed via msi, setup.exe, inno, msu etc.

## Useful commands

Chocolatey has a number of commands and the help system is great, just like PowerShell's. Use /? to find options on the command line parameters.

The chocolatey command can be abbreviated to choco to save typing!

These commands are all available in chocolatey version 0.10.8

### Help

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

### Installing

```powershell
# Install a package - you'll be prompted to accept licences etc
chocolatey install 'package name'
# Install with licences accepted so no prompt
choco install 'package name' -y
# Shorter way to install with confirmation of licence acceptance
cinst 'package name' -y
```

### Uninstalling

```powershell
# Uninstall a package that is managed by chocolatey
chocolatey uninstall 'package name'
# Uninstall confirming prompts
choco uninstall 'package name' -y
# Shorter way to uninstall with confirmation of prompts
cuninst 'package name' -y
```

### Searching for Packages

```powershell
# List locally installed packages
choco list -lo
# Search for a package on a repo (default repo is chocolatey.org)
choco list 'package name'
# Find packages that exactly match the search query
choco list -exact 'package name'
```

Package Repos

The main chocolatey package repository is currently found at the feed: https://chocolatey.org/api/v2/

You could also host your in-house choco repository using software such as [proget], [nexus] or [artifactory]. You can also set your chocolatey source to point to a file share and install packages from there.

For hosting files on a share, you can download the nupkg file from the chocolatey website.

```powershell
# View current source
choco source
# Add a local directory as a package source
choco source add -name="local" -source="c:\choco-packages"
# List local source the was added above
choco list -source="local"
# Install from local source
choco install notepadplusplus.install -source="local" -y
```

Checking packages / security

There are a number of ways to verify the integrity of the package and chocolatey professional licence gives you more peace of mind.

On the website you can:
* View the install.ps1 script
* Check for this package was approved as a trusted package (date)
* View the package source (usually a link to the code in source control)

Some packages also have checksum checks that will fail if the package has been changed. You'll see the checksum in the output of the package install from the command line.

### Using with Ansible

Ansible has the module [win_chocolatey] that will install chocolatey packages from Ansible playbooks. If on the first run through Ansible detects that chocolatey is not installed, it will install chocolatey as part of the task. 

If you don't specify a source, the chocolatey repo will be used

```yaml
- name: Install notepadplusplus
  win_chocolatey:
    name: notepadplusplus
    source: //fileshare/packages
```

[Chocolatey package manager]:https://chocolatey.org/
[guide]:https://chocolatey.org/docs/installation#non-administrative-install
[proget]:https://inedo.com/proget
[nexus]:https://www.sonatype.com/nexus-repository-oss
[artifactory]:https://jfrog.com/artifactory/
[win_chocolatey]:https://docs.ansible.com/ansible/latest/modules/win_chocolatey_module.html