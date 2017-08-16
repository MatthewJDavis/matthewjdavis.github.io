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
published: false
---

I've been learning chef for configuration management through their excellent learning resources at [learnchef.io]. It's a great way to learn chef and has some excellent resources and tutorials that cover Linux and Windows configuration management with chef.

I've completed a number of the tracks and I'm currently working through the [Local Development and Testing] track and although inspec was cool, I'd much prefer to run integration tests with Pester, the PowerShell testing framework (which I'm also learning and incorporating it as much as possible).

It took me a bit of time and googling to get this up and running so here's an overview.
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

For example, a recipe call uk_settings, create a Pester test file like so:
team_city_agent/test/integration/uk_settings/pester/uk_settings.Tests.ps1
 

Now when you run ```ruby kitchen verify``` your Pester tests will run for the integration tests.

I'm enjoying learning chef and am glad I can use Pester tests within the chef framework to test the infrastructure created.

[learnchef.io]: https://learn.chef.io/
[Local Development and Testing]: https://learn.chef.io/tracks/local-development-and-testing#/
[kitchen-pester]: https://github.com/test-kitchen/kitchen-pester