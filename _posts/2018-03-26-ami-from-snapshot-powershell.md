---
title: Create an AWS AMI from an EC2 snapshot using PowerShell
author: Matthew Davis
date: 2018-03-26
excerpt: 
categories: 
    - PowerShell
tags:
    - AWS
    - powershell
published: false
---
March 2018

```PowerShell
$snapShotid = 'snap-12345678'
$amiName = 'demoAmi'
$deviceName = '/dev/sda1'
$virtType = 'hvm'
$arch = 'x86_64'


# Create a device mapping object for the new ami
$DeviceMapping = New-Object Amazon.EC2.Model.BlockDeviceMapping
$DeviceMapping.DeviceName = $deviceName
$DeviceMapping.Ebs = @{"snapshotid"= $snapShotid}

# Register the ami from the snapshot
Register-EC2Image -Architecture $arch -Name $amiName -RootDeviceName $deviceName -VirtualizationType $virtType -BlockDeviceMapping $DeviceMapping

```