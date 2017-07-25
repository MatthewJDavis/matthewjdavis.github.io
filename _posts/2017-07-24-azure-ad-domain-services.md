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

## Why
I've been using Desired State Configuration (DSC) to manage the configuration of VMs and one of the most commonly used task is joining a VM to a domain. I wanted to practice and evaluate different ways to automate the process including using [Azure Automation] and [Chef] for configuration management which I've been implementing recently. 
I could have configured a VM to be a domain controller like I've done in the past but thought this would be a great opportunity to see what AADDS offered functionality wise for a domain controller and to see if it would met my needs as a testing platform for configuration management in a domain environment.

## Focus on automation - slight problems
I wanted to script all / as much of the process as possible so I can keep it in source control allowing reuse for other projects and for re-deploying if necessary but hit two slight problems with this:
1. Deploying AADDS
I looked through the Azure PowerShell commands to see if there are any for AADDS and as of July 2017, you can't create this via PowerShell or with an ARM template so this step has to be done via the portal.
2. Retrieving the DNS server values for AADDS
Once AADDS is created you need the two DNS server values to set in your VNET(s) so the VMs will be able to resolve the domain, I couldn't find a way to access the information programmatically. I couldn't even get them from the [Azure Resource Explorer].

## Classic VNET required
As of July 2017 you have to deploy Azure Active Directory Domain Services into a classic VNET. This method uses the old Azure APIs ([Azure Service Management]) to interact with Azure and is no longer the recommended way to deploy resources unless the services is not supported by the Azure Resource Manager deployment method. See the [deployment docs] for more information.

## Planning the VNETs
As always, it is wise to plan your VNETs and subnets before deployment. Using [VNET peering] it's possible to use only one Classic VNET for AADDS and then peer that to ARM VNETs so we can deploy and manage ARM resources as normal.
![vnet layout](/images/azure-ad-domain-services/vnet.png)

Classic VNET
- Address range: 10.0.0.0/16
- Subnet: 10.0.0.0/24

ARM VNET
- Address range: 10.1.0.0/16
- Subnet 1: 10.1.0.0/28 (Gateway subnet to peer with classic VNET)
- Subnet 2: 10.2.0.0/24
- Subnet 3: 10.3.0.0/24

## Creating the ARM VNET

Here we will setup the Resource Group to contain all the resources and while logged into AzureRM, create the VNET and subnets to.
1. Login to AzureRM
{% highlight powershell %}
Add-AzureRmAccount
{% endhighlight %}
2. Select correct AzureRM subscription (if you need to)
{% highlight powershell %}
Select-AzureRmSubscription -SubscriptionName
{% endhighlight %}
3. Run the following (adjust the names / location / addresses as required)
<script src="https://gist.github.com/MatthewJDavis/6fad491b929572afd4c01170eb888242.js"></script>

![ARM vnet with subnets](/images/azure-ad-domain-services/az-portal-vnet.png)

## Classic VNET deployment
Now it's time to deploy the classic VNET, this means authenticating against the ASM API, using the Azure PowerShell module (not the AzureRM module).

1. Login to Azure
{% highlight powershell %}
Add-AzureAccount
{% endhighlight %}
2. Select correct Azure subscription (if you need to)
{% highlight powershell %}
Select-AzureSubscription -SubscriptionName
{% endhighlight %}

To deploy the classic VNET, you need to have the config in XML format like below:

<script src="https://gist.github.com/MatthewJDavis/d515c177fec80ed2297f5a241c155953.js"></script>

If you look at the VirtualNetworkSite element, you will see the resource group name with the VNET name, this will place it in the Resource group created. 
Save this file to the local computer and then run the following Azure PowerShell command with the full path to the file:
{% highlight powershell %}
Set-AzureVNetConfig -ConfigurationPath C:\temp\classic-network-config.xml
{% endhighlight %}

You should now see the classic VNET in the resource group with the ARM VNET in the portal.
![Classic vnet in ARM resource group](/images/azure-ad-domain-services/az-portal-classic.png)


## Setting up VNET Peering


[Azure Active Directory Domain Services]: https://azure.microsoft.com/en-gb/services/active-directory-ds/
[Azure Automation]: https://docs.microsoft.com/en-us/azure/automation/automation-intro
[Chef]: https://www.chef.io/
[Azure Resource Explorer]: https://resources.azure.com/
[Azure Service Management]: https://msdn.microsoft.com/en-us/library/azure/ee460799.aspx
[deployment docs]: https://docs.microsoft.com/en-gb/azure/azure-resource-manager/resource-manager-deployment-model
[VNET peering]: https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview