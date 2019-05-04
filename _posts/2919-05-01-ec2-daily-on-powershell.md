---
title: Starting EC2 instances by tag & EC2 Filters
author: Matthew Davis
date: 2019-05-01
toc: true
excerpt: Use PowerShell core to start AWS EC2 instances on a daily schedule and some useful EC2 filters for PowerShell
categories:
    - aws
tags:
    - ec2
    - aws
    - powershell
published: publish
---

## Overview

I had previously written about [shutting down Azure VMs] with a PowerShell script using Azure Automation runbook and recently needed to fix a broken script at work that started certain EC2 instances.
The broken script was storing all of the EC2 instances in a variable then filtering by tag, but when I rewrote the script I used the Format parameter of the [Get-EC2Instance] Cmdlet which filters the objects on the AWS side and testing on the dev account showed a large performance gain and also easier to read and understand.
Below the script are a few helpful filters for EC2 Instances, Volumes and Snapshots.

All examples run on PowerShell Core installed on a Mac with the AWSPowerShell Core module:

```powershell
PSVersion                      6.1.1
PSEdition                      Core
OS                             Darwin 18.5.0
Platform                       Unix
AWSLambdaPSCore                1.2.0.0
AWSPowerShell.NetCore          3.3.485.0  
```

## Daily On script

The script has one parameter, the AWS region (set to eu-west-1 as the default as this is where the majority of the instances are).

Instances the are required to start up in the morning are tagged with the key pair: DailyOn = True.

![instance tagged with daily on](/images/aws-daily-on/daily-on-tag.png)

Using the AWS filter to get instances with the specified tags, instances are added to the ec2List variable.
The instances in the list are iterated over and any that are not in the state of running are started. Instances that are already running are written to the output (it's currently run in TeamCity and the output appears in the 'build' log).
That's it, nice and simple and wouldn't take much changing to shutdown instances too (it could shutdown all instances except those with a LeaveOn = True tag).

<script src="https://gist.github.com/MatthewJDavis/ed1f0a99c933bfa28ffbea49d2c6023c.js"></script>

![instance tagged with daily on](/images/aws-daily-on/start-daily-on-output.png)

## Using AWS EC2 Filters

As I wrote this script, I took a look again at the EC2 filter parameter.

The filter can be defined in two ways, as Filter object type or supplied directly to the parameter as hashtable or array of hashtables.

**Filter Name and Value values are case sensitive!!**

### EC2 Filter Object

```powershell
$filter = New-Object -TypeName Amazon.EC2.Model.Filter
$filter.Name = 'virtualization-type'
$filter.Value = 'hvm'
(Get-EC2Instance -Filter $filter).count
```

![instance tagged with daily on](/images/aws-daily-on/filter-object.png)

### Hashtable filter

```powershell
(Get-EC2Instance -Filter @{name = 'virtualization-type'; values = 'hvm' }).count
```



### Array of Hashtables = multiple filter

```powershell
# Multiple filters
(Get-EC2Instance -Filter @(@{name = 'availability-zone'; values = 'eu-west-1b' } ; @{name ='tag:DailyOn'; values = 'True'})).count
```

![instance tagged with daily on](/images/aws-daily-on/multi-filter.png)

## A useful example filters

### EC2 Instances

### Count number of instances in each AZ

```powershell
(Get-EC2Instance -Filter @{name = 'availability-zone'; values = 'eu-west-1a' }).count
```

![instance tagged with daily on](/images/aws-daily-on/az-count.png)

### Search by key

```powershell
(Get-EC2Instance -Filter @{name ='tag-key'; values = 'LeaveOn'}).count
```

![instance tagged with daily on](/images/aws-daily-on/tag-key.png)

### Wildcard search

```powershell
(Get-EC2Instance -Filter @{name ='reason'; values = 'User*'}).count
(Get-EC2Instance -Filter @{name ='reason'; values = '*ser*'}).count
```

![filter by reason](/images/aws-daily-on/filter-reason.png)

### Search for a key with no value

```powershell
(Get-EC2Instance -Filter @{name ='reason'; values = ''}).count
```

![filter by reason](/images/aws-daily-on/filter-reason-blank.png)

### Platform type

```powershell
(Get-EC2Instance -Filter @{name ='platform'; values = 'windows'}).count
```

![filter by platform](/images/aws-daily-on/filter-platform.png)

### Instance type

```powershell
(Get-EC2Instance -Filter @{name ='instance-type'; values = 't2.medium'}).count
(Get-EC2Instance -Filter @{name ='instance-type'; values = '*'}).count
```

![ec2 filtered by type](/images/aws-daily-on/filter-instance-type.png)

### EC2 Volumes

```powershell
# Get vol mounted at sda1 for an instance (normally O/S drive)
Get-EC2Volume -Filter @(@{ Name = 'attachment.instance-id' ; Values = "$($ec2Instance.Instances.instanceid)"} ;
    @{Name = 'attachment.device' ; values = '/dev/sda1'})

Get-EC2Volume -Filter @( @{ Name = 'attachment.instance-id' ; Values = "$($ec2Instance.Instances.instanceid)"} ; 
    @{Name = 'attachment.device' ; values = '/dev/sda2'})
```

![volume filtered by attachment](/images/aws-daily-on/attachment.png)


### EC2 Images

```powershell
# https://docs.aws.amazon.com/powershell/latest/reference/items/Get-EC2Image.html
# OwnerID '801119661308' = Amazon Windows
# OwnerID '099720109477' = Canonical
# OwnerID '137112412989' = Amazon linux

# Filter by image id - handy to get owner info, name details etc
Get-EC2Image -Filter @{ Name="image-id"; Values="ami-01ac8cd0e2853e2be" }

# Windows
#2019 Full
Get-EC2Image -Filter @(@{ Name="platform"; Values="windows" } ; @{Name='owner-id' ; Values = '801119661308'} ; @{Name='name'; Values = 'Windows_Server-2019-English-Full-Base-*'})

#2019 Core
Get-EC2Image -Filter @(@{ Name="platform"; Values="windows" } ; @{Name='owner-id' ; Values = '801119661308'} ; @{Name='name'; Values = 'Windows_Server-2019-English-Core-Base-*'})

# Ubuntu

Get-EC2Image -Filter @(@{Name='owner-id' ; Values = '099720109477'} ; @{Name='name'; Values = '*ubuntu-bionic-18.04-amd64-server-*'})

Get-EC2Image -Filter @(@{Name='owner-id' ; Values = '099720109477'} ; @{Name='name'; Values = '*ubuntu-xenial-16.04-amd64-server-*'})

# Amazon
Get-EC2Image -Filter @(@{Name='owner-id' ; Values = '137112412989'} ; @{Name='name'; Values = '*amzn2-ami-hvm-2.0.2019*'})
```

### Time difference between a filter and getting all instances (only 68 instances in this case)

![instance tagged with daily on](/images/aws-daily-on/filter-vs-all-instances.png)

## Summary

Updating the script to turn on EC2 instances was a fun exercise that didn't take too long. The script is basic but does the job well.
The AWS filters are handy to use to speed up the filtering of EC2 objects and very useful to find volumes and images. Before I would get all instances / volumes / images from AWS and use PowerShell to sort and filter etc but now I've learnt more about filtering and how filtering at the API makes manipulating with PowerShell quicker due to less objects being sent in the pipeline and to sort etc.

[shutting down Azure VMs]: https://matthewdavis111.com/azure/azure-auto-stop-vm-with-tag/
[Get-EC2Instance]: https://docs.aws.amazon.com/powershell/latest/reference/