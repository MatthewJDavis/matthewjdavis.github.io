---
title: updating AMI with PowerShell
author: Matthew Davis
date: 2017-07-14
categories: 
    - powershell
tags:
    - powershell
    - aws
    - ami 
---

# Updating AMI IDs in Cloudformation

Amazon release updates to their [Windows AMIs][ami-update] monthly, within 5 days of Microsoft's [patch Tuesday][ms-update] updates.


<script src="https://gist.github.com/MatthewJDavis/3bdbe9fa8fe4a3657308d0799a92f57a.js"></script>

[ami-update](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/windows-ami-version-history.html)
[ms-update](https://technet.microsoft.com/en-us/security/bulletins.aspx)