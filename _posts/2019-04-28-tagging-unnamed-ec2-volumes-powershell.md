---
title: Tagging AWS EC2 unnamed EC2 volumes with a PowerShell Core lambda
author: Matthew Davis
date: 2019-04-28
toc: true
excerpt: This post will cover how to use PowerShell core running in a lambda to tag unnamed EC2 volumes.
categories:
    - aws
tags:
    - backup
    - aws
    - powershell
    - lambda
published: true
---
April 2019

# Overview

Following on from my [last post] on implementing AWS Backup, we have started to run it in production with a few instances after successful tests of backup and restores in a development environment. Some of the volumes that were being backed up did not have any name tags which made identifying the snapshots created by AWS backup time consuming (volumes should be tagged when the provisioning tasks are run but there were quite a few old instances in the account that had volumes attached with no name tag).
I created a lambda using PowerShell Core (Windows PowerShell is not supported) to tag all the volumes in the account that are attached to an EC2 instance to make sure all volumes have a name. The lambda can be run periodically just in case an instance is created and the volume does not have a name tag applied.

This lambda was created on a Mac using PowerShell core and the AWSCore and AWSLambda modules:

```powershell
PSVersion                      6.1.1
PSEdition                      Core
OS                             Darwin 18.5.0
Platform                       Unix
AWSLambdaPSCore                1.2.0.0
AWSPowerShell.NetCore          3.3.485.0  
```

## The code

Full code can be found in my PowerShell [Github repo]. See the AWS post on [Setting up a PowerShell Development Environment].

Below is the complete code for the lambda,

The #requires statement is needed for any modules that will be used by the lambda. For this one it is just the AWS module, but you can include any modules that are PowerShell core compatible.

The list of volumes is saved into the variable VolumeList and a new list object is created to hold the volumes that don't have a name tag. The VolumeList variable is iterated over to check for a name tag, if it doesn't exist, the instance id and volume id is stored in the NoNameTagVolumeList variable.

The NoNameTagVolumeList is iterated over and name tags are created from either the name tag of the instance the volume the snapshot was created from is attached to, or if the instance does not have a name tag, then the instance ID is used and this is applied to the volume.

### TagVolumesWithNoName.ps1

<script src="https://gist.github.com/MatthewJDavis/f71fbab9c17448434c4e7c536f53a096.js"></script>

## Lambda permissions

The lambda needs to the ability to be able to tag the volumes.

A policy is created with the permission to apply tags.

<script src="https://gist.github.com/MatthewJDavis/5d65e61e92af74d7c4683585eb64eb51.js"></script>

A trust policy is also required.

<script src="https://gist.github.com/MatthewJDavis/bda438ab8ab5bceb93d8b8b8223ce446.js"></script>

With the two policies, a role can be created for the lambda to use that will allow it to tag EC2 resources and also log what actions have taken place.
The code below uses Get-Content and the two json files above to create the policy and role in the AWS account (the account running the commands but have permission to create IAM policies and roles).

<script src="https://gist.github.com/MatthewJDavis/5a8f9c79e34ec5abfd736bc3391cac99.js"></script>

## Creating the lamdba

```powershell
New-AWSPowerShellLambda -ScriptName 'TagVolumesWithNoName' -Template Basic
```

After running the command the directory structure looks like:

![aws modules](/images/aws-lambda-volume-tagging/dir-structure-after-create.png)

The lambda code should be placed in the TagVolumesWithNoName/TagVolumesWithNoName.ps1 script. This file has comments and template code when created and the readme.txt has more information and can be deleted once it has been read.

Copy the code from above TagVolumesWithNoName.ps1 into the newly created file TagVolumesWithNoName.ps1 and save it.
The lambda is now ready to be published.

## Publish the lambda

```powershell
Publish-AWSPowerShellLambda -ScriptPath .\TagVolumesWithNoName\TagVolumesWithNoName.ps1 -Name TagVolumesWithNoName -Region 'eu-west-1'
```

As the lambda is published, you will be prompted to select an iam role. Enter the number of the IAM role that was created above.

![Select iam role for the lambda](/images/aws-lambda-volume-tagging/select-iam-role.png)

I did run into errors publishing a couple of times and had to restart my PowerShell Core session and try again.

Once the lambda has been published, you will see it in the AWS console.

![Lambda published to the aws console](/images/aws-lambda-volume-tagging/lambda-console.png)

## Test the lambda

Select the lambda that has been published from the lambda section in the AWS console.

To test the lambda from the console:

- Select configure test events from the drop down

![Create a new lambda event](/images/aws-lambda-volume-tagging/new-lambda-event.png)

- On the configure test event screen, give the event a name such as test and click save.

![Configure the event](/images/aws-lambda-volume-tagging/configure-test-lambda-event.png)

- You will be taken back to the main lambda view, click the test button to run the lambda.

![Test button](/images/aws-lambda-volume-tagging/test-button.png)

- The lambda will run and tag any volumes that don't have a name tag.

The below example volume did not have a name tag and the instance name tag value was 'Matt-test-ubuntu-splunk'

![Volume with no name](/images/aws-lambda-volume-tagging/vol-no-name.png)

After the lambda ran, the volume was tagged with the instance name.

![Volume tagged by the lambda](/images/aws-lambda-volume-tagging/vol-tagged-by-lambda.png)

## Summary

Volumes should be tagged appropriately on provisioning of an EC2 instance, however sometimes they are not and currently to name the root ebs volume using cloudformation, requires a [hacky workaround]. Using PowerShell core running on a lambda is a good option to make sure volumes are tagged with a name tag which can be created as from the attached instance. This makes managing volumes easier and also helps identify snapshot that are created from these volumes. This lambda could be run on a schedule or started on another event from [cloudwatch event].

[last post]: https://matthewdavis111.com/aws/aws-backup-powershell/
[Github repo]: https://github.com/MatthewJDavis/PowerShell/tree/master/AWS/lambda-for-tagging-volumes
[Setting up a PowerShell Development Environment]: https://docs.aws.amazon.com/lambda/latest/dg/lambda-powershell-setup-dev-environment.html
[hacky workaround]: https://serverfault.com/questions/876942/create-new-ec2-instance-with-existing-ebs-volume-as-root-device-using-cloudforma
[cloudwatch event]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/WhatIsCloudWatchEvents.html