---
title: Get DynamoDB table names with Ansible and AWS Cli by tag values
excerpt: 
date: 2020-02-27
toc: false
classes: wide
categories:
- aws
tags:
- ansible
- aws cli
published: false
---
February 2020

# Overview

I was working today on an Ansible playbook to remove a DynamoDB table for a tear down job for features using the Ansible [DynamoDB module] which was going well until it came to deleting multiple tables. For the module to work, you need to supply the table's name and there is no native Ansible module that allows you to get the name of the DynamoDB tables in the AWS accounts. A way to get around this and what I show below is to use the [AWS Cli] and Ansible [command module].

Below is the script I used to achieve this, including how to create a new list of just the table names that I read about on [Jeff Geerling's blog].


### Playbook

<script src="https://gist.github.com/MatthewJDavis/c839c9619a0245f602b39ab7619ced2b.js"></script>

### To remove the tables

I add the code below to use the DynamoDB table module to remove all the tables in the list I had created by looping over that list and passing the module each table name to remove.

```yml
  - name: remove DynamoDB table
    dynamodb_table:
      name: "{{ item }}"
      region: "{{ region }}"
      state: absent
    with_items: "{{ dynamodb_table_names }}"
```

Output of deleting tables playbook run

![Output of removed playbook run](/images/get-dynamodb-ansible/remove-dynamodb.png)

Removed from the console

![Removed tables from the console](/images/get-dynamodb-ansible/console-removed.png)


## Summary

[DynamoDB module]: https://docs.ansible.com/ansible/latest/modules/dynamodb_table_module.html
[AWS Cli]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html
[command module]: https://docs.ansible.com/ansible/latest/modules/command_module.html#command-module
[Jeff Geerling's blog]: https://www.jeffgeerling.com/blog/2017/adding-strings-array-ansible