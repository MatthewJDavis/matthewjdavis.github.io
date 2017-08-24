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

![download the powershell image from docker hub](/images/pester-docker/docker-pull-powershell.png)

The process for running Pester tests in a Docker container:
1. Create a container from the microsoft/powershell image.
2. Mount the folder where your PowerShell script and tests are as a volume in the container
3. Set the working directory of the container to the mounted volume
3. Invoke-Pester against the script in the container 
4. Output the results to the mounted volume which will be accessible from your client machine
5. Exit the container

Here is the command that I ran:

```bash
docker run --rm -it -w /tests -v ~/Documents/PowerShell:/tests microsoft/powershell
```
![docker run](/images/pester-docker/docker-run.png)

In the container run
 ```bash
 Invoke-Pester -OutputFile results.xml -OutputFormat NUnitXml
```

![invoke-pester with outputfile](/images/pester-docker/invoke-pester.png)

I run docker to create a container with a directory mapped to a volume in the container from my host laptop. The directory on my laptop is ~/Documents/PowerShell and the directory in the container is /tests.The -w option is used to set the working directory in the container to /tests.
Once the container is created, Invoke-Pester is executed in the directory containing the tests and the results are outputted to a file in the /tests directory which will write to the local directory on the laptop once the container is run and has exited (type exit in the container) and has been removed (the rm command).

## Creating a Docker Image with the AWS module installed

As I was also testing some code that used the AWS module, instead of installing the module each time the test ran which would add significantly more time to the test, creating a new docker image with the module already installed with a DockerFile allowed me to create a container with the necessary module. 

Create a text file called Dockerfile.
The following code:
1. Creates a container from the microsoft/powershell image
2. Installs the AWS PowerShell module in the container
3. Creates an image from the above container with the AWS module installed

<script src="https://gist.github.com/MatthewJDavis/1aef9a47bc804b5a8e118a97b3ec32b8.js"></script>

Navigate into the directory where you created the file (cd ~/Documents/Docker/PowerShell) run the following to create the image:

```bash
docker build -t awspowershell
```
![build the docker file](/images/pester-docker/docker-build.png)

You should now be able to see your new image and create containers from it.
The -t parameter adds the tag of awspowershell to the image.

```bash
docker image list
```
![docker image list](/images/pester-docker/docker-image-list.png)

[PowerShell docker image]: https://hub.docker.com/r/microsoft/powershell/
[Docker course on lynda.com]: https://www.lynda.com/Docker-tutorials/Learning-Docker/485649-2.html
[PowerShell editions]: https://docs.microsoft.com/en-us/powershell/gallery/psget/script/scriptwithpseditionsupport
[Docker Hub]: https://hub.docker.com/
[docker docs on installing]: https://docs.docker.com/engine/installation/#desktop
