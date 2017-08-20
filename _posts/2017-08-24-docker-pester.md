---
title:  "Using Docker container for Pester testing"
author: Matthew Davis
date: 2017-08-24
categories: 
    - PowerShell
tags:
    - pester
    - docker
    - testing
published: false
---


docker run --rm -ti -v ~/Documents/PowerShell:/tests microsoft/powershell Invoke-Pester -Script /tests/New-Test.Tests.ps1 -OutputFile /tests/results.xml -OutputFormat NUnitXml
