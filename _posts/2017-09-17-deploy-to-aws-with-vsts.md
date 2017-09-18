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
Click **New** and then **Folder** from the dropdown and call it S3.

We need 2 files to create the S3 bucket, one to create the resource and one for the parameters to pass to it so it is a reusable template.

Add the following code:

The template:

<script src="https://gist.github.com/MatthewJDavis/4cd996382161a809ef14fdc8055e1e06.js"></script>

The parameter files:

<script src="https://gist.github.com/MatthewJDavis/634af98f2f219b68f3c82519e47f9519.js"></script>

## build step

## release step

## 