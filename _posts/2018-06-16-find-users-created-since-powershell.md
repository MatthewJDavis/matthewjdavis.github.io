---
title: PowerShell to return users created in the last 31 days
author: Matthew Davis
date: 2018-04-16
excerpt: How to find AD users in a specific OU created in the last 31 days
categories: 
    - powershell
tags:
    - active directory
    - powershell
published: true
---

This will return all the users in the users OU that have bee created in the last 31 days:

```powershell
$createdSinceDate = ((Get-Date).AddDays(-31)).Date
$ou = 'OU=users ,OU=Company,DC=matthewdavis111,DC=com'

Get-ADUser -Filter {whenCreated -ge $createdSinceDate} -Properties whenCreated -SearchBase $ou
```

Use Sort-Object to sort by created date

```powershell
Get-ADUser -Filter {whenCreated -ge $createdSinceDate} -Properties whenCreated -SearchBase $ou | Select-Object userprincipalname, whencreated | Sort-Object whencreated
```

Output to CSV

```powershell
Get-ADUser -Filter {whenCreated -ge $createdSinceDate} -Properties whenCreated -SearchBase $ou | Select-Object userprincipalname, whencreated | Sort-Object whencreated | Export-Csv C:\temp\skip-created-users.csv -NoTypeInformation
```