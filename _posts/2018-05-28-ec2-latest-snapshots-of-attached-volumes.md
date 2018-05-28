---
title:  Use PowerShell to get the lastest EC2 snapshots of all volumes attached to an instance
author: Matthew Davis
date: 2018-05-28
categories: 
    - PowerShell
tags:
    - PowerShell
    - aws
    - ec2
---

I needed to share some AWS EC2 snapshots with another account. To do this, first I needed to find out the volumes that were attached to the instance,
then share the snapshots. This is going to be on a daily schedule so needed PowerShell to get the details and then use a scheduling system to run it daily.

Here's the script that will get all attached volumes of an EC2 instance and then get the latest snapshots. This can be modified to update the snapshot properties to share with another account etc.

<script src="https://gist.github.com/MatthewJDavis/d20816b36f253fe5c71b070708c94d50.js"></script>
