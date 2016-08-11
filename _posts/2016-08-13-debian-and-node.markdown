---
layout: post
title:  "Ubuntu / Debian and NodeJS : a weird relationship?"
date:   2016-08-13 14:00:00 +0200
categories: NodeJS Ubuntu Debian
---

# Running NodeJS tools with Debian why is this still an issue?
Recently, I had to use my CI to host a new node project. Up until now I had it all setup and was already able to build & test deploy other node projects.
But, of course, this project was a little different from the others (no "special snowflake", but still different ;) as it used gulp / browserify. 
My problem was that I could not run my gulp jobs on their own and would always get a "could not find 'node' : no such file or directory" error.
Same thing happened when I ran a few tests on my development machine (wich runs ubuntu). But Node runs on those machines! So why the error message? :-/

<!-- more -->

# The root cause / the (quick) fix

Turns out that, as it's packaged today (at least in standard repos, Nodejs' binary is called `nodejs` and not `node`. 
Hence the error gulp and other tools burp at you in that precise context.
Another thing I did not expect was that the reason jobs would run okay using `npm` but not directly was that my npm has been configured to use `nvm` but apparently gulp ignores it.


[This issue sums up the problem](https://github.com/gulpjs/gulp/issues/436).


Right now, if you do not use `n` or `nvm`, the smartest thing to do is probably not to use distibution standard packages but to add node repos ([The actual documentation today](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)) which should provide you with correctly named binaries.

Otherwise, you can create a `node` symlink (typically in /usr/bin) that either points to the nodejs binary you need (I ended up doing that).

# Why the hell "nodejs" ?

What remains is the not so trivial issue of why are those .debs packing an admittedly wrongly named binary? At some point the issue was mostly that
 the most up to date official binaries were quite lagging "version-wise" (a bit like for Ruby at some point, I don't know if it's still the case). 
Now we have decent versions but naming issues, and for the main binary, it's a bit sad. Moreover, it's quite strange, coming from such serious people.

At that point, I'm not exactly sure why they package NodeJS this way but, as tools like gulp are expecting the "node" binary and not "nodejs",
 I'm guessing there's a communication issue somewhere. Unfortunately the "lost cause" mentioned in the issue I've linked seems to have been lost. So it's a bit difficult to figure out. 
 But it's still a shame that there does not seem to be some kind of guidelines on how to package software that is not a novelty anymore (and it's not a one time fluke as it has been at least 2 years apparently).
 

The node documentation states : 

```
A Node.js package is also available in the official repo for Debian Sid (unstable), Jessie (testing) and Wheezy (wheezy-backports) as "nodejs". It only installs a nodejs binary.

The nodejs-legacy package installs a node symlink that is needed by many modules to build and run correctly. The Node.js modules available in the distribution official repositories do not need it.
```

So, you do have a known workaround, but I'm not quite sure what to think of this as it seems that Debian packagers moved from "node" to "nodejs" at some point and are probably not going back (and, in that case, I'd like my `nodejs-legacy` package to be automatically installed when I install `nodejs`).

# Is it a big deal?
 
Yes and no, Although one might argue that maintainers choose how they package the apps they're responsible for, it's always better when everyone agrees on such things (imagine the same issue for core binaries like `ls`, `find` ...). Also, it arguably makes Debian based distributions look like oddities for Nodejs devs, even though these distributions are very nice and sturdy (not to mention hugely widespread).

In my opinion, the situation as it exists today is a bit strange (as I don't see another linux distro doing the same), so I'd probably rather have debian packages use the same binary name as everyone else (i.e: `node`), but I might be wrong as I haven't tested out all existing distros but more something like 3 (+OSX).


# In the end : Who's wrong ? ;)

Just to be sure, I've compiled NodeJS from source to see what binary is produced (`nodejs` or `node`)


;afdg;asfkgj;alkfgj;dfg
dfgsdfg
sdfgsdfgsdfgsdfgsdfg
sdfgsdfgsdfg
s


sdfgsdfgsdfgsdfgsdfgsdfg



sdfgsdfgsdfgdsfgsdfgdsfgdfgsdfg


# What could be done?

One might argue that the responsibility of having an ubiquitous name for a specific binary is a shared responsibility between the packagers and the "packagee" as you need to find a name that works for every system. That would probably imply finding a name that is specific enough so that it does not conflict with other binaries. `NodeJS` is definitely more specific than just `Node`, so that might be an explanation (or at least part of it). 

## What to do initially?

1. Ockham's razor would suggest to just package the binaries "as is" from the compilation process
2. Maybe it could the the packagee's responsibility to state what name is valid, but then what if a specific distro has a conflict?
3. If we needed a more complex system, my guess is it could be done through some kind of registry (like what is done for the DNS system : you apply for a name and if it's not yet taken you can take it and it's yours), but it looks like a lot of work for a naming issue, but I'm not sure that the binaries naming space is _that_ crowded at the moment though that it'd require a dedicated system.


## What if the binaries change names from a version to another (and should it be possible)?
This is probably the trickiest part of it all (and probably NodeJS' issue). Changing the name of a binary is a compatibility issue as dependent software might break upon upgrade, and the problem is that dependent software packages will probably specify a minimum version, but I'm not quite sure it's possible to specify a maximum one. With package managers like APT you can freeze a package's version through some configuration, but it's still a manual operation, so it doesn't really solve the problem.



====> Qu'en pense dicosmo ? =)