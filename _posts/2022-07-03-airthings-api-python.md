---
title: Airthings consumer API with Python
excerpt: How to authenticate and query the Airthings API with Python
date: 2022-07-03
toc: false
classes: wide
categories:
- python
tags:
- airthings
- python
- api
published: true
---
July 2022

# Overview

Quick guide on how to authenticate and query the [Airthings consumer API] to pull data from the air quality monitoring devices registered with the account.

## Code

Here's the code to authenticate and pull sensor data to get going.

<script src="https://gist.github.com/MatthewJDavis/e26f388b68d27dfede3ba8bbb1d213fa.js"></script>

## Script details

Requires Python 3.6 and above due to use of [f-strings].

The requests package is a requirement and should be installed in a virtual environment or container with ```pip install requests```.
The script calls the authentication server to request a bearer token to authenticate the calls to the API, using the client credentials [Oauth2 flow].

You'll need the API token and client id which can be created: <https://dashboard.airthings.com/integrations/api-integration> with the scopes '''read:device:current_values''' checked.

Before running the script, the secret should be exported to an environment variable. That can be done by:

```bash
# Linux and Mac
 export secret="secret-key"
# Windows
 $Env:secret = 'secret-key'
```

Update the *client_id* and *device_id* variable values - these can be viewed in the Airthings website.

```bash
python main.py
```

That should then pull the latest data from the API for that particular device. Individual values can be accessed via the dictionary returned from the API as demonstrated in the script with the co2 value.

[Oauth2 flow]: https://www.rfc-editor.org/rfc/rfc6749#section-4.4.2
[Airthings consumer API]: https://developer.airthings.com/consumer-api-docs
[f-strings]: https://realpython.com/python-f-strings/