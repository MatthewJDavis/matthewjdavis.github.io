---
title: Create AWS IAM instance profile for the CloudWatch agent with PowerShell.
author: Matthew Davis
date: 2019-07-20
toc: false
classes: wide
excerpt: Use PowerShell to create the required IAM EC2 instance profile for the CloudWatch agent to collect more system level metrics and send them to CloudWatch.
categories:
    - aws
tags:
    - iam
    - aws
    - powershell
published: true
---
July 2019

# Overview

When I first started learning AWS, to get an EC2 instance's RAM metrics into CloudWatch logs required you to download and run some pearl scripts. I had a requirement to do this again this week at work and while looking for the scripts I came across that AWS had released the [CloudWatch agent].

The instance needs permission to be able to send logs to CloudWatch and this is achieved in AWS by creating an [instance profile] and attaching it to the instance the agent will be installed on as described by the [documentation].

AWS has a predefined policy called 'CloudWatchAgentServerPolicy' with the correct access rights, so this post will show how to create a instance profile that can be applied to an EC2 instance so that it can report back system level metrics including RAM and disk space usage.

## Code

Note: this was done on a Mac running PowerShell Core 6.1.2 and AWS tools version 3.3.485.0 but should work on Windows PowerShell with the AWS module installed too.

Make sure you can authenticate to AWS via PowerShell and that you have the correct permissions to update IAM roles.

<script src="https://gist.github.com/MatthewJDavis/1e1d225e09687044429b76890b85e8d2.js"></script>

A here-string variable is used to hold JSON of the EC2 [trust policy].

The CloudWatchAgentServerPolicy created by AWS is filtered with the `Where-Object` Cmdlet so it can be attached to the new role.

The role is created using the trust policy stored in the here-string variable and the CloudWatchAgentServerPolicy is registered to that role.

Finally, an instance profile is created then the role with the CloudWatchAgentServerPolicy policy is added to it.

Now the role will be available to be attached to an EC2 instance in the account so that they can push their logs to CloudWatch via the CloudWatch agent.

Below is an example CloudWatch dashboard with metrics gathered via the agent.

![CloudWatch dashboard showing agent metrics](/images/aws-iam-profile-powershell/cw-agent-dash.png)


## Summary

I have only just found out about the CloudWatch agent and have installed it on two Ubuntu EC2 instances so far however it does look promising. Using PowerShell to create the instance profile allowed me easily to get this set up in a development environment first for testing then get the same policy attached to the instance profile in the production account.

Instance profiles mitigate the need to store credentials on the EC2 instances to interact with the various APIs and now setting them up with PowerShell makes it easier, more repeatable and the roles can be stored in source control if they need to be deployed to other AWS accounts.


[CloudWatch agent]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html
[documentation]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-CloudWatch-agent-commandline.html
[instance profile]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#ec2-instance-profile
[trust policy]: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html
