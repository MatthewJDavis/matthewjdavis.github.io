---
title: Create a self signed certificate for local development
author: Matthew Davis
date: 2018-09-03
excerpt: Create a self signed certificate and use it with HAProxy and in the Windows store for local development with HTTPS sites
categories: 
    - certificates
tags:
    - certificates
    - haproxy
    - powershell
published: false
---

I have been creating a Jenkins runbook with Ansible to configure a Jenkins server. I wanted to make this installation as realastic as possible so that meant using https to connect to Jenkins. To achive this I used an HAProxy loadbalancer to do the SSL offloading. For this, I needed a PEM certificate to apply to the loadbalancer but as I didn't want to payout for a certificate and you can't use let's encrypt on localhost, I followed the let's encrypt guide on how to create a self signed certificate.

I written down the process here and also added how to extract the PEM from the cert and the x509 certificate that can then be imported into the keystore so the self signed certificate is trusted.
This shows how to do it using Windows 10 with Windows Subsytem for Linux (WSL) in stalled.

In WSL, create the certificate and private key using open-ssl (this is taken directly from the let's encrypt article)
```bash
#variable to hold domain name, replace with desired domain
domain='jenkins.matthewdavis111.com' 

#create the certificate and private key
openssl req -x509 -out $domain.crt -keyout $domain.key \
  -newkey rsa:2048 -nodes -sha256  \
  -subj "/CN=$domain" -extensions EXT -config <( \printf "[dn]\nCN=$domain\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:$domain\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")
```

![Creating the certificate and private key using Windows Subsystem for Linux](/images/self-signed-cert/create-cert.png)

Create the PEM file by combining the certificate and the private key

Note: because this file contains the private key, extra care should be taken with it and should be treated as a password.

```bash
cat $domain.crt $domain.key >> jenkins.pem
```

# Create cer cert from the pem for Windows root store
```bash
openssl x509 -outform der -in jenkins.pem -out jenkins.matthewdavis111.cer
```

Copy the cer file to a location Windows can access (you can access the file from within Windows, but it is reccommended not to modify any of them and to make it easier, I just copy to a temp location at the root of the c directory which is found in the directory path of /mnt/c in WSL) https://www.howtogeek.com/261383/how-to-access-your-ubuntu-bash-files-in-windows-and-your-windows-system-drive-in-bash/

```
cp jenkins.matthewdavis111.cer /mnt/c/TEMP/
```

To import the certificate to the Windows store, use PowerShell running as an administrator

```
#Import self signed cert to trusted root
Import-Certificate -FilePath C:\TEMP\jenkins.matthewdavis111.cer -CertStoreLocation Cert:\LocalMachine\Root\
```

Now you'll be able to use the pem file with HAProxy (or whatever you are testing) and the connection will be trusted over SSL when using Google Chrome (you'll need to shutdown and restart chrome). This doesn't work with Firefox which has it's own separate certificate store.

Finally to tidy up and remove the self signed cert after testing has finished, run the following in PowerShell as an administrator

```powershell
#Clean up the cert from the root store
Get-ChildItem Cert:\LocalMachine\Root\ | Where-Object -FilterScript {$_.subject -like "*jenkins.matthewdavis111.com*"} | Remove-Item
```