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
---

In [part 1] we configured the networking, in [part 2] set up [Azure Active Directory Domain Services] (AADDS) and in [part 3] we setup a management VM. 
In this post we'll join the management VM to the domain to enable DNS and Active Directory management.

## User
To join the domain in the first instance and manage it, you'll need to use an account that is a member of the **AAD DC Administrators** group. A user (or users) were specified when setting up AADDS, to check what users are members of this group you can use AzureRM PowerShell cmdlets:

{% highlight PowerShell %}
(Get-AzureRmADGroup -SearchString AAD).Id.guid | Set-Clipboard
Get-AzureRmADGroupMember -GroupObjectId paste clipboard here
{% endhighlight %}
Or via the Azure Portal

Search for "Directory" and select **Azure Active Directory**

![Azure Active Directory](/images/azure-ad-domain-services/directory.png)

Find the AAD DC Administrators group and select it to see the users and add more users

![admin group in portal](/images/azure-ad-domain-services/aad-dc-admin-group.png)

I've created a user called domain-join which is a member of the AAD DC Administrators group and I'll be using this account to join the domain and manage it for this post. In future I plan to use this user to domain join VMs to the domain via different configuration management software such as Chef and PowerShell DSC for testing.

Start the management VM either through the portal or via PowerShell

{% highlight PowerShell %}
Get-AzureRmVM -Name ds-manag-vm -ResourceGroupName DOMAIN-SERVICES-RG | Start-AzureRmVM
{% endhighlight %}

![start Azure VM with PowerShell](/images/azure-ad-domain-services/start-vm.png)

Connect to the VM via RDP, you can use PowerShell to set the public IP of the VM to your clipboard and open the RDP prompt as before:

{% highlight PowerShell %}
(Get-AzureRmPublicIpAddress -Name domain-services-vm-pip -ResourceGroupName domain-services-rg).PublicIpAddressVersion | Set-Clipboard
mstsc
{% endhighlight %}
Login with the **admin user** that was created with the vm **dsadmin** (or whatever you called it).

Start a PowerShell session as Administrator

You'll need to authenticate to the domain when joining it so use the Get-Credential cmdlet to store the domain credentials in a variable. 

**Note** I've always had success using the **UserPrincipalName** of the user. The samaccountname may not be as expected in Azure AD as outlined at the bottom of this [Microsoft document] on domain joining to Azure AD Domain Services.

{% highlight PowerShell %}
$creds = Get-Credential
Add-Computer -Credential $creds -DomainName yourdomain.com
Restart-Computer -Force
{% endhighlight %}
You should now be able to login with the domain user you used to join the domain with (or any other user who is a member of the AAD DC Administrators)

Open Server Manager (search for Server Manager) if it doesn't open automatically on start and go to **Tools** on the top right.
Select **Active Directory Administrative Centre**

![open active directory admin centre from server manager](/images/azure-ad-domain-services/ad-admin-centre.png)

Click on yourdomain (local) on the left hand side and then AADDC computers. There you should see your management VM.

![management computer in AADDC OU](/images/azure-ad-domain-services/ad-admin-centre-comps.png)

[Azure Active Directory Domain Services]: https://azure.microsoft.com/en-gb/services/active-directory-ds/
[part 1]: http://matthewdavis111.com/azure/azure-ad-domain-services-1/
[part 2]: http://matthewdavis111.com/azure/azure-ad-domain-services-2/
[part 3]: http://matthewdavis111.com/azure/azure-ad-domain-services-3/
[Microsoft Document]: https://docs.microsoft.com/en-us/azure/active-directory-domain-services/active-directory-ds-admin-guide-join-windows-vm