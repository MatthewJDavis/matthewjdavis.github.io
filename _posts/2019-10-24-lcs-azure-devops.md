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
