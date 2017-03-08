---
layout: post
title:  "NodeJS Best Practices: Callbacks"
date:   2014-06-08 12:03:49
categories: nodejs
uuid: b54a5ee7-36e7-4996-8d2e-ed760344b260
---

As an experienced programmer, I've found that NodeJS is easy to learn with many tutorials to cover the basics.  However, I was (and still am) unclear about what are the best practices for NodeJS. Here are a few of the things I've learned so far about callbacks:

## _Everything_ in Node should be done with callbacks.

It can be tempting to write functions that just return results like:

{% highlight javascript %}
  function calculateScore(someInput) {
    …
    return score;
  }
{% endhighlight %}

But that’s generally a bad idea because you’ve closely tied your interface to your implementation.  That means if you decide to change your implementation to do something asynchronous, like fetching a cached value, you now have to change all of your callers.  In practice you’ll run into a lot of bugs where the function actually returns nothing (undefined) or fails to call a callback, making your program hang.

NodeJS requires you to invert the usual sequential way of solving problems into an asynchronous set of callbacks.  Embrace it.  Once you wrap your head around this inverted idea, it’s actually a lot of fun and powerful.  Have you ever tried to run parallel database queries in PHP?

## Use the async library to organize your callbacks

There are some lots of ways to prevent “callback soup,” but here’s what worked for me in practice:

1.  Move most of your anonymous callbacks to clearly named functions, following good object-oriented design.
2.  Use the awesome [async library](https://github.com/caolan/async) to handle all of your common practices like calling functions in parallel or in a series.  Promises and generators are both cool ideas that falter in practice.  Async just works.

## Callbacks should have **exactly two** arguments

What's wrong with this code?

    fs.exists('/etc/passwd', function (err, exists) {
      util.debug(exists ? "it's there" : "no passwd!");
    });

`fs.exists` doesn't follow the convention of invoking a callback with an `err` parameter so `exists` is undefined and silently cast to false. Oops.

One of my biggest frustrations is how unpredictable callbacks are in NodeJS, because the number of arguments is arbitrary. While most authors respect the convention that the first argument is an error (or null), you can't rely on it.  The problem is compounded if you use the async library, which expects the standard (err, data) format:

    async.waterfall([
      function(next) {
        fs.exists('/etc/passwd', next);
      },
      function(next) {
        // next is undefined and the script hangs...
      }
    ], ...);

Furthermore, I believe strongly that you should pass the actual response as one argument.  This is the same reason you should not return multiple values from a function.  For example this is bad practice in Python:

    # changing this array will probably break callers
    return [users, total_users]

This happens often enough in Node libraries to be infuriating.  For example:

    // mongoose js model
    user.save(function(err, user, num) {
      ...
    });

The real reason this is a problem is that you are violating the **Single Responsibility Principle**: your function returns multiple arguments because it’s trying to do more than one thing.  The awkward callback is a symptom of OO design that needs rethinking.  In this case it would really be better to return some type of Response object that encapsulates the saved model, number saved, etc.  In the future you can add information like query time without breaking the contract.

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

Rigidly sticking with the convention of `callback(err, data)` saves a lot of headache.

## Conclusion

NodeJS is awesome, but the best practices are stilling emerging.  I hope these thoughts are helpful to others who are adopting NodeJS and I'd love to hear more about other best practices.





