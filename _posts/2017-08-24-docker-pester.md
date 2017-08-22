---
title:  "Using Docker container for Pester testing"
author: Matthew Davis
date: 2017-08-24
categories: 
    - PowerShell
tags:
    - pester
    - docker
    - testing
published: false
---

When running Pester tests, it's nice to know that you are running against the correct version of the module you are testing and there are no environment variables that could influence the test in an unexpected way. You can unload all modules and check the environment variables but I was following a [Docker course on lynda.com] and thought it would be pretty cool to use the [PowerShell docker image] from Microsoft to spin up a container, run the tests on a "clean" system and then output the results. 

Note: The docker image runs the PowerShell Core Edition which doesn't have all the modules as the Desktop Edition (see Microsoft overview of [PowerShell editions]). 
So you may not be able to use this method to test your scripts if they rely on cmdlets only available in the Desktop Edition.

For a guide to installing Docker, see the [docker docs on installing].

I did this using PowerShell Core 6 running on a Ubuntu laptop.

I had downloaded the microsoft/powershell Docker image from [Docker Hub] to my local machine from the docker repository.

```bash
docker pull microsoft/powershell
```

I run docker to create a container with a shared drive from my host laptop. The directory on my laptop is ~/Documents/PowerShell and the directory in the container is /tests. 
Once the container is created, Invoke-Pester is executed against the designated script and the results are outputted to a file in the /tests directory which will write to the local directory on the laptop once the container is run and has been removed (the rm command)

docker run --rm -ti -v ~/Documents/PowerShell:/tests microsoft/powershell Invoke-Pester -Script /tests/New-Test.Tests.ps1 -OutputFile /tests/results.xml -OutputFormat NUnitXml

If you want to run the test manually you can add the -ti parameter to docker run to have an interactive terminal session to the container. Then you can change directory to the shared directory (in this example the /tests directory) and access any scripts there for more interactive testing

```bash
docker run --rm -ti -v ~/Documents/PowerShell:/tests microsoft/powershell
```

## Creating a Docker Image with the AWS module installed

As I was also testing some code that used the AWS module, instead of installing the module each time the test ran which would add significantly more time to the test, creating a new docker image with the module already installed with a DockerFile allowed me to create a container with the necessary module. 


DockerFile

[PowerShell docker image](https://hub.docker.com/r/microsoft/powershell/)
[Docker course on lynda.com](https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html)
[PowerShell editions](https://docs.microsoft.com/en-us/powershell/gallery/psget/script/scriptwithpseditionsupport)
[Docker Hub](https://hub.docker.com/)
[docker docs on installing](https://docs.docker.com/engine/installation/#desktop)
