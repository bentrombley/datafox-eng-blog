---
layout: post
title:  "10 Killer JavaScript Utilities You Should Stop Reinventing"
date:   2014-09-01
categories: nodejs
---

I've wasted more time than I care to admit reinventing this kind of functionality, and not doing as good a job.  These utilities all do one things and do it well with solid documentation.  Just use them.


## 1.  [NumeralJS](http://numeraljs.com/)

Ever written code to format 31235892 as "31,235,892" or "$31.2m"?  This is a simple as:

    var numeral = require('numeral');
    numeral(31235892).format('$0.00a');  // $31.2m

And [NumeralJS](http://numeraljs.com/) has support for internationalization if you need it.


## 2.  [MomentJS](http://momentjs.com/)

Formatting and manipulating dates/times makes often creates confusing and bug-ridden code.   Replace this kind of grossness with:

    // ugh
    var yesterday = new Date();
    yesterday.setDate(yesterday.getDate() - 1);

    // way clearer
    var yesterday = moment().subtract(1, 'days');

Look how easy it is:

    var trialMessage = "Your trial expires in " + moment(expirationDate).fromNow();

    // prints "Your trial expires in 14 days" or "Your trial expires in 2 months" as appropriate


## 3. [UnderscoreJS](http://underscorejs.org/)

Underscore is a great way to avoid boilerplate code when manipulating arrays and objects.  Some examples:

    // remove duplicate values (NOTE: this is not an efficient implementation)
    values = _.uniq(values);

    // sort by all the user objects by the 'name' field
    users = _.sortBy(users, 'name');

    // remove null/empty values from the list
    names = _.compact(names);

    // get all of the values from an object
    var values = _.values(myObject);

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

If you write scripts in NodeJS it's trivial to add command-line arguments:

    Commander = require('commander');
    Commander
      .option('--company-id <id>', 'company id to retrieve')
      .parse(process.argv);

    companyId = Commander.companyId;

Plus you automatically get help:

    $ coffee my_script.coffee --help

    Usage: my_script.coffee [options]

    Options:

      -h, --help         output usage information
      --company-id <id>  company id to retrieve


## 6.  [Request](https://github.com/mikeal/request)

[Request](https://github.com/mikeal/request) is more than a nice-to-have, it's practically a requirement for making HTTP requests in Node.  The library is extensively documented and handles the more arcane aspects like following redirects.









