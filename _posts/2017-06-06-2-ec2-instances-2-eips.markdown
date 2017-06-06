---
layout: post
title:  "2 EC2 Instances and EIPs with AWS Cloudformation YAML"
date:   2017-06-06 20:40:00 +0000
categories: aws cloudformation yaml "infrastructure as code"
---

### Provision 2 EC2 instances with an Elastic IP address attached to each using YAML and cloudformation

**you could be charged using the free tiers because of the Elastic IPs**

After yesterday's post about using YAML, today at work I had to set up two jump servers, with public IP addresses to access resources in a disaster recovery account. 
I didn't have time to finish the set up, so I decided to write out the cloudformation script this evening so I can get things up and running in the morning for our disaster recovery test. If this script can be saved, then in a DR failover the jump servers will be able to be provisioned in any AWS account (providing the script is accessible... hello github, S3 storage and Azure Blob storage).

Here's the script that will provision 2 Amazon Linux instances with public IP addresses and a security group allowing SSH from anywhere (this should be locked do to a home address or office).

Be warned, there will be a charge for the elastic IP addresses if you're using the free tier.

<script src="https://gist.github.com/MatthewJDavis/6bc2803209ed334bc3a5d2388476e66d.js"></script>



