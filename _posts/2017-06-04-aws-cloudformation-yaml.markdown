---
layout: post
title:  "AWS Cloudformation with YAML"
date:   2017-06-05 21:20:00 +0000
categories: aws cloudformation yaml "infrastructure as code"
---

### What is YAML  

[YAML Ain't Markup Language] (YAML) was added to AWS cloudformation in September last year for provisioning resources through code via cloudformation.  

I definitely find it easier to work with and understand what is going on than JSON, so have decided to write future cloudformation templates in YAML.  

 

YAML allows code comments and also the standard AWS functions. Below I will demonstrate the code to provision a VM instance in AWS using JSON then YAML with cloudformation.  

 

Difference between JSON and YAML (code snippet  

 

VM in JSON  

VM in YAML with comments and parameters  

 

 

Wrap up 

Coming from Azure, I used JSON for ARM templates a lot and found the best way to work with it was to install an add on for Visual studio. I find working with YAML much easier to understand and quicker to use, plus I can use my favourite code editor, Visual Studio code to get things done without the need for starting up Visual Studio. 

[YAML Ain't Markup Language]: http://www.yaml.org/