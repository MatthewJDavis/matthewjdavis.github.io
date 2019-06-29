---
title: Get EC2 details with PowerShell
author: Matthew Davis
date: 2019-06-26
toc: true
excerpt: Use AWS PowerShell module to get basic EC2 information including the AMI name used to create the instance.
categories:
    - powershell
tags:
    - aws
    - powershell
    - ec2
published: true
---
June 2019

# Overview

Today I needed to go through and check the details of EC2 instances and the [AMI] they were created from for an audit. You can get this information via the AWS console, however this is laborious and I wanted to share the results with others so used PowerShell and the [AWS PowerShell Module] to get the required information and save it to a CSV file.

![AWS console showing ami name](/images/ec2-details-powershell/aws-console-ami.png)

## The Script

<script src="https://gist.github.com/MatthewJDavis/1bfcc34d62a9a57899215dc9f0ce60cc.js"></script>

I use an EC2 filter to get only running instances, then create a new list ($noAgentList) of instances that do *not* have the name like TeamCityAgent (I am not interested in these instances).

Next I create an [ordered hashtable] to store the properties that are required. I use the ami id to get the AMI Name. 

Note: The AMI name may not be available, I found that older instances that used Windows didn't always have a an AMI name and in the console it was unavailable too.

Once the hashtable is created, a custom object is created with the properties made up of the hashtable values and saved to the list variable. This is done for each of the objects in the no agent list.

Below we can see the output in the PowerShell terminal for the same instance in the console screenshot above.

![Custom PowerShell object output with ami name](/images/ec2-details-powershell/ec2-details-posh.png)

For the last step, I sort the instance details by ami image name and save them to a CSV file.

![EC2 instance details saved to csv](/images/ec2-details-powershell/ec2-details-csv-output.png)

## Summary

Working with EC2 instances and outputting the required data takes a bit extra work but once you get the hang of it, you can use it to extract information quickly and easily and build useful reports of the details of the EC2 instances. This script could be modified to get other details such as volume attachment information, snapshot dates and be run across different AWS regions.

[AMI]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html
[AWS PowerShell Module]: https://aws.amazon.com/powershell/
[ordered hashtable]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-6