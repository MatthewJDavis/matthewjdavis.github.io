---
title: updating AMI with PowerShell
author: Matthew Davis
date: 2017-07-14
published: false
categories: 
    - powershell
tags:
    - powershell
    - aws
    - ami 
---

# Updating AMI IDs in Cloudformation

Amazon release updates to their [Windows AMIs][ami-update] monthly, within 5 business days of Microsoft's [patch Tuesday][ms-update] updates. Making sure you are provisioning the latest patched Windows AMI images can be labour intensive, you have to get the latest AMI ID from either the website, AWS console, commandline or via AWS PowerShell then update the JSON or YAML cloudformation.


<script src="https://gist.github.com/MatthewJDavis/3bdbe9fa8fe4a3657308d0799a92f57a.js"></script>

[ami-update](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/windows-ami-version-history.html)
[ms-update](https://technet.microsoft.com/en-us/security/bulletins.aspx)