---
layout: post
title:  "PowerShell Profile"
date:   2017-03-01 20:35:00 +0000
categories: powershell
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



[sysinternals]: https://technet.microsoft.com/en-gb/sysinternals/bb545021.aspx
[rss feed]: https://blogs.technet.microsoft.com/sysinternals/feed/
[ZoomIt]: https://technet.microsoft.com/en-us/sysinternals/zoomit.aspx
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/