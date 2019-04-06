---
title: Create AWS Backup Vault, Plan & Rule with PowerShell
author: Matthew Davis
date: 2019-04-07
toc: true
excerpt: Overview of managing IAM Users and Roles in AWS using PowerShell
categories:
    - aws
tags:
    - backup
    - aws
    - powershell
published: false
---

# Overview

New AWS backup service

## Prerequistis 

AWS PowerShell installed along with an authenticated user with permissions for AWS backup

## Create a Vault

![AWS backup before vault creation](/images/aws-backup/new-backup-console.png)

<script src="https://gist.github.com/MatthewJDavis/f975b48e1ad41a665d817c50e910658b.js"></script>

![demo vault created](/images/aws-backup/demo-vault.png)

## Create a backup plan

<script src="https://gist.github.com/MatthewJDavis/3a56103be8bfd3bbd62c799625fd79c2.js"></script>


![AWS backup plan](/images/aws-backup/backup-plan.png)

![AWS backup rule](/images/aws-backup/backup-rule.png)


Adds the tag to the snapshot 'created:by:aws:backup:plan', '4-AM-7-Day-Retention' (can be used to run lambdas for tagging up snapshots with the volume names).

```powershell
$RecoveryTags = New-Object -TypeName 'system.collections.generic.dictionary[string,string]'
$RecoveryTags.Add('created:by:aws:backup:plan', '4-AM-7-Day-Retention')
```

Schedule is set by an AWS cron expression:

```powershell
$BackupRule.ScheduleExpression = 'cron(0 4 * * ? *)' # https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html

```

## Assign resources to the rule

<script src="https://gist.github.com/MatthewJDavis/8d37b23e6d8bf4a5c0f9be7a8d9e99bc.js"></script>

![AWS backup resource tag assignment](/images/aws-backup/resource-tag.png)

You can assign resources via their ARN or by a specific tag or tags. To use the tagging method a 'Condition' object must be created that has 3 properties, ConditionKey, ConditionValue and ConditionType as demonstrated in the code below.

```powershell
$BackupCondition = New-Object -TypeName Amazon.Backup.Model.Condition 
$BackupCondition.ConditionKey = 'BackupPolicy'
$BackupCondition.ConditionValue = '4AM-7-Day-Retention'
$BackupCondition.ConditionType = 'STRINGEQUALS'
```

![AWS backup full plan](/images/aws-backup/full-plan.png)