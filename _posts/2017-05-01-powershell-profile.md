---
title:  PowerShell Profile
date: 2017-03-01
update: 2017-07-15
categories: 
    - powershell
tags:
    - Powershell
    - profile
---

The PowerShell profile can be used to customise your PowerShell environment to how you like it. I use my profile to add aliases to common commands and modules I use.

The profile in PowerShell can have four different settings:
AllUsersAllHosts
AllUsersCurrentHost
CurrentUserAllHosts
CurrentUserCurrentHost 

To view your profiles, you can use the $PROFILE variable and the relevant NoteProperty:

{% highlight powershell %}
$PROFILE.AllUsersAllHosts
$PROFILE.AllUsersCurrentHost
$PROFILE.CurrentUserAllHosts
$PROFILE.CurrentUserCurrentHost
{% endhighlight %}

You can just use the $PROFILE variable to see the current profile in use in the current host

![PowerShell profiles](/images/powershell-profile/profile-output.png)

Here's the code that will create the output in the image above:

<script src="https://gist.github.com/MatthewJDavis/da273783cb85c15fa6975c173e1ec7d3.js"></script>
