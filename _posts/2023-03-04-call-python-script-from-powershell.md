---
title: How to call a Python script from PowerShell
excerpt: Creating a JSON Web Key Sets (JWKS) with Python for use with an Okta service application to authenticate with Terraform.
date: 2023-03-04
toc: false
categories:
- powershell
tags:
- python
published: true
---
March 2023

# Overview

A quick one today to remind me how to call a Python script via PowerShell, passing in an argument and saving the output of the script to a variable in PowerShell using a couple of demo scripts that do nothing more than pass a string to the Python script and return it with text added.

Test machine specs:

```bash
Ubuntu 20.04.5 LTS
PSVersion 7.3.2
Python 3.10.5 
```

This should work as is with Python 3.6+ (uses [f-string](https://realpython.com/python-f-strings/) formatting).

I have a virtual environment created with the required Python packages installed.
The Python script simply outputs the text I need via the print function and exits.

Here is the Python script that takes an argument and prints it out.

<script src="https://gist.github.com/MatthewJDavis/07ff817b79348d3ff7a19745a82f7983.js"></script>

The output can be saved to a variable in PowerShell by calling the script like so.

<script src="https://gist.github.com/MatthewJDavis/ba97157071ed2bc8cdfaa0432b3448f7.js"></script>

The PythonProgramPath can either point to the main install of Python (or just Python if it is added to the Path environment variable) or a virtual environment if extra packages are required / installed for the script to run.

I was using this method to work with JWTs. I was struggling to so with PowerShell and already had the Python scripts written so I only had to grab the output of the script to use the JWT values in my PowerShell script.
