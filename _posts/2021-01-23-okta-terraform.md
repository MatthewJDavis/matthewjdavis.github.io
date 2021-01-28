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

## Create a service account

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

In the directory ```okta-demo-svc-account`` there are the following two files.

<script src="https://gist.github.com/MatthewJDavis/41a04b3a6d29b3eab129df821128a9dd.js"></script>

<script src="https://gist.github.com/MatthewJDavis/03a179d056c33081db5c36a4ad1dbb72.js"></script>

Running a plan first with terraform will show you what is going to be added, changed or destroyed and is always a good idea.

```bash
terraform plan
```

![Terraform plan output](/images/terraform-okta/terraform-plan.gif)

To create the user run apply with terraform, you'll be prompted to enter ```yes``` and terraform will create the user for you in Okta.

```bash
terraform apply
```

![Terraform init](/images/terraform-okta/terraform-apply.gif)

The user will now be visible in the Okta portal.

![Okta user portal](/images/terraform-okta/users.gif)

## Creating the Application

Now the user is created, login to the Okta portal with that user and generate an API token, same process as before and keep it in a safe place. This token will be scoped to the user that created it so only has application permissions and can't manage users etc.

Update the token environment variable so it is using the application service account user API token.

```bash
 export OKTA_API_TOKEN="token_here"
```

In the directory ```okta-demo-application`` there are the following two files.

<script src="https://gist.github.com/MatthewJDavis/9741a1ca57c53a682f1378976e2c4f7b.js"></script>

<script src="https://gist.github.com/MatthewJDavis/54f794ff60ed97c77f6ab3bd65067f3e.js"></script>

Because this is a new directory, terraform require initialisation again.

```bash
terraform init
```

As before, we can run a plan to see what is going to be added.

```bash
terraform plan
```

![terraform plan output](/images/terraform-okta/terraform-plan-app.gif)

```bash
terraform apply
```

![terraform apply output](/images/terraform-okta/terraform-apply-app.gif)

Below is a screenshot of the application created in the Okta dashboard.

![okta portal with created appliation](/images/terraform-okta/apps-portal.gif)


[Install Terraform]: https://learn.hashicorp.com/tutorials/terraform/install-cli
[Okta developer account]: https://developer.okta.com/signup/
[Okta user resource]: https://registry.terraform.io/providers/oktadeveloper/okta/latest/docs/resources/user