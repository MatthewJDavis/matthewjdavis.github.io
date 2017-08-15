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

1. Install kitchen-pester
 - Open up the chefdk 
 - Run ```ruby gem install kitchen-pester ```

2. Open the kitchen.yml file
  Edit the verifier and change it to pester 
  Specify a path to the tests if you want to change from the default location of test/integration
  

  verifier:
      name: pester
      test_folder: test/integration

[learnchef.io]: https://learn.chef.io/
[Local Development and Testing]: https://learn.chef.io/tracks/local-development-and-testing#/