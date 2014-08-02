---
layout: post
title:  "NodeJs Best Practices: Avoid Optional Parameters"
date:   2014-06-08 12:03:49
categories: nodejs
---

NodeJS authors really love taking advantage of JavaScript's looseness to create ambiguous functions.  For example, mongoose (MongoDB ORM)'s `populate` function can be invoked any of these ways:

    .populate('path')

    .populate('path1 path2')

    .populate('path1').populate('path2')

    .populate({path: 'path', ...})

with subtly varying results.

I also frequently see code like this:

    myFunction(api, options, callback) {
      if (typeof callback === "undefined")
        callback = options
      ...
    }


This feels very clever, but really it just achieves the convenience of being able to write

    myFunction(‘my/api/endpoint/‘, callback)

instead of:

    myFunction(‘my/api/endpoint/‘, {}, callback)


I’m not sure why these patterns are so common, but I suspect the author believes they are supporting a wider variety of inputs and shorthands.  In practice this has makes code:

* harder to document and search
* harder to predict, since it may accept more than you expect
* harder to catch mistakes, since the code will tries to accomodate those unintended mistakes.

Experience has taught me the importance of writing explicit code.  Ambiguity causes bugs.