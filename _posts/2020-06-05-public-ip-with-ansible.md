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
published: true
---
June 2020

# Overview

Short post on how to use Ansible to get the public IP address of the network you are currently using, querying the [ipify api].

```yaml
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

The IP address can then be used in other tasks such as creating the inbound rules in an Azure Network Security Group to lock down ssh access to a VM to your IP address.

```yaml
    - name: Create security group that allows SSH
      azure_rm_securitygroup:
        resource_group: "{{ rg_name }}"
        name: "{{ nsg_name }}"
        rules:
          - name: SSH
            protocol: Tcp
            destination_port_range: 22
            access: Allow
            priority: 101
            direction: Inbound
            source_address_prefix: "{{ public_ip.json.ip }}/32"
```

[ipify api]: https://www.ipify.org/