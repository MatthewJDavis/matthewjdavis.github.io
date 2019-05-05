---
title: Using an Ansible role from github enterprise where authentication is required in a Jenkins job 
author: Matthew Davis
date: 2018-09-09
excerpt: How to use the SSH Agent plugin for Jenkins to authenticate to a Github Enterprise server to install an Ansible role
categories: 
    - ansible
tags:
    - ansible
    - jenkins
    - git
published: true
---
September 2018

# Intro

I've been setting up a Jenkins instance to run Ansible playbooks and need to install a playbook that was hosted on a Github enterprise server which required authentication.

The playbook is hosted in a public repository on the server, however authentication is required before being allowed to see any of the repos which took me a while to figure out how to specify the credentials in a Jenkins job, then use them to authenticate and install the role for use on the Jenkins build agent during an Ansible playbook run.

## Prerequists

A SSH key should be stored in the Jenkins credential store for a user that has rights to github enterprise. If the repo the playbook is stored in is private, make sure the user has at least read rights the repo.

## Install SSH Agent plugin

The [SSH Agent plugin] for Jenkins allows you to provide SSH credentials to the builds and should be installed form the Jenkins plugin menu (no reboot was required).

![Plugin menu for jenkins, search for ssh agent](/images/jenkins-github-enterprise/ssh-agent-plugin.png)

Now when you open a freestyle project, the build environment section has and option to check for SSH Agent.

![Plugin menu for jenkins, search for ssh agent](/images/jenkins-github-enterprise/build-env-ssh.png)

## Build Environment

Check the box for SSH Agent and select the SSH key for the user that has rights to login to github enterprise.

![Checked ssh agent plugin box and selected ssh key](/images/jenkins-github-enterprise/ssh-key.png)

## Shell build step

Now that the build will be able to authenticate to github enterprise via SSH, you'll be able to download the Ansible role using the [ansible-galaxy] command.

Add an 'Execute shell' build step in Jenkins and run the ansible-galaxy command.

For example, your enterprise github server and role is configured as so:
server address: github.example.com
project name: ansible-roles
repo name: example-ping

The ansible-galaxy install command would look like this:

```bash
ansible-galaxy install git+git@github.example.com:ansible-roles/example-ping.git
```

## Requirements file
If you're using an Ansible requirements file for ansible-galaxy, the file and command would look like:

requirements.yml

```yaml
---
- src: git+git@github.example.com:ansible-roles/example-ping.git
```

```bash
ansible-galaxy install -r requirements.yml
```

The important bit here is the git+ then you can get the rest of the command from the repository when selecting clone and choosing ssh from the drop down (example below taken from main github.com, doing the same from a private github enterprise server will give the correct URL).

![Get SSH url from github repo](/images/jenkins-github-enterprise/ssh-url.png)

## Ansible step

Now in the next step of executing the Ansible playbook that uses that role, it will be able to execute the role that has been saved to the Jenkins build agent.

## Summary

Looking at the best practices of creating Ansible roles, the suggestions are to create a separate repository per role (this is also the case for roles installed from the main Ansible Galaxy). This caused me a problem at first because github enterprise required a login to download the role to run locally even if the repository was public. This wasn't a problem when testing Ansible from my terminal as I already had my SSH key loaded to authenticate with github enterprise but when running the playbook from a Jenkins job, this required the job to be able to authenticate with github enterprise and download the role.

Thankfully, the SSH Agent plugin allowed me to do this and then using the ansible-galaxy utility, the role was downloaded and available to use in the Ansible playbook run.

[SSH Agent plugin]: https://wiki.jenkins.io/display/JENKINS/SSH+Agent+Plugin
[ansible-galaxy]: https://galaxy.ansible.com/docs/using/installing.html