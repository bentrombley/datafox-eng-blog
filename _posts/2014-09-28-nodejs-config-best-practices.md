---
layout:       post
uuid:         497214c7-8a26-471c-968b-c1f1f8bb4295
categories:   nodejs
tags:         [nodejs, APIs]
title:        "NodeJs Best Practices: Environment-Specific Configuration"
date:         2014-09-28
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2014-09-28T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

When you find yourself writing code like:

    if (NODE_ENV === 'production') {
      stripeApiKey = 'prod-abc-123';
    } else {
      stripeApiKey = 'dev-def-456';
    }

it's a bad sign because you're probably duplicating this if statement every time you need this api key, violating
the DRY principle.  And you're also making your code less clear because you've just stuck some environment logic
in the middle of payment code, violating the single responsibility principle.  And what happens when you now
want to add a staging environment?

It's time to refactor your configuration code to be separate from your logic.

There are plenty of solutions, like environment variables and .conf files, but these are probably overkill unless
you have a large ops team. A simpler solution is to add a `conf` directory like this:

    conf/
      index.js
      development.js
      staging.js
      production.js

    src/
    lib/
    ...

Your conf files are simply JS objects.  For example, `development.js`:

    module.exports = {

      # api keys and secrets
      STRIPE_API_KEY: 'dev-def-456',
      ...

      # control flags
      logLevel: 'debug'
      ...
    };

This works because when you `require` a directory, Node will automatically load the `index.js`
file, which then requires the appropriate config for the environment:

    switch (process.env.NODE_ENV) {
      case 'development':
        module.exports = require('./development');
        break;
      case 'staging':
        module.exports = require('./staging');
        break;
      case 'production':
        module.exports = require('./production');
        break;
      default:
        console.error("Unrecognized NODE_ENV: " + process.env.NODE_ENV);
        process.exit(1);
    }

Now conf values are all in place (DRY) and your code is much cleaner:

    Conf = require('../conf');
    stripeApiKey = Conf.STRIPE_API_KEY;

This also makes it easy to support new environments like testing and staging.  For us staging
should look exactly like production, except it uses a different database.  Our staging conf "inherits" from production:

    config = require('./production');

    # override specific values
    config.DB_URL = "mongodb://staging-db.datafox.co";

    module.exports = config;

You can do the same to create dev-testing and staging-testing setups without making your code messy.
