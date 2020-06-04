---
title: Get the public IP address of a network with Ansible
excerpt: Use Ansible and the ipify api to get the public IP address of the network you are currently connected to.
date: 2020-06-06
toc: false
classes: wide
categories:
- ansible
tags:
- ansible
published: false
---
June 2020

# Overview

```ansible
---
- hosts: localhost
  gather_facts: false
  tasks:
    - name: Get the public IP address of the network.
      uri:
        url: https://api.ipify.org?format=json
        method: Get
      changed_when: false
      register: public_ip
      until: public_ip.status == 200
      retries: 6
      delay: 10

    - name: Display the public IP address of the network.
      debug:
        msg: "{{ public_ip.json.ip }}"
```

Now the public IP address is stored in the public_ip variable, it can be accessed and used in later tasks. The returned result is a dictionary with numerous keys and one of those keys is json which has the value of a dictionary. To access the IP address value requires access the nested dictionary: ``` public_ip.json.ip ```.
