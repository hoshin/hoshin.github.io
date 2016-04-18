---
layout: post
title:  "Testing with current time involved"
date:   2016-04-18 10:34:05 +0200
categories: node.js testing Sinon.JS trick
---

At some point when you're writing tests, you come across frustrating bits that you know are going to rain on your parade and be somewhat of a pain to test (and not for the good reasons). 
On my end, Dates, do fall in that category, and the JS universe is no exception.

Generally, what I want to do when writing my tests is to avoid having to deal with Dates if I can or, on the other hand, 
be able to just set the Dates I need to work with (by design, or by stubbing), which generally solves my problem.

Sometimes, you still have trouble setting up your date, and don't want to write a getter for the current Date just to stub it 
for tests and it doesn't seem to make much sense to inverse that "dependency". 
That's where [Sinon.Js's fake timers](http://sinonjs.org/docs/#clock) come in handy =)

{% highlight javascript %}
var clock = sinon.useFakeTimers(now); //sets the date to 'now' (UNIX timestamp)

[... testing occurs ...]

clock.restore(); //get back to the default Date behaviour
{% endhighlight %}

It's not like that behaviour is a hidden feature of Sinon or anything, but it's not the first thing you bump into when reading the Sinon.Js documentation either (in fact, it's roughly at the middle of the huge doc page).

It's not much, but it helps =)