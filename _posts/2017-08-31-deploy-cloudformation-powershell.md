---
title: Deploy an AWS CloudFormation stack with PowerShell
author: Matthew Davis
date: 2017-08-31
excerpt: Deploy a CloudFormation template into AWS with PowerShell and the AWS PowerShell module
categories: 
    - aws
tags:
    - aws
    - cloudformation
    - powershell
---
August 2017

# Overview

Following from my last post about testing CloudFormation, I'm going to write up how to use PowerShell to deploy resources to AWS with a CloudFormation stack.
I use yaml templates but the process is the same for templates written in JSON.

The AWS PowerShell module is required and can be installed via the [PowerShell gallery] for both the desktop and core versions of PowerShell.

The below commands will find the module on the PSGallery and pipe the result to Install-Module to install (you can change the scope to the current user if you need to - the default will install the module for all users of the machine)

```powershell
# For Desktop PowerShell:
Find-Module -Name AWSPowerShell | Install-Module

# For Core PowerShell:
Find-Module -Name AWSPowerShell.NetCore | Install-Module
```

You'll need to set your [AWS credentials] to authenticate to the AWS APIs with PowerShell (your access key, secret key, and possibly your session token) which are located in "Security Credentials" within your "Users" area of the "[IAM]" console.

To set your credentials, use the Set-AWSCredentials cmdlet. See the [AWS PowerShell docs] for details of how the credentials are set and stored.

Now set the default AWS region for the session:

```powershell
Set-DefaultAWSRegion -Region eu-west-1
```

The following yaml template will create a basic S3 bucket with the owner having full access to it and no other properties set. See the [AWS CloudFormation S3 docs] for all the properties that can be set. This template will be used to deploy a CloudFormation stack via PowerShell.

<script src="https://gist.github.com/MatthewJDavis/c60e7558d4adbba4b1e40eb5dbd061cf.js"></script>

With that file saved and the PowerShell session authenticated with AWS, save the template in a variable, using the Get-Content with the Raw switch to read the whole of the file as a [single object]:

```powershell
$template = Get-Content -Path C:\temp\simple-s3-bucket.yml -Raw
```

Now we need to create 2 parameters to pass to the template (BucketName and ProjectTag). The New-CFNStack requires a parameter type data type (not a string) so a custom object is required:

```powershell
# bucketname parameter
$bucketname = New-Object -TypeName Amazon.CloudFormation.Model.Parameter
$bucketname.ParameterKey = 'BucketName'
$bucketname.ParameterValue = 'globally-unique-bucket-name'

# project value for the project tag
$project = New-Object -TypeName Amazon.CloudFormation.Model.Parameter
$project.ParameterKey = 'ProjectTag'
$project.ParameterValue = 'demo'
```

Now it's time to create the CloudFormation stack that will be deployed with the New-CFNStack cmdlet:

```powershell
New-CFNStack -StackName s3-demo-stack -TemplateBody $template -Parameter $bucketname, $project
```

If you go to the console, under the CloudFormation you'll see the full details of the stack deployment, including events and outputs (make sure you're in the same region as the region set in the PowerShell session).

![console output of deployed CloudFormation stack](/images/cfn-powershell/deployed-stack.png)

You can also get events and outputs with PowerShell. For example to get the outputs specified in the yaml template with PowerShell:

```powershell
(Get-CFNStack -StackName s3-demo-stack).Outputs
```

You can also remove stacks with the Remove-CFNStack command that will delete all the resources created with the stack so be careful when using it, especially with the force switch!

That's it, it took me a while to figure out how to deploy CloudFormation with PowerShell but it makes for testing and standing up infrastructure in AWS much quicker and reliable for testing and can be used as part of a deployment pipeline.

[PowerShell gallery]: https://www.powershellgallery.com/api/v2/
[IAM]: https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html
[AWS credentials]: https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html
[AWS PowerShell docs]: https://docs.aws.amazon.com/powershell/latest/userguide/specifying-your-aws-credentials.html
[AWS CloudFormation S3 docs]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html#aws-properties-bucket-prop
[single object]: https://powershell.org/2013/10/21/why-get-content-aint-yer-friend/
