---
layout:       post
uuid:         bf5cb3a1-93fa-407b-ac76-98eac0635fc0
categories:   mongodb
tags:         null
title:        "Protect Your Database with .mongorc"
date:         2014-10-05
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2014-10-05T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

Your database is the most import piece of your infrastructure and also your most vulnerable.  When it's down, everything is down.  Anything you can do to protect against errors or mistakes is worth the effort.

Here is one simple step, courtesy of [MongoDB: the Definitive Guide](http://shop.oreilly.com/product/0636920001096.do), that protects you from accidentally typing a dangerous command from the mongo command line: use your `~/.mongorc.js` file to disable dangerous operations.  For example add this:

    // disable dropDatabase
    db.dropDatabase = DB.prototype.dropDatabase = function() {
      print("dropDatabase is disabled on this environment");
    };

So you can't accidentally drop your database while connecting from the command-line:

    some-db> db.dropDatabase()
    dropDatabase is disabled on this environment

You can also disable other potentially dangerous commands like `dropCollection` or `dropIndex`.

If you have multiple users you can apply this setting globally by editing the `/etc/mongorc.js` file instead.  At DataFox we use [ansible](http://www.ansible.com/), so we have a simple task to apply this protection to all servers:

    - name: Update /etc/mongorc.js
    template:
      src: mongorc.js
      dest: /etc/mongorc.js
      mode: 0644
    sudo: yes

Of course, this is not a replacement for best-practices like [user roles that respect the "principle of least privilege."](http://docs.mongodb.org/manual/core/security-introduction/#role-based-access-control) or backing up your system (I highly recommend [MMS](https://mms.mongodb.com/)), but it can still save you from a very costly mistake.
