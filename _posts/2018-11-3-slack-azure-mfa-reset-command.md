---
title: Reset Azure MFA settings with a slack command
author: Matthew Davis
date: 2018-11-11
excerpt: How to delegate the resetting of Azure MFA to others via a Slack command without the need for elevating their user account to Global Admin
categories: 
    - powershell
tags:
    - poweshell
    - azure
    - slack
    - mfa
published: flase
---

Currently only users in the [Global Admin] role can reset the Azure MFA details of users which is used for Azure, D365 and Office 365. There has been an [Azure feedback request] with Mircrosoft to open this up to other roles for over 3 years with no movement, it would make sense that the Password/ Helpdesk Administrator role would be able to do this and was a big pain point for us recently, sometimes getting up to 5 requests a day for resets from the helpdesk because we couldn't give them Global Admin rights to our Azure tenant!


[Global Admin]: https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/directory-assign-admin-roles
[Azure feedback request]: https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/10072839-allow-the-user-admin-role-to-enable-disable-mfa-fo