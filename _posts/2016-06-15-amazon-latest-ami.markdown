---
layout: post
title:  "Get the lastest Amazon AMIs using PowerShell"
date:   2017-06-15 21:20:00 +0000
categories: AWS PowerShell AMIs
---

Need to find the latest AMI ID for Amazon Linux but don't want to use web console (click, click, click...)? This simple PowerShell script will return the latest AMI IDs for Amazon Linux. I use this script to update AMI IDs in cloudformation maps.

<script src="https://gist.github.com/MatthewJDavis/9618049c2b75a36b8c8ee58b7a364dfd.js"></script>

Of course, you can change or add the regions as needed, you can use the following command to get the regions:

{% highlight powershell %}
Get-AWSRegion
{% endhighlight %}

Which as of 15th June 2017, gives the following:

| Region | Name |
| :--- | ---: |                      
|ap-northeast-1| Asia Pacific (Tokyo)|
|ap-northeast-2| Asia Pacific (Seoul)|      
|ap-south-1|     Asia Pacific (Mumbai)|     
|ap-southeast-1| Asia Pacific (Singapore)|  
|ap-southeast-2| Asia Pacific (Sydney)|     
|ca-central-1|   Canada (Central)|          
|eu-central-1|   EU Central (Frankfurt)|    
|eu-west-1|      EU West (Ireland)|         
|eu-west-2|      EU West (London)|          
|sa-east-1|      South America (Sao Paulo)|
|us-east-1|      US East (Virginia)| 
|us-east-2|      US East (Ohio)|
|us-west-1|      US West (N. California)|
|us-west-2|      US West (Oregon)|
