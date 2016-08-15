---
layout: post
title:  "Stubbing/Mocking JS module internals (almost) seamlessly"
date:   2016-06-15 12:00:00 +0200
categories: NodeJS testing
---

# The issue

When you come from strongly typed compiled languages, you often use 
specific tools to get your stubbing / mocking going (not that they're mandatory but they generally make your life easier). 
Many options exist : 
 
* google test for your C / C++
* Rhino mocks for C#
* Mockito for Java 
* and more ...

Usually, you get your favourite library, set it up, import it in your test 
setup, and then the fun can begin.

In some regard in the Javascript world, SinonJS allows you to do this, at least partially. 
It allows mocking in your class / object prototype and does a nice job at that.
But what if you want to stub internals you don't want exposed like some DB connector, a passport strategy ... ?

<!-- more -->
  
  
# What's the issue there?

## 1st, you probably want to question your design, just to be sure
Sometimes, you just take a hard look at your overall design and realize it's not the most sensible, change it around 
a bit and your problems are gone, sometimes it still doesn't quite cut it.

## Ok my design looks sound, what now?
The aim here is not to test those unexposed properties / functions (and it's probably some kind of a bad smell if you intend to do that). 
Here, we're going to look at one way to aid testing by allowing you to "mock all the things" that would get in the way of
 unit testing the functions that matter (the ones you chose to expose).

![Mock all the things!](/images/mock_all_the_things.jpg){: .center-image }

What can end up happening is that you expose your imported module 
you wish to stub (via a getter function or a property, for example) so that you can plug Sinon to it.
You could also use libs like [rewire](https://www.npmjs.com/package/rewire) that might get the job done although probably being a bit too complex if you just want to 
 stub a single component.

# What you can do

## Stubbing through the tested functions context

One way to almost get away with anything is to remember 
[`function.prototype.call`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) 
and how it works.

That way, you can have a class that looks like this (ES6 style): 

{% highlight javascript %}
import configurationManager from '../helpers/configurationManager'

class Foo {
	constructor(targetEnvironment) {
        this.configuration = new configurationManager(targetEnvironment);
	}

	concatenateConfigurationFooAnd42 (suffix) {
        return `${this.configurationManager.get('foo')}-42-${suffix}`;
	}
}

export {Foo};

{% endhighlight %}

Or like this (ES5)

{% highlight javascript %}
var configurationManager = require('../helpers/configurationManager')

var Foo = function (targetEnvironment){
    this.configuration = new configurationManager(targetEnvironment);
}

Foo.prototype.concatenateConfigurationFooAnd42 = function(suffix) {
    return `${this.configurationManager.get('foo')}-42-${suffix}`;
}


module.exports.Foo = Foo;

{% endhighlight %}

And a test that is written this way (ES6) : 

{% highlight javascript %}

describe('Foo.concatenateConfigurationFooAnd42', () => {
    it('should concatenate the foo attribute gotten from config manager with 42-bar', () => {
        //setup
        const context = {configurationManager:{get:sinon.stub().returns('myfoo')}}
        //action
        const actual = foo.concatenateConfigurationFooAnd42.call(context, 'bar');
        //assert
        assert.equal(actual, 'myfoo-42-bar');
    });
});

{% endhighlight %}

This example is, of course, a bit silly for the sake of brevity ;) (with no other context for that example, you'd probably be happier directly passing the needed configuration items at function call, for instants)

This allows to bypass issues like libs you want to load but not expose and do something that somewhat resembles DI for testing. 
That way, you avoid dependency issues to libs you'd rather import than inject (you still have to set an internal property leading to your imported lib, though).

Some actual code for the above example would probably look like : 

{% highlight javascript %}

// [ ... somewhere in a JS function of the code ... ]
const foo = new foo();
const concatenatedValuesForReturnState = foo.concatenateConfigurationFooAnd42('bar'); // no use of "call"
// [ ... ]

{% endhighlight %}

## F*** it I'm exposing this!

Of course, you can also expose the properties / methods you want to be able to set up when testing but if it's not part
of the normal behaviour of your object, you'd probably rather avoid explicitly exposing what doesn't need to be.

{% highlight javascript %}

import configurationManager from '../helpers/configurationManager'

class Foo {
	constructor(targetEnvironment) {
		this.configuration = new configurationManager(targetEnvironment);
	}
	
	get configurationManager () {
	    return configurationManager;
	}

	concatenateConfigurationFooAnd42 (suffix) {
	    return `${this.configurationManager().get('foo')}-42-${suffix}`;
	}
}

export {Foo};
{% endhighlight %}

{% highlight javascript %}
var configurationManager = require('../helpers/configurationManager')

var Foo = function (targetEnvironment){
    this.configuration = new configurationManager(targetEnvironment);
}

Foo.prototype.configurationManager = configurationManager;

Foo.prototype.concatenateConfigurationFooAnd42 = function(suffix) {
    return `${this.configurationManager.get('foo')}-42-${suffix}`;
}

module.exports.Foo = Foo;
{% endhighlight %}

And a test that is written this way : 

{% highlight javascript %}

describe('Foo.concatenateConfigurationFooAnd42', () => {
    it('should concatenate the foo attribute gotten from config manager with 42-bar', () => {
        //setup
        sinon.stub(foo, 'configurationManager', {get:() => {
            return 'myfoo';
        }})
        //action
        const actual = foo.concatenateConfigurationFooAnd42('bar');
        //assert
        assert.equal(actual, 'myfoo-42-bar');
    });
});

{% endhighlight %}

# Wrap up

At the moment I quite like the approach of using the function's context 
as it allows you to keep a consistent interface and write as few "test boilerplate code" 
as possible into what will be your production code (hence reducing general clutter) and it resembles the Dependency Injection principles 
one can find in Object Oriented projects. The second method works, but feels less explicit (even a bit cryptic at times), 
especially when you discover the code. There are probably other options available. 
Faking context through `Function.call` is just the latest I came up with and seemed to reduce overall clutter / improved readability a bit on my current work.
If you've came up with different methods / think that's cool / think I'm a maniac using JS like this, feel free to hit me on Twitter [@MartinBahier](http://www.twitter.com/MartinBahier) =)