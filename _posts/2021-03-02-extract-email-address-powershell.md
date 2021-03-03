---
title: Use PowerShell to extract an email address from a string
excerpt: Short post on how to use PowerShell and regex to extract an email address from a string.
date: 2021-03-02
toc: false
classes: wide
categories:
- powershell
tags:
- powershell
published: true
---
March 2021

Short post on how to use PowerShell and regex to extract an email address from a string.

I was recently given a spreadsheet of Azure AD groups and users that should added to those groups. The users were in various formats but did include their email address per line (which matches their UPN in Azure AD).

I used the following script to extract the user's email address to a list and then used the list to update the AzureAD group (I used [Import-CSV] Cmdlet to assign the ```$userList values``` - in the below example the strings are assigned directly for demonstration purposes).

<script src="https://gist.github.com/MatthewJDavis/74743afd13e54afd171289e1c4f70a3d.js"></script>

## How it works

- A [list] of [string types] is created, initialised and assigned to the variable ```$emailList```.
- A foreach loop is used to iterate over the user details assigned to the ```$userList``` variable. 
- Each string is matched against the [email regex pattern] and if a portion of the string matches the regex, it is stored in the [$Matches] automatic variable value property. 
- This value is added to the $emailList.

![Showing output of email addresses from script](/images/ps-email-extract/ps-email-extract.png)

[list]: https://docs.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1?view=net-5.0
[string types]: https://docs.microsoft.com/en-us/dotnet/api/system.string?view=net-5.0
[$Matches]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_automatic_variables?view=powershell-7.1#matches
[email regex pattern]: https://emailregex.com/
[Import-Csv]: https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-csv?view=powershell-7.1