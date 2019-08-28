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

There is a cost for the log storage space used by CloudTrail and also if these are additional trails in the account, check the latest AWS documentation for details

## Set up

You will need PowerShell core and the AWS PowerShell core module (this is what I tested on, it should work on desktop PowerShell with the AWS module installed too).
A user that has permissions to create S3 buckets and CloudTrail trails.

## The Script

I'll run through some of the parts of the script, the full script can be found below.

Firstly some variables are set up for creating the S3 bucket name which needs to be globally unique and the account number gives a good chance of this being achieved.

```powershell
$AccountNumber = (Get-STSCallerIdentity).account
$BucketName = "cloudtrail-$AccountNumber"
$CloudTrailName = 'matt-cloudtrail'
```

### S3 Bucket

The CloudTrail policy is created in a here string variable and applied to the S3 bucket after creation to allow the CloudTrail service to write logs to the bucket.

```powershell
# Bucket Policy to allow CloudTrail to write logs to the bucket
$S3CloudTrailPolicy = @"
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AWSCloudTrailAclCheck20150319",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudtrail.amazonaws.com"
            },
            "Action": "s3:GetBucketAcl",
            "Resource": "arn:aws:s3:::$BucketName"
        },
        {
            "Sid": "AWSCloudTrailWrite20150319",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudtrail.amazonaws.com"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::$BucketName/AWSLogs/$AccountNumber/*",
            "Condition": {
                "StringEquals": {
                    "s3:x-amz-acl": "bucket-owner-full-control"
                }
            }
        }
    ]
}
"@


# Create S3 bucket and apply the bucket policy
New-S3Bucket -BucketName $BucketName
Write-S3BucketPolicy -BucketName $BucketName -Policy $S3CloudTrailPolicy
```

The bucket is set to private so no items can be made public.

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

To create the CloudTrail trail, the parameters are [splatted] for easier reading with the name and bucket name variables passed to the hashtable and the required settings set to true. The New-CTTrail creates the desired trail.

```powershell
#region CloudTrail
$params = @{
    Name                      = $CloudTrailName
    S3BucketName              = $BucketName
    EnableLogFileValidation   = $true
    IsMultiRegionTrail        = $true
    IncludeGlobalServiceEvent = $true
}

# Create trail
New-CTTrail @params
```

Next a tag is created and applied to the trail

```powershell
# Create description tag and add to trail
$tag = New-Object -TypeName Amazon.CloudTrail.Model.Tag
$tag.Key = 'Description'
$tag.Value = 'All region Cloudtrail logging all data events for S3 and Lambda.'

Add-CTTag -ResourceId  (Get-CTTrail -TrailNameList $CloudTrailName).TrailARN -TagsList $tag
```

### Write-CTEventSelector

To capture all of the S3 and Lambda, an [EventSelector] is required and the Write-CTEventSelector takes a list of EventSelector objects for the EventSelector parameter value.

```powershell
[-EventSelector <Amazon.CloudTrail.Model.EventSelector[]>]
```

```powershell
# Create CloudTrail event selector object
$EventSelector = New-Object -TypeName Amazon.CloudTrail.Model.EventSelector
$EventSelector.IncludeManagementEvents = $true
$EventSelector.ReadWriteType = 'All'
```

Looking at the [EventSelector] documentation, along with the two properties specified above, it also requires a [DataResource] object (again can be a list). The [DataResource] objects created below are for S3 and Lambda.

```powershell
# S3 data resource
$S3DataResource = New-Object -TypeName Amazon.CloudTrail.Model.DataResource
$S3DataResource.Type = 'AWS::S3::Object'
$S3DataResource.Values = 'arn:aws:s3'

# Lambda data resource
$LambdaDataResource = New-Object -TypeName Amazon.CloudTrail.Model.DataResource
$LambdaDataResource.Type = 'AWS::Lambda::Function'
$LambdaDataResource.Values = 'arn:aws:lambda'
```

The DataResources created are added to the EventSelector and this is applied to the trail and the trail is started.

```powershell
# Add data resources to event selector and update CloudTrail
$EventSelector.DataResources = $S3DataResource, $LambdaDataResource
Write-CTEventSelector -TrailName $CloudTrailName -EventSelector $EventSelector

# Start CloudTrail logging
Start-CTLogging -Name $CloudTrailName
```

### Full Script

<script src="https://gist.github.com/MatthewJDavis/cb16c1edadd2e3ba5fbc23658bcaf58a.js"></script>

## Summary

CloudTrail trails and logs are an important auditing tool in AWS, allowing you to see who is doing what in your account and can be used to set up alerts when certain actions take place. Using PowerShell allowed me to quickly deploy a standardised CloudTrail trail to many AWS accounts and made sure the S3 buckets were set to only private objects and the trail covered all regions and capturing actions on the services we required.

[AWS CloudTrail]: https://aws.amazon.com/cloudtrail/
[Splunk]: https://www.splunk.com/en_us/cyber-security/siem-security-information-and-event-management.html
[Azure Sentinel]: https://azure.microsoft.com/en-ca/services/azure-sentinel/
[future regions]: https://aws.amazon.com/blogs/aws/aws-cloudtrail-update-turn-on-in-all-regions-use-multiple-trails/
[splatted]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_splatting?view=powershell-6
[EventSelector]: https://docs.aws.amazon.com/sdkfornet/v3/apidocs/index.html?page=CloudTrail/TCloudTrailEventSelector.html&tocid=Amazon_CloudTrail_Model_EventSelector
[DataResource]: https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/CloudTrail/TDataResource.html