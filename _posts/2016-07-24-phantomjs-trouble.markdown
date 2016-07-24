---
layout: post
title:  "PhantomJS trouble"
date:   2016-07-24 18:00:00 +0200
categories: PhantomJS testing NodeJS
---

# Why the hell doesn't PhantomJS work anymore! I've just reinstalled my modules ...
I've been using PhantomJS on various projects in the past months. PhantomJS is definitely a useful tool but,
more often than you'd like, it is related to weird build or test errors. Last time for me, I was unable to properly install
my modules and even when npm gave me an OK after multiple attempts, test builds using PhantomJS would still crash.

<!-- more -->

# The last symptoms to date

Just when I had other things to do than fix my build chain, I had to deal with that issue when running part of my tests :

{% highlight shell %}
martin@tuxbox:/buildDirectory$ npm run test:unit:kiosk

> u1@ test:unit:kiosk /buildDirectory
> gulp test-kiosk

[16:54:37] Using gulpfile /buildDirectory/gulpfile.js
[16:54:37] Starting 'browserify-test-kiosk'...
[16:54:55] Finished 'browserify-test-kiosk' after 19 s
[16:54:56] Starting 'test-kiosk'...
internal/child_process.js:274
  var err = this._handle.spawn(options);
                         ^

TypeError: Bad argument
    at TypeError (native)
    at ChildProcess.spawn (internal/child_process.js:274:26)
    at exports.spawn (child_process.js:362:9)
    at Object.<anonymous> (/buildDirectory/node_modules/gulp-mocha-phantomjs/node_modules/phantomjs-prebuilt/bin/phantomjs:22:10)
    at Module._compile (module.js:398:26)
    at Object.Module._extensions..js (module.js:405:10)
    at Module.load (module.js:344:32)
    at Function.Module._load (module.js:301:12)
    at Function.Module.runMain (module.js:430:10)
    at startup (node.js:141:18)
[16:54:56] 'test-kiosk' errored after 495 ms
[16:54:56] Error in plugin 'gulp-mocha-phantomjs'
Message:
    test failed. phantomjs exit code: 1

npm ERR! Linux 3.2.0-4-amd64
npm ERR! argv "/usr/local/bin/node" "/usr/local/bin/npm" "run" "test:unit:kiosk"
npm ERR! node v5.3.0
npm ERR! npm  v3.3.12
npm ERR! code ELIFECYCLE
npm ERR! u1@ test:unit:kiosk: `gulp test-kiosk`
npm ERR! Exit status 1
npm ERR! 
npm ERR! Failed at the u1@ test:unit:kiosk script 'gulp test-kiosk'.
npm ERR! Make sure you have the latest version of node.js and npm installed.
npm ERR! If you do, this is most likely a problem with the u1 package,
npm ERR! not with npm itself.
npm ERR! Tell the author that this fails on your system:
npm ERR!     gulp test-kiosk
npm ERR! You can get their info via:
npm ERR!     npm owner ls u1
npm ERR! There is likely additional logging output above.

npm ERR! Please include the following file with any support request:
npm ERR!     /buildDirectory/npm-debug.log

{% endhighlight %}

At that point, I'm seeing "phantomjs" everywhere in my logs so I say to myself : "Ok, phantom's install has gone wrong again somehow, let's fix it, it's simple".
Obviously, that did not quite go as planned, but the fix I found is actually pretty simple, honest =)

# What I attempted
Like most problems where it seems a module was not isntalled properly, I basically went on and removed my "node_modules" directory and attempted a fresh install.
At that point, phantom would not install directly and it'd take me 2 `npm i`s to have npm give me the OK. I though "meh ... good enough for the moment",
 boy was I wrong =)


Tests using phantom would still not run, I attempted a second "delete node_modules and re-install everything" just in case and,
 as logic would have it, I had the same issues at install and testing.


At that point, I turned to one of the developer's best friends (aka : StackOverflow) and, surprisingly, did not find much topics
on that kind of issue. I tried globally installing phantom (yuck!), and it obviously did not do me any good.
Presumably,when using phantomjs with other components like casper, the test runner sometimes mixes up which binary to run. I wasn't convinced
(the same tests ran fine on my laptop without any global install + it reminds me of those developers that install everything globally
"just in case" and end up messing up their whole configuration) and ... well ... I'm still not =).

# What seems to fix everything

One thing I forgot is that, as npm uses a cache, if I get a weird version of a package for some reason, it can stay in the cache
 even if I think I'm reinstalling everything. [https://github.com/Medium/phantomjs/issues/435](This GH issue) brought me back on track.


In the end, I deleted my cache on top of removing my node_modules before re-installing :
{% highlight shell %}
martin@tuxbox:/buildDirectory$ rm -rf node_modules && npm cache clean && npm install
{% endhighlight %}

... and PhantomJS did not complain anymore and I was able to run my tests again =)


# Wrap-up
When in doubt with install issues, never forget `npm cache clean` before bothering with reinstalls.
I'll try and make my CI delete that cache if the "build" steps of my apps fail from now on, so that it'll, hopefuly, allow me to just re-run a build immediately =)

Hope that helps ;)