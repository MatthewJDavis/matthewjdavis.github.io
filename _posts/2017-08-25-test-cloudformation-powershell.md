---
title: Test-CFNTemplate PowerShell
author: Matthew Davis
date: 2017-08-17
excerpt: How to test an AWS cloudformation template with PowerShell
categories: 
    - aws
tags:
    - aws
    - cloudformation
    - powershell
---

This stumped me for a few hours yesterday while wanting to deploy a cloudformation stack to AWS via PowerShell using New-CFNStack instead of the console, I kept getting the following error:

```powershell
Test-CFNTemplate : Template format error: unsupported structure.
```

Not overly helpful, I checked the path was correct, used relative and full file paths and tried with file:// before the file name but the error was still the same.

To check that there wasn't an error in my cloudformation yaml file, I deployed it successfully via the AWS console and found the Test-CFNTemplate command that accepted the -TemplateBody parameter but still failed with the same error message as New-CFNStack

Checking the help I realised the parameter was requires the type System.String and not a file


>    -TemplateBody <System.String>
>        Structure containing the template body with a minimum length of 1 byte and a maximum length of 51,200 bytes. For more information, go to Template Anatomy in the AWS CloudFormation User Guide. 
>        Conditional: You must specify either the TemplateBody or the TemplateURL parameter, but not both.

>        Required?                    False
>        Position?                    Named
>        Default value                None
>        Accept pipeline input?       False
>        Accept wildcard characters?  false

