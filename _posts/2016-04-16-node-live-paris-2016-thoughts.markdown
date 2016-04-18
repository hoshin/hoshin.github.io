---
layout: post
title:  "Thoughts on Node Live Paris 2016"
date:   2016-04-16 22:34:05 +0200
categories: node.js live Paris npm benchmarking
---

Last thursday afternoon, I went to the node.js Live Paris event. It was less crowded than I expected but it was probably due to the fact that the event took place midweek in the middle of the afternoon. Not a big deal though, it left more room in the comfy IBM conference room for the geeks like me that made it ;)

In my opinion, this event was a bit of a mixed bag but there were some interesting info here and there, which is always cool.

In this piece, I'll quickly go through what happened 

# Andy Watson - "Lightning/welcome talk"
Andy greeted us and talked a bit about IBM's involvement in the node.js project as of today. For those like me who either did not care much or didn't notice, here are some figures : 

* 10 contributors (with push privileges) to the node.js project are actually IBM-ers
* 1 of the node.js board members is an IBM-er
* IBM has worked with node.js for about 3 years
* IBM's work in the Node ecosystem revolves around projects like expressjs and "Node Application Metrics"

# Ashley Williams - "You don't know npm"

Ashley told us various tips and tricks she thought we probably didn't knew. On my end is was pretty much dead on (heard of some but definitely not all).
For our convenience, the slides are available on [one of her repos](https://github.com/ashleygwilliams/node-live), so let's focus on a couple of nifty things :

## Things you might not know about `npm init` but that are pretty nice

{% highlight shell %}
    npm init #<= allows the creation of a basic package.json file
{% endhighlight %}
this is probably what most of us do when creating a project. It just works, there's not too much to do, you can even use it with the `--yes` option to get a default `package.json` file, 
it avoids copy/pasting another file from a similar project but with differences that we'll forget about and **will** show up at an undesirable moment.


But what if we could tailor make that process to, say, get rid of the copy/pasting or, more practically, to enforce certain rules amongst projects in a team?
That's where the `.npm_init.js` file comes in handy, it allows for you to specify which questions to ask when running `npm init`, plus it fills in stuff for you. Ain't that nice? =)


There's some documentation about this [here](https://docs.npmjs.com/misc/config) (around the middle of the page, look for `init-module`). Basically this is a module loaded when using `npm init` 
that overrides the default behaviour. The module seems to export what'll become your package.json but in the mean time you can do whatever you please whether it be asking questions to the user or 
inspecting stuff programatically or generating something. 


Of course, this requires a bit of elbow grease to get started as you need to actually write that `.npm_init.js` file, 
but it can probably be useful on the long run if you don't want to think about certain aspects of your projects. Say you always init using the default tool and then 
add exactly the same testing scripts / basic devdependencies... that's probably not that useful for a couple of projects, but for a little more it might just be worth the effort.


## npm3, the good parts (?)

### What about cache management?
Even though npm3 is not exactly the most popular thing about node.js right now, Ashley talked a bit about it and 
showed us this little trick npm3 users can do to avoid network lookups when installing modules if they so desire (and that is not possible w/ npm2). 
npm3 uses a sort of "module cache" (the `~/.npm` directory) where every module you downloaded for install is kept. 
I say it's sort of a cache only as npm3 still tries to access the network 
to get the latest modules or do other npm things but, although it's not especially recommended, you can trick npm 
into tapping into the cache directory directly using the `--cache_min` option.

{% highlight shell %}
    npm i --cache_min=999999999999 my_module # or whatever huge number you fancy
{% endhighlight %}

### Semver FTW
Ashley also talked about various topics around npm3, the most notable other probably being how npm3 now handles building the dependency tree of your application, 
and especially the fact that it does not break your shrinkwrap anymore (i.e:you do not have to re-shrinkwrap your dependencies after npm3 has had a go at them (after a new install, for example).
Why talking about this, she brings the fact that some people ask the question "Why not shrinkwrap all the time, so we lock down everything then?". 
Her answer to that boils down to : "we believe in [semver](http://semver.org/) ant the fact that packagers abide by it". 


That makes sense, IMHO, as semver is pretty straightforward and you probably do not want to manage each dependency of your project individually 
and npm allows you to manage wether or not you want a precise version or to go as far as update even if the version number suggests breaking changes have occurred.


## A word about the [node-together](http://www.nodetogether.org/) initiative
I didn't know about this initiative until Node Live, but it sounds like fun. The aim of that initiative is to encourage diversity
 in the node.js ecosystem and to do that by teaching Node to whoever feels like learning something new.
 
 
One might argue that the fact that the community around Node is probably not as diverse (described as "a 30yo white dude") 
as it could be probably applies to pretty much every technology out there today, but it certainly is nice to 
try and bring people together and get some coding done =)


# Thomas Watson - Instrumenting node.js in production

