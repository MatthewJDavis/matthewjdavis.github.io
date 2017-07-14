---
title: updating AMI with PowerShell
author: Matthew Davis
date: 2017-07-14
header:
  teaser: "images/powershell-profile/profile.png"
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

### You will need:
- [AWS PowerShell][aws-powershell] installed / imported
- [AWS credentials][aws-creds] set up


<script src="https://gist.github.com/MatthewJDavis/3bdbe9fa8fe4a3657308d0799a92f57a.js"></script>

I do struggle with regular expressions. I've not used them that much throughout my career, however I find the [RegExr][regexr-site] site really handy when writing and testing.

{% highlight powershell %} Select-String {% endhighlight %} is a useful PowerShell cmdlet, more info in the PowerShell [online docs][select-string] or by running {% highlight powershell %}Get-Help -Name Select-String -Full{% endhighlight %} in PowerShell.

I've not found documentation, but from researching the AWS AMI IDs seem to follow the pattern: ami-xxxxxxxx where x is a lowercase letter or number. I will raise a support request to find this out.

[ami-update]:http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/windows-ami-version-history.html
[ms-update]:https://technet.microsoft.com/en-us/security/bulletins.aspx
[aws-powershell]:https://aws.amazon.com/powershell/
[aws-creds]:http://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html
[regexr-site]:http://regexr.com/
[select-string]:http://go.microsoft.com/fwlink/?LinkId=821853