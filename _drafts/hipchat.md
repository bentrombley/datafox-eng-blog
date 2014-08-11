---
layout: post
title:  "Automate Everything Using Hipchat"
date:   2014-08-09
categories: hipchat
---



We're huge fans of [HipChat](https://www.hipchat.com/) here at DataFox and it has rapidly become our central means of communication.

We started by using it as a simple instant messenger for 1:1 conversations, and then expanded to using the chat rooms as a way to send out customer feedback without clogging our inboxes.  And it has become the way to broadcast announcements like `@all order lunch from [doordash](http://doordash.com)`.

But that's just the beginning.  HipChat is also _the_ simplest way to communicate with our automated tools and processes.  No more e-mails or dashboards spread across 10 different systems.  By consolidating our messaging through HipChat we get all of our alerts in one place and can instantly discuss everything that is happening.


### Deployment and Configuration

#### Continuous Integration

[Jenkins](http://jenkins-ci.org/) has become the standard way to run continuous integration and deployment, and it defaults to e-mailing you when builds or tests fail.  But if you're like me, you've already set up e-mail filters to ignore this noise.

Fortunately, it takes less than 5 minutes to install and configure the [HipChat plugin](https://wiki.jenkins-ci.org/display/JENKINS/HipChat+Plugin) to broadcast when builds fail.  Now everyone sees a growl notification on failure (configurable, of course) and you can discuss the issue right there with your team.  No more e-mail threads!

#### Deployment
We use [Ansible](http://docs.ansible.com/index.html) for all of our deployment and configuration, and highly recommend it.  Ansible is designed to simple and powerful, with almost no setup.  For example, Ansible comes with a hipchat module that makes it trivial to broadcast when you've deployed code to a server:

    ---
    # ... steps to deploy your code ...

    - name: "Broadcast deployment in hipchat"
      hipchat: token={{ hipchat_api_key }} room={{ hipchat_engineering_room_id }} msg="{{ inventory_hostname }} server deployed."

And in your `vars.yml` file you define these two variables (inventory_hostname is the server name, like "db.datafox.co")

    ---
    hipchat_api_key: ...
    hipchat_engineering_room_id: ...


This shows up as an alert like:
(screenshot of deployment notifications, blur out actual server name?)

Similarly, it's trivial to broadcast changes in configuration.  Have you ever wasted 4 hours debugging an issue only to learn that someone has had just updated a conf file?  With a few lines in ansible you can create a log of all changes to your production servers.


### Monitoring

[Monit](http://mmonit.com/monit/) is a very lightweight and powerful tool for monitoring your servers and services.  For example, we can alert if a server is running out of storage, by creating a file at `/etc/monit/conf.d/diskspace`:

    check filesystem rootfs with path /
    if space usage > 80% then alert

similarly, it's easy to monitor our various backend services and remote connections.  By default monit will send alert e-mails, but it's far more useful to have the messages sent to hipchat where they can trigger a growl notification and show up in context of any recent deployments or other changes.

To make this work we just wrote a quick-and-dirty python script to send hipchat messages:

    import sys, urllib, urllib2, json

    values = {
      'room_id'        : 12345,
      'message'        : 'message to send',
      'color'          : 'red',
      'from'           : 'server.datafox.co',
      'notify'         : 0,
      'message_format' : 'text'
    }

    # yes, this should use optparse...
    last_command = ''
    for i, arg in enumerate(sys.argv):
      if i % 2 == 1:
        last_command = arg
      elif i != 0:
        if last_command == '-m':
          values['message'] = arg
        elif last_command == '-f':
          values['from'] = arg
        elif last_command == '-r':
          room_id = arg
        elif last_command == '-c':
          values['color'] = arg
        elif last_command == '-n':
          values['notify'] = arg
        elif last_command == '-fm':
          values['format'] = arg
        else:
          print 'Unrecognized argument: ' + last_command
          sys.exit()

    url = 'https://api.hipchat.com/v1/rooms/message?auth_token= ...'
    req = urllib2.Request(url, urllib.urlencode(values))
    urllib2.urlopen(req)


And then invoke it from monit like:

    # if service is unavailable 3 times in a row, send an alert that we are restarting
    if failed host localhost port 12345 protocol http for 3 cycles then restart
    then exec "/var/hipchat/hipchat_cli.py -m 'some-server process failed on server.datafox.co -- restarting.' -f 'Monit' -n 1"


### Bugs and Uncaught Exceptions

We use [Bugsnag](https://bugsnag.com) to report uncaught errors in our frontend JavaScript, as well as in our backend NodeJS and Python services.  I haven't used any of the competing products, but Bugsnag was extremely easy to setup and automatically groups all errors,  so we are alerted the 1st, 10th, 100th, etc. time an error occurs.

Connecting Bugsnag to HipChat takes less than 60 seconds, and now we can immediately see if the errors started occurring after a deployment.


### And More...

With widespread adoption it seems like most cloud-based services support HipChat.  We particularly rely on NewRelic alerts, but also use Github.  The only concern is drowning out the useful signal with noise (e.g. github commits).

<img src="/img/hipchat-screenshot.png" style="width: 100%" />
<small><b>Pictured Above:</b> HipChat showing a git push to our Master branch, followed by an automated Jenkins build and Ansible deployment. Unfortunately, this deployment resulted in a bug which was subsequently caught by bugsnag and displayed prominently in HipChat.</small>
