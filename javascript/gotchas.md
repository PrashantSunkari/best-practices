# Common JavaScript "Gotchas"

## Introduction:

JavaScript has a lot of weird behaviours that trip up noobs to the language - especially those acquainted with more traditional OOP languages.  Hopefully this guide will provide a quickly scannable, easily understood list to save a lot of pain to those getting acquainted with the language.

The intended audience for this article are engineers from other programming languages who are working with JavaScript for the first time.  This article will not focus on detailed explanations of why the language works the way it does.  It's merely intended to get you past the early hurdles quickly.

JavaScript is a flexible language with many ways to achieve the same result.  What I document below is not the single "best" way.  Rather, it's a series of strategies I use to write code in such a way that it is less confusing to people new to the language.

If you take nothing else away from this article, I __highly__ recommend you read [JavaScript: The Good Parts, by Douglas Crockford][good-parts].  It's the single best resource I'm aware of for getting new JavaScript developers up to speed.

Author:  
Steve Kwan, Project Lead, EASPORTS.com  
Electronic Arts  
<mail@stevekwan.com>  
<http://www.stevekwan.com/>

Originally from my GitHub:  
<https://github.com/stevekwan/best-practices/>

## Common Javascript "Gotchas":

Let's talk about some of the JavaScript 101 problems that noobs to the language encounter.  If you're relatively unfamiliar with JavaScript, I highly recommend you read this section - it'll save you a lot of pain down the road.

### The `var` keyword: what exactly does it do?
`var` declares a variable.  But...you don't need it.  Both these will work:

    var myFunction = function()
    {
        var foo = 'Hello'; // Declares foo, scoped to myFunction
        bar = 'Hello';     // Declares bar...in global scope?
    };

But you should never, EVER use the latter.  See the comment for the reason why.

If you ever forget the var keyword, your variable will be declared...but it will be scoped __globally__ (to the `window` object), rather than to the function it was declared in.

This is extremely bizarre behaviour, and really an unfortunate design decision in the JavaScript language.  There are no pragmatic situations where you would want local variables to be declared globally.  So remember to __always__ use `var` in your declarations.

### Why are there so many different ways to declare a function?  What are the differences?
There are three common ways to define a function in JavaScript:

    myFunction1 = function(arg1, arg2) {};     // NEVER do this!
    function myFunction2(arg1, arg2) {};       // This is OK, but...
    var myFunction3 = function(arg1, arg2) {}; // This is best!

The first option is bad because it declares a function like a variable, but without the `var` keyword.  Read the above point to understand why that's bad...it gets created as a global function!

The second option is better, and is properly scoped, but it leads to a slightly more complicated syntax once you get into closures.  It can also cause your code to behave in ways you may not expect due to [JavaScript variable hoisting][javascript-hoisting].

The third option is best, and is syntactically consistent with the rest of your variables.

As an alternative to the third option, you can use:

    var myFunction4 = function myFunction4(arg1, arg2) {};

Although more verbose, this option can be useful because it provides a little more context when debugging.

### The `this` keyword: how does it behave?
`this` in JavaScript does __not__ behave the way you would expect.  Its behaviour is very, very different from other languages.

If you are merely looking for a way to encapsulate and scope your code, I suggest avoiding the `this` keyword for now.  But if you truly need to use `this` to write object-oriented JavaScript, here's an explanation...

In more sane languages, `this` gets you a pointer to the current object.  But in JavaScript, it means something quite different: it gets you a pointer to the __calling context__ of the given function.

This will make much more sense with an example:

    var myFunction = function()
    {
        console.log(this);
    };
    var someObject = {};                // Create an empty object.  Same as: new Object();
    someObject.myFunction = myFunction; // Give someObject a property
    someObject.myFunction();            // Logs Object
    myFunction();                       // Logs...Window?

That's bizarre, isn't it?  Depending on how you call a function, its `this` pointer changes.

In the first example, because `myFunction` was a property of `someObject`, the `this` pointer referred to `someObject`.

But in the second example, because `myFunction` wasn't a property of anything, the `this` pointer defaults to the "global" object, which is `window`.

The important take-away here is that __`this` points to whatever is "left of the dot."__

Note that you can actually override what `this` points to by using the built-in `call()` and `apply()` functions.

As you can imagine, this causes a ton of confusion - particularly for new JavaScript developers.  My recommendation is to avoid writing code in such a way that relies on the intricacies of `this`.

### WTF are .constructor and .prototype?

If you're asking this question, it means you're getting knee-deep into JavaScript OOP.  Good for you!

