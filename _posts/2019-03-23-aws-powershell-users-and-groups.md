---
title: Manage AWS IAM Users and groups with PowerShell
author: Matthew Davis
date: 2019-03-23
toc: true
excerpt: Overview of managing IAM Users and Roles in AWS using PowerShell
categories:
    - aws
tags:
    - iam
    - aws
    - powershell
published: true
---
March 2019

# Overview

AWS offers the familiar users, groups and polices via Identity Access Management (IAM).
These can be managed via the console, the AWS CLI or the AWS PowerShell module. I'll run through a demo of creating an IAM user, a group, adding the user to the group, create a policy and attach it to the group so the user will have the access the policy specifies by being a member of a group.

I am using PowerShell core version 6.1.3 with the AWSPowerShell.NetCore version 3.3.462.0 installed which are the latest versions at time of writing. The operating system is Windows 10.

## Install AWS PowerShell Core module

Easiest way is to use the Install-Module Cmdlet.

``` powershell
Install-Module -Name AWSPowerShell.NetCore -Scope CurrentUser
```

## Set the credentials

The IAM user used to connect to the AWS account will need to have credentials created for API access (AccessKey and SecretKey). These should be kept secured and never shared. 
I created my local AWS profile by using the Set-AWSCredential cmdlet which stores the credential in an encrypted JSON file stored in the app data location.

See the [AWS credential documents] for more details on setting up the credentials.

```powershell
Set-AWSCredential -AccessKey 'exampleKey' -SecretKey 'exampleKey' -StoreAs 'demo'

ls C:\Users\Demo\AppData\Local\AWSToolkit\

    Directory: C:\Users\Demo\AppData\Local\AWSToolkit


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       23/03/2019     15:40           1360 RegisteredAccounts.json
```

This file is encrypted with the Windows cryptographic APIs and on Linux and Mac the credentials are stored in plaintext in the ~/.aws/credentials file and should be protected using another method (you can specify alternative path to a credential file when setting the credentials).

### Users

We can get a list of IAM user cmdlets by using the Get-AWSCmdletName

```powershell
Get-AWSCmdletName -Service iam | Where-Object -Property CmdletName  -Like '*user*'
```

![List of users](/images/aws-iam/aws-powershell-iam-user-cmdlets.png)

To Create a User and give access to console and /or AWS API

```powershell
New-IAMUser -UserName 'Jack' -Tag @{Key='Project';Value='Demo'}

# Give console access
New-IAMLoginProfile -UserName 'Jack' -PasswordResetRequired $true -Password (read-host)

# Give API access - the access key and secret key will be displayed so the output could be redirected to a file to securely send to the user.
New-IAMAccessKey -UserName Jack
```

To get the IAM Users

```powershell
Get-IAMUserList | Select-Object -Property UserName
```

![List of users](/images/aws-iam/get-iam-user-list.png)

### Groups

To create a new group

```powershell
New-IAMGroup -GroupName 'S3DemoUpload'
```

Add a user to the group

```powershell
Add-IAMUserToGroup -GroupName 'S3DemoUpload' -UserName 'Jack'
```

List users in a group

```powershell
(Get-IAMGroup s3DemoUpload).Users | select UserName
```

![List of users in a group](/images/aws-iam/get-iam-users-in-group.png)

List all groups

```powershell
Get-IAMGroupList | Select GroupName
```

### Policy

Now that we have a user and a group, we can attach a [policy] to that group to allow the users in that group to upload to an S3 bucket.

```powershell
# Create the policy with a here string

$policy = @'
 {
     "Version": "2012-10-17",
     "Statement": [
         {
             "Effect": "Allow",
             "Action": [
                 "s3:PutObject"
             ],
             "Resource": [
                 "arn:aws:s3:::demo-aws-s3-example",
                 "arn:aws:s3:::demo-aws-s3-example/*"
             ]
         }
     ]
 }
 '@

$p = New-IAMPolicy -PolicyName 'DemoUploadPolicy' -Description 'Allow upload to the demo bucket' -PolicyDocument $policy

# Make a note of the policy arn or save to a variable like above to use later
```

![Creating the new IAM Policy](/images/aws-iam/new-iampolicy.png)

Register the policy to the group

```powershell
Register-IAMGroupPolicy -GroupName 'S3DemoUpload' -PolicyArn $p.Arn
```

![Creating the new IAM Policy](/images/aws-iam/new-iampolicy.png)

```powershell
Get-IAMAttachedGroupPolicyList -GroupName 'S3DemoUpload'
```

Now that the policy is attached, users in that group have the permissions of the policy.

### Removing the policy and group

```powershell
Unregister-IAMGroupPolicy -GroupName 'S3DemoUpload' -PolicyArn $p.Arn

Remove-IAMPolicy -PolicyArn $p.Arn -Force

# Remove the users from the group
(Get-IAMGroup s3DemoUpload).Users | select UserName | 
ForEach-Object {Remove-IAMUserFromGroup -GroupName 'S3DemoUpload' -UserName $_.UserName -force}

# Remove the group
Remove-IAMGroup -GroupName 'S3DemoUpload' -force
```

## Summary

Managing IAM users, groups and policies is relatively straightforward with PowerShell core and allows you to manage users by adding them to groups, similar to Active Directory and then applying policies to those groups. You could also apply policies directly to a group by creating an in-line policy but I like to have separate policies that could also be attached elsewhere such as an EC2 IAM role or a role that is used for cross account access, just one place to update the policy.

Protecting the access and secret key should be a priority, it is encrypted on Windows, but extra measures are needed on Mac or Linux, I usually create an encrypted directory and store the creds in there and only decrypt and mount when they are needed.

[AWS credential documents]: https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html
[different profiles]: https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html#managing-profiles
[policy]: https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html