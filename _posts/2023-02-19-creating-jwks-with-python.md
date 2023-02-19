---
title: How to create JSON Web Key Sets (JWKS) with Python
excerpt: Creating a JSON Web Key Sets (JWKS) with Python for use with an Okta service application to authenticate with Terraform.
date: 2023-02-19
toc: false
classes: wide
categories:
- jwks
tags:
- python
- jwks
- okta
published: true
---

# Overview

Something that I've had to do more of recently is create [JSON Web Key Sets](https://auth0.com/docs/secure/tokens/json-web-tokens/json-web-key-sets) commonly known as JWKS -private and public keys in JSON format for [Terraform to authenticate to Okta](https://registry.terraform.io/providers/okta/okta/latest/docs) via [OAuth service applications](https://developer.okta.com/docs/guides/implement-oauth-for-okta-serviceapp/main/). There are plenty of online options out there but when running in production, you'll want to create them locally and not rely on a third party solution. This works for Okta service accounts but the JWKS can be used for other applications and the script updated to create keys with other algorithms and settings.

The script below shows how to achieve this with Python and OpenSSL running on Ubuntu and can be used as a starting point for other operating systems to generate keys locally.

Because the keys are being used in automation (with Terraform to provision Okta resources) they are not protected with a password. These keys should be moved to secure storage such as a password manager / key vault for automation ASAP.

Tested with:

- Python 3.8.10
- Ubuntu 20.04.5 LTS

In a virtual environment - install the jwcrypto module if not already installed.

```bash
pip install jwcrypto
```

Copy the contents of main.py to a local main.py file.

<script src="https://gist.github.com/MatthewJDavis/bca7426df03eb74695b206a7201869f1.js"></script>

```bash
python main.py
```

4 keys will be created in the 'keys' directory.

The keys are now ready to be used and can be setup to authenticate Terraform with Okta. Terraform requires the private key in RSA format so use the key ```service_app_keys_rsa.pem``` for any Terraform scripts.
