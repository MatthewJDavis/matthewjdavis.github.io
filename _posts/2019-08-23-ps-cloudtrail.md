---
title: Create an AWS CloudTrail trail for all regions with PowerShell
author: Matthew Davis
date: 2019-08-23
toc: false
classes: wide
excerpt: Use PowerShell and the AWS module to create a CloudTrail trail for all regions that captures S3 and Lamda events and saves the logs to S3.
categories:
    - powershell
tags:
    - aws
    - powershell
    - cloudtrail
published: true
---
August 2019

# Overview

[AWS CloudTrail] allows you to log and monitor activity in your AWS account, giving insight into actions carried out in the account from a range of sources from the AWS console, command line, AWS PowerShell and other AWS services.
The default settings for CloudTrail is to keep the logs for 90 days but by setting up a trail, you can keep the logs longer and the logs can be ingested by a Security Information and Event Management (SIEM) such as [Splunk] or [Azure Sentinel] for monitoring and alerting.

Having just set this up in a number of AWS accounts, I created a PowerShell script that would create an S3 bucket (public access denied on it) and a new CloudTrail trail that covers events in all current and [future regions] along with collecting events from the S3 and Lambda services.

## Set up
You will need PowerShell core and the AWS PowerShell core module (this is what I tested on, it should work on desktop PowerShell with the AWS module installed too).
A user that has permissions to create S3 buckets and CloudTrail trails.

## The Script

I'll run through some of the parts of the script, the full script can be found below.

Firstly some variables are set up for creating the S3 bucket name which needs to be globally unique and the account number gives a good chance of this being achieved.

```powershell
$AccountName = Get-IAMAccountAlias
$AccountNumber = (Get-STSCallerIdentity).account
$BucketName = "cloudtrail-$AccountNumber"
$CloudTrailName = 'matt-cloudtrail'
```

### S3 Bucket

The CloudTrail policy is created in a here string variable and applied to the S3 bucket after creation to allow the CloudTrail service to save logs to the bucket.

The bucket is made private so no items can be made public.

```powershell
# Block public access for the bucket and objects
$s3PublicParams = @{
    BucketName                                          = $BucketName
    PublicAccessBlockConfiguration_BlockPublicAcl       = $true
    PublicAccessBlockConfiguration_BlockPublicPolicy    = $true
    PublicAccessBlockConfiguration_IgnorePublicAcl      = $true
    PublicAccessBlockConfiguration_RestrictPublicBucket = $true
}

Add-S3PublicAccessBlock @s3PublicParams
```

### CloudTrail



### Full Script

<script src="https://gist.github.com/MatthewJDavis/cb16c1edadd2e3ba5fbc23658bcaf58a.js"></script>

## Summary

[AWS CloudTrail]: https://aws.amazon.com/cloudtrail/
[Splunk]: https://www.splunk.com/en_us/cyber-security/siem-security-information-and-event-management.html
[Azure Sentinel]: https://azure.microsoft.com/en-ca/services/azure-sentinel/
[future regions]: https://aws.amazon.com/blogs/aws/aws-cloudtrail-update-turn-on-in-all-regions-use-multiple-trails/