The first thing you need to know is that JavaScript does NOT use classical OOP.  It uses something called prototypal OOP.  This is very, very different.  If you really want to know how JavaScript OOP works, you need to read [Constructors considered mildly confusing, by Joost Diepenmaat][constructors-confusing].  Joost does a better job of explaining it than I ever will.

But for the lazy, I'll summarize: JavaScript does not have any classes.  You don't create a class and spawn new objects off of it like in other languages.  Instead, you create a new object, and set its `prototype` property to point to the old one.

When you refer to an object's property, if that property doesn't exist JavaScript will look at the prototype object...and its prototype, and its prototype, all the way up until it hits the `Object` object.

So unlike the class/object model you see in other languages, JavaScript relies on a series of "parent pointers."

`constructor` is a pointer to the function that gets called when you use the `new` keyword.  If you want to learn more about the prototype/constructor relationship, read [The Surprisingly Elegant JavaScript Type Model, by Kannan Vijayan][javascript-type].  But be prepared to be confused.

### WTF is a closure?
Closures are a concept that appear in functional languages like JavaScript, but they have started to trickle their way into other languages like PHP and C#.  For those unfamiliar, "closure" is a language feature that ensures variables never get destroyed if they are still required.

This explanation is not particularly meaningful in and of itself, so here's an example:

    // When someone clicks a button, show a message.
    var setup = function()
    {
        var clickMessage = "Hi there!";
        $('button').click
        (
            function()
            {
                window.alert(clickMessage);
            }
        );
    };
    setup();

In a language without closures such as C/C++, you'd likely get an error when that click handler fires because `clickMessage` will be undefined.  It'll have fallen out of scope long ago.

But in a language with closures (like JavaScript), the engine _knows_ that `clickMessage` is still required, so it keeps it around.  When a user clicks a button, they'll see, "Hi there!"

As you can imagine, closures are particularly useful when dealing with event handling, because an event handler often gets executed long after the calling function falls out of scope.

Because of closures, we can go one step further and do something cool like this!

    (function()
    {
        var clickMessage = "Hi there!";
        $('button').click
        (
            function()
            {
                window.alert(clickMessage);
            }
        );
    })();

In the above example, we don't even need to give the function a name!  Instead, we execute it once with the () at the end, and forget about it.  Nobody can ever reference the function again, but _it still exists_.  And if someone clicks that button, it will still work!

### Why is parseInt() mucking up when I get into big numbers?
Despite looking like it, JavaScript doesn't actually have an integer data type - it only has a floating point type.  This isn't an issue when you do:

    parseInt("1000000000000000",10)<parseInt("1000000000000001",10); //true

but add one more zero:

    parseInt("10000000000000000",10)<parseInt("10000000000000001",10); //false

And you'll see where the difference between integers and floating points manifests.

### Why does JavaScript have so many different ways to do the same thing?
So you can choose which way is best for you. Some parts of JavaScript are not designed that well. Your best guide to muddle through it is to read [JavaScript: The Good Parts, by Douglas Crockford][good-parts].  He clearly outlines which pieces of the language you should ignore.

<!--
Break into DOM vs straight-up JS?
### How do I create a class?  How do I create public/private members? (should provide an experiement)
### How do I do inheritance? Overwriting automatically-created prototype object (should provide an experiement)
### WTF is \_\_proto\_\_?  (Chrome Inspector)  How is it different from prototype?
### What are the differences between ready(), load(), and inline JavaScript?
### What are the differences between bind(), delegate(), live(), and on()?
### My function is always returning null, even though I'm providing a return value.  Why?
### Pushing console.log() into production
### JavaScript never executing because an error was thrown earlier
### I'm trying to select a DOM element and it's coming back as undefined, even though I know it's there.  Why?
-->

[javascript-gotchas]: https://github.com/stevekwan/best-practices/blob/master/javascript/gotchas.md
[javascript-best-practices]: https://github.com/stevekwan/best-practices/blob/master/javascript/best-practices.md
[good-parts]: http://shop.oreilly.com/product/9780596517748.do
[javascript-hoisting]: http://www.adequatelygood.com/2010/2/JavaScript-Scoping-and-Hoisting
[constructors-confusing]: http://joost.zeekat.nl/constructors-considered-mildly-confusing.html
[jquery-api]: http://api.jquery.com/
[javascript-type]: http://vijayan.ca/blog/2012/02/21/javascript-type-model/
[partial-application]: http://benalman.com/news/2012/09/partial-application-in-javascript/
[js-adolescence]: http://james.padolsey.com/javascript/js-adolescence/
[object-function-experiment]: https://github.com/stevekwan/experiments/blob/master/javascript/object-vs-function.html
[constructor-prototype-experiment]: https://github.com/stevekwan/experiments/blob/master/javascript/constructor-vs-prototype.html
