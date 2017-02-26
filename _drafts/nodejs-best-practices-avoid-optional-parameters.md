---
layout: post
title:  "NodeJS Best Practices: Avoid Optional Parameters"
date:   2017-02-26
categories: nodejs
---

NodeJS authors really love taking advantage of JavaScript's looseness to create ambiguous functions.  For example, mongoose, a popular ORM for MongoDB, a function to populate a model that can invoked any of these ways:

    .populate('path')

    .populate('path1 path2')

    .populate('path1').populate('path2')

    .populate({path: 'path', ...})

with subtly varying results.

I also frequently see code like this:

    myFunction(api, options, callback) {
      if (typeof callback === "undefined") {
        callback = options;
      }
      ...
    }


This feels very clever, but really it just achieves the convenience of being able to write

    myFunction(‘my/api/endpoint/‘, callback);

instead of:

    myFunction(‘my/api/endpoint/‘, {}, callback);


Perhaps the authors are imitating the pattern of overloading functions in languages like C, although in these statically-typed languages your IDE makes them far less ambiguous.  And, I would argue, this is not a great pattern to imitate in the first place.

I believe this pattern is so common because library authors are trying to be accomodating, which is noble but misguided.  Unlike frameworks, libraries should not be opinionated and should accept the widest variety of inputs.  It's definitely better for your library to accept a generic `Stream`, for example, than to expect a `FileStream` [fix this example].

However, accepting a wide variety of inputs is different from accepting a wide variety of syntax.  The latter means supporting a wide variety of requirements.  In the stream example it could mean accepting a stream from S3 rather than the filesystem.  Accepting a wide variety of syntax, however, just accomodates programmers typing, and that is not a valid concern.

The problem is that the library now has an ambiguous interface.  Ambiguity is wonderful in literature and art, but it's anathema in code.  In practice these ambigous interfaces make your code:

* harder to document and search
* harder to predict, since it may accept more than you expect
* harder to catch mistakes, since the code will tries to accomodate those unintended mistakes.
* harder to refactor, because parameters may or may not be used.

Writing short-hand versions of a function feels clever, but it is ultimately an attempt to do the IDE's job.

In the case where you truly do have a large number of optional parameters, I suggest this syntax using ES6:


    function sendEmail({subject, content, from, to, cc, bcc}, callback) {
      ... set default values for any omitted fields ...
    }

In this case you may or may not want to include a set of e-mails to cc or bcc, and I can imagine other options.  However, it's clear which values are being passed in, and which are omitted.



