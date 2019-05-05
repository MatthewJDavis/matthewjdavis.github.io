---
title: Azure Active Directory Domain Services Part 2
author: Matthew Davis
date: 2017-07-27
excerpt: Setting up Azure Active Directory Domain Services in the Portal
categories: 
    - azure
tags:
    - azure
    - active directory domain services
---
July 2017

# Overview

In [part 1], we set up the networking for Azure Active Directory Domain Services. In this part we will implement AADDS using the portal. It was announced on July 2017 that AADDS will be available in the portal in this [Microsoft blog post].

This is the manual bit at the moment (no PowerShell available), so login to the [Azure portal].

Click the "+" sign in the menu and search for "Domain Services" (Make sure you select "Azure AD Domain Services).

![search for domain services](/images/azure-ad-domain-services/az-search-aadds.png)

Click "Create"

![click create](/images/azure-ad-domain-services/aadds-create.png)

Fill in the basic config:

- DNS Name: yourdnsname.com 
- Subscription: same one you created the VNETs in
- Resource Group: Use exsisting or create new, up to you (I'm managing everything in one resource group)
- Location: Same as where you created the VNETs

![fill out basic config](/images/azure-ad-domain-services/aadds-basic-config.png)

Select Virtual Network: Select the Classic VNET created

![select classic vnet](/images/azure-ad-domain-services/aadds-select-vnet.png)

Administrators Group: Click "Add members" to select users or groups to be added to the AAD DC Administrators group

![add members](/images/azure-ad-domain-services/aadds-add-members.png)

View the summary and click "OK" to create the domain.

![summary](/images/azure-ad-domain-services/aadds-summary.png)

You should now see the AADDS as a resource in your resource group

![aadds resource](/images/azure-ad-domain-services/aadds-resource.png)

It will take some time to deploy, it took an hour before mine had finished... time to get a brew.

![aadds deploying](/images/azure-ad-domain-services/aadds-deploying.png)

## Update the DNS Server values of the VNET
Now we need to update the VNET DNS server values so VMs in the VNET can resolve the domain name to be able to join it.

Get the DNS server values from the Portal. 
Click on the AADDS resource and Overview to see the DNS server values:

![aadds dns server addresses](/images/azure-ad-domain-services/aadds-overview.png)

You can update the VNET DNS settings via the Portal or use the PowerShell script below.
The script needs the VNET name, resource group name and two DNS server addresses.
The script will save the current ARM VNET in a variable, update the DNS server addresses then update the VNET

<script src="https://gist.github.com/MatthewJDavis/85b93d15ef222f504524cc4d0aae033f.js"></script>

![dns servers added to vnet](/images/azure-ad-domain-services/aadds-dns-servers.png)

That's it, you now have your Azure Active Directory Domain Services set up and VNETs configured, ready for VMs to be added and joined to the domain. In the next part, we'll set up a management server, allowing you to manage the Directory.

[part 1]: http://matthewdavis111.com/azure/azure-ad-domain-services-1/
[Microsoft blog post]: https://blogs.technet.microsoft.com/enterprisemobility/2017/07/11/new-public-preview-azure-ad-domain-services-admin-ux-in-the-new-azure-portal/
[Azure portal]: https://portal.azure.com