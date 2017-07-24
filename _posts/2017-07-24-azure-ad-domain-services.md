---
title: Azure Active Directory Domain Services Part 1
author: Matthew Davis
date: 2017-07-24
excerpt: VNET setup for Azure Active Directory Domain Services
categories: 
    - azure
tags:
    - azure
    - active directory domain services
    - azure resource manager
    - azure vnets
---

In the next few posts I'm going to go through the process of setting up [Azure Active Directory Domain Services] (AADDS).  This service is a managed domain controller in Azure, removing the overhead of configuring and maintain a virtual machine (VM) to act as a domain controller for your Azure VMs. This post will focus on setting up the networking requirements to implement AADDS.

**Note**: this will incur costs, the AADDS has been costing around £3 a day of my Azure credit. Then there's the cost of the management VM and the associated costs with it (public IP, storage etc).

##Why
I've been using Desired State Configuration (DSC) to manage the configuration of VMs and one of the most commonly used task is joining a VM to a domain. I wanted to practice and evaluate different ways to automate the process including using [Azure Automation] and [Chef] for configuration management which I've been implementing recently. 
I could have configured a VM to be a domain controller like I've done in the past but thought this would be a great opportunity to see what AADDS offered functionality wise for a domain controller and to see if it would met my needs as a testing platform for configuration management in a domain environment.

##Focus on automation - slight problems
I wanted to script all / as much of the process as possible so I can keep it in source control allowing reuse for other projects and for re-deploying if necessary but hit two slight problems with this:
1. Deploying AADDS
I looked through the Azure PowerShell commands to see if there are any for AADDS and as of July 2017, you can't create this via PowerShell or with an ARM template so this step has to be done via the portal.
2. Retrieving the DNS server values for AADDS
Once AADDS is created you need the two DNS server values to set in your VNET(s) so the VMs will be able to resolve the domain, I couldn't find a way to access the information programmatically. I couldn't even get them from the [Azure Resource Explorer].

##Classic VNET required
As of July 2017 you have to deploy Azure Active Directory Domain Services into a classic VNET. This method uses the old Azure APIs ([Azure Service Management]) to interact with Azure and is no longer the recommended way to deploy resources unless the services is not supported by the Azure Resource Manager deployment method. See the [deployment docs] for more information.

##Planning the VNETs
As always, it is wise to plan your VNETs and subnets before deployment. Using VNET peering it's possible to use only one Classic VNET for AADDS and then peer that to ARM VNETs so we can deploy and manage ARM resources as normal.
![vnet layout](/images/azure-ad-domain-services/vnet.png)


[Azure Active Directory Domain Services]: https://azure.microsoft.com/en-gb/services/active-directory-ds/
[Azure Automation]: https://docs.microsoft.com/en-us/azure/automation/automation-intro
[Chef]: https://www.chef.io/
[Azure Resource Explorer]: https://resources.azure.com/
[Azure Service Management]: https://msdn.microsoft.com/en-us/library/azure/ee460799.aspx
[deployment docs]: https://docs.microsoft.com/en-gb/azure/azure-resource-manager/resource-manager-deployment-model