---
layout: post
title:  "What We Wish We Had Known About Mongoose"
date:   2015-06-01
categories: mongoose
---

At DataFox we use [mongoose](http://mongoosejs.com/) as an ORM layer that abstracts our MongoDB calls. Generally, we benefit from cleaner, more modular code that hides the nitty-gritty details of the MongoDB driver - but mongoose isn’t perfect and it certainly doesn’t abstract all of the problems that can occur when using MongoDB.

As such, we decided to collect a list of "gotchas" that might be useful to others beginning to use mongoose in a production environment...

## Beware of ensureIndex()

One major benefit to using Mongoose is that you can define the schema for your collections (i.e. SQL tables) in code, making it easy to modify the schema without a database migration. Mongoose further adds support for defining indexes in code like so:

{% highlight coffeescript %}
schema = new mongoose.Schema({
  name: String
  ...
})

# index on name
schema.index({name: 1})

module.exports = mongoose.model('Company', schema)
{% endhighlight %}

Mongoose supports all of the advanced index parameters, so you can create unique indexes, compound indexes and indexes on subdocuments or arrays.

In practice this works because Mongoose automatically calls `ensureIndex()` when the module is required. If the index exists, the call is ignored, but if the index is new, MongoDB will immediately build the index and replicate the command to all secondaries.

In most cases this is fine, but on a large table the command will lock all writes for a significant amount of time. Furthermore if you are applying a unique index to a field that is not currently unique, the command will fail -- or if a duplicate occurs on the secondary the entire database will shut down.

The drastic solution is to disable ensureIndex on production environments like:

{% highlight coffeescript %}
new Schema({...}, {autoIndex: false})
{% endhighlight %}

A compromise (while you’re small) is to simply expect the call as part of a code deployment and ensure it happens in off-peak hours after testing.

## Ensure Index Can Fail Silently

A far more insidious problem, however, is that when a call to `ensureIndex()` fails the error is swallowed, leading to bizarre behavior where `find()` queries return truncated results. There is an open [mongoose issue](https://github.com/Automattic/mongoose/issues/609) for this and it points to the root cause as this as-yet unresolved [MongoDB issue](https://jira.mongodb.org/browse/SERVER-4462).

## Catch Connection Failures

Be careful to subscribe to the error hooks when you connect to the database, or you’ll be stuck with silent failures.

{% highlight coffeescript %}
dbInstance = new mongoose.Mongoose()
dbInstance.connect(DB_URL, options)
dbInstance.connection.on("error", (err) ->
  # log error...
)
{% endhighlight %}

## Prevent CPU Spikes by Using Lean Models
By default when you call a method like `find()` or `findOne()`, Mongoose returns a full model complete with mongoose methods and virtuals. For example:

{% highlight coffeescript %}
User.findOne({email: "data@datafox.co"}, (err, user) =>
  ...
  user.status = 1
  # save() method works
  user.save(callback)
)
{% endhighlight %}

This works in most cases, but in bulk the work of constructing thousands of mongoose objects from the database, results can become CPU-bound, causing CPU thrashing which blocks Node (and everything else on the server).

The solution in most cases is to add the `lean()` parameter to your query so mongoose returns only the raw JS object. For example:

{% highlight coffeescript %}
Company.find().lean().exec((err, allTheCompanies) =>
  ...
)
{% endhighlight %}

You no longer can call methods like `save()` or use virtuals\ fields, but in most bulk cases this is acceptable.

Unfortunately, this means sacrificing some the encapsulation afforded by defining methods on your models, but in the long run it's better to treat mongoose models as simple MongoDB documents, rather than full-blown classes.

## Learn to Love Streams

Streams are one of the best parts of NodeJS, allowing you to process large data sets efficiently, but it remains buggy and awkward in many libraries... including Mongoose. Here’s how to make it work:

### Use StreamWorker

StreamWorker makes using streams trivial by hiding the awkward query hooks. To improve our previous example:

{% highlight coffeescript %}
stream = Company.find().lean().stream() # note that the lean() command works here too
PARALLELISM = 5
StreamWorker(stream, PARALLELISM, processOneCompany, callback)
{% endhighlight %}

Now we can define a callback like `processOneCompany` which will handle the companies one-at-a-time, and StreamWorker will handle the annoying on finish hooks.

### Prevent Timeouts

One disastrous and silent error can occur when processing a very long streaming query: a timeout. This can happen at either the network level or the database level, so you need to guard against both.

First when connecting to the database specify the "keepAlive" parameter to prevent a network disconnect on a long-running query.

{% highlight coffeescript %}
mongooseInstance = new mongoose.Mongoose()
options = {
  user: config.DB_USER
  pass: config.DB_PASS
  server: {socketOptions: {keepAlive: 1}}
  replset: {socketOptions: {keepAlive: 1}}
}
mongooseInstance.connect(config.DB_URL, options)
{% endhighlight %}

Second and just as importantly, don’t forget to set the `timeout` parameter to false.

{% highlight coffeescript %}
User.find().setOptions({timeout: false})
{% endhighlight %}

Note that you may still run into configurations in MongoDB or your network that can cause similar disconnect issues, but this will solve most cases.

### Don’t Forget to Catch Errors

You must subscribe to the stream’s `on('error')` or you can experience silent failures as the error is caught and never returned to the callback.

## Use Schema Plugins to add Created and Updated Timestamps

Mongoose supports "plugins" which behave like a mixin or trait. This let's you easily define methods and attributes across all of your models. One of the most useful, in our experience, is automatically saving the created and last-modified times on all models, which makes debugging vastly simpler.

Here is is the plugin:

{% highlight coffeescript %}
# plugin for mtime/ctime
module.exports.timestamps = (schema, options) ->
  schema.add({ctime: {type: Date}})
  schema.add({mtime: {type: Date}})

  schema.pre('save', (next) ->
    @ctime = new Date if @isNew and not @ctime?
    @mtime = new Date
    next()
  )
{% endhighlight %}

Use it like so:

{% highlight coffeescript %}
plugins = require('plugins')
schema.plugin(plugins.timestamps)
{% endhighlight %}

## Conclusion

We hope you found these mongoose gotchyas and solutions useful. Both mongo and mongoose are evolving and hopefully it will become easier and easier for new users to employ best practices which avoid some of the pitfalls mentioned above. Please feel free to contact us if you have any others that you'd like to share!
