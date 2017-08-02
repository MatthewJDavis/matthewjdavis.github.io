---
title: Azure Active Directory Domain Services Part 4
author: Matthew Davis
date: 2017-08-02
excerpt: Domain joining the management VM
categories: 
    - azure
tags:
    - azure
    - active directory domain services
published: false
---

In [part 1] we configured the networking, in [part 2] set up [Azure Active Directory Domain Services] (AADDS) and in [part 3] we setup a management VM. 
In this post we'll join the management VM to the domain to enable DNS and Active Directory management.

## User
To join the domain in the first instance and manage it, you'll need to use an account that is a member of the **AAD DC Administrators** group. A user (or users) were specified when setting up AADDS, to check what users are members of this group you can use AzureRM PowerShell cmdlets:

```PowerShell
(Get-AzureRmADGroup -SearchString AAD).Id.guid | Set-Clipboard
Get-AzureRmADGroupMember -GroupObjectId *paste clipboard here*
```
Or via the Azure Portal

![admin group in portal](/images/azure-ad-domain-services/aad-dc-admin-group.png)


[Azure Active Directory Domain Services]: https://azure.microsoft.com/en-gb/services/active-directory-ds/
[part 1]: http://matthewdavis111.com/azure/azure-ad-domain-services-1/
[part 2]: http://matthewdavis111.com/azure/azure-ad-domain-services-2/