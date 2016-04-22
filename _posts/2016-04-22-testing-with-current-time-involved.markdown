---
layout: post
title:  "Testing with current time involved"
date:   2016-04-22 13:14:05 +0200
categories: node.js testing Sinon.JS trick
---

At some point when you're writing tests, you come across frustrating bits that you know are going to rain on your parade and be somewhat of a pain to test (and not for the good reasons). 
On my end, Dates do fall in that category, and the JS universe is no exception.

Generally, what I want to do when writing my tests is ibeing able to control them (so that I don't have to deal with things like a `Date.now` somewhere I can't reach which would lead to unreliable tests.
In general, I end up testing that by controlling the date (it's a parameter of the function I'm testing) or stubbing (a function controls the date I'm using, and I've not created it just for the sake of testing).

Sometimes, though, you still face issues where you can't really properly set your date (not accessible because it's lost somewhere in a lib you don't control maybe, or it just does not really make sense to extract it). That's where [Sinon.Js's fake timers](http://sinonjs.org/docs/#clock) can come in handy =)

{% highlight javascript %}
var clock = sinon.useFakeTimers(now); //sets the date to 'now' (UNIX timestamp)

[... testing occurs ...]

clock.restore(); //get back to the default Date behaviour
{% endhighlight %}

It's not like that behaviour is a hidden feature of Sinon or anything, but it's not exactly the first thing you bump into when reading the Sinon.Js documentation either (in fact, it's roughly at the middle of the huge doc page).

It's not much, but it helps =)
