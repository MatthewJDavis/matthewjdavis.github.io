---
layout: post
title:  "PowerShell Profile"
date:   2017-03-01 20:35:00 +0000
categories: powershell
---

The profile in PowerShell can be found can have four different settings:
AllUsersAllHosts
AllUsersCurrentHost
CurrentUserAllHosts
CurrentUserCurrentHost 

To view your profiles, you can use the $profile variable

{% highlight powershell %}
$PROFILE.AllUsersAllHosts
$PROFILE.AllUsersCurrentHost
$PROFILE.CurrentUserAllHosts
$profile.CurrentUserCurrentHost

{% endhighlight %}



[sysinternals]: https://technet.microsoft.com/en-gb/sysinternals/bb545021.aspx
[rss feed]: https://blogs.technet.microsoft.com/sysinternals/feed/
[ZoomIt]: https://technet.microsoft.com/en-us/sysinternals/zoomit.aspx
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/