---
layout: post
title:  "Ubuntu / Debian and NodeJS : a weird relationship?"
date:   2016-08-13 14:00:00 +0200
categories: NodeJS Ubuntu Debian
---

# Running NodeJS tools with Debian why is this still an issue?
Recently, I had to use my CI to host a new node project. Up until now I had it all setup and was already able to build, test & deploy other node projects.
But, of course, this project was a little different from the others (no "special snowflake", but still different ;) as it used gulp / browserify. 
My problem was that I could not run my gulp jobs on their own and would always get a "could not find 'node' : no such file or directory" error.
Same thing happened when I ran a few tests on my development machine, it came as a bit of a surprise but then I remembered that I usually always use npm to run everything, not directly tools. 
But Node runs on those machines! So why the error message? :-/

<!-- more -->

# The root cause / the (quick) fix

Turns out that, as it's packaged today (at least in standard repos, NodeJS' binary is called `nodejs` and not `node`, hence the error gulp and other tools burp at you in that precise context.
Another thing I did not expect was that the reason jobs would run okay using `npm` but not directly was that my npm has been configured to use `nvm` but apparently gulp ignores it.


After digging a little, I found [this gulp issue sums up the problem](https://github.com/gulpjs/gulp/issues/436). Apparently, it is indeed linked to the way the official .deb package is made.

You can work your way around this by creating a `node` symlink (typically in /usr/bin) that either points to the nodejs binary you need (I ended up doing that).

Although, if you do not use `n` or `nvm`, the smartest thing to do might be not to use distibution standard packages but to add node repos ([as the current NodeJS documentation suggests](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)) which should provide you with correctly named binaries.

# Why the hell "nodejs" ?

What remains is the not so trivial issue of why are those .debs packing containing an admittedly wrongly named binary? At some point the issue was mostly that
 the most up to date official binaries were quite lagging "version-wise" (a bit like for Ruby at some point, I don't know if it's still the case). 
Now we have decent versions (at least the LTS in SID) but naming issues, and for the main binary, it's a bit sad. Moreover, it's quite strange, coming from such serious people. So what happened there?

At that point, I'm not exactly sure why they package NodeJS this way but, as tools like gulp are expecting the "`node`" binary and not "`nodejs`",
 I'm guessing there's a communication issue somewhere. Unfortunately the "lost cause" mentioned in the issue I've linked above seems to have been lost. So it's a bit difficult to figure out. 

The node documentation states : 

> A Node.js package is also available in the official repo for Debian Sid (unstable), Jessie (testing) and Wheezy (wheezy-backports) as "nodejs". It only installs a nodejs binary.
>
> The nodejs-legacy package installs a node symlink that is needed by many modules to build and run correctly. The Node.js modules available in the distribution official repositories do not need it.

So, you do have a known workaround, but I'm not quite sure what to think of this as it seems that Debian packagers moved from "node" to "nodejs" at some point (as the "legacy" package would suggest) and are probably not going back (and, in that case, I'd like my `nodejs-legacy` package to be automatically installed when I install `nodejs`).

If we dig a little more, we even see that it is still an [open issue in the Ubuntu BTS](https://bugs.launchpad.net/ubuntu/+source/nodejs/+bug/1221904), and that it indeed affects every installation that relies on standard Debian packages.
# Is it a big deal?
 
One might argue that maintainers choose how they package the apps they're responsible for, at least that if they encounter an issue they need to fix for their specific distro, it's their call on how to go about that. 
Though, it's always better when everyone agrees on such things (imagine the same issue for core binaries like `ls`, `find` ...). 
Also, it arguably makes Debian based distributions look like oddities for NodeJS devs, even though these distributions are very nice and sturdy (not to mention hugely widespread).

In my opinion, the situation as it exists today is a bit strange, so I'd probably rather have debian packages use the same binary name as everyone else (i.e: `node`), but I might be wrong as I haven't tested out all existing distros but more something like 3 (+OSX).

# In the end : Who's wrong ? ;)

Just to be sure, I've compiled NodeJS from source to see what binary is produced (`nodejs` or `node`) and, well, a freshly compiled version outputs an `out/Release` directory with a `node` binary in it :-/ 

Lets ask aptitude what it thinks of all that : 

{% highlight shell %}
octo-mba@localhost:~ % ssh home
Welcome to tuxbox
Welcome to tuxbox
Last login: Sat Aug 13 23:40:48 2016 from freebox-server.local
martin@tuxbox:~$ aptitude search node
#[ ... lots of packages ... ]   
p   node                            - Amateur Packet Radio Node program (transit
p   node-abbrev                     - Get unique abbreviations for a set of stri
p   node-abstract-leveldown         - Abstract prototype matching the LevelDOWN 
p   node-accepts                    - higher-level content negotiation for Node.
#[ ... tons of other NodeJS packaged modules ...]

{% endhighlight %}

And there's our culprit! There's a package named "node" that has more to do with radio than a javascript v8 runtime =). 
What aptitude has cut in the output is that the "node" package is transitional as it has been [renamed](https://packages.debian.org/wheezy/node), 
and that it indeed provides a binary named node [as we can see here](https://tracker.debian.org/pkg/node).

At that point, we can safely assume that Debian packagers named the NodeJS binary `nodejs` instead of `node` to not break the actual "ax25-node" package 
that was probably there before someone even thought that serverside JS could be a good idea. That's a fairly reasonable explanation.

So what about RedHat, Fedora, Suse and all those other RPM users? 
I'm pretty sure I did not have the issue on my Fedora, so did I just go past the issue by not using the stock package at all? Or have they devised a workaround?

[RedHat's Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=815018#c49) suggests that they encountered the issue themselves. 
What seems to get out of the discussion that ensued is that "as long as the 2 binaries are unlikely to be installed on the same machine, it is somewhat okay if they conflict", at least for a certain amount of time. 
The thread even links to [the exact decision from Debian tech committee on the conflict](https://lists.debian.org/debian-devel-announce/2012/07/msg00002.html), which always handy ;)

In the end, it appears that the ax25-node maintainers agreed to rename but that the process is a bit longer on Debian and all its distros (as the decision was made by 2012 and we still have the transitional package).

# What could be done?

One might argue that the responsibility of having an ubiquitous name for a specific binary is a shared responsibility between the packagers and the "packagee" as you need to find a name that works for every system. 
That would probably imply finding a name that is specific enough so that it does not conflict with other binaries. `nodejs` is definitely more specific than just `node`, so that might be an explanation.

# Wrap up

* There are so many things packaged for widespread distributions like Fedora or Debian that it is very difficult to ensure that nothing conflicts with nothing name-wise.
* Even if you manage not to conflict at some point, there might be a time, later on, where you'll encounter the issue, and possibly have to rename.
* Redhat and Debian maintainers saw the issue, they just handled it differently. In the end, we'll have an identical naming for everybody.