---
title:  "AWS Cloudformation with YAML"
author: Matthew Davis
date: 2017-06-05
categories: 
    - aws
tags:
    - cloudformation
    - provisioning infrastructure
    - infrastructure as code
---

[YAML Ain't Markup Language] (YAML) was added to AWS cloudformation in [September last year] for provisioning resources through code via cloudformation.

I definitely find YAML easier to work with and understand than JSON, so going forward will write future cloudformation templates in YAML.

Here's a comparison of the code for parameters in JSON and YAML:

## JSON

<script src="https://gist.github.com/MatthewJDavis/a4cc7f80a5954a7cbd9bc39f5d33b1af.js"></script>

## YAML

<script src="https://gist.github.com/MatthewJDavis/e1bb0ed8ddd45fbe66199a397872b019.js"></script>


Not only is YAML clearer it also allows code comments and also the standard [AWS functions] such as replacing values with parameters. Below I will demonstrate the code to provision a VM instance in AWS using JSON then YAML with cloudformation.  

Here is the complete code to provision an EC2 instance in AWS in an existing VPC and subnet, made reusable with parmaeters (this template will work for the free tier, [check out the details])

<script src="https://gist.github.com/MatthewJDavis/edcaa9d2c362464b7e5f7bced50df1b1.js"></script>

## Wrap up

Coming from Azure, I used JSON for ARM templates a lot and found the best way to work with them was to install the [Azure Resource Manager tools] for Visual studio. I find working with YAML much easier to understand and quicker to use, plus I can use my favourite code editor, [Visual Studio code] to get things done without the need for starting up Visual Studio. 

[YAML Ain't Markup Language]: http://www.yaml.org/
[September last year]:https://aws.amazon.com/about-aws/whats-new/2016/09/aws-cloudformation-introduces-yaml-template-support-and-cross-stack-references/
[AWS functions]: https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-sub.html
[check out the details]: https://aws.amazon.com/free/
[Visual Studio Code]: https://code.visualstudio.com/
[Azure Resource Manager tools]: https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools