---
title: Use Terraform to import and manage existing Okta applications
excerpt: 
date: 2021-01-23
toc: false
classes: wide
categories:
- terraform
tags:
- terraform okta
published: false
---
January 2021

# Overview

This week I've started using Okta for work and have been getting familiar with it. I followed the tutorial on integrating a dotnet application with Okta and was aware that there is a Terraform provider for Okta so decided to take a look on how it works. Terraform allows you to manage resources such as AWS, Azure and many other providers with the Hashicorp language.
This post will cover getting setup with an Okta developer account and using Terraform to manage the applications in it.

[Install Terraform] following the official guide. I am using the latest version available at this time: v0.14.5. This was carried out on a Ubuntu system using bash so adjust environment variables to other systems as required.

## Set up Okta dev account

Register for an [Okta developer account].

Login to the portal and create an API token for the user keep this safe and secure, it's need to create a less privilege users shortly.

## Create an Application admin

The best practice is to use least privilege accounts so I will create a specific account for Terraform to manage applications. This can be done via the portal or in production the account could be mastered from another source such as Active Directory. I'll show how to create the account in terraform using the super admin account as a one off.

```bash
terraform init
```

![Terraform init](/images/terraform-okta/terraform-init.gif)

Next is to set two environment variables. If you put a space before the export statement then they stay out of your bash history.

```bash
 export OKTA_API_TOKEN="token_here"
 export TF_VAR_password="service_account_password_here"
```

terraform plan
terraform apply






[Install Terraform]: https://learn.hashicorp.com/tutorials/terraform/install-cli
[Okta developer account]: https://developer.okta.com/signup/
[Okta user resource]: https://registry.terraform.io/providers/oktadeveloper/okta/latest/docs/resources/user