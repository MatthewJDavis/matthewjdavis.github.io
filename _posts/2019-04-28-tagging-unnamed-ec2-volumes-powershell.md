---
title: Tagging AWS EC2 unnamed EC2 volumes using PowerShell core in a lambda
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
published: false
---

# Overview

Following on from my last post on implementing AWS Backup, we have started to run it in production with a few instances after successful tests of backup in restores in a development environment. I found that some of the volumes that were being backed up did not have any name tags which made identifying the snapshots created by AWS backup time consuming (volumes should be tagged when the provisioning tasks are run but there were quite a few old instances in the account that for whatever reasons had volumes attached that had no name tag).
I created a lambda using PowerShell core to tag all the volumes in the account that are attached to an EC2 instance to make sure all volumes have a name and the lambda can be run periodically just in case an instance is created and the volume does not have a name tag applied.

This lambda was created on a Mac using PowerShell core and the AWSCore and AWSLambda modules:

![psversiontable output](/images/aws-lambda-volume-tagging/psversiontable.png)

## The code

Below is the complete code for the lambda,

The #requires statement is needed for running on AWS lambda.

The list of volumes is saved into the variable VolumeList and a new list object is created to hold the volumes that don't have a name tag. The list is iterated over to check for a name tag, if it doesn't exist, the instance id and volume id is stored in the NoNameTagVolumeList variable.

The NoNameTagVolumeList is iterated over and name tags are created from either the name tag of the instance the volume the snapshot was created from is attached to, or if the instance does not have a name tag, then the instance ID is used and this is applied to the volume.

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




## Publish the lambda

## Test the lambda

## Summary