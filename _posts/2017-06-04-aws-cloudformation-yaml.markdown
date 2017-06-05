---
layout: post
title:  "AWS Cloudformation with YAML"
date:   2017-06-05 21:20:00 +0000
categories: aws cloudformation yaml "infrastructure as code"
---

### Provision an EC2 instance with YAML and cloudformation  

[YAML Ain't Markup Language] (YAML) was added to AWS cloudformation in [September last year] for provisioning resources through code via cloudformation.  

I definitely find YAML easier to work with and understand than JSON, so going forward will write future cloudformation templates in YAML.

Here's a comparison of the code for parameters in JSON and YAML:

<script src="https://gist.github.com/MatthewJDavis/a4cc7f80a5954a7cbd9bc39f5d33b1af.js"></script>


YAML allows code comments and also the standard AWS functions. Below I will demonstrate the code to provision a VM instance in AWS using JSON then YAML with cloudformation.  


Difference between JSON and YAML (code snippet)

 

VM in JSON  

VM in YAML with comments and parameters  

 

 

Wrap up 

Coming from Azure, I used JSON for ARM templates a lot and found the best way to work with it was to install an add on for Visual studio. I find working with YAML much easier to understand and quicker to use, plus I can use my favourite code editor, Visual Studio code to get things done without the need for starting up Visual Studio. 

[YAML Ain't Markup Language]: http://www.yaml.org/
[September last year]:https://aws.amazon.com/about-aws/whats-new/2016/09/aws-cloudformation-introduces-yaml-template-support-and-cross-stack-references/