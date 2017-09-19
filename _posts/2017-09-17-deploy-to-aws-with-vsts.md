---
title: Deploy S3 bucket to Amazon Web Services with Visual Studio Team Services
author: Matthew Davis
date: 2017-09-17
excerpt: Guide on how to deploy an S3 bucket on Amazon Web Services from the build release pipeline in Visual Studio Team Services
categories: 
    - vsts
tags:
    - aws
    - vsts
published: false
---
## 
set up endpoint - previous blog post link
In the last post I outlined how to set up a service endpoint from Visual Studio Team Services (VSTS) to Amazon Web Services (AWS). In this post, I'll go through deploying an S3 bucket via VSTS though the method will be similar to different AWS resources.

## templates
With the endpoint created to AWS, either create a new project to upload the templates to or if the templates are in another reposititory (git or bitbucket) skip ahead to the build step and link those as the build source. This post will show you how to create a new project and add the templates too them.
From the main screen, click the **New Project** Button.
Enter the following details:

- Project name
- Details
- Git Version Control
- Agile Work item process

![New VSTS project](/images/vsts-aws-deploy/new-vsts-project.png)

Click on the newly create project and select **or initialize with a README or git ignore**
Click **Initialize** (you can select an ignore file item, however it is not needed in this case).
Click **New** and then **Folder** from the drop down and call it **S3**.
In the New Filename text box, enter: **create-bucket.yml**

![New folder](/images/vsts-aws-deploy/new-folder.png)

Copy the template code below into the file, click the **Commit** button. The commit dialogue box will appear, here you can change the commit message if you like and then click **Commit**

The template:

<script src="https://gist.github.com/MatthewJDavis/4cd996382161a809ef14fdc8055e1e06.js"></script>

![Committing the template](/images/vsts-aws-deploy/commit-template.png)

The template is a cloudformation yaml file which will create an S3 bucket, give the account owner full access and tag it with a Project tag. It requires 2 input parameters, BucketName and ProjectTag

Now we need a parameter file.

Click on the **S3** folder, click on the **+ New** button and select **File** from the drop down.
For the name, enter: **create-bucket-params.json**

![New file](/images/vsts-aws-deploy/new-file.png)
![New file name](/images/vsts-aws-deploy/new-file-name.png)

Add the parameter file code below (you must change the value of the BucketName from "matt-vsts-bucket" to something globally unique).

<script src="https://gist.github.com/MatthewJDavis/634af98f2f219b68f3c82519e47f9519.js"></script>

The parameter file is used to pass the two parameters required by the cloudformation template of BucketName and ProjectTag. The parameters use key value pairs and the ParameterValue for BucketName should be changed so the S3 bucket is globally unique.

### Change the following:

- ParameterValue of the BucketName from "matt-vsts-bucket" to something globally unique (See [Rules for bucket naming])
- ParameterValue of Project (if you have other projects you tag by, if not leave as VSTS)

Click the **Commit** button. The commit dialogue box will appear, here you can change the commit message if you like and then click **Commit**

![Complete file and folder](/images/vsts-aws-deploy/complete-code.png)

## build step



## release step

## 

[Rules for bucket naming]: http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html