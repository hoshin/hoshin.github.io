---
layout: post
title:  "Virtualized puzzle with Vagrant & Virtualbox"
date:   2016-07-09 00:00:00 +0200
categories: VirtualBox Vagrant Ubuntu16.04
---

# Oh noez! Mai VM iz not bootin after distro updatz! Halp!
Well, sometimes you update something on your distro and suddently your VMs stop working. 
The fix can be very easy but I tend to always forget it so "I'll remember it so you don't have to" as the <a href="https://www.youtube.com/user/achannelthatsawesome">Nostalgia Critic</a> would say ;) 

<!-- more -->

# The symptoms
After some updates and a reboot, your VMs will not start up anymore. If you're using Vagrant, it usually complains like this :

{% highlight shell %}
hoshin@inawen:~/dev/ansiTest % vagrant up
The provider 'virtualbox' that was requested to back the machine
'default' is reporting that it isn't usable on this system. The
reason is shown below:

VirtualBox is complaining that the installation is incomplete. Please
run `VBoxManage --version` to see the error message which should contain
instructions on how to fix this error.
{% endhighlight %}

And, VBoxManage will shout at you like this
{% highlight shell %}
hoshin@inawen:~/dev/ansiTest % VBoxManage --version 
WARNING: The character device /dev/vboxdrv does not exist.
	 Please install the virtualbox-dkms package and the appropriate
	 headers, most likely linux-headers-generic.

	 You will not be able to start VMs until this problem is fixed.
5.0.18_Ubuntur106667
{% endhighlight %}

On a more general note, Vagrant or VirtualBox will say that the `/dev/vboxdrv` is not available anymore and that `modprobe` might be your friend, 
or that you should ensure that `virtualbox-dkms` & `linux-headers-generic` should be installed. But, if you're like me, 
those packages are already installed so spamming installation commands won't do you any good.

# The (very simple) fix

On my end, the fix was really simple ... reinstalling : 

{% highlight shell %}
hoshin@inawen:~/dev/ansiTest % aptitude reinstall virtualbox-dkms
{% endhighlight %}

**Caution** : for those of you running the "OSE" version of VirtualBox, don't forget that you'll proably want to reinstall the `virtualbox-ose-dkms` parckage rather than the `virtualbox-dkms` one

Hope that helps ;)