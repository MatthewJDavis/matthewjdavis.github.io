---
title: Configure OU permissions for Okta Active Directory agent
author: Matthew Davis
date: 2019-02-26
toc: true
excerpt: User PowerShell and dsacls to update the OU permissions for the Okta Active Directory agent.
categories:
    - PowerShell
tags:
    - okta
    - dsacls
    - cacerts
published: true
---

# Overview

The user that runs the Okta Active Directory agent requires a number of different permissions to the desired OU(s) that are set out in the [docs] under the *Minimum Okta Service Account permission requirements* section (you may not need to give all permission depending on what you want Okta to do with your users but this example takes the full permissions outlined in the docs).

The docs lists the commands needed using the [dsacls] cmd.exe program to set the Organisational Unit permissions. You can use the [Set-ACL] PowerShell cmdlet to update the permissions, but seeing as Okta has done the work, it makes sense to wrap the dsacls in PowerShell to make it easy to update for target OUs, service account names etc.

All the script does is build the dsacls command syntax and save each one to a PowerShell ArrayList, then pipe each of the dsacls to cmd to be executed in a foreach loop.

```powershell
# Set the permissions on the Users OU for the Okta agent as per docs:
# https://help.okta.com/en/prod/Content/Topics/Directory/ad-agent-install.htm
# Using a group managed service account for the Okta agent so the account name has a $ 

$OU = '"OU=Users,DC=ad,DC=matthewdavis111,DC=com"'
$ServiceAccount = 'gmsa-okta-sync$'
$Domain = 'ad.matthewdavis111'

$dsaclsCmd = "dsacls $OU /I:S /G $Domain\$ServiceAccount"

# Array list to hold all of the dsacls commands for the various permissions for Okta
$commandList = New-Object System.Collections.ArrayList

# Create or Update user
# include additional attributes that are mapped in your org within Okta
$commandList.Add($dsaclsCmd + ':WP;mail;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;userPrincipalName;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;sAMAccountName;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;givenName;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;sn;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;userAccountControl;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;pwdLastSet;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;lockoutTime;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;cn;user') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;name;user') | Out-Null
# Create user/Password Reset
$commandList.Add($dsaclsCmd + ':CA;Reset Password;user') | Out-Null
#Group Push
$commandList.Add($dsaclsCmd + ':CCDC;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;sAMAccountName;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;description;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;groupType;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;member;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;cn;group') | Out-Null
$commandList.Add($dsaclsCmd + ':WP;name;group') | Out-Null

# Run the commands piping to cmd
foreach ($command in $commandList) {
    $command | cmd
}

```

## Summary

Using PowerShell to wrap the cmd.exe program dsacls makes it easy to target different OUs and different service accounts. As Okta has already provided the necessary commands and permissions, it didn't make sense to me to rewrite them in PowerShell as time was of the essence for this particular project, but using PowerShell to build the commands make it less prone to typos and error and easier to update.

[docs]: https://help.okta.com/en/prod/Content/Topics/Directory/ad-agent-install.htm
[dsacls]: https://ss64.com/nt/dsacls.html
[Set-Acl]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-acl?view=powershell-6