Thomas works for Opbeat and came here to give a talk ([presentation here](https://github.com/watson/talks/tree/master/2016/04%20Node.js%20Live%20Paris)) 
about how their product does manage Node in a production environment. This talk focused on the main hurdles Opbeat had to jump 
to be able to provide performance monitoring (and more specifically, how to "follow" an http request during the time it's in your servers, and have enough context to make something out of it) 
for Node applications in production. 

## The problem
The main challenges boiled down to being able to get a context that "follows" asynchronous requests 
across multiple services as there is currently no API to pass that context across the async boundary (and be as seamless as possible as we don't want to be handling the context trip manually).

## The solution
In order to do that, they "monkey patched" `Async` and now use a lib called `async_wrap` that allows to record context when a callback is queued 
and restore it when it's de-queued (which is pretty much what they wanted to achieve). `Async_wrap` is not, yet, your usual module, so you can't just require it and start cracking at your problem, but it's getting there ([https://github.com/nodejs/tracing-wg](https://github.com/nodejs/tracing-wg)). 
From what I understood, it is currently shipping with Node, it's just a bit hidden as it is not quite ready yet.

## Caveats
One might not realize that until it blows up in their hands, but basic functions like `console.log` are asynchronous, 
which makes the use of `async_wrap` a bit tricky (for some console logging you can just use `process._rawDebug`, but you probably will encounter other calls you were certain were synchronous but in fact are not ;).


# Gareth Ellis - Introduction to benchmarking

Here we tackled the subject of how to benchmark a Node application and what things to look for that could lead you to false assumptions.
If we sum this up, that gives us : 


* "Do not adjust multiple things at the same time". It seems pretty straightforward and applies to pretty much anything you want to study the behaviour, but I guess it's better to put this out there if people tend to forget it.
* Node components like the V8 JIT might influence the readings you're getting when tuning one thing or another.
* There are various tools out there to help w/ benchmarking. You can use your favourite UNIX tools, but also Javascript / V8 profilers

# Igor Soarez - Anti-patterns in node.js

Here, we dive into a more "development-oriented" territory (\o/). Igor shares with us 15 or so patterns that he regularly sees 
when helping other teams. Some of those patterns are tightly linked to node.js and how it works, others are more related to 
best pratices in development as a general rule. With the increasing use of node.js, more and more developers 
coming from very diverse technologies come to Node because it's a bit "the technology to work with" right now. 
Everybody comes w/ their old habbits, and some of those tend to die hard. Other habbits just don't really exist if you haven't ever
worked with Node, and it's not always easy to get those habbits going.

Some of the most "node.js-y" include : 

* Callback hell (of course), and why it's bad and you should avoid it
* Ignoring callback errors ("because, you know, errors do not occur that often, and my coding is perfect anyhow so that won't break")
* Concurrency issues ("This works on my machine, I don't know why this doesn't in production")
* Over-tooling (npm scripts will probably get the job done just as well as those 5 grunt plugins you just added)
* Unlimited async operations (can clog the event loop, usually does not blow up until in production w/ a real load).

Some more "software craftsmanship" oriented ones include : 

* Code monoliths should be avoided like the Black Death
* An endless lists of arguments for your function is a (bad) smell
* Too much "kitchen sink" modules that do not respect the [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle)
* You should not test the implementation details of a module but rather its functionality


There were many other points, some overlapped a bit, but this should give you a pretty good idea.


# Mikeal Rogers - node.js Everywhere

Mikeal Rodgers talked about how the meaning of the word "fullstack" has changed in the last years, and how now it encompases many things, 
from the good old browser, to your last "IoT" gizmo.

What he says, is that node.js, seems to be prepared to be a unified platform for all those applications.

* On the desktop side, you have projects like electron that allow you to create native applications based on node.js
* Front-end wise, Node is present in tools like Gulp, Grunt, Less, ESLint ...
* For mobiles, services like Cordova rely on node.js
* In the cloud, most service vendors like Amazon, Heroku, Google ... have invested to provide good support for node.js
* Looking at the "IoT"/embedded systems world, node.js seems completely appropriate. The way these systems work is mostly event-driven, which matches what Node is about. 
node.js is now supported by various low power boards (like Pis or Arduinos)
* APIs : No surprises there, node.js is generally a good choice when it comes to creating an API.

Mikeal thinks that, as it has become so ubiquitous, node.js could be some sort of "universal transferable skillset across platforms". 

I'm not sure I completely agree with that. There are and will be things where node.js is just not the best option available, 
or JS for that matter (you probably don't want to perform complex intensive calculations with it, for example). 
Languages and frameworks are tools you use to get things done, and when you need a hammer, you don't want to feel obligated to use a toaster oven instead ;)
But I guess that node.js (and, more broadly Javascript) offer today something quite unique in terms of ubiquity (and that is very nice). 
It probably isn't optimal but that's not always what all of this is about.