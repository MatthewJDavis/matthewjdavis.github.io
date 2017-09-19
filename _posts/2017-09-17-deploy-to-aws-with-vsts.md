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
published: true
---

In the last post I outlined how to set up a [service endpoint] from Visual Studio Team Services (VSTS) to Amazon Web Services (AWS). In this post, I'll go through deploying an S3 bucket via VSTS though the method will be similar to different AWS resources.

## New VSTS Project

With the endpoint created to AWS, either create a new project to upload the templates to or if the templates are in another repository (git or bitbucket) skip ahead to the build step and link those as the build source. This post will show you how to create a new project and add the templates too them.
From the main screen, click the **New Project** Button.
Enter the following details:

- Project name
- Details
- Git Version Control
- Agile Work item process

![New VSTS project](/images/vsts-aws-deploy/new-vsts-project.png)


## CloudFormation Templates

Click on the newly create project and select **or initialize with a README or git ignore**
Click **Initialize** (you can select an ignore file item, however it is not needed in this case).
Click **New** and then **Folder** from the drop down and call it **S3**.
In the New Filename text box, enter: **create-bucket.yml**

![New folder](/images/vsts-aws-deploy/new-folder.png)

Copy the template code below into the file, click the **Commit** button. The commit dialogue box will appear, here you can change the commit message if you like and then click **Commit**

The template:

<script src="https://gist.github.com/MatthewJDavis/4cd996382161a809ef14fdc8055e1e06.js"></script>

![Committing the template](/images/vsts-aws-deploy/commit-template.png)

The template is a CloudFormation yaml file which will create an S3 bucket, give the account owner full access and tag it with a Project tag. It requires 2 input parameters, BucketName and ProjectTag

Now we need a parameter file.

Click on the **S3** folder, click on the **+ New** button and select **File** from the drop down.
For the name, enter: **create-bucket-params.json**

![New file](/images/vsts-aws-deploy/new-file.png)
![New file name](/images/vsts-aws-deploy/new-file-name.png)

Add the parameter file code below (you must change the value of the BucketName from "matt-vsts-bucket" to something globally unique).

<script src="https://gist.github.com/MatthewJDavis/634af98f2f219b68f3c82519e47f9519.js"></script>

The parameter file is used to pass the two parameters required by the CloudFormation template of BucketName and ProjectTag. The parameters use key value pairs and the ParameterValue for BucketName should be changed so the S3 bucket is globally unique.

### Change the following:

- ParameterValue of the BucketName from "matt-vsts-bucket" to something globally unique (See [Rules for bucket naming])
- ParameterValue of Project (if you have other projects you tag by, if not leave as VSTS)

Click the **Commit** button. The commit dialogue box will appear, here you can change the commit message if you like and then click **Commit**

![Complete file and folder](/images/vsts-aws-deploy/complete-code.png)

## build step

From the top menu, select **Build and Release**, then **Builds**

![Select build from the build release tab](/images/vsts-aws-deploy/build-tab.png)

Click on **+ New Definition**
In the build template, select **Empty Process**

![Select empty process](/images/vsts-aws-deploy/build-empty-process.png)

Select "Hosted Agent"
The **Get Sources** section should be set to your master branch from the code you added to you repository earlier. This is the step where you can specify another source for your code such as Github or Bitbucket.

Add the following tasks:

- Copy Files to:
- Publish Artifacts:

![adding build tasks](/images/vsts-aws-deploy/build-tasks.png)

### Copy build step

- Source Folder: S3 (you can click on the ellipsis to navigate to the folder too)
- Contents: ** (this copies all the files in the folder)
- Target Folder: $(build.artifactstagingdirectory)

![copy build step](/images/vsts-aws-deploy/copy-build-step.png)

### Publish Artifact build step

- Path to Publish: $(build.artifactstagingdirectory)
- Artifact Name: s3-drop
- Artifact Type: Server

![publish artifact build step](/images/vsts-aws-deploy/publish-artifact-build-step.png)

Click **Save and Queue**
Click **Queue**

## release step

From the top menu, select **Build and Release**, then **Releases**
Click **+ New Definition**
Select **Empty Process**

Call the **Environments** what you like, I'm calling mine Dev1 (this could be part of a dev - qa - staging - production pipeline).
Click **Save**
Enter a comment if you like then click **OK**

![New release definition](/images/vsts-aws-deploy/new-release-definition.png)

### Artifacts

You should be taken to the pipeline view, if not click on **Pipeline** from the menu.
Click **+ Add** in the Artifacts box
Choose the **Source (build definition)** from the drop down (you can leave the remaining settings as default)

![Add artifact screen](/images/vsts-aws-deploy/add-artifact.png)

### Release Tasks

Click on the **Phase, Task ** link under the environment name

![Release tasks](/images/vsts-aws-deploy/release-tasks.png)

Select the **AWS CloudFormation Create/Update Stack**, click **Add**

![CloudFormation task](/images/vsts-aws-deploy/cloudformation-task.png)

Enter in the following values

- Display Name: Create/Update Stack: s3-stack-vsts
- AWS Credentials: AWS-VSTS (or select whatever you called your service endpoint to AWS)
- AWS Region: eu-west-1 (or any other valid AWS region)
- Stack Name: s3-stack-vsts
- Template File: $(System.DefaultWorkingDirectory)/aws-resource-demo-CI/s3-drop/create-bucket.yml (you can use the ellipsis to navigate to the template file in the artifacts)
- Template Parameters File: $(System.DefaultWorkingDirectory)/aws-resource-demo-CI/s3-drop/create-bucket-params.json

![CloudFormation task settigns](/images/vsts-aws-deploy/cloudformation-task-settings.png)

The rest can be left as default.
Click **Save**
Enter a comment if you like and click **OK**

Click **Release**
From the drop down select **Create Release**

![Create new release](/images/vsts-aws-deploy/create-new-release.png)

Add a description if you like
Click **Queue**

Click on the Release-1 (or could be another number if there are previous releases) to access the newly created release. This will open the releases in a new browser tab.

![Successful release](/images/vsts-aws-deploy/released.png)

If there are any errors, check on the log tab to see them.

![Release logs](/images/vsts-aws-deploy/release-logs.png)

## Check release and cleaning up 

The last step is to go to the AWS console and check the bucket.

Here's the bucket created with the Project tag.

![VSTS deployed bucket](/images/vsts-aws-deploy/vsts-s3-bucket.png)

If we go to the CloudFormation section, we can see the CloudFormation stack created from the VSTS release.

![CloudFormation Console](/images/vsts-aws-deploy/cloudformation-console.png)

To clean up, with the stack checked, click **Actions** and select **Delete Stack** from the drop down.

![Delete stack](/images/vsts-aws-deploy/delete-stack.png)

## The end

This was an overview how to use VSTS to deploy an AWS resource. The resource was a simple S3 bucket but the steps would be the same to deploy more complicated resources. Creating deployment pipelines in VSTS is super easy and brings powerful and configurable build and releases to both small and large teams with a small time investment to get it up and running. Once you have one environment being deployed to, it's easy to copy that task and deploy to others.

[service endpoint]: https://matthewdavis111.com/vsts/vsts-aws-service-endpoint/
[Rules for bucket naming]: http://docs.aws.amazon.com/AmazonS3/latest/dev/BucketRestrictions.html