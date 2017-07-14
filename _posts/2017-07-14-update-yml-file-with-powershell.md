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

Amazon release updates to their [Windows AMIs][ami-update] monthly, within 5 business days of Microsoft's [patch Tuesday][ms-update] updates. Making sure you are provisioning the latest patched Windows AMI images from your scripts can be labour intensive, you have to get the latest AMI ID from either the website, AWS console, commandline or via AWS PowerShell, copy it and then update the JSON or YAML cloudformation.

Below is a script I came up with to update the AMI version using PowerShell. The script gets the latest AMI ID from AWS, then replaces it in the specified file (in the example code the file is ec2.yml). This takes out the manual step of updating the AMI ID to the latest version.

<script src="https://gist.github.com/MatthewJDavis/3bdbe9fa8fe4a3657308d0799a92f57a.js"></script>

I will admit, I struggle with regular expressions. I've not used them that much throughout my career, however I find the [RegExr][regexr-site] site really handy when writing and testing.

I've not found documentation, but from researching the AWS AMI IDs seem to follow the pattern: ami-xxxxxxxx where x is a lowercase letter or number. I will raise a support request to find this out.

[ami-update](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/windows-ami-version-history.html)
[ms-update](https://technet.microsoft.com/en-us/security/bulletins.aspx)
[regexr-site](http://regexr.com/)