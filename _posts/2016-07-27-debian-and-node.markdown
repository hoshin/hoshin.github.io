---
layout: post
title:  "Ubuntu / Debian and NodeJS : a weird relationship?"
date:   2016-08-07 14:00:00 +0200
categories: NodeJS Ubuntu Debian
---

# Running NodeJS tools with Debian why is this still an issue?
Recently, I had to use my CI to host a new node project. Up until now I had it all setup and was already able to build & test deploy other node projects on that server.
But, of course, this project was a little different from the others as it used gulp / browserify. 
My problem was that I could not run my gulp jobs on their own and would always get a "could not find 'node' : no such file or directory" error.
Same thing happened when I ran a few tests on my development machine (wich runs ubuntu). But Node runs on those machines! So why the error message? :-/

<!-- more -->

# The root cause / the (quick) fix

Turns out that, as it's packaged today (at least in standard repos, Nodejs' binary is called `nodejs` and not `node`. 
Hence the error gulp and other tools burp at you in that precise context.


[This issue sums up the problem](https://github.com/gulpjs/gulp/issues/436).

The quickest fix you can think about (and what I did, for lack of a better option) is to create a `node` symlink (typically in /usr/bin) that either points to the nodejs binary that probably sits in the same directory or towards the node version you've installed via some tool like `n` or `nvm`. 

# "There we go, done!" or is it ?

At this point, you're technically out of the woods, this fix will work for the time being and will probably require some more tampering at the next upgrade,
 but in the mean time everything should work ok.
What remains is a not so trivial issue of why are those .debs packing an admittedly wrongly named binary? At some point the issue was mostly that
 the most up to date official binaries were quite lagging "version-wise" (a bit like for Ruby at some point, I don't know if it's still the case). 
Now we have decent versions but naming issues, and for the main binary, it's a bit of a bummer. Moreover, it's quite strange, coming from such serious people.

At that point, I'm not exactly sure why they package NodeJS this way but, as tools like gulp are expecting the "node" binary and not "nodejs",
 I'm guessing there's a communication issue somewhere. Unfortunately the "lost cause" mentioned in the issue I've linked seems to have been lost. So it's a bit difficult to figure out. 
 But it's still a shame that there does not seem to be some kind of guidelines on how to package software that is not quite a novelty anymore (and it's not a one time fluke as it has been at least 2 years apparently).
 
 
Right now the smartest thing to do is probably not to use distibution standard packages but to add node repos ([The actual documentation today](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)) which should provide you with correctly named binaries.

The node documentation states : 

```
A Node.js package is also available in the official repo for Debian Sid (unstable), Jessie (testing) and Wheezy (wheezy-backports) as "nodejs". It only installs a nodejs binary.

The nodejs-legacy package installs a node symlink that is needed by many modules to build and run correctly. The Node.js modules available in the distribution official repositories do not need it.
```

So I'm not quite sure what to think of this as it seems that Debian packagers moved from "node" to "nodejs" at some point and are probably not going back.

# Who's responsibility is this?
 
It's not a big deal, per se, but it's always better when everyone agrees on such things (imagine the same issue for core binaries like `ls`, `find` ...) 
and it makes Debian based distributions oddities for Nodejs devs, even though these distributions are very nice and sturdy (not to mention hugely widespread).

# Est-ce que la responsabilité ne serait pas au moins un peu partagée entre tous les acteurs (mon problème est hyper précis, dès qu'on rentre dans une chaine ća pose des problèmes


# Cette position est-elle tenable? Est-ce que cela n'est pas un peu dommage ?

# Que pourrait-on faire ?


https://github.com/gulpjs/gulp/issues/436