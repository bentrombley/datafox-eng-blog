---
layout: post
title:  "NodeJS Best Practices: Error Handling in Express"
date:   2017-05-29
categories: nodejs
---

There are already many great articles about how to do error handling in NodeJS.  They boil down to this:

  - _Never_ throw an exception, since this can't be caught correctly with asynchronous code
  - Use promises to handle errors
  - _Always_ use `Error` objects to return errors, not strings or objects
  - Callbacks should always include an error or null as the first parameter

However, if you're creating an API using Express, this is only a start.  Any good API returns clear status codes and error messages.  For example, if the user lacks permission to perform an action they should receive a 403 status code, while a missing parameter should return a 400 status code with a nice message pointing them to the appropriate documentation.

This is easier said than done.  The key is to build proper error handling into your entire system.

## Use Better Errors

### Stop using Generic Errors

Hopefully, you are already using `Error` objects every time you return an error.  But the default error object is very limited, passing only a string value.  Fortunately, you can extend the Error object with your own subclasses.

### Use Common-Errors

Rather than reinvent the wheel, I suggest you start by adding the `common-errors` npm module and using it everywhere.  This library includes common error cases like `ArgumentNullError` and `NotFoundError` and `NotPermittedError`.  You can also define your own Errors using its `generateClass()` method.

### Or Better: Fork Common-Errors

The Common Errors library is great, but it unfortunately hard-codes a lot of behavior that may not match up with your requirements.  We ended up forking common-errors to support our own needs.  In our version all errors have this standard format:

    new Error(message, errorCode, data)

Where `errorCode` and `data` are optional.  The intention is that they include information you would need to display back to the user.  For example, on a `NotPermittedError` `errorCode` could be `account_disabled` and `data` could be `{account: <account Id>, disable_reason: "credit card expired"}`.


## Use Error Middleware

## Use the Express Error Handler Middleware

Let's start with a simple route in Express for retrieving users that just calls a `getUsers()` function and returns the results as JSON:

    app = express();
    app.get('/users', function(res, req, next) {
      getUsers(function(err, users) {
        res.json(users);
      });
    });

If the call to `getUsers()` errors, we should return an error status.  The temptation is to do this:

    app = express();
    app.get('/users', function(res, req, next) {
      getUsers(function(err, users) {
        if (err) {
          // something went wrong, I guess we'll send a 500 status code...
          return res.send(500);
        }
        res.json(users);
      });
    });

But a 500 error status means that something generally went wrong on our side -- basically we crashed.  But what if the error is that you aren't logged in or don't have permission to call `getUsers()`?  A better solution is this:

    app = express();
    app.get('/users', function(res, req, next) {
      getUsers(function(err, users) {
        if (err) {
          return next(err);
        }
        res.json(users);
      });
    });

Simply call `next()` with the error.  This passes the error to the Express error handler, which then can translate into an error code.

One massive benefit is that you can eliminate almost all error handling code in your routes; simply pass along the error for someone else to process.  A further benefit is that you now consolidate your error handling in one place.

## Better Still, use the Common-Errors Error Handler

If you are using the `common-errors` library you can leverage it's error handler to automatically translate your errors into the correct HTTP status codes with generic messages by adding this in your Express app after you define your routes:

    app.use(errors.middleware.errorHandler);


### Rolling Your Own

Alternatively, you can define your own handler based on these.  For example this one we use in our internal apps:

    app.use((err, req, res, next) =>
      if (!err) {
        if (next) {
          return next();
        }
        return res.end();
      }

      // use common-errors mapping between error type and status code
      httpStatusError = new errors.HttpStatusError(err, req);
      messageMap = errors.HttpStatusError.message_map;
      statusCode = httpStatusError.status_code;
      errorCode = messageMap[statusCode] || messageMap[500];    // default to a 500 error

      response = {
        object_type: "error"
        status_code: statusCode
        error_code: errorCode
        details: []
      };

      // for validation errors, fill in the details so we can report on all fields that were invalid
      if (err instanceof errors.ValidationError) {
        errorJson = err.toJSON();
        if (_.isArray(errorJson.errors)) {
          _.each(errorJson.errors, function(message) {
            if (message && message.text) {
              response.details.push({text: message.text});
            }
          });
        }
      }

      res.status(statusCode);
      res.json(response);
    );

This gives you more control on the exact format -- in our case we want to return a full JSON response with details.


## Final Thoughts

Error handling is a deceptively-simple topic, and there are more ways you can improve.

First, you can add an error handler to handle uncaught errors and gracefully return a 500 error.  I suggest you also use a crash reporting tool to alert you when your code is crashing.  We use [Bugsnag](https://www.bugsnag.com/) and I recommend it.

Next, design your application to fail gracefully.  For example, add a global error handler in your frontend code to tell the user if they don't have permission to perform an action or the item could not be found.  Similarly, on the backend design your code with the expectation that other systems will fail -- your application shouldn't crash just because the e-mail server went down.

Lastly, make error handling a key part of your design process.  When you are building systems and services, think about how they will fail, not just how they will succeed.  And when things do go wrong, learn from your mistakes with in-depth fixes rather than patching the superficial problem.  Add logging so you can quickly detect and debug future issues.  But most importantly, emphasize that errors are as much a part of engineering as any feature specification.

