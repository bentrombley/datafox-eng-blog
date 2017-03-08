---
layout: post
title:  "10 Great JavaScript Utils You Should Stop Reinventing"
date:   2014-09-01
categories: nodejs
uuid: bfd880de-7ee4-4912-a144-266d2ab57666
---

I've wasted more time than I care to admit reinventing these wheels. These utilities are all small and well documented.  Just use them.


## 1.  [NumeralJS](http://numeraljs.com/)

Ever written code to format 31235892 as "31,235,892" or "$31.2m"?  This is a simple as:

    var numeral = require('numeral');
    numeral(31235892).format('$0.0a');  // $31.2m

[NumeralJS](http://numeraljs.com/) has support for localization and can parse strings like "32m" back into numbers.


## 2.  [MomentJS](http://momentjs.com/)

Formatting and manipulating dates creates confusing and bug-ridden code like:

    var yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);


[MomentJS](http://momentjs.com/) makes this all ridiculously easy:

    var yesterday = moment().subtract(1, 'days');

Or better still:

    var trialMessage = "Your trial expires in " + moment(expirationDate).fromNow();
    // "Your trial expires in 14 days"

MomentJS also has support for localization so you can delight your international customers like a pro.


## 3. [UnderscoreJS](http://underscorejs.org/)

Underscore is a great way to avoid boilerplate code when manipulating arrays and objects.  For example:

    // remove duplicate values
    values = _.uniq(values);

    // sort by all the user objects by the 'name' field
    users = _.sortBy(users, 'name');

    // remove null/empty values from the list
    names = _.compact(names);

    // get all of the values from an object
    var values = _.values(myObject);

Nothing complicated, just the sort of code you shouldn't bother reinventing.

## 4. [async](https://github.com/caolan/async)

Async simplifies many common node practices like calling functions in parallel or series and handling their errors.  For example we can simplify this indentation pyramid:

    User.findOne({name: 'Helen'}, function(err, user) {
      if (err) {
        callback(err);
      } else {
        Account.findOne({user_id: user._id}, function(err, account) {
        if (err) {
          callback(err);
        } else {
          makeAnApiCall(account, function(err, response) {
            if (err) {
              callback(err);
            } else {
              callback(null, response);
            }
          }
        }
        })
      }
    });

to a simple list `async.waterfall` call:

    async.waterfall([
      function(next) {
         User.findOne({name: 'Helen'}, next);
      },

      function(user, next) {
        Account.findOne({user_id: user._id}, next);
      },

      function(account, next) {
        makeAnApiCall(account, next);
      }
    ], callback);


## 5. [Commander](https://github.com/visionmedia/commander.js)

Add proper command-line arguments to your scripts in Node:

    Commander = require('commander');
    Commander
      .option('--user-id <id>', 'user id to retrieve')
      .parse(process.argv);

    userId = Commander.userId;

[Commander](https://github.com/visionmedia/commander.js) automatically fills in the --help:

    $ coffee my_script.coffee --help

    Usage: my_script.coffee [options]

    Options:

      -h, --help         output usage information
      --user-id <id>     user id to retrieve


## 6.  [Request](https://github.com/mikeal/request)

[Request](https://github.com/mikeal/request) is more than a nice-to-have, it's practically a requirement for making HTTP requests in Node.  The library lets you make simple GET requests easily:

    var request = require('request');
    request('http://www.google.com', function (error, response, body) {
      ...
    });

And it supports advanced options like multi-part POSTs, streaming, and more.

## 7.  [Helmet](https://github.com/evilpacket/helmet)

[Helmet](https://github.com/evilpacket/helmet) adds security best practices to your Express app painlessly, without requiring you to muck with headers and the various browser compatibility.  For example:

    var helmet = require('helmet');
    app = express();

    // use default security settings
    app.use(helmet());

    // or enable one at a time
    app.use(helmet.hsts());  // HTTP Strict Transport Security

To quickly check your site's security headers and settings, try the free [ExtensionRecx Security Analyser Chrome Extension](https://chrome.google.com/webstore/detail/recx-security-analyser/ljafjhbjenhgcgnikniijchkngljgjda).


## 8. [Stream Worker](https://github.com/goodeggs/stream-worker)

[Stream Worker](https://github.com/goodeggs/stream-worker) simplifies the routine task of processing a Node stream.  For example, iterating over a large results from MongoDB:

    // stream over all users
    var stream = Users.find().stream();
    var CONCURRENCY = 5;
    var processUser = function (user, callback) { ... };
    StreamWorker(stream, CONCURRENCY, processUser, callback);

StreamWorker handles the `pause()` and `resume()` in the stream as well as handling any errors, including thrown Exceptions.


## 9.  [Colors](https://github.com/Marak/colors.js)

<img src="/img/nyan-cat-console.png" style="width: 100%" />
<small>Running tests is always fun with nyan cat!</small>

Okay, you may not _need_ to add colors to your console output, but it's definitely fun.  [Colors](https://github.com/Marak/colors.js) saves you the hassle of dealing with ANSI color codes:

    var colors = require('colors');
    console.error("make this text red".red);


## 10.  [ShouldJS](https://github.com/shouldjs/should.js)

ShouldJS helps you add clear assert statements to your unit tests while eliminiating lots of boilerplate.  Some examples:

    functionToTest(function(err, user) {

      // easy static asserts
      should.not.exist(err);
      should.exist(user);

      // and easy object-oriented checks
      user.should.have.property('name', 'Expected Name');

      // assertions can easily be chained in a nice readable format
      user.age.should.be.greaterThan(18).and.lessThan(25);

      ...
    });

[ShouldJS](https://github.com/shouldjs/should.js) plugs smoothly into test frameworks like [mocha](http://visionmedia.github.io/mocha/) and you can choose whether to write simple asserts or fully English-like semantic assertions.









