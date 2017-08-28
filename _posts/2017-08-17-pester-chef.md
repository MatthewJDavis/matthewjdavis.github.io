---
title: Setting up Pester to use with Chef
author: Matthew Davis
date: 2017-08-17
excerpt: Use Pester for integration testing with Chef
categories: 
    - chef
tags:
    - chef
    - powershell
    - pester
    - integration testing
---

I've been learning chef for configuration management through their excellent learning resources at [learnchef.io]. It's a great way to learn chef and has some excellent resources and tutorials that cover Linux and Windows configuration management with chef.

I've completed a number of the tracks and I'm currently working through the [Local Development and Testing] track and although inspec was cool, I'd much prefer to run integration tests with [Pester], the PowerShell testing framework (which I'm also learning and incorporating it as much as possible).

It took me a bit of time and googling to get this up and running but it's easy once you know how, so here's an overview.
You'll need:
1. chefdk installed
2. A kitchen.yml file for your cookbook

1. Install [kitchen-pester]
 - Open up the chefdk 
 - Run ```ruby gem install kitchen-pester ```

2. Open the kitchen.yml file
   - Edit the verifier and change it to pester 

My kichen.yml file looks like this

<script src="https://gist.github.com/MatthewJDavis/43ecc7e3b81d42b9d260a06b33de233f.js"></script>

Now create the Pester tests under the directory structure:
cookbook/test/integration/recipeName/pester

For example, a recipe called uk_settings, create a Pester test file like so:
team_city_agent/test/integration/uk_settings/pester/uk_settings.Tests.ps1

![Directory tree of tests](/images/chef-pester/directory-layout.png)
 

Now when you run ``` kitchen verify```, your Pester tests will run for the integration tests.

I'm enjoying learning chef and am glad I can use Pester tests within the chef framework to test the infrastructure created.

For reference, the following versions are being used on my Windows 10 machine. I also got this set up easily on my mac at work after I had added the chef install of ruby to my bash_profile (which it tells you to do in the setup but I had originally missed that step and was using the system installed version which was too old to install kitchen-pester).

>- Chef Development Kit Version: 2.0.28
> - chef-client version: 13.2.20
> - berks version: 6.2.0
> - kitchen version: 1.16.0
> - inspec version: 1.31.1
> - kitchen-pester: 0.8.0




[learnchef.io]: https://learn.chef.io/
[Pester]: https://github.com/pester/Pester
[Local Development and Testing]: https://learn.chef.io/tracks/local-development-and-testing#/
[kitchen-pester]: https://github.com/test-kitchen/kitchen-pester