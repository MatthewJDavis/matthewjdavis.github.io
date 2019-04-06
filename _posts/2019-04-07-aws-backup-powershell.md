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

The [AWS Backup] service that was announced in January 2019 in this [AWS blog post] provides a centralised way to manage backups across a number of AWS  services which are currently (April 2019): Amazon EBS volumes, Amazon RDS databases, Amazon DynamoDB tables, Amazon EFS file systems, and AWS Storage Gateway volumes (allows backup of on prem systems via the storage gateway).

```powershell
PSVersion: 6.1.3
PSEdition: Core
Module used: AWSPowerShell.NetCore
Module Version: 3.3.462.0
O/S: Windows 10
```

## Prerequisites

[AWS PowerShell Module] installed and [credentials configured] to allow you to access AWS via PowerShell (and have the appropriate permissions to carry out the commands successfully).

## Cmdlets

As of April 2019, there are 43 PowerShell Cmdlets in the AWSPowerShell.NetCore module and follow the standard PowerShell naming conventions.

```powershell
Get-AWSCmdletName -Service backup | Measure-Object | Select-Object -Property Count
```

![Count of 43 AWS backup cmdlets](/images/aws-backup/back-cmdlets.png)

```powershell
New-BAKBackupPlan
New-BAKBackupSelection
New-BAKBackupVault
```

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


[AWS backup]: https://aws.amazon.com/backup/
[AWS blogpost]: https://aws.amazon.com/blogs/aws/aws-backup-automate-and-centrally-manage-your-backups/
[AWS PowerShell Module]: https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-set-up-windows.html
[Credentials Configured]: https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html