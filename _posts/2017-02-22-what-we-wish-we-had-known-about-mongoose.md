---
layout: post
title:  "What We Wish We Had Known About Mongoose"
date:   2017-02-22
categories: mongoose
uuid: a508a6b5-03f7-4a51-a31a-30c725c83a5e
---

At DataFox we use [mongoose](http://mongoosejs.com/) as an ORM layer that abstracts our MongoDB calls. Overall, we recommend using it because it improves code clarity and effectively adds schemas to MongoDB without the overhead of table migrations.  However, we've learned a few things that you should be aware of when using in production, particularly at scale.

## Mongoose is a CPU hog at scale

By default when you call a method like `find()` or `findOne()`, Mongoose returns a full model complete with mongoose methods and virtuals. For example:

{% highlight javascript %}
User.findOne({email: "data@datafox.co"}, function(err, user) {
  user.status = 'active';
  user.save(callback);
});
{% endhighlight %}

In most cases this code works fine.  However, Mongoose is very inefficient in how it constructs the model from the raw MongoDB JSON response.  This not only causes CPU spikes on your server, but it blocks the NodeJS event loop, effectively blocking all other work and bringing your entire site to its knees.

## Use lean() to avoid CPU spikes

If you are simply reading data from the database, we highly recommend you use the `lean()` method like this:

{% highlight javascript %}
Company.find().lean().exec(function(err, allTheCompanies) {
  // CPU is not a problem, though memory is now an issue...
});
{% endhighlight %}

This essentially returns the raw JSON object from MongoDB.  The compromise is that you can no longer can call methods like `save()` or use virtual fields.  Mongoose is modeled after Rails' ActiveRecord with "smart" models, but it's really better to treat models as simple JSON objects and move those functions to other classes.


## Learn to Love Streams

In the above example, if you call `Company.find()` on a big collection you will quickly load more data into memory than you can handle.  Fortunately, both NodeJS and Mongoose support streaming results so you can handle large results efficiently.  Unfortunately, the syntax is awkward and easy to confuse.  Here's our solution:

### Use StreamWorker

[StreamWorker](https://github.com/goodeggs/stream-worker) makes using streams trivial by hiding the awkward query hooks. To improve our previous example:

{% highlight javascript %}
// note that the lean() command works here too
let stream = Company.find().lean().stream();
const PARALLELISM = 5;
StreamWorker(stream, PARALLELISM, processOneCompany, callback);
{% endhighlight %}

Now we can define a function like `processOneCompany` which will handle the companies one at a time, and StreamWorker will handle the annoying onFinish hooks.

### Prevent Timeouts

One disastrous and silent error can occur when processing a very long streaming query: a timeout. This can happen at either the network level or the database level, so you need to guard against both.

First when connecting to the database specify the `keepAlive` parameter to prevent a network disconnect on a long-running query.

{% highlight javascript %}
mongooseInstance = new mongoose.Mongoose();
options = {
  user: config.DB_USER_NAME,
  pass: config.DB_PASSWORD,
  server: {socketOptions: {keepAlive: 1}},
  replset: {socketOptions: {keepAlive: 1}},
};
mongooseInstance.connect(config.DB_URL, options);
{% endhighlight %}

Second, and just as importantly, don’t forget to set the `timeout` parameter to `true` (the name "timeout" is an unforgivable shorthand for "noCursorTimeout" in MongoDB).

{% highlight javascript %}
User.find().setOptions({timeout: true});
{% endhighlight %}

Note that you may still run into configurations in MongoDB or your network that can cause disconnect issues, but this will solve most cases.


### Don’t Forget to Catch Errors

You must subscribe to the stream’s `on('error')` or you can experience silent failures as the error is caught and never returned to the callback.


## Add a Backtrace on Your MongoDB Queries

Hopefully, you have alerting on your MongoDB logs to track slow queries and tables scans (called "COLLSCAN" in the logs).  However, it is often frustratingly difficult to figure out what code is creating the inefficient query so you can fix it.

Solve this problem forever by appending a stack trace in the `$comment` field of all queries using Mongoose's pre-find hook.  For example:

{% highlight javascript %}
schema.pre('find', function(next) {
  this.comment(getShortStackTrace());
  next();
});
{% endhighlight %}

The `$comment` is very short, so you'll want a very short version of the back trace without the Mongoose internals:


{% highlight javascript %}
StackTraceParser = require('stacktrace-parser');

/**
  @return {String} short-version of stack trace that excludes mongoose or mongo
  connection frames, for use in mongodb $comment
*/
getShortStackTrace = function() {
  stack = StackTraceParser.parse(new Error().stack);
  stackString = "";
  count = 0;
  for (var i = 0; i < stack.length; i++) {
    var frame = stack[i];
    if (count > 2) break;
    // ignore stack trace that just shows this file or mongoose internals
    if (frame.file.indexOf('mongoose') === -1 &&
        frame.file.indexOf('mongo_connection') === -1 &&
        frame.file.indexOf('kareem') === -1) {
      stackString += "#{frame.methodName} (#{frame.file}:#{frame.lineNumber}\t";
      count++;
    }
  return stackString;
};
{% endhighlight %}

## Use Schema Plugins to add Created and Updated Timestamps

Mongoose supports "plugins" which behave like a mixin or trait. This lets you easily define methods and attributes across all of your models. One of the most useful in our experience is automatically saving the created and last-modified times on all models to help in debugging and reporting.

Here is is the plugin:

{% highlight javascript %}
// plugin for mtime/ctime
module.exports.timestamps = function(schema, options) {
  schema.add({ctime: {type: Date}});
  schema.add({mtime: {type: Date}});

  schema.pre('save', funtion(next) {
    if (this.isNew and !this.ctime) {
      this.ctime = new Date();
    }
    this.mtime = new Date()
    next();
  );
};
{% endhighlight %}

Use it like this:

{% highlight javascript %}
plugins = require('plugins');
schema.plugin(plugins.timestamps);
{% endhighlight %}



## Create a Wrapper for Creating Models

As your create more models, it becomes difficult to enforce consistent rules across them -- for example that all models should use the `timestamps` plugin above.

We've found that it is simplest to create a centralized `defineModel()` method, and use linters that error if you try to call `mongoose.model()` anywhere else.

In a model class it looks like:
{% highlight javascript %}
schema = new mongoose.Schema({ ... fields ... });
module.exports = MongoConnection.defineModel('Company', schema);
{% endhighlight %}

`defineModel()` then handles adding the timestamps plugin and applying the hooks to `find()` calls.




## Be Aware of ensureIndex()

One major benefit to using Mongoose is that you can define the schema for your collections in code, making it easy to modify the schema without a database migration. Mongoose further adds support for defining indexes in code like so:

{% highlight javascript %}
schema = new mongoose.Schema({
  name: String,
  ...
});

// index on name (ascending order)
schema.index({name: 1});

module.exports = mongoose.model('Company', schema);
{% endhighlight %}

Mongoose supports all of the advanced index parameters, so you can create unique indexes, compound indexes, partial indexes, and indexes on subdocuments or arrays.

In practice this works because Mongoose automatically calls `ensureIndex()` when the module is required. If the index exists, the call is ignored, but if the index is new, MongoDB will immediately build the index in the background and replicate the command to all secondaries.

In most cases this is fine, but on a large table the command will have a performance impact on the primary database and can lock the secondary databases for a significant amount of time. Furthermore, if you are applying a unique index to a field that is not currently unique, the command will fail.  Or worse, if a duplicate is introduced before the index is built on the secondary database, the entire secondary will be shut down.

The drastic solution is to disable ensureIndex on production environments like:

{% highlight javascript %}
new Schema({...}, {autoIndex: false});
{% endhighlight %}

and run the `ensureIndex()` call manually as part of a migration process.

A compromise (while you’re small) is to simply expect the call as part of a code deployment and ensure it happens in off-peak hours after testing.

## Ensure Index Can Fail Silently

A far more insidious problem, however, is that when a call to `ensureIndex()` fails the error is swallowed, leading to bizarre behavior where `find()` queries return truncated results. There is an closed but not fixed [mongoose issue](https://github.com/Automattic/mongoose/issues/609) for this and it points to the root cause as this as-yet unresolved [MongoDB issue](https://jira.mongodb.org/browse/SERVER-4462).

## Catch Connection Failures

Be careful to subscribe to the error hooks when you connect to the database, or you’ll be stuck with silent failures.

{% highlight javascript %}
dbInstance = new mongoose.Mongoose();
dbInstance.connect(DB_URL, options);
dbInstance.connection.on("error", function(err) {
  // log error...
});
{% endhighlight %}




## Conclusion

Mongoose and MongoDB are both great tools and we hope these hints save you time as you deploy them to production.  Both MongoDB and Bongoose are evolving and hopefully it will become easier and easier for new users to employ best practices which avoid some of the pitfalls mentioned above. Please feel free to contact us if you have any others that you'd like to share!
