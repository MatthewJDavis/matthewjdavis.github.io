---
title: Azure Active Directory Domain Services Part 3
author: Matthew Davis
date: 2017-07-29
excerpt: Creating a VM to manage Azure Active Directory Domain Services
categories: 
    - azure
tags:
    - azure
    - active directory domain services
---

In [part 1] we configured the networking and in [part 2] set up [Azure Active Directory Domain Services] (AADDS), a managed domain controller for Microsoft Azure.

Microsoft take care of the infrastructure running the domain controller so you can concentrate on managing the domain. To do this, you'll need a domain joined Virtual Machine (VM) with the [management tools] installed. 

1. Provision a Windows 2016 VM
2. Install the following tools for domain management
    - Remote Server Administration Tools
    - AD PowerShell
    - DNS Server Tools
3. Domain Join the VM

The VM needs to be placed within one of the subnets create in the ARM VNET in a subnet that is **not** the gateway subnet.

Provision the VM either through the portal, via PowerShell or you can deploy via the ARM templates I've created and are stored on Github:
- [Azure ARM deployment Template]
- [Azure ARM deployment Parameters]



[Azure Active Directory Domain Services]: https://azure.microsoft.com/en-gb/services/active-directory-ds/
[part 1]: https://matthewdavis111.com/
[part 2]: https://matthewdavis111.com/
[management tools]: https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-admin-guide-administer-domain
[Azure ARM deployment Template]: https://github.com/MatthewJDavis/Azure/blob/master/Domain-Services/vm-templates/windows-management-vm/azuredeploy.json
[Azure ARM deployment Parameters]: https://github.com/MatthewJDavis/Azure/blob/master/Domain-Services/vm-templates/windows-management-vm/azuredeploy.parameters.json
