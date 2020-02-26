---
title: Use the Azure VM Extension provider in Terraform to run a PowerShell script on deployment
excerpt: 
date: 2020-02-28
toc: false
classes: wide
categories:
- azure
tags:
- azure virtual machine
- powershell
- terraform
published: false
---
February 2020

# Overview

I'm not going to cover the basics or setting up Terraform, the [official docs] are great and Hashi Corp provide a guide for [Azure] and so do Microsoft with this post on how to set up [Terraform for Azure].

https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/custom-script-windows

https://www.terraform.io/docs/providers/azurerm/r/virtual_machine_extension.html

## Azure VM Extension resource

```
resource "azurerm_virtual_machine_extension" "main" {
  name                 = "init"
  location             = "${azurerm_resource_group.main.location}"
  resource_group_name  = "${azurerm_resource_group.main.name}"
  virtual_machine_name = "${azurerm_virtual_machine.main.name}"
  publisher            = "Microsoft.Compute"
  type                 = "CustomScriptExtension"
  type_handler_version = "1.9"

  settings = <<SETTINGS
    {
        "fileUris": [
            "https://gist.githubusercontent.com/MatthewJDavis/4dd470641f69316e8655f7fe4c6eb13d/raw/ffd92866b724d6b96acd9ef701bc4226c7eaed2a/init.ps1"
        ],
        "commandToExecute": "powershell -ExecutionPolicy Unrestricted -File init.ps1"
    }
  SETTINGS
}
```


Full script

<script src="https://gist.github.com/MatthewJDavis/8e99e42678c190ec6c044aca24a36398.js"></script>

## Summary

[Azure]: https://learn.hashicorp.com/terraform?track=azure#azure
[official docs]: https://www.terraform.io/intro/index.html
[Terraform for Azure]: https://docs.microsoft.com/en-us/azure/terraform/terraform-install-configure