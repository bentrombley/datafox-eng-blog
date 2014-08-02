---
layout: post
title:  "NodeJs Best Practices: Streams"
date:   2014-06-08 12:03:49
categories: nodejs
---

## Streams are better in theory than practice

Streams are an elegant and powerful idea in NodeJS, allowing you to process large data sets efficiently without loading them all into memory at once.  You can even pipe streams like unix, to chain together processing.

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

