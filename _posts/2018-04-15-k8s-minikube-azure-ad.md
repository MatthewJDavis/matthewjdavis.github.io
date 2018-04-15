---
title: Running Kubernetes on minikube and using authentication with Azure AD
author: Matthew Davis
date: 2018-04-15
excerpt: 
categories: 
    - Kubernetes
tags:
    - active directory
    - minikube
published: false

# Intro

I was recently asked to investigate how to authenticate our proof of concept [Kubernetes (K8s)] cluster with our Active Directory. 

One of the nice to have requirements was to make it not reliant on our internal network where are AD servers reside. We sync our AD with Azure AD so it would be great if this could be used as authentication to a Kubernetes cluster and it can be done. 

I was using [minikube], which is a great way to get up and running quickly with K8s so this post will show you how to configure k8s running on minikube to authenticate with Azure AD. 

Once you get the work flow with minikube, it is straight forward to set it up with a K8s cluster hosted elsewhere.

## Azure AD applications
We need two Azure AD apps:
1. For the [kube-apiserver]
2. For [kubectl], the kubernetes commandline client

### Kube server API
Sign into the Azure Portal
Select the Azure Active Directory Blade
Select App registrations
Click New application registration



[Kubernetes (K8s)]:https://kubernetes.io/
[minikube]:https://kubernetes.io/docs/getting-started-guides/minikube/
[kube-apiserver]:https://kubernetes.io/docs/reference/generated/kube-apiserver/
[kubectl]:https://kubernetes.io/docs/reference/generated/kubectl/kubectl/