-- -
title: Create AWS IAM instance profile for the cloudwatch agent with PowerShell.
author: Matthew Davis
date: 2019-07-20
toc: false
excerpt: Use PowerShell to create the required IAM EC2 instance profile for the cloudwatch agent to collect more system level metrics and send them to cloudwatch.
categories:
- aws
tags:
- iam
- aws
- powershell
published: false
---

# Overview

When I first started learning about AWS, to get an EC2 instance's RAM metrics into cloudwatch logs and be able to create a graph on them required you to download and run some pearl scripts. I had a requirement to do this again this week at work and while looking for the scripts I came across that AWS had released the [CloudWatch agent].

The instance needs permission to be able to send logs to CloudWatch and this is achieved in AWS by creating an [instance profile] and attaching it to the instance the agent will be installed on.

AWS has a predefined policy called 'CloudWatchAgentServerPolicy' with the correct access rights, so this post will show how to create a instance profile

## Summary

[CloudWatch agent]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html
[instance profile]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#ec2-instance-profile