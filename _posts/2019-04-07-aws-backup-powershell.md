---
title: Create AWS Backup Vault, Plan & Rule with PowerShell
author: Matthew Davis
date: 2019-04-07
toc: true
excerpt: How to use PowerShell to create a backup strategy with AWS backup.
categories:
    - aws
tags:
    - backup
    - aws
    - powershell
published: true
---

# Overview

The [AWS Backup] service announced in January 2019 in this [AWS blog post] provides a centralised way to manage backups across a number of AWS  services which are currently: Amazon EBS volumes, Amazon RDS databases, Amazon DynamoDB tables, Amazon EFS file systems, and AWS Storage Gateway volumes (allows backup of on premises systems via the storage gateway).
As of April 2019, there is not a charge for setting up the backup vaults and plans. You [pay] for the amount of data stored and the amount of data restored.

The examples in this post were run using the following environment:

```powershell
PSVersion: 6.1.3
PSEdition: Core
Module used: AWSPowerShell.NetCore
Module Version: 3.3.462.0
OS: Windows 10
```

## Prerequisites

[AWS PowerShell Module] installed and [credentials configured] to allow you to access AWS via PowerShell (and have the appropriate permissions to the AWS backup service).

## Cmdlets

Currently there are 43 PowerShell Cmdlets in the AWSPowerShell.NetCore module and follow the standard PowerShell naming conventions.

```powershell
Get-AWSCmdletName -Service backup | Measure-Object | Select-Object -Property Count
```

![Count of 43 AWS backup cmdlets](/images/aws-backup/back-cmdlets.png)

For creating resources, there are three commands that start with new:

```powershell
New-BAKBackupPlan
New-BAKBackupSelection
New-BAKBackupVault
```

## Creating a Vault

A vault is a container that the backups are stored in. It is encrypted (this can also be done with keys stored AWS Key management service) and policies can be applied to the vault to control access. There is a vault created called 'default' and you can create new vaults if you wanted to use different encryption keys and set different access policies for certain backups.

Before creating any resources, the console looks like the image below.

![AWS backup before vault creation](/images/aws-backup/new-backup-console.png)

Below is the PowerShell code to create a new vault called demo, with a project tag with the value demo.

<script src="https://gist.github.com/MatthewJDavis/f975b48e1ad41a665d817c50e910658b.js"></script>

Now a vault called 'demo' has been created.

![demo vault created](/images/aws-backup/demo-vault.png)

## Create a backup plan

A backup plan defines when backups take place, retention time and backup lifecycle (you can move old backups to Glacier cold storage or expire them).

Below is how to create a backup plan with PowerShell. The backup is set to run at 4AM daily and keeps backups for 7 days.

<script src="https://gist.github.com/MatthewJDavis/3a56103be8bfd3bbd62c799625fd79c2.js"></script>

![AWS backup plan](/images/aws-backup/backup-plan.png)

![AWS backup rule](/images/aws-backup/backup-rule.png)

A [Lifecycle object] needs to be created to set how long the backups should be kept for and if they are moved to cold storage.  If you wanted to move the backup to cold storage, you would set the MoveToColdStorageAfterDays property.

```powershell
$BackupLifeCycle = New-Object -TypeName Amazon.Backup.Model.Lifecycle
$BackupLifeCycle.DeleteAfterDays = 7
# $BackupLifeCycle.MoveToColdStorageAfterDays = 0 no cold storage
```

Tagging is important to help organise and identify resources, the code below adds the tag to the snapshot 'created:by:aws:backup:plan', '4-AM-7-Day-Retention'.

```powershell
$RecoveryTags = New-Object -TypeName 'system.collections.generic.dictionary[string,string]'
$RecoveryTags.Add('created:by:aws:backup:plan', '4-AM-7-Day-Retention')
```

The backup schedule is set by an [AWS cron expression] and is set to run with 60 minutes of the designated start time:

```powershell
$BackupRule.ScheduleExpression = 'cron(0 4 * * ? *)'
$BackupRule.StartWindowMinutes = 60
```

## Assign resources to the rule

<script src="https://gist.github.com/MatthewJDavis/8d37b23e6d8bf4a5c0f9be7a8d9e99bc.js"></script>

![AWS backup resource tag assignment](/images/aws-backup/resource-tag.png)

You can assign resources via their ARN or by a specific tag or tags. To use the tagging method a [Condition object] must be created that has 3 properties, ConditionKey, ConditionValue and ConditionType as demonstrated in the code below.

```powershell
$BackupCondition = New-Object -TypeName Amazon.Backup.Model.Condition
$BackupCondition.ConditionKey = 'BackupPolicy'
$BackupCondition.ConditionValue = '4AM-7-Day-Retention'
$BackupCondition.ConditionType = 'STRINGEQUALS'
```

Now that the vault and plan have been created and resources assigned, any resources that match the value of the tag specified will be backed up at 4AM daily and have the backups kept for 7 days. I did notice that for EC2 instances, the volumes had to be tagged and it wouldn't work with just tagging the instance.

The fully created plan in the console can be seen below.

![AWS backup full plan](/images/aws-backup/full-plan.png)

## Snapshot naming

At the present, the snapshots are not named when they are created. I am working on a lambda to run through the account, find any snapshots that don't have a name and then create a tag containing a name of the volume they were created from. The idea is that the lambda will check all of the snapshots created by AWS backup using the custom Key I assign to the snapshot: 'created:by:aws:backup:plan'. It will then check to see if it has name, if not it will look up the volume it was created from and create a tag from the volume name to make identifying the snapshot easier.

## Summary

AWS backup is a relatively new service that lets you centrally manage backups of AWS resources in one location. It still needs work doing to it such as naming the created snapshots but it's a solid start and you can work around some of the limitations / annoyances of it with PowerShell and lambdas.
Managing the backup policies and rules with PowerShell is straight forward once you read through the docs and understand the objects that are required to create the backup plans and I'm planning on creating a function in the future to standardise the creation of backup plans and assigning resources for when it is used in production.

[AWS backup]: https://aws.amazon.com/backup/
[pay]: https://aws.amazon.com/backup/pricing/
[AWS blog post]: https://aws.amazon.com/blogs/aws/aws-backup-automate-and-centrally-manage-your-backups/
[AWS PowerShell Module]: https://docs.aws.amazon.com/powershell/latest/userguide/pstools-getting-set-up-windows.html
[Credentials Configured]: https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html
[Lifecycle object]:  https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/Backup/TLifecycle.html
[Condition object]: https://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=Backup/TBackupCondition.html&tocid=Amazon_Backup_Model_Condition
[AWS cron expression]: https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html