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
published: true
---
February 2020

# Overview

I was working on an Ansible playbook to remove a [DynamoDB] table for a tear down job for features based on their tag values using the Ansible [DynamoDB module] which was going well until it came to features using multiple tables. For the module to work, you need to supply the table's name and there is no native Ansible module that allows you to get the name of the DynamoDB tables in the AWS accounts. A way to get around this and what I show below is to use the [AWS Cli] and Ansible [command module].

Below is the script I used to achieve this, including how to create a new list of just the table names that I read about on [Jeff Geerling's blog].

### Playbook

Console with 4 DynamoDB tables

![Console showing tables](/images/get-dynamodb-ansible/dynamodb-console.png)

<script src="https://gist.github.com/MatthewJDavis/c839c9619a0245f602b39ab7619ced2b.js"></script>

Running the script above, returns 3 of the tables that match the tag values provided

![Output of playbook to get table names](/images/get-dynamodb-ansible/output-playbook.png)

### To remove the tables

I add the code below to the script. This use the DynamoDB table module to remove all the tables in the list I had created by looping over that list and passing the module each table name to remove.

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

Console has been updated

![Removed tables from the console](/images/get-dynamodb-ansible/console-removed.png)

## Summary

Although you currently you can't use the dynamodb_table module to get a list of tables, using the command module, AWS CLI and the Ansible split function to get the table name is a good solution to get DynamoDB table names by given tag values.
This can be used in the automation of Dynamodb tables such as deleting them or modifying them using Ansible playbooks.

[DynamoDB]: https://aws.amazon.com/dynamodb/
[DynamoDB module]: https://docs.ansible.com/ansible/latest/modules/dynamodb_table_module.html
[AWS Cli]: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html
[command module]: https://docs.ansible.com/ansible/latest/modules/command_module.html#command-module
[Jeff Geerling's blog]: https://www.jeffgeerling.com/blog/2017/adding-strings-array-ansible