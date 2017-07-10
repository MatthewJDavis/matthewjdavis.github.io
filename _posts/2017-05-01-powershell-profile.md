---
title:  PowerShell Profile
author: Matthew Davis
date: 2017-03-01
categories: 
    - powershell
tags:
    - PowerShell
---

The PowerShell profile can be used to customise your PowerShell environment to how you like it. I use my profile to add aliases to common commands and modules I use.

The profile in PowerShell can have four different settings:
AllUsersAllHosts
AllUsersCurrentHost
CurrentUserAllHosts
CurrentUserCurrentHost 

To view your profiles, you can use the $profile variable:

{% highlight powershell %}
$PROFILE.AllUsersAllHosts
$PROFILE.AllUsersCurrentHost
$PROFILE.CurrentUserAllHosts
$PROFILE.CurrentUserCurrentHost

{% endhighlight %}

You can just use the $PROFILE variable to see the current profile in use in the current host

![PowerShell profiles](/images/powershell-profile/profile.png)