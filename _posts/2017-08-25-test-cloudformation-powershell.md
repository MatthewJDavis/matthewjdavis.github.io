---
title: Test-CFNTemplate PowerShell
author: Matthew Davis
date: 2017-08-25
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

To check that there wasn't an error in my cloudformation yaml file, I deployed it successfully via the AWS console and thought I'd try the Test-CFNTemplate command that accepted the -TemplateBody parameter but still failed with the same error message as New-CFNStack

Checking the help I realised the parameter was requires the type System.String and not a file (I know, RTFM first!!)


>    -TemplateBody '<System.String>'
>        Structure containing the template body with a minimum length of 1 byte and a maximum length of 51,200 bytes. For more information, go to Template Anatomy in the AWS CloudFormation User Guide. 
>        Conditional: You must specify either the TemplateBody or the TemplateURL parameter, but not both.

Next I tried:

```powershell
Test-CFNTemplate -TemplateBody (get-content .\amazon-linux-vm.yaml)
```

This gave the error:
```powershell
Test-CFNTemplate : Cannot convert 'System.Object[]' to the type 'System.String' required by parameter 'TemplateBody'.
```

What gives here?
Get-Content reads the file in and creates a list of objects for each line. If you were to save get content to a variable, then access the first item, you'd get the **whole** first line of the file returned.
```powershell
$template = Get-Content -Path .\amazon-linux-vm.yaml
$template[0]
AWSTemplateFormatVersion: 2010-09-09
```
## Use the -Raw switch
The Raw switch, although not well documented at present, will read in the whole file as one big string object instead of a separate object per line. To demonstrate this:
```powershell
$template1 = Get-Content -Path .\amazon-linux-vm.yaml
$template1[0]
A
```

See this post about Get-Content and why it's not your friend on [powershell.org]

To read the file without line breaks, use the -Raw switch and the cmdlet should now work correctly.

```powershell
Test-CFNTemplate -TemplateBody (Get-Content .\amazon-linux-vm.yaml -Raw)
```

The Test-CFNTemplate will now run correctly and you'll be able to use New-CFNStack to deploy your resources via cloudformation with PowerShell making it easy to change the values of the parameters you pass in.

[powershell.org]: https://powershell.org/2013/10/21/why-get-content-aint-yer-friend/