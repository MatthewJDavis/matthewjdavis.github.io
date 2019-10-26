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
published: false
---
October 2019

![Azure DevOps project screen](/images/lifecycle-services/project-screen.png)

# Overview

Having spent a lot of time recently figuring out a Microsoft Lifecycle services, Azure DevOps and Azure integration one of the main challenges was removing the user's account who had set it all up and integrate it all together and replacing it with a service account.

One area that we kept getting stuck on was the deployment of new environments to the azure subscription, there was an error between LCS and Azure DevOps 

"The user first.last@domain.com does not have administrator level privilege to agent pool 'Default'......"

After calls with support and a bit of trial and error the fix was to give administrator permissions on the default build pool to the account at the **organisational** level (not the project). This seems far too much permission for the user as the project is specified in the Azure DevOps (still says VSTS) settings in the portal, but having tried giving the user lots of different permissions, only organisational level ones fixed the issue.

## Summary

Steps to grant permissions (not UI will likely be different as they release frequent updates, this is how it's done as of Oct 2019)Login to Azure DevOps (formally VSTS) with the organisation owner account. 

Click on **Organisational Settings** at the bottom left side with the correct organisation selected

![Azure DevOps organisation settings](/images/lifecycle-services/org-settings.png)


Select permissions

![Azure DevOps settings screen](/images/lifecycle-services/org-settings1.png)


Under Security , click Permissions, Project Collection Build Administrators

![Azure DevOps organisation permissions](/images/lifecycle-services/org-permissions.png)

Search for the service account and click add

![Azure DevOps project collection build administrators](/images/lifecycle-services/pcba-add.png)

Click on **Default** pool

![Azure DevOps agent pools tab](/images/lifecycle-services/agent-pools.png)

Click on **Security** then the **Add** button

![Azure DevOps agent security tab](/images/lifecycle-services/agent-security.png)

Find the user, change the role to **Administrator** and click **Add**

![Azure DevOps add user as admin to agent pool](/images/lifecycle-services/add-user.png)

You should now be able to deploy to the connected Azure subscription via LCS.

