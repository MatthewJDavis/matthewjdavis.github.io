---
title: How to delete a project in Visual Studio Team Services
author: Matthew Davis
date: 2017-09-20
excerpt: Sometimes you need to delete a project if you've been testing or using it for demos, here's how to.
categories: 
    - vsts
tags:
    - vsts
published: true
---
September 2017

# Overview

I use VSTS to create test projects and demo projects then have a clean up every now and then but sometimes forget how to delete the project, sometime having to ask myself if I delete it from within the project itself or elsewhere? I forget. This quickie will remind me in the future when I've not deleted a project in a while and have forgotten (hopefully the interface won't have changed too drastically then, it's updated on such a fast cycle there's always something new to discover!).

**This action can't be undone, so make sure you have backups / definitely don't mind all of the work being deleted**

You need to been in the main account overview page before clicking on the cog (settings) icon.

[VSTS main account screen](/images/vsts-delete-project/vsts-account-main.png)

If you're in a project this will take you to the settings for the project and not the overall account. To get to the main account page from a project either:

1. Click on the VSTS icon on the top right
2. Click on the **cog** icon and select **Account Settings**

![settings in a VSTS project](/images/vsts-delete-project/vsts-project-settings.png)

On the main screen, select the **cog** icon and select **Overview**

![overview option from cog menu](/images/vsts-delete-project/vsts-account-overview.png)

Hover the mouse over the project, click the ellipsis and select **Delete**

![delete option for project on main screen](/images/vsts-delete-project/vsts-delete-option.png)

A confirmation screen pops up, write the name of the project to confirm deletion.

![confirm deletion of project](/images/vsts-delete-project/vsts-delete-confirm.png)

Goodbye project, easy when you know how!