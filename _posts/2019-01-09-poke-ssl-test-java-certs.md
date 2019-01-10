---
title: Use SSL Poke to test Java SSL connection
author: Matthew Davis
date: 2019-01-09
toc: true
excerpt: How to use SSL Poke to test Java SSL connections and verify the cacerts file has the correct certificates.
categories:
    - java
tags:
    - java
    - ssl
    - cacerts
published: false
---

# Overview

Java uses the cacerts file as its certificate authority to validate https connections made with Java application to sites. It is useful to be able to verify that the cacerts file has the correct certificates added to it to connect securely to internal sites (when using internal PKI to issue certificates).
Atalassian has created a small Java program to test connectivity.

Here's how to use it.

This is tested on Ubuntu with Java default-jdk 1.7 installed and on the user's path.

# Create SSLPoke.java and paste the code below into it

vim SSLPoke.java

<script src="https://gist.github.com/MatthewJDavis/50f3f92660af72c812e21b7ff6b56354.js"></script>

![The created poke file](/images/java-ssl-poke/poke-file.png)

If you haven't got the jdk, you can install it with on ubuntu with:

```bash
apt-get install default-jdk -y
```

Now run Javac:

```bash
javac SSLPoke.java
```

This produces a Java class file.

![Running javac with the class file output](/images/java-ssl-poke/ssl-poke-class-file.png)


To run the test, run the following with a hostname and port:

```bash
java SSLPoke hostname port
# Example for google
java SSLPoke google.com 443
```

Here is a successful connection to google.com.

![Connecting successfully to google.com with SSL Poke](/images/java-ssl-poke/connection-success.png)

And here is the error if the cert is not in the cacerts file (this is my test Jenkins instance running with a self signed certificate)

```bash jenkins.matthewdavis111.com 443
java SSLPoke google.com 443
```

![Connection failed to my test Jenkins instance with SSL Poke](/images/java-ssl-poke/connection-failed.png)

Below is the same Jenkins instance being trusted by chrome because I added it into the Windows certificate store. To get Java to trust it, I would have to do the same with the cacerts file.

![Connection to Jenkins ok with chrome](/images/java-ssl-poke/jenkins-cert-trusted.png)
