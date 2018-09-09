---
title: Using an Ansible role from github enterprise where authentication is required in a Jenkins job 
author: Matthew Davis
date: 2018-09-09
excerpt: 
categories: 
    - ansible
tags:
    - ansible
    - jenkins
    - git
published: false
---

# Intro

I've been setting up a Jenkins instance to run Ansible playbooks and need to install a playbook that was hosted on a Github enterprise server that required authentication. 
The playbook was hosted in a public repository on the server, however authentication is required before being allowed to see any of the repos which took me a while to figure out how to specify the credentials in a Jenkins job, then use them to authenticate and install the role for use on the Jenkins build agent during an Ansible playbook run.

## Prerequists

An SSH key should be stored in the Jenkins credential store for a user that has rights to github enterprise. If the repo the playbook is stored in is private, make sure the user had read rights the repo, if the repo is public then it should be OK.

## Install SSH Agent plugin

The [SSH Agent plugin] for Jenkins allows you to provide SSH credentials to the builds and should be installed form the Jenkins plugin menu (no reboot was required).

![Plugin menu for jenkins, search for ssh agent](/images/jenkins-github-enterprise/ssh-agent-plugin.png)

Now when you open a freestyle project, the build environment section has and option to check for SSH Agent.
![Plugin menu for jenkins, search for ssh agent](/images/jenkins-github-enterprise/build-env-ssh.png)

## Build Environment

Check the box for SSH Agent and select the SSH key for the user that has rights to login to github enterprise.

## Shell build step

Now that the build will be able to authenticate to github enterprise via SSH, you'll be able to download the Ansible role using the [ansible-galaxy] command.
Add an 'Execute shell' build step in Jenkins and run the ansible-galaxy command.

For example, your enterprise github server and role is configured as so:
server address: github.example.com
project name: ansible-roles
repo name: example-ping

The ansible-galaxy install command would look like this:

```bash
ansible-galaxy install 'git+git@github.example.com:ansible-roles/example-ping.git'
```

## Ansible step

Now in the next step of executing the Ansible playbook that uses that role, it will be able to execute the role that has been saved to the Jenkins build agent


[SSH Agent plugin]: https://wiki.jenkins.io/display/JENKINS/SSH+Agent+Plugin
[ansible-galaxy]: https://galaxy.ansible.com/docs/using/installing.html