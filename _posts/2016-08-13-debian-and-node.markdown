---
layout: post
title:  "Ubuntu / Debian and NodeJS : why the odd naming?"
date:   2016-08-15 16:00:00 +0200
categories: NodeJS Debian/Ubuntu
---

# Why doesn't nodejs Debian package provide the "node" binary used by all my tools?
Recently, I had to use my CI to host a new node project. Up until now I had it all setup and was already able to build, test & deploy other node projects.
But, of course, this project was a little different from the others (no "special snowflake", but still different ;) as it used gulp / browserify. 
My problem was that I could not run my gulp jobs on their own and would always get a "could not find 'node' : no such file or directory" error.
I went ahead and checked and the same thing happened when I ran a few tests on my Ubuntu machine, it came as a bit of a surprise but then I remembered that I usually always use npm to run everything, not directly tools. 
But Node runs on those machines! So why the error message? :-/

The debian package deploys a `nodejs` binary instead of `node` ... that's why! But why the odd naming? Everybody else uses `node`! 

Let's have a look at why =)

<!-- more -->

#First I was afraid, I was petrified ... 

As I did not have any issues w/ my Fedora, the first thing that came to mind was something like this :

<p style="text-align:center">
    <img src="/images/yuno.jpg" alt="Y U no use node?"/>
</p>

And a couple seconds later, I remembered that the Debian folks are generally not doing what could look like peculiar things without a good reason, so I started looking into it a bit =)

# The surface problem

## The issue itself
Turns out that, as it's packaged today in standard repos, NodeJS' binary is renamed and tools like gulp burp an error message at you in that precise context as they specifically expect a `node` binary to run.
Another thing I did not expect was that the reason jobs would run okay using `npm` but not directly was that my npm has been configured to use `nvm` but apparently gulp ignores it. I ran in the same issue trying to run gulp jobs from my IDE (Webstorm).

After digging a little, I found [this gulp issue that sums up the problem](https://github.com/gulpjs/gulp/issues/436). Apparently, it is indeed linked to the way the official .deb package is made. This also means I'm not alone! So I should probably find threads talking about this if I look out long enough : good.

If we dig a little more, we even see that it is still (at least) one [open issue in the Ubuntu BTS](https://bugs.launchpad.net/ubuntu/+source/nodejs/+bug/1221904), and that it indeed affects every installation that relies on standard Debian packages.

So, at this point it seems that the binary has been renamed, and that it hurts tools like gulp that rely on it, but it is still unclear why the rename was made.

## The (quick'n'dirty) fix
You can work your way around this by creating a `node` symlink (typically in /usr/bin) that points to the nodejs binary you need. This is also the most common workaround I've seen in various threads. Not the prettiest way to fix the problem, I would not recommend it but it gets the job done.

{% highlight shell %}
martin@tuxbox:~$ sudo ln -s /usr/bin/nodejs /usr/bin/node
{% endhighlight %}

## A cleaner fix
If you do not to use distibution standard packages but node repos ([as documented here](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)) they will provide you with correctly named binaries "out of the package". 
In addition, you'll probably get more recent versions of node ahead of what is in your favorite distribution's package repos, which might be something to consider.

## The clean fix (for Debian based systems, only using official repos)
The node documentation states : 

> A Node.js package is also available in the official repo for Debian Sid (unstable), Jessie (testing) and Wheezy (wheezy-backports) as "nodejs". It only installs a nodejs binary.
>
> The nodejs-legacy package installs a node symlink that is needed by many modules to build and run correctly. The Node.js modules available in the distribution official repositories do not need it.

Good news! You can install a `node-legacy` package that'll create that symlink for you, and with that you don't have to rely on external repos.

A simple :
{% highlight shell %}
martin@tuxbox:~$ sudo apt-get install node-legacy
{% endhighlight %}
should do the trick.

I'd like my `nodejs-legacy` package to be automatically installed when I install `nodejs`, but it's not that bad. At least you can sort your issue in a civilized fashion, not manually creating symlinks where you should not if you don't want to end up with a unnecessarily cluttered system.


# Is it a big deal?
 
We can safely assume that maintainers choose how they package the apps they're responsible for using their best judgement (especially to avoid breaking anything in the package itself or elsewhere). So, in itself, it's not that big of a deal as there are workarounds available.

The only concern for me is that it arguably makes Debian based distributions look like oddities for NodeJS devs, even though these distributions are very nice and sturdy (not to mention hugely widespread), which is a bigger deal imho.

The situation as it exists today is a bit strange, and I'd rather have Debian packages use the same binary name as everyone else (i.e: `node`), as RPM based systems seem to be doing just fine packaging a `node` binary.

# The root cause / In the end : Who's wrong ? ;)

Just to be sure, I've compiled NodeJS from source to see what binary is produced (`nodejs` or `node`) and, well, a freshly compiled version outputs an `out/Release` directory with a `node` binary in it (which comes as little surprise after reading the NodeJS documentation about installing from various package managers, but better safe than sorry).

Let's ask aptitude what it thinks of all that : 

{% highlight shell %}
martin@tuxbox:~$ aptitude search node
#[ ... lots of packages ... ]   
p   node                            - Amateur Packet Radio Node program (transit
p   node-abbrev                     - Get unique abbreviations for a set of stri
p   node-abstract-leveldown         - Abstract prototype matching the LevelDOWN 
p   node-accepts                    - higher-level content negotiation for Node.
#[ ... tons of node-something modules ...]

{% endhighlight %}

And there's our culprit! There's a package named "node" that has much more to do with radio than a javascript v8 runtime =). 
What aptitude has cut in the output is that the "node" package is transitional as it has been [renamed](https://packages.debian.org/wheezy/node), 
and that it indeed provides a binary named node [as we can see here](https://tracker.debian.org/pkg/node) in the "binaries" section (bottom left).

At that point, we can safely assume that Debian packagers named the NodeJS binary `nodejs` instead of `node` to not break the actual "ax25-node" package 
that was probably there long before someone even thought that server-side JS could be a thing, and would be called NodeJS, which is a fairly reasonable explanation =)

So what about RedHat, Fedora, SUSE and all those other RPM users? 
Did I just go past the issue on my Fedora by not using the default package at all? Or have they devised a workaround?

[RedHat's Bugzilla](https://bugzilla.redhat.com/show_bug.cgi?id=815018#c49) suggests that they encountered the issue themselves. 
What seems to get out of the thread is that "as long as the 2 binaries are unlikely to be installed on the same machine, it is somewhat okay if they conflict", at least during the process of finding an adequate solution (and, indeed, standard nodejs .rpms now contain `node` binaries). 
The thread even links to [the exact decision from Debian tech committee on the conflict](https://lists.debian.org/debian-devel-announce/2012/07/msg00002.html), which always handy, as they faced the issue at pretty much the same time ;)

In the end, it appears that the ax25-node maintainers agreed to rename but that the process is a bit longer on Debian and all its distros (as the decision was made by 2012 and we still have the transitional package). 

So nobody is "wrong" (yay \o/) it's just that different solutions take different amounts of time to propagate =)

Last issue, in the end, is that Debian still lives with the "node" transitional package that provides ax25-node instead of nodejs even though the rename was agreed upon quite a while back and everybody seemed happy with it.

# What could be done?

One might say that the responsibility of having an ubiquitous name for a specific binary is a shared responsibility between the packagers and the "packagee" as you need to find a name that works for every system. 
Ochkam's razor would probably suggest coming up with a name that is specific enough so that it does not conflict with other binaries (`nodejs` is definitely more specific than just `node`, in our example) but issues might still arise later on.

In the end, the process as it is is probably more than enough to deal with the issues that will sometimes occur, even though it's frustrating / unclear when it happens to you.
Trying to formalize / create a tool for such a thing would probably entangle packagers in unnecessary processes the whole time just to better handle some edge cases.

# Wrap up

## About the initial issue

* There is no real anomaly about the ax25-node and nodejs packages, issue has been identified and decisions were made ages ago (yeah, 2012 is already "ages" for me ;)
* Redhat and Debian packagers handled the issue differently, which explains the current disparity.
* Issue is not yet sorted out Debian-wise (with removal of transitional packages & such). 
I have not yet found an ETA or transitional package recommendations about a maximum amount of time before cleaning this up which is too bad.

## About distribution packages issues

* There are so many things packaged for widespread distributions like Fedora or Debian that it is very difficult to ensure that nothing conflicts with nothing name-wise.
* Even if you manage not to conflict at some point, there might be a time, later on, where you'll encounter the issue, and possibly have to rename.
* As per usual : there are multiple solutions to the same issue but they're not necessarily equivalent in terms of timing
* Even if you are the first, it doesn't necessarily mean you'll get to keep your binaries names forever, if a more popular / sensible app that wants the same bin name later on, you'll have to have a little chat with the package maintainers of distros you're packaged for.

Hope this helps, feel free to hit me on Twitter if you want to share additional info / correct me ;)