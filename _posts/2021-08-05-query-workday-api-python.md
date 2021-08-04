---
title: Query Workday API using Python and the Zeep package
excerpt: How to authenticate with the Workday SOAP API and query for a worker by worker ID using Python and Zeep.
date: 2021-08-05
toc: false
classes: wide
categories:
- python
tags:
- zeep
- python
- api
published: false
---
August 2021

# Overview

I have spent the last 2 days attempting and finally succeeding in querying the Workday API for a user by their ID using Python and Zeep and couldn't find a solid example so have recorded here the solution and the process of getting to it. I am by no means a Python expert and definitely not a Workday expert having only used it as an employee in the past.

## Script

Requires the Zeep package to be installed

```bash
pip install zeep
```

<script src="https://gist.github.com/MatthewJDavis/5823fa942fe753d320417eab2bc36a03.js"></script>

## Process

Having not had the joy of using a SOAP based API in a long long time, I set off Googling ways to do this with Python. I found the Zeep package so started there then I came across the Workday package but found that it caused an error for me and the project is now archived in GitHub. Going back to Zeep I read a few tutorials that got me on my way and once I was given the correct credentials for the Workday API user, I was able to return a list of workers however was still stuck on how to actually specify a particular user.

Reading these posts, including this one on Stack Overflow that shows the required xml and going back to the documentation a lot, I finally managed to figure out what was required to send the request to the SOAP endpoint and get back the correct worker data.

### Dumping the wdsl

First thing to do was dump the wdsl using Zeepv

```bash
workday_url = 'https://<hostname>.workday.com/ccx/service/<tenant>/Human_Resources/v36.2?wsdl' 
python -m zeep  $workday_url >> hr.txt
```

Then referencing the Workday API docs and the hr.txt I found that [Worker_Request_References] is used *"to retrieve a specific instance(s) of Worker and its associated data."* The parameter name is Worker_Reference and takes the [type WorkerObjectID].

Following along with the Zeep docs on [datastructures] I found we can use the get_type method of the client to create a query. *"Most of the times you need to pass nested data to the soap client. These Complex types can be retrieved using the client.get_type() method."*

Referencing the hr.txt wdsl dump I searched for 'Worker_Request' and found 'ns0:Worker_Request_ReferencesType'. Inspecting the object looked like I was on the right track so I added the ID I was looking for as per the docs and tried that but got an exception.

```python
ref_type = client.get_type('ns0:Worker_Request_ReferencesType'). 
request = ref_type(Worker_Reference='1234')
client.service.Get_Workers(request)

zeep.exceptions.Fault: Validation error occurred. Invalid ID 'type'  attribute: NotSet.  Valid Types: WID 
```

Inspecting the request object I could see that it was kind of right but also missing the type as mentioned in the docs and also in the xml example on the stackoverflow post: https://stackoverflow.com/questions/53440759/workday-get-worker-from-user-id-or-email-filter and what I had be shown in Workday Studio.

![Output of the object in Python](/images/python-workday/ref-object.png)

[Further down] in the Zeep datastructures docs and mentioned in this blog post it shows how to set a nest value so after using the request object as a starting point and some trial and error creating the dictionary that finally worked.

```python
request_dict = { 
    'Worker_Reference': { 
        'ID': { 
            'type': 'Employee_ID', 
            '_value_1': employee_id 
        }, 
        'Descriptor': None 
    }, 
    'Skip_Non_Existing_Instances': None, 
    'Ignore_Invalid_References': None 
}

client.service.Get_Workers(request_dict) 
```

This returned the data for my user in the test instance.

![data returned from Workday](/images/python-workday/returned-data.png)

## Summary

Hopefully this will help someone who is struggling querying the Workday API like I was.

[Worker_Request_References]: https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v36.2/Get_Workers.html#Worker_Request_ReferencesType
[type WorkerObjectID]: https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v36.2/Get_Workers.html#WorkerObjectIDType
[datastructures]: https://docs.python-zeep.org/en/master/datastructures.html
[Further down]: https://docs.python-zeep.org/en/master/datastructures.html#xsd-choice