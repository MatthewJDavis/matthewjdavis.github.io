---
title: "Deploy Storage to Azure via VSTS"
date: 2017-07-07
categories: 
    - azure
tags:
    - visual studio team services
    - devops
    - build
    - release
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

Now you select your source, this can be your VSTS project repo or a remote repo
See the [VSTS][vsts-repo] documentation for setting up a remote source. 
![VSTS sources](/images/azure-storage-vsts/vsts-sources.png)

Click on add task 
Under the utilities section Add the following tasks:
- Copy Files
- Publish Build Artifacts

![VSTS add tasks](/images/azure-storage-vsts/vsts-build-add-task.png)

Click the Copy Files task
Source Folder: Select the ellipses and select the folder containing the template and parameters JSON files

[![VSTS copy files](/images/azure-storage-vsts/vsts-file-copy.png)](/images/azure-storage-vsts/vsts-file-copy-big.png)

Contents: **\\*.json (we only want the JSON files) See [file copy docs][vsts-file-copy] for more info.

Target Folder:  $(build.artifactstagingdirectory) (this where the files will be copied before we upload them as an artifact to be used in the release)

![VSTS copy files](/images/azure-storage-vsts/vsts-file-copy-1.png)

Click the Publish Artifact task
- Name: Rename if you like
- Path to publish:  $(build.artifactstagingdirectory)
- Artifact Name: storage-demo-artifact-drop
- Artifact Type: Server

![VSTS publish artifacts](/images/azure-storage-vsts/vsts-publish-artifact.png)

## Build Time
Click Save and Queue
![VSTS queue build](/images/azure-storage-vsts/vsts-queue-build.png)

This will queue a new build and once it has completed you should have an artifact containing the two JSON files that can be used in the release phase.

Go back to the builds main page and click on the build. You should see a build number, click on that number and you will be taken into the details of the build.

![VSTS build result](/images/azure-storage-vsts/vsts-build-result.png)

Here you can click on the artifacts and see what the build produced
![VSTS build artifacts](/images/azure-storage-vsts/vsts-build-artifacts.png)

## Release
Click on the Releases tab and click New Definition

Select Empty
![VSTS new release](/images/azure-storage-vsts/vsts-new-release.png)

Make sure the build definition is the one you've just created.
You can click the Continuous deployement if you like, then every time a successful build is created, the deployment will happen automatically (note: if you're doing lots of little changes you may not want this and deploy manually otherwise you can use up your free monthly build minutes pretty quickly).

![create release definition](/images/azure-storage-vsts/vsts-release-definition.png)

First, let's setup the Variable for the release, these will apply to all of the environments

[![click on variables on the top menu](/images/azure-storage-vsts/vsts-release-variables.png)](/images/azure-storage-vsts/vsts-release-variables-big.png)

Add the following variables (paying attention to the case):
Location: uksouth (or the Azure region nearest you)
blobEncryptionEnabled: true

Click on Environment 1 and rename to Development or similar
[![set release variables](/images/azure-storage-vsts/vsts-release-var-setting.png)](/images/azure-storage-vsts/vsts-release-var-setting-big.png)

Click Add Task
Select Azure Resource Group Deployment

Azure Subscription: Select your Azure subscription from the drop down
Action: Create or update resource group
Resource Group: demo-storage-rg (or call it what you like or choose an existing resource group to deploy into)
Location: $(location) Here we are using the release variable that we set up earlier for the location

Template: Linked artifact
Template: click the elipses and select the azuredeploy.json from the artifacts we create from the build

[![select templates from artifacts](/images/azure-storage-vsts/vsts-select-artifact.png)](/images/azure-storage-vsts/vsts-select-artifact-big.png)

Template Parameters: Repeat the previous process and select azuredeployparameters.json
Override Template Parameters: -location $(location) -storageAccountName $(storageAccountName) -storageAccountType $(storageAccountType) -blobEncryptionEnabled $(blobEncryptionEnabled)
Here we specify the variables we are going to use to override the template parameters

[![override template parameters](/images/azure-storage-vsts/vsts-override-params.png)](/images/azure-storage-vsts/vsts-override-params-big.png)

Deployment mode: Incremental is fine

Environment Variables
Now we set the variables for the individual environment.
Click the ellipses by the environment name and select Configure variables…

[![select configure variables](/images/azure-storage-vsts/vsts-environment-vars.png)](/images/azure-storage-vsts/vsts-environment-vars-big.png)

Enter the following:

storageAccountName: storageacctname242 - this needs to be unique to Azure and has specific naming conventions
storageAccountType: Standard_LRS

[![configure variables for storageAccountName and storageAccountType](/images/azure-storage-vsts/vsts-environment-vars-conf.png)](/images/azure-storage-vsts/vsts-environment-vars-conf-big.png)

Click the pencil icon to rename the release if you wish.

Save the release

Click the Release button and select Create Release

[![select create release from release menu drop down](/images/azure-storage-vsts/vsts-create-release.png)](/images/azure-storage-vsts/vsts-create-release-big.png)

Hit create

Click on Release-1
[![click on release-1](/images/azure-storage-vsts/vsts-created-release.png)](/images/azure-storage-vsts/vsts-created-release-big.png)

You should now see the release deploying
[![release in progress](/images/azure-storage-vsts/vsts-release-in-progress.png)](/images/azure-storage-vsts/vsts-release-in-progress-big.png)

If you go to your Azure subscription, you should see the resource group created and shortly after the storage account and the release Deployment Status should show Success

[![release success](/images/azure-storage-vsts/vsts-release-succeeded.png)](/images/azure-storage-vsts/vsts-release-succeeded-big.png)

In the Azure portal:

[![azure portal](/images/azure-storage-vsts/vsts-azure-deploy.png)](/images/azure-storage-vsts/vsts-azure-deploy-big.png)


[vsts]: https://https://www.visualstudio.com/team-services/
[GitHub]: https://github.com/MatthewJDavis/Azure/tree/master/Azure-Storage/Storage-Account-Deployment-Demo
[vsts-repo]:https://www.visualstudio.com/en-us/docs/build/define/repository
[vsts-file-copy]:https://www.visualstudio.com/en-gb/docs/build/steps/utility/copy-files