---
layout: post
title:  "zypper behind bars ... well behind proxies"
date:   2016-08-07 13:00:00 +0200
categories: SUSE/Zypper proxy
---

# Sometimes, things just go the Murphy's law way
Consulting often means you can't do whatever you like in terms of infrasturcture.
Here I'm going to talk about my current assignment, the continuous integration platform we need to use, 
and how upgrading it turned out to be a bit of a puzzle =)

<!-- more -->

# The context
At first, there was joy. We were ditching our old SVN for Git. The first steps were easy, create a repo in the private GitLab, convert the svn repo to a git repo, push it all into gitlab ... things were great.

Then, we needed to update our jenkins slave so that it could use git, and things started to get a bit messy :-/

Our predicament pretty much unfolded this way : 

1. Project Manager : "Good news guys! We're migrating our VCS to Git!"
2. Infrastructure guys : "Well, you need to update your environments on your own. No worries, just explicitly set 
the proxy through environment variables and you should be good to go"
3. Us : "What the heck ?! I've set my proxy settings up. How come zypper does not pick it up?"
4. ... an hour or two of fiddling and asking for help
5. "Oh, ok, I get it ..."

# The setup

* The CI we used is provided by our client.
* It sports a whopping SUSE (SLES 11 ... but at least it runs a kernel 3.x which is handy to be able to run a decent Git version ;)
* Internet access is provided through an unconfigured proxy

# So? What's the big deal? You configure your environment variables for the proxy and there you go. Right?

Well ... not really. It seems that zypper does not give a damn about the `https_proxy` / `http_proxy` env variables /o\\.

You need to "write down" the configuration into 2 separate files :

* `/etc/sysconfig/` for the proxy settings, which pretty much means to define the `HTTP_PROXY` & `HTTPS_PROXY`. 
If the file has not been edited yet, chances are the lines defining those variables just need uncommenting and editing. 
At that point, it seems you only need to set them to the URL of the proxy you're going to go through, credentials setup being done in another file.
* `~/.curlrc` for your credentials: You need to specify this in the file `proxy-user="proxyuser:pass"` 
(note that I talk about your `~/.curlrc` file, but that is because I edited it and ran zypper as `root`, it might be different if you're sudo-ing.

# Wrap up

Worst part of this is we've gone through all this for practically nothing as our SUSE was so old and peculiar we ended up installing special packages directly via RPM. 
But at least it allowed us to understand a little bit how zypper works (and now we can install packages with that machine ;) 