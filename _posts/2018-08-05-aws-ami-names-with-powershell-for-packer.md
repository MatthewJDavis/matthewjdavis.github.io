---
title: Find AMI names with AWS PowerShell for Packer Filter
author: Matthew Davis
date: 2018-08-05
excerpt: How to find the name of an AWS AMI using PowerShell that can be then used for a filter by Packer
categories: 
    - PowerShell
tags:
    - aws
    - powershell
    - packer
published: false
---

# How to find the name of an AWS AMI using PowerShell that can be then used for a filter by Packer

I have been using Packer again and went back through the [packer getting started] documents that use the example with a Ubuntu server AMI for AWS. This works great and got me back up and running with Packer but I needed to do the same using the latest Windows 1803 AMI. The goal was to create a small Windows DNS server using 1803.

The parts I needed to know was how to wild card the name correctly so it would use the Windows 1803 AMI as the AMI to build the image from and the Owner number.

![Example code displayed on Packer website](/images/aws-ami-names-packer/example-packer.png)

I need the **name** and **owner**, but for Windows 1803.

## Using AWS PowerShell to get the correct name and Owner ID

I am using PowerShell core with the AWSPowerShell.NetCore module installed

![PowerShell Core version with AWS PowerShell Net Core module loaded](/images/aws-ami-names-packer/psversion.png)

After authenticating with AWS, I set my default region to EU-West-1

```powershell
Set-DefaultAWSRegion -Region eu-west-1
```

In the past, I have used the [Get-EC2ImageByName Cmdlet] to get Windows AMI details. It returns the Windows AMIs on offer by Amazon but as of today, does not return the 1803 offering. What we can see from the output of this command is the naming convention for Windows servers.

![Get-EC2ImageByName output](/images/aws-ami-names-packer/get-ec2imagebyname.png)

### Get-EC2Image -Filter

To get the information I require, I will use the Get-EC2Image Cmdlet with the Filter parameter.
There are a number of parameters that can be filtered on, the two that made sense to me are the platform and name.

First, the filterPlatform variable is created with the object type of Amazon.EC2.Model.Filter. This has two properties, name and value. In this case, they are:

Name = 'platform'
Value = 'windows'






![](/images/aws-ami-names-packer/.png)
![](/images/aws-ami-names-packer/.png)
![](/images/aws-ami-names-packer/.png)
![](/images/aws-ami-names-packer/.png)





[packer getting started]: https://www.packer.io/intro/getting-started/build-image.html
[Get-EC2ImageByName Cmdlet]:https://docs.aws.amazon.com/powershell/latest/userguide/pstools-ec2-get-amis.html#pstools-ec2-get-ec2imagebyname
[Get-EC2Image Cmdlet]: