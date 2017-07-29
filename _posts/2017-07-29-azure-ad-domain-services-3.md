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

In part 1 we configured the networking and in part 2 set up Azure Active Directroy Domain Serices, a managed domain controller for Microsoft Azure.

Microsoft take care of the infrastructure running the domain controller so you can concentrate on managing the domain. To do this, you'll need a domain joined VM with the management tools installed. This post will
1. Provision a Windows 2016 VM
2. Install the following tools for domain management
    - Remote Server Administration Tools
    - AD PowerShell
    - DNS Server Tools
3. Domain Join the VM