---
title: Create AWS EC2 instance with custom volume size
author: Matthew Davis
date: 2019-03-02
toc: true
excerpt: Create an AWS EC2 instance with a volume size instead of the default size and tag with a Name and Description tag.
categories:
    - aws
tags:
    - ec2
    - aws
    - powershell
published: false
---

# Overview

There are plenty of examples on how to create a basic EC2 instance with PowerShell in AWS, the built in help from the AWS module gives a perfect example on how to quickly create one. However creating an EC2 instance this way means you get the default volume size of the specified AMI, usually 8GB for Linux or 30GB for Windows and the volume created is not tagged.

I like to provision a larger volume for an O/S drive (and will also provision one or more volumes to use as data drive(s)) so below is a guide on how to specify the size of the volume to be used for the EC2 O/S drive and also tags the volume.

Here's the full function if you just want to see how it's done (don't forget to update the params or specify them when running the function) and I'll go through it below.
 
Code below is saved as a script module called EC2CustomVolSiz.psm1.

```powershell
# Script Module
# Create a new EC2 Instance that has a volume size that can be specified

function New-MDEC2Instance {
    [CmdletBinding()]
    [OutputType('PSCustomObject')]
    param (
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $ImageId = 'ami-09226bf1895277020',
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $KeyName = 'Dev.Key',
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $SecurityGroupId = 'sg-9a60f5e5', # All local network SG
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $AvailabilityZone = 'eu-west-1a',
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $SubnetId = 'subnet-7167f77d',
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $InstanceType = 't2.micro',
        # Volume Size in GB of OS drive
        [Parameter(Mandatory = $false)]
        [string]
        $VolumeSize = '50',
        # Parameter help description
        [Parameter(Mandatory = $false)]
        [String]
        $InstanceName = 'DemoInstance'
    )

    begin {
        Import-Module -Name AWSPowerShell.NetCore -Force
        #Tag helper function
        Function New-MDEC2Tag ($Key, $Value) {
            $tag = New-Object -TypeName Amazon.EC2.Model.Tag
            $tag.Key = $key
            $tag.Value = $value

            return $tag
        }
    }

    process {

        # Set volume size and block mappings of OS drive
        $blockDeviceMapping = New-Object -TypeName Amazon.EC2.Model.BlockDeviceMapping
        $ebsBlockDevice = New-Object -TypeName Amazon.EC2.Model.EbsBlockDevice
        $blockDeviceMapping.DeviceName = '/dev/sda1'
        $ebsBlockDevice.VolumeSize = $VolumeSize
        $ebsBlockDevice.VolumeType = 'standard'
        $blockDeviceMapping.Ebs = $ebsBlockDevice

        # Create the tags
        # EC2
        $InstanceNameTag = New-MDEC2Tag -key 'Name' -value $InstanceName
        # OS Volume
        $OSVolName = New-MDEC2Tag -key 'Name' -value "$InstanceName-os-volume"
        $OSVolDesc = New-MDEC2Tag -key 'Description' -value "$InstanceName OS Volume"



        $params = @{
            'ImageId'          = $ImageId
            'KeyName'          = $KeyName
            'SecurityGroupId'  = $SecurityGroupId
            'AvailabilityZone' = $AvailabilityZone
            'SubnetId'         = $SubnetId
            'InstanceType'     = $InstanceType
            'BlockDeviceMapping' = $blockDeviceMapping
        }

        $ec2Instance = New-EC2Instance @params

        # Tag the EC2 Instance
        New-EC2Tag -Resource $ec2Instance.Instances.InstanceId -Tag $InstanceNameTag

        # Tag the OS volume
        $osVolume = Get-EC2Volume -Filter @{ Name='attachment.instance-id' ; Values="$($ec2Instance.Instances.InstanceId)"}
        New-EC2Tag -Resource $osVolume.VolumeId -Tag $OSVolName
        New-EC2Tag -Resource $osVolume.VolumeId -Tag $OSVolDesc

        # Output a custom object with the Instance details
        [PSCustomObject]@{
            'Name' = $InstanceName
            'PrivateIP' = $ec2Instance.Instances.PrivateIpAddress
            'InstanceId' = $ec2Instance.Instances.InstanceId
        }
    }

    end {
    }
}
```

```powershell
# Import the script module then run
Import-Module .\EC2CustomVolSize.psm1

New-MDEC2Instance -InstanceName 'MyInstance' -VolumeSize '75'

```

The above code create and EC2 instance called MyInstance with a Volume Size of 75GB using the default param values in the script.