---
layout: post
title:  "NodeJs Best Practices"
date:   2014-06-08 12:03:49
categories: nodejs
---

# What I Wish I’d Read When I Started Node

As an experienced programmer, I've found that NodeJS is easy to learn with many tutorials to cover the basics.  However, I was (and still am) unclear about what are the best practices for NodeJS. Here are a few of the things I've learned so far...

## Everything in Node should be done with callbacks.

It can be tempting to write functions that just return results like:

{% highlight javascript %}
  function calculateScore(someInput) {
    …
    return score;
  }
{% endhighlight %}

But that’s generally a bad idea because you’ve closely tied your interface to your implementation.  That means if you decide to change your implementation to do something asynchronous, like fetching a cached value, you now have to change all of your callers.  In practice you’ll run into a lot of bugs where the function actually returns nothing (undefined) or fails to call a callback, making your program hang.

NodeJS requires you to invert the usual sequential way of solving problems into an asynchronous set of callbacks.  Embrace it.  Once you wrap your head around this inverted idea, it’s actually a lot of fun and powerful.  Have you ever tried to run parallel database queries in PHP?

Use the async library to organize your callbacks

There are some nice articles [link] on how to prevent the “callback soup,” but here’s what worked for me in practice:

1.  Move most of your anonymous callbacks to clearly named functions that belong to the class*, following good object oriented design.
2.  Use the async library to handle all of your common practices like calling functions in parallel or in a series.  Skip promises, which are cooler in theory than in practice.  Async just works.

## Invoke all callbacks with exacty two arguments like this: callback(err, response)

One of my biggest frustrations is how unpredictable callbacks are in NodeJS, because the number of arguments is arbitrary. Most Node authors respect the convention that the first argument is an error (or null), although you can omit it:

    callback(data);

But your caller (which is probably you) will then write:

    oddCallback(function(err, data) {
      if (err) {
        // false alarm, this is the data :(
      }
    });

Sadly, a few Node core libraries like filter [cite] follow this pattern.  The problem is compounded if you use the async library, which expects the standard (err, data) format:

    async.waterfall([
      (next) => someFunction()..
      // broken :(
    ], ...)

Furthermore, I believe strongly that you should pass the actual response as one argument.  This is the same reason you should not return multiple values from a function.  For example this is bad practice in Python:

return [users, total_users]  # <— God help you if you change this return value.

This happens often enough in Node libraries to be infuriating.  For example:

    // mongoose js model
    user.save(function(err, user, num) {
      ...
    });

The real reason this is a problem is that you are violating the *Single Responsibility Principle*: your function returns multiple arguments because it’s trying to do more than one thing.  The awkward callback is a symptom of OO design that needs rethinking.  In this case it would really be better to return some type of Response object that encapsulates the saved model, number saved, etc.  In the future you can add information like query time without breaking the contract.

In practice this ambiguity has caused more bugs for me than anything else.  For example:

    async.waterfall([
      function(callback) {
        user.save(callback);
      },
      function(user, callback) {
       // oops, next is actually a number, so there is an error or the script just hangs
      },
      ...
    )




## Don’t Create Optional Parameters

I’m not sure why this is so common, but I think the idea is that the author is supporting a wider variety of inputs and shorthands.  In reality, I think this only supports non-deterministic behavior:

    myFunction(api, options, callback) {
      if typeof callback === “undefined”
        callback = options

    }

This feels very clever, but really it just achieves the convenience of being able to write

    myFunction(‘my/api/endpoint/‘, callback)

instead of:

    myFunction(‘my/api/endpoint/‘, {}, callback)

Just because JavaScript is very loose about parameters doesn’t mean you should take advantage of this fact to make ambiguous interfaces.  Experience has taught me to make my code very explicit.  Ambiguity = bugs.

## Streams are better in theory than practice

Streams are an elegant and powerful idea in Node, allowing you to process large data sets efficiently, and send network responses on-the-fly.

The only issue is that while Node v.10 vastly improved streams, the libraries (e.g. MongooseJS and csv-node) have not caught up.  To use a simple example, this is how you should be to process millions of collections:

    // users is a massive table with millions of documents
    stream = Users.find().stream();
    stream.on(‘data’, function(user) {
      stream.pause();
      doSomethingExpensive(user, function() {
        stream.resume();
      });
    });
    stream.on(‘end’, function() {
      // all done, disconnect from the db
    });

...except the stream.pause() function is “strictly advisory” and therefore doesn’t work :(.  I’ve experienced the same problems in other libraries.

I hope this is rapidly solved as the libraries mature, because this is a much better way to think about data problems.  However, I will reserve judgment until then.


## Warning: Event Emitting is Blocking

There is a very unfortunate inconsistency at the heart of NodeJS.  As you’ve probably read, everything in Node is non-blocking, which can be extremely efficient without multithreading.  At the same time, JavaScript is built around event emitting, which is a powerful way to decouple code, whether you want to call it Pub/Sub or whatever.

The issue is that doing...

    myObject.emit(‘some notification…’)

...is a blocking operation while all listeners respond.

I’ve talked to core contributors that argue this is a fundamental design flaw in Node, and are arguing to change it.  I’ve also read good arguments that the listeners should be blocking, otherwise there is no guarantee they’ll have a chance to respond to the event before it’s too late.  The important point is that you should be aware of this inconsistency and plan accordingly.  Which brings me to my next point:

## Most NodeJS Examples Assume a Persistent Server

A related problem I have encountered with events is that Node authors tend to assume that you are working with a persistent instance, like a web server, so all events and callbacks will have unlimited time to complete.  This assumption falls apart when you start to write background scripts that are intended to run and shutdown, releasing resources like DB connections.

My issue occurred because I want to emit and listen for events (to keep my code nicely decoupled), while ensuring that any subsequent asynchronous behavior completes.  To make this more concrete:

When you create a company in [DataFox](http://datafox.co), we trigger many background jobs like crawling the corporation’s website and hitting various APIs.  Rather than have my Company class know about all of these ever-changing jobs, I have it emit a “new company” event, which those jobs can listen for.  So for example:

    // in WebsiteCrawler
    Company.on(‘new company’, function(company) {
      // the request is non-blocking.
      request.get(company.url, function(err, response) {
        // process the response and write to the db…
      }
    });

So what happens if my script completes before the http response gets back?  Yes, Node will not terminate running while there are outstanding callbacks, but I can still have issues if I’ve disconnected from the database.


## Error Handling in Node is a Mess
Error handling in asynchronous code is tricky and Node compounds the problem by being very inconsistent.  The problem is being addressed in new version of Node, but here are some pitfalls to avoid:

## Don’t Ever Throw Exceptions
You’ll quickly discover that when you throw an exception, it’s probably not going to be caught:

    function() {
      try {
        asynchronousFunctionThatThrowsException();
      } catch (Exception e) {
        // won’t catch anything
     }
    }

Unfortunately, many third party libraries do exactly this, causing your code to die.  The solution is Domains and the proposed zones (?), but they’re not fun.

## Always Return Proper Error() Objects
This was not clear to me at first, but if you have an error you should always return a new Error().  You can subclass the Error.

## Create a Logging Class
If you try to log an object, you’ll get the useless “[object Object]” so I recommend doing JSON.stringify(object) instead.  But be warned that JSON.stringify(new Error(‘error message’)) returns “{}”, so here you want to just “” + error.

I suggest creating a simple logging library which tests what you’re logging and does the right thing.  Plus, you can have it log a stack trace on errors, to help in debugging where they came from.

You can use a 3rd party logging library, but I ran into a surprising number of issues here and just wrote my own.  In particular, I’d warn you against using winston, which has an inexplicable number of bugs (such as not actually writing all logs) for such a simple library.


## Avoid Circular Requires in NodeJS

I recently ran into this error:

  “getUser is not defined”.

My code looked this:

    UserService = require(‘./user’);

    module.exports = class CompanyService

      getCompaniesForUser:  =>
      userService = new UserService;
      userService.getUser(…);


Nothing looks wrong.  After debugging I found that UserService = {}.  What?

The issue turns out to be a circular dependency.  If you have

    # file a

    B = require ‘./file_b’
    module.exports = …

    # file b

    A = require ‘./file_a’

    # A == {}

    module.exports = …


When B tries to require A, A is not initialized, so you actually import an empty object!  I’m curious if there is a good explanation for this silent failure, but it seems like a serious design flaw to me.

The solution is to either avoid the circular import if possible, or to lazy-require the files.  For example:

    getB = function() {
      B = require(‘./file_b’);
      return new B;
    }

(Note: the Node modules are cached, so it’s okay to call the function repeatedly.)



## Conclusion

This is not meant to be an authoritative list, and I haven’t covered other important topics such as installing packages with NPM or deploying and monitoring NodeJS.  I’m still learning the right way to tackle Node and would love to hear feedback.





* technically, the prototype of the function






