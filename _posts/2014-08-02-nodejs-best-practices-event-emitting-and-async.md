---
layout: post
title:  "NodeJs Best Practices: Event Emitting and Async Callbacks"
date:   2014-08-02
categories: nodejs
---

## The Issue

There is a very unfortunate inconsistency at the heart of NodeJS.  As you’ve probably read, everything in Node is non-blocking, which can be extremely efficient without multithreading.  At the same time, JavaScript is built around event emitting, which is a powerful way to decouple code, whether you want to call it Pub/Sub or whatever.

The issue is that doing...

    myObject.emit("some notification...")

...is a blocking operation while all listeners respond.

I’ve talked to core contributors that argue this is a fundamental design flaw in Node, and are arguing to change it.  I’ve also read good arguments that the listeners should be blocking, otherwise there is no guarantee they’ll have a chance to respond to the event before it’s too late.  The important point is that you should be aware of this inconsistency and plan accordingly.  Which brings me to my next point:

## Most NodeJS Examples Assume a Persistent Server

A related problem I have encountered with events is that Node tutorials tend to assume that you are working with a persistent instance, like a web server, so all events and callbacks will have unlimited time to complete.  This assumption falls apart when you start to write background scripts that are intended to run and shutdown, releasing resources like DB connections.

My issue occurred because I want to emit and listen for events (to keep my code nicely decoupled), while ensuring that any subsequent asynchronous behavior completes.  To make this more concrete:

When you create a company in [DataFox](http://www.datafox.co), we trigger many background jobs like crawling the corporation’s website and hitting various APIs.  Rather than have my `Company` class know about all of these ever-changing jobs, I have it emit a “new company” event, which those jobs can listen for.  So for example:

    // in WebsiteCrawler
    Company.on('new company', function(company) {
      // the request is non-blocking.
      request.get(company.url, function(err, response) {
        // process the response and write to the db...
      }
    });

So what happens if my script completes before the http response gets back?  Yes, Node will not terminate running while there are outstanding callbacks, but I can still have issues if I’ve disconnected from the database.

## My Solution

The solution is to avoid doing any real work (asynchronous calls) in an event listener.  For me this means a listener should enqueue the work to be done, using [async.queue](https://github.com/caolan/async#queue):

    fetchUrlQueue = async.queue(functionThatWillFetchUrl);
    Company.on('new company', function(company) {
      fetchUrlQueue.push({url: company.url});
    });

Then I can check that this queue is drained before disconnecting from the database, etc. and shutting down my script.
