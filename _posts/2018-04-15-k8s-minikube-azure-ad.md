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

### Kube server API application

1. Sign into the Azure Portal
2. Select the Azure Active Directory Blade
3. Select App registrations
4. Click New application registration

[api server azure ad](/images/k8-minikube-azure-ad/api-server-app-new.png)

1. Name: demo-k8s-api-server
2. Application type: Web app  / API
3. Sign-on URL: https://localhost
4. Click Create

[create api app](/images/k8-minikube-azure-ad/api-server-app-create.png)

Copy the application id and make a note of it

Api server = ba75cd2a-26ef-4aae-840c-02a00d0ad4c5

[copy server app id from settings](/images/k8-minikube-azure-ad/api-server-app-id.png)

### Kubectl app
Same as api server app:
1. Select the Azure Active Directory Blade
2. Select App registrations
3. Click New application registration

Fill out:
Name: demo-k8s-kubectl
Application type: Native
Redirect URI: https://localhost
Click Create

[create api app](/images/k8-minikube-azure-ad/kubectl-app-create.png)

Copy the application id and make a note of it

Kubectl = 076e3eaf-cf92-4478-a533-5faf5d5f7c96

[kubectl app id from settings](/images/k8-minikube-azure-ad/kubectl-app-id.png)

### App Permissions




[](/images/k8-minikube-azure-ad/.png)

[](/images/k8-minikube-azure-ad/.png)

[Kubernetes (K8s)]:https://kubernetes.io/
[minikube]:https://kubernetes.io/docs/getting-started-guides/minikube/
[kube-apiserver]:https://kubernetes.io/docs/reference/generated/kube-apiserver/
[kubectl]:https://kubernetes.io/docs/reference/generated/kubectl/kubectl/



[](/images/k8-minikube-azure-ad/.png)
