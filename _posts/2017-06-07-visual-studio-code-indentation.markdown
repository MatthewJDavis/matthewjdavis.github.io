---
title:  "View indentation guides in Visual Studio Code"
author: Matthew Davis
date:   2017-06-07
categories: Visual Studio Code
---

[Visual Studio code] is my go to editor now for most work including creating PowerShell scripts, terraform deployments, cloudformation templates and general file editing. It has a great selection of useful [extensions], nice git integration and I use it on Windows, Mac and Ubuntu (the debugging of PowerShell on the Mac crashes from time to time but on the whole it's a solid experience, and to be fair [PowerShell on Mac] is still an Alpha release).

While editing a cloudformation YAML file in the VS Code insders release, I notice that the indentations were highlighted and this was really handy to make sure everything was correctly aligned. When I got to work and opened up the file on the mac I noticed they were not visible, this was solved by going into the user settings (File, Preferences, Settings) and adding the following line to the file:
 "editor.insertSpaces": true
My settings file now looks like:

{%highlight JSON%}
{
    "git.enableSmartCommit": true,
    "git.confirmSync": false,
    "window.zoomLevel": 0,
    "editor.insertSpaces": true
}
{% endhighlight %}

With that line added, the YAML file now looked like:

![VS Code Indentation](/images/vscode/vscode-indentation.png)


[Visual Studio code]:https://code.visualstudio.com/
[extensions]:https://code.visualstudio.com/docs/editor/extension-gallery
[PowerShell on Mac]:https://github.com/PowerShell/PowerShell