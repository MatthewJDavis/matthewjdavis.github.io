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
published: true
---

I have been using Packer again and went back through the [packer getting started] documents that use the example with a Ubuntu server AMI for AWS. This works great and got me back up and running with Packer but I needed to do the same using the latest Windows 1803 AMI. The goal was to create a small Windows DNS server using 1803.

I need the **name** and **owner** values for Windows Server 1803 so Packer will use that as the base AMI (I also need to change from ssh and use winrm for configuring the AMI, but that is not in the scope of this post).

![Example code displayed on Packer website](/images/aws-ami-names-packer/example-packer.png)

## Using AWS PowerShell to get the correct name and Owner ID

I am using PowerShell core with the AWSPowerShell.NetCore module installed

![PowerShell Core version with AWS PowerShell Net Core module loaded](/images/aws-ami-names-packer/psversion.png)

After authenticating with AWS, I set my default region to EU-West-1

```powershell
Set-DefaultAWSRegion -Region eu-west-1
```

In the past, I have used the [Get-EC2ImageByName Cmdlet] to get Windows AMI details. It returns the Windows AMIs on offer by Amazon, however as of today the 1803 offering is not returned. What we can see from the output of this command is the naming convention for Windows servers.

![Get-EC2ImageByName output](/images/aws-ami-names-packer/get-ec2imagebyname.png)

### Get-EC2Image -Filter

To get the information I require, I will use the [Get-EC2Image Cmdlet] with the [Filter] parameter.
There are a number of parameters that can be filtered on and those parameters can be found on the [API documentation], the two that made sense to me are the platform and name.

Here's the code:

<script src="https://gist.github.com/MatthewJDavis/29d31954fac1b586f9069d3298450586.js"></script>

First, the filterPlatform variable is created with the object type of Amazon.EC2.Model.Filter. This has two properties, name and value. In this case, they are:

* Name = 'platform'
* Value = 'windows'

*I noticed that the results return the platform as Windows with an uppercase W, however using Windows for the filter does not work, I needed to use windows with a lowercase w, strange.*

Second, another variable is created with the object type of Amazon.EC2.Model.Filter. The same two properties are filled out:

* Name='name'
* Value = '*Windows*1803*Base*'

*This was also case sensitive but this time matches the output displayed.*

Running the [Get-EC2Image Cmdlet] with the two filter objects passed to it, gave me the name and OwnerID that I needed to add to my Packer script and I could also check that the rootdevice and virtualisation type were the same as the example.

![Output using the EC2 filters](/images/aws-ami-names-packer/ec2-filter.png)

Now that I have the name and ownerID to get the latest Window Server 1803 AMI and looking at the packer example for Ubuntu, I changed the script with the new owner number and so the name filter is: 

```json
name: "Windows_Server-1803-English-Core-Base*",
```

This worked, with the most recent property set to true my packer build now uses the latest released Windows Server 1803 AMI.

Here's the work in progress of the packer build for an 1803 AMI to run as a DNS server. It's not finished yet, I still need to work on the sysprep side of things but it's getting there and I have got it building via a Jenkins job, which could be scheduled to run monthly to build the small DNS server from the latest AMI.

<script src="https://gist.github.com/MatthewJDavis/e2bb26bb7a90265e292d18250d231fa7.js"></script>

And the final result from the code above used in a Jenkins build:

![AWS console showing the created AMI](/images/aws-ami-names-packer/created-ami.png)

[packer getting started]: https://www.packer.io/intro/getting-started/build-image.html
[Get-EC2ImageByName Cmdlet]: https://docs.aws.amazon.com/powershell/latest/userguide/pstools-ec2-get-amis.html#pstools-ec2-get-ec2imagebyname
[API documentation]: https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html
[Get-EC2Image Cmdlet]: https://docs.aws.amazon.com/powershell/latest/userguide/pstools-ec2-get-amis.html#pstools-ec2-get-image
[filter]: https://docs.aws.amazon.com/powershell/latest/reference/index.html?page=Get-EC2Image.html&tocid=Get-EC2Image