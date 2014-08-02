---
layout: post
title:  "NodeJs Best Practices: Avoiding Circular Dependencies"
date:   2014-06-08 12:03:49
categories: nodejs
---

## Avoid Circular Requires in NodeJS

I recently ran into this error: "getUser is not defined".

My code looked this:

    UserService = require(‘./user’);

    module.exports = function CompanyService() {

      this.getCompaniesForUser = function() {
        userService = new UserService;
        userService.getUser(...);
      }


Nothing looks wrong.  After debugging I found that `UserService = {}`.  What?

The issue turns out to be a circular dependency.  If you have two files that require each other like:

    B = require('./file_b');
    module.exports = ...

and

    A = require('./file_a');
    // A == {}

    module.exports = ...


When B tries to require A, A is not initialized, so you actually import an empty object!  I’m curious if there is a good explanation for this silent failure, but it seems like a serious design flaw to me.

The solution is to either avoid the circular import if possible, or to lazy-require the files.  For example:

    getB = function() {
      B = require('./file_b');
      return new B;
    }

(Note that NodeJS modules are cached, so it’s okay to call the function repeatedly.)
