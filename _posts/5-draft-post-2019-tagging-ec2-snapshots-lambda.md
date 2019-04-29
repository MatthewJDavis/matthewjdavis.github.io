---
title: Tagging AWS EC2 snapshots using PowerShell core in a lambda
author: Matthew Davis
date: 2019-04-28
toc: true
excerpt: This post will cover how to use PowerShell core running in a lambda to tag unnamed snapshots.
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

Following on from my last post on implementing AWS Backup, we have started to run it in production with a few instances after successful tests of backup in restores in a development environment. As mentioned in the previous post, AWS backup does not name the snapshots it creates which makes identifying the snapshots harder than it should be.
At first my lambda was just tagging those snapshots created by the AWS lambda backup service but I changed this to tag all volumes that don't have a name because unnamed snapshots are a bit of a mystery to what they contain and take some time investigating which could be better spent elsewhere.

## The code

Below is the complete code for the lambda,

The #requires statement is needed for running on AWS lambda.
The list of volumes is saved into a variable and a new list object is created to hold the volumes that don't have a name tag.
The 

The list of volumes with no name tag is iterated over and name tags are created from either the name tag of the instance the volume the snapshot was created from is attached to, or if the instance does not have a name tag, then the instance ID is used.

```powershell
#Requires -Modules @{ModuleName='AWSPowerShell.NetCore';ModuleVersion='3.3.485.0'}

$VolumeList = Get-EC2Volume

# Get the volumes that do not have a name tag and add them to a list
$NoNameTagVolumeList = New-Object 'collections.generic.list[object]'
```
<script src="https://gist.github.com/MatthewJDavis/f71fbab9c17448434c4e7c536f53a096.js"></script>

## Lambda permissions

## Creating the lamdba

## Publish the lambda

## Test the lambda

## Summary