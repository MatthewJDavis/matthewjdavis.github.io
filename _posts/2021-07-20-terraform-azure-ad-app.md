---
title: Use Terraform to deploy an Azure AD application
excerpt: 
date: 2021-07-03
toc: false
classes: wide
categories:
- terraform
tags:
- azuread
- terraform
published: false
---
July 2021

# Overview

How to deploy an Azure AD application using Terraform.

## Set up Terraform

Local state

## Create application with permissions to deploy applications

Service principal.
Graph permissions.

```bash
az ad sp create-for-rbac --skip-assignment --name 'test-terraform-ad'
```
Assign application admin role (or create custom role).

## Create the application and deploy

## Update the application

## Destroy the application

## Azure storage for remote state

##