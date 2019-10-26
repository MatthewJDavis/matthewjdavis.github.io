---
title: Permissions required to deploy to Azure from Dynamics 365 lifecycle services
excerpt: How to set the correct permissions for the service account linking Microsoft Dynamics 365 lifecycle services and Azure DevOps to deploy to Azure.
date: 2019-10-24
toc: false
classes: wide
categories:
- lifecycle services
tags:
- azure
- devops
published: true
---
October 2019

# Overview

Having spent a lot of time recently figuring out the integration between [Microsoft Lifecycle services], [Azure DevOps] and [Azure] one of the main challenges was removing the user account who had set it all up and link it all together and replace it with a service account.

One area that we kept getting stuck on was the deployment of new Lifecyle services (LCS) dev and test environments to the azure subscription, there was an error between LCS and Azure DevOps:

"User 'username@domain.com' does not have administrator level privilege to agent pool 'Default'. Please try again with an account with administrator level privilege or contact support if the issue persists. Follow this link for more information - https://go.microsoft.com/fwlink/?LinklD=817307"

After calls with support and a bit of trial and error the fix was to **give administrator permissions on the default build pool** to the account at the **organisational** level (not the project level - that didn't work). 

This goes against the principal of 'Just Enough Administration' and seems over the top, but having tried giving the user lots of different permissions at the project level, only organisational level ones fixed the issue.

## Setting the correct permission

Steps to grant permissions (note UI will likely be different as they release frequent updates, this is how it's done as of Oct 2019)

* Login to Azure DevOps (formally VSTS) with the organisation owner account (or one that is member of the group set for organisation owner).

* Click on **Organisational Settings** at the bottom left side with the correct organisation selected.

![Azure DevOps organisation settings](/images/lifecycle-services/org-settings.png)

* Select **Agent Pools**, Click on **Default** pool

![Azure DevOps agent pools tab](/images/lifecycle-services/agent-pools.png)


Click on **Security** then the **Add** button

![Azure DevOps agent security tab](/images/lifecycle-services/agent-security.png)

Find the user, change the role to **Administrator** and click **Add**

![Azure DevOps add user as admin to agent pool](/images/lifecycle-services/add-user.png)

The error should now be fixed and you should now be able to deploy to the connected Azure subscription via LCS.

## Summary

This took a while to sort out especially as the organisation was set up with a user as the owner who happened to be away, so this will be changed to a group to prevent this happening again. The permissions required for the seem excessive and hopefully the LCS team will fix this to at least be project specific or I guess the think it behind it could be because you set up a separate organisation just for your LCS work, at the moment this looks to be the best solution.

[Microsoft Lifecycle services]: https://lcs.dynamics.com/Logon/Index
[Azure DevOps]: https://azure.microsoft.com/en-ca/services/devops/
[Azure]: https://azure.microsoft.com/en-us/