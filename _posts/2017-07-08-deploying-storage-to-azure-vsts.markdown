---
layout: post
title:  "Deploy Storage to Azure via VSTS"
date:   2017-07-07 21:30:00 +0000
categories: Azure VSTS CI CD
---

In this post, we're going to deploy an Azure Storage Account to Azure using Microsoft's [Visual Studio Team Services][vsts] (VSTS).

The code for the storage account is available on my [GitHub][Github], it consists of two ARM templates:
- azuredeploy.json - the template containing details of the resource (storage account in this example)
- azuredeployparameters.json - this file is used to override the parameters in the template file to allow us to reuse it in different scenarios / projects

You'll need:
- An active Azure Subscription
- A VSTS account
These are free as long as you stay within the limits. Microsoft are very good at warning / disabling resources so you don't spend any money but you should keep a close eye on costs if you have monthly credit to use.
This example doesn't store any data in the storage account so there will be no charges, if you do store data, you'll get charge / have credit reduced.

Login to VSTS and create a New Project

![VSTS create New Project](/images/azure-storage-vsts/vsts-new-project.png)

I'm not going to be using the source control provided by VSTS because I'm using GitHub for this demo. VSTS offers free private repositories and integrates very nicely with the build tasks.


From the top menu I select "Build & Release" then click the + New definition

![VSTS create New build](/images/azure-storage-vsts/vsts-new-build.png)

Now select Empty process
![VSTS build empty process](/images/azure-storage-vsts/vsts-build-empty-process.png)

Click on the Process:
Name: rename this build if you like
Agent: Hosted

![VSTS build agent](/images/azure-storage-vsts/vsts-build-agent.png)





[vsts]: https://https://www.visualstudio.com/team-services/
[GitHub]: https://github.com/MatthewJDavis/Azure/tree/master/Azure-Storage/Storage-Account-Deployment-Demo
