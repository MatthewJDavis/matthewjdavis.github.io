---
title: Azure Active Directory Domain Services Part 2
author: Matthew Davis
date: 2017-07-24
excerpt: Setting up Azure Active Directory Domain Services in the Portal
categories: 
    - azure
tags:
    - azure
    - active directory domain services
published: false
---

In [part 1], we set up the networking for Azure Active Directory Domain Services. In this part we will implement AADDS using the portal. It was announced on July 2017 that AADDS will be available in the portal in this [Microsoft blog post].

This is the manual bit at the moment (no PowerShell available), so login to the [Azure portal].

Click the "+" sign in the menu and search for "Domain Services" (Make sure you select "Azure AD Domain Services).
Click "Create"
![](/images/azure-ad-domain-services\az-search-aadds.png)

Fill in the basic config:

DNS Name: yourdnsname.com 
Subscription: same one you created the VNETs in
Resource Group: Use exsisting or create new, up to you (I'm managing everything in one resource group)
Location: Same as where you created the VNETs

Select Virtual Network: Select the Classic VNET created

Administrators Group: Click "Add members" to select users or groups to be added to the AAD DC Administrators group

View the summary and click "OK" to create the domain.



[part 1]: http://matthewdavis111.com/azure/azure-ad-domain-services-1/
[Microsoft blog post]: https://blogs.technet.microsoft.com/enterprisemobility/2017/07/11/new-public-preview-azure-ad-domain-services-admin-ux-in-the-new-azure-portal/
[Azure portal]: https://portal.azure.com