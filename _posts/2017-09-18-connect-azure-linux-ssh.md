---
title: Connect to an Azure Linux vm with ssh and Windows Subsytem for Linux
author: Matthew Davis
date: 2017-09-18
excerpt: Set up a connection between Microsoft Visual Studio Team Services and Amazon Web Services to deploy AWS resources from VSTS
categories: 
    - azure
tags:
    - azure
    - linux
published: false
---

Windows 10 64bit version has the Windows Subsystem for Linux (WSL) installed allowing you to easily run Linux tools from within Windows 10 which is great for managing Linux VMs in Azure and AWS. When creating an EC2 instance in AWS, you specify which key pair to use so you can connect to your AWS EC2 instance via ssh. You create the initial key pair in the AWS Identity and Access Management (IAM) console and providing you keep the key safe and still have access to it, you'll be able to easily connect to the EC2 instance. 

Azure Linux VMs work slightly differently though the concept to connect is the same via ssh, you need to specify the public key to the vm and authenticate to it with the private key. The main difference is you must create the key pair on your computer and then paste the public key into the Azure VM.

This post will show you how to create an ssh key pair on Windows using the Windows Subsystem for Linux (it will also work if you're just using an installation of Ubuntu Linux) and how to set up an Azure Linux VM to be able to connect using the ssh key.

## Installing WSL

Follow this guide to [Installing Windows Subsystem for Linux]

## Creating the key pair

## Creating the Linux VM

### Pasting the public key

### Network Security Group

## Connecting

[Installing Windows Subsystem for Linux]: https://msdn.microsoft.com/en-gb/commandline/wsl/install_guide