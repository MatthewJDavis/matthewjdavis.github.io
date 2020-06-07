---
title: Get the public IP address of a network with Ansible
excerpt: Use Ansible and the ipify api to get the public IP address of the network you are currently connected to.
date: 2020-06-06
toc: false
classes: wide
categories:
- ansible
tags:
- ansible
published: true
---
June 2020

# Overview

Short post on how to use Ansible to get the public IP address of the network you are currently using, querying the [ipify api].

<script src="https://gist.github.com/MatthewJDavis/6f28b1a88f226bfc781ce2c200b05ea3.js"></script>

Now the public IP address is stored in the public_ip variable, it can be accessed and used in later tasks. The returned result is a dictionary with numerous keys and one of those keys is json which has the value of a dictionary. To access the IP address value requires access the nested dictionary: ``` public_ip.json.ip ```.

The IP address can then be used in other tasks such as creating the inbound rules in an Azure Network Security Group to lock down ssh access to a VM to your IP address.

<script src="https://gist.github.com/MatthewJDavis/7f2c440f17738cadf09ccf07726191fc.js"></script>

[ipify api]: https://www.ipify.org/