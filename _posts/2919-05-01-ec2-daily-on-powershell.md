---
title: Starting EC2 instances by tag
author: Matthew Davis
date: 2019-05-01
toc: true
excerpt: Use PowerShell core to start AWS EC2 instance on a daily schedule.
categories:
    - aws
tags:
    - ec2
    - aws
    - powershell
published: false
---




Count number of instances in each AZ
```powershell


(Get-EC2Instance -Filter @(@{name = 'availability-zone'; values = 'eu-west-1a' })).count


# Create a filter object
$filter = New-Object -TypeName Amazon.EC2.Model.Filter
$filter.Name = 'virtualization-type'
$filter.Value = 'hvm'
(Get-EC2Instance -Filter $filter).count

# Multiple filters
(Get-EC2Instance -Filter @(@{name = 'availability-zone'; values = 'eu-west-1b' },@{name ='tag:DailyOn'; values = 'True'})).count

# Search by key
(Get-EC2Instance -Filter @(@{name ='tag-key'; values = 'LeaveOn'})).count

# Wildcard search
(Get-EC2Instance -Filter @(@{name ='reason'; values = 'User*'})).count
(Get-EC2Instance -Filter @(@{name ='reason'; values = '*ser*'})).count

# Search for a key with no value
(Get-EC2Instance -Filter @(@{name ='reason'; values = ''})).count

# Platform type 
(Get-EC2Instance -Filter @(@{name ='platform'; values = 'windows'})).count

# Instance type
(Get-EC2Instance -Filter @(@{name ='instance-type'; values = 't2.medium'})).count
```
