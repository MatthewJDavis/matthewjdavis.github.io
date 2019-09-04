---
title: Use PowerShell to see UPN lengths of Active Directory users.
excerpt: Small script that will get the UserPrincipalName lengths of all users in Active Directory and sort them by size.
categories:
- powershell
tags:
- active directory
- powershell
published: true
---
August 2019

# Overview

Quick one today, had an application that was using an [Azure Enterprise application] for Single Sign On (SSO) via SAML that would log the user in and straight back out.
Troubleshooting with the user provided no indications of what was going on and after checking the Active Directory account, I noticed the UserPrincipalName (UPN) was rather long.
This got me thinking on how it compared to other UPNs for length in Active Directory so wrote a quick script to find out.

## Script

Obviously the time it takes depends on how many users there are in the directory, use the SearchBase parameter to specify which OUs to get the users from.

```powershell
# Gets all the AD users under the users OU and sorts them by length of UserPrincipalName.

$upnList = (Get-ADUser -Filter * -Properties userprincipalname -SearchBase 'OU=Users,DC=matthewdavis111,DC=com').userprincipalname

$upnDetails = foreach ($upn in $upnList) {
    if ($upn.Length -gt 0) {
        [pscustomobject]@{
            'Name'  = $upn
            'Count' = $upn.ToCharArray().count
        }
    }
}

$sorted = $upnDetails | sort count -Descending

# find the top ten longest UPNS
$sorted | Select-Object -First 10

# output to csv
$sorted | Export-Csv upn-length.csv -NoTypeInformation
```

### List of UPNs and length

![sorted output](/images/ps-upn-count/sorted.png)

### Ten longest UPNS

![ten longest upns](/images/ps-upn-count/first-10.png)


## Summary

Nothing too difficult here but was handy to find out and the user in question who was having issues with SSO login had the 3rd longest UPN in the directory being a couple of service accounts.

After contacting support with the logs and my theory that the lenght of the UPN was causing the issue, a fix was released and the user can now SSO into the application.

[Azure Enterprise application]: https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/what-is-application-management
