-- -
title: Use PowerShell to see UPN lengths of Active Directory users.
excerpt: Small script that will get the UserPrincipalName lengths of all users in Active Directory and sort them by size.
categories:
- powershell
tags:
- active directory
- powershel
published: false
-- -
August 2019

# Overview

Quick one today, had an application that was using an Azure Enterprise application for Single Sign On via SAML that would let the user log in then log them straight back out.
Troubleshooting with the user provided no indications of what was going on and after checking the Active Directory account, I noticed the UserPrincipalName (UPN) was rather long.
This got me thinking on how it compared to other UPNs for length in Active Directory so wrote a quick script to find out.

## Script

Obviously the time it takes depends on how many users there are in the directory, use the SearchBase parameter to specify which OUs to get the users from.

```powershell
# Gets all the AD users under the users OU and sorts them by length of UserPrincipalName.

$upnList = (Get-ADUser -Filter * -Properties userprincipalname -SearchBase 'OU=Users,DC=matthewdavis111,DC=com').userprincipalname

$upnDetails = foreach($upn in $upnList){
                  [pscustomobject]@{
                      'Name' = $upn
                      'Count' =  $upn.ToCharArray().count
                  }
              }

$sorted = $upnDetails | sort count -Descending

# Few the top ten longest UPNS
$sorted | Select-Object -First 10

# output to csv
$sorted | Export-Csv upn-length.csv -NoTypeInformation
```

## Summary

Nothing too difficult here but was handy to find out and the user in question who was having issues with SSO login had the 3rd longest UPN in the directory being a couple of service accounts.

