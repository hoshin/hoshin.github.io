---
layout: post
title:  "Configuring a shortcut for LightDM's app launcher"
date:   2016-06-15 12:00:00 +0200
categories: LightDM Gnome3 launcher shortcut
---

# We do it, then we forget

That's it! I've got a new shiny tool that I want to use for my day to day work. It's so pretty, it has nice features that'll help me a lot... 
But it is not packaged / has a custom installer that'll create launcher shortcuts /o\ ! (I kinda like to just have to type
 `<META>+"my app name"+<ENTER>` and launch my app instead of having to use my mouse). I always forget how to set it up manually so now I'll 
 write it down so I (and, hopefuly some readers) don't have to =)

 ![Ooooh! Shiny!](/images/shiny.png){: .center-image }


<!-- more -->

# It's all about configuration files, really

One of the ways to make this work is in the `~/.local/share/applications` directory. LightDM seems to read the contents of
 that directory to augment the list of applications it is able to suggest when you type something while in the launcher. Mine looks like this :
 
{% highlight shell %}

-rw-rw-r-- 1 hoshin  346 april  4  14:56 defaults.list
-rw------- 1 hoshin  259 march  12 12:00 jetbrains-webstorm.desktop
-rw-rw-r-- 1 hoshin 1,7K may    24 10:00 mimeinfo.cache
-rw-rw-r-- 1 hoshin  272 april  5  10:43 wine-extension-application.desktop
[...]
-rw-rw-r-- 1 hoshin  258 april  5  10:47 wine-extension-xsl.desktop

{% endhighlight %}


and a typical `.desktop` file looks something like this (here a webstorm config file): 


{% highlight shell %}

[Desktop Entry]
Version=1.0
Type=Application
Name=WebStorm
Icon=/home/hoshin/Webstorm/bin/webstorm.svg
Exec="/home/hoshin/Webstorm/bin/webstorm.sh" %f
Comment=Develop with pleasure!
Categories=Development;IDE;
Terminal=false
StartupWMClass=jetbrains-webstorm

{% endhighlight %}

This one was actually created automatically when installing webstorm 2016, but the point is that we can basically take that file,
copy it, and then edit the name / path / icon / whatever values, and we should almost be good to go 
(it seens to take a logout / login to get LightDM to reload the configuration)

