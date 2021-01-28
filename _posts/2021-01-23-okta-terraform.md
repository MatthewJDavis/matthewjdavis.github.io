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

This post will cover adding an application to Okta with Terraform.

I was following the tutorial [ASP.NET Core 3.0 MVC Secure Authentication] that required an application to be created, I had heard there was a Terraform provider available for Okta so decided to try it out and have documented the process below.

Terraform allows you to manage resources such as AWS, Azure and many other providers including Okta with the Hashicorp language.

[Install Terraform] following the official guide. I am using the latest version available at this time: v0.14.5. This was carried out on a Ubuntu system using bash so adjust environment variables and commands to other systems as required.

This is not an extensive tutorial on Terraform, the [official tutorial site] is a great resource and I highly recommend checking it out. The details in this post should be enough to get you up an running with Terraform and Okta.

## Set up Okta dev account

Register for an [Okta developer account] if you've not already done so - currently it is free and allows you to create 5 applications to test out.

Login to the dev portal and create an API token for the user keep this safe and secure, it's need to create a less privilege user shortly (alternatively you can create another user via the portal and assigning it to the Application Administrator role and skip down to the Create Application section).

![Create API token in portal](/images/terraform-okta/api-token.png)

## Local directory structure

I created two directories, one for the service account Terraform and one for the application Terraform. This is because the service account user that I create for managing applications will not have the required permissions to manage users accounts so I have separated out the files. You don't have to create the okta-service-account directory if you create the application service account through the portal.

![Directory structure](/images/terraform-okta/tree.png)

```bash
# Create application dir and files
mkdir okta-application
touch okta-application/variables.tf
touch okta-application/main.tf
#Create service account dir and files
mkdir okta-service-account
touch okta-service-account/vairables.tf
touch okta-service-account/main.tf
```

## Create a service account

The best practice is to use least privilege accounts so I will create a specific account for Terraform to manage applications. This can be done via the portal or in production the account could be mastered from another source such as Active Directory. I'll show how to create the account in Terraform using the super admin account as a one off.

```bash
terraform init
```

![Terraform init](/images/terraform-okta/terraform-init.png)

Next is to set two environment variables. If you put a space before the export statement it stays out of your bash history.

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

![Terraform plan output](/images/terraform-okta/terraform-plan.png)

To create the user run apply with Terraform, you'll be prompted to enter ```yes``` and Terraform will create the user for you in Okta.

```bash
terraform apply
```

![Terraform init](/images/terraform-okta/terraform-apply.png)

The user will now be visible in the Okta portal.

![Okta user portal](/images/terraform-okta/users.png)

## Creating the Application

Now the user is created, login to the Okta portal with that user and generate an API token, same process as before and keep it in a safe place. This token will be scoped to the user that created it so only has application permissions and can't manage users etc.

Update the token environment variable so it is using the application service account user API token.

```bash
 export OKTA_API_TOKEN="token_here"
```

In the directory ```okta-application`` there are the following two files.

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

![terraform plan output](/images/terraform-okta/terraform-plan-app.png)

Now we can run the apply command to create the application.

```bash
terraform apply
```

![Terraform apply output](/images/terraform-okta/terraform-apply-app.png)

Below is a screenshot of the application created in the Okta dashboard.

![okta portal with created appliation](/images/terraform-okta/apps-portal.png)

[ASP.NET Core 3.0 MVC Secure Authentication]: https://developer.okta.com/blog/2019/11/15/aspnet-core-3-mvc-secure-authentication
[Install Terraform]: https://learn.hashicorp.com/tutorials/terraform/install-cli
[Okta developer account]: https://developer.okta.com/signup/
[Okta user resource]: https://registry.terraform.io/providers/oktadeveloper/okta/latest/docs/resources/user
[official tutorial site]: https://learn.hashicorp.com/terraform?utm_source=terraform_io