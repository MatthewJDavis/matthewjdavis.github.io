---
title: Creating a VSTS and AWS Service Endpoint
author: Matthew Davis
date: 2017-08-31
excerpt: Set up a connection between Microsoft Visual Studio Team Services and Amazon Web Services to deploy AWS resources from VSTS
categories: 
    - vsts
tags:
    - aws
    - vsts
---

Like VSTS? Yes, Like AWS? Yes. Want to use the greatness of VSTS to deploy to AWS? Of course. Here's how to get started by installing the [AWS Tools for Microsoft Visual Studio Team Services] from the VSTS market place and then creating a Service Endpoint to link VSTS and AWS together.

# AWS Tools Extension

Login to your VSTS account and access the market place.

The easiest way to do this is click on the shopping bag icon by the search box and select browse market place.

![shopping bag by the search bar at the top of the page](/images/vsts-aws-endpoint/market-place-icon.png)

Another way to do this is from the main account, click on the settings cog then select extensions and click on the browse market place icon.
Note, this is not from within an individual project, you'll get the project settings. Click on the VSTS logo at the top left of the page to get back to the main account screen.

![project settings then select extensions](/images/vsts-aws-endpoint/settings-extensions.png)

This will open up a new tab, search for AWS Tools, and click on the AWS Tools for Microsoft Visual Studio result tile.

On the main screen, click install.
![aws tools in the vsts market place](/images/vsts-aws-endpoint/aws-tools-market-place.png)

It will check you have the correct permissions to install the extension, click confirm.
Click get started and you'll be taken to the AWS page with the details of the extension, or click close and go back to the tab with your VSTS account loaded in it.

# Service Endpoint

Click on one of your projects. Once you are in a project, click on the settings cog icon and select services from the drop down.

![project settings then select services](/images/vsts-aws-endpoint/settings-services.png)

Click on the + New Service Endpoint button on the left of the screen and from the drop down choose AWS

![New service endpoint drop down list, aws, azure classic](/images/vsts-aws-endpoint/aws-endpoint.png)

You'll need your access key ID and secret key ID for an IAM user who has programmatic access to AWS with the correct permissions to deploy resources.

![AWS connection settings](/images/vsts-aws-endpoint/aws-connection-settings.png)

You now have a connection that will allow you to deploy from VSTS releases into AWS. In my next post, I'll demonstrate how to deploy a simple resource such as a storage account from VSTS to AWS.

[VSTS market place]: https://marketplace.visualstudio.com/vsts
[AWS Tools for Microsoft Visual Studio Team Services]: https://marketplace.visualstudio.com/items?itemName=AmazonWebServices.aws-vsts-tools