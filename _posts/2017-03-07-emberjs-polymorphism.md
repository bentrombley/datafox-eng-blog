---
layout:       post
uuid:         0f8caf84-9204-421b-bdc1-1b6706091801
categories:   emberjs
tags:         null
title:        "Polymorphism in Ember Data"
date:         2017-03-07
author:       
  name:       null
  twitter:    null
  github:     null
feature_img:  null
sitemap:
  lastmod:    2017-03-07T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

One of the core powers of DataFox is our ability to show you a customized feed
of recent events affecting companies you care about.  For example, today I can
see that my former employer, Box, launched a new product, while my friend's
company, FireEye, just announced a new partnership with Thomson Reuters.

<img src="/img/events_feed.png" style="width: 100%" />
<small>My news feed on 3/7/17</small>

When it comes to displaying an event in our app we create a new Ember data model:

{% highlight javascript %}
var Event = DS.Model.extend({
  title: DS.attr('string'),
  date: DS.attr('date'),
  ... ??? ...
});
{% endhighlight %}

and my template would look like:

{% highlight handlebars %}{% raw %}
{{#each event in company.events}}
  <h1>{{event.title}}</h1>
  <p>Date: {{formatDate event.date}}</p>
  ...
{{/each}}
{% endraw %}{% endhighlight %}

But what if I want each event to have different display data and behavior (which they do)?
This is a classic case of polymorphism: we need a base `Event` class for the common behavior
and lots of subclasses like `ProductLaunchEvent` and `CompanyAcquisitionEvent` that define
the specific data and actions of that event type.  Ember doesn't explain how to handle this
common case, so what do we do?

Fortunately, a friend in the Ember core team pointed us to an undocumented `polymorphic` parameter
in Ember data that does exactly what we need:

{% highlight javascript %}
var Event = DS.Model.extend({
  title: DS.attr('string'),
  date: DS.attr('date'),
  object: DS.belongsTo('eventData', {polymorphic: true});
  ...
});
{% endhighlight %}


The corresponding JSON response looks like:

{% highlight json %}{% raw %}
{
  "events": [
    {
      "id": 12345,
      "date": "2014-10-26T06:01:28.700Z",
      "object": {
        "id": 456789,
        "type": "CompanyAcquisitionEvent"
      }
    },
    ...
  ],
  "company_acquisition_events": [
    {
      "id": 456789,
      "acquirer": "Acquirer Name"
    }
  ]
}
{% endraw %}{% endhighlight %}


Ember data automatically wires up the `id` and `type` fields so the `event.object` points directly to
the corresponding `CompanyAcquisition` model.  And we can update our generic event template to include
specific rendering for each event by adding a `template` variable to each model we define*:


{% highlight handlebars %}{% raw %}
{{#each event in company.events}}
  <h1>{{event.title}}</h1>
  <p>Date: {{formatDate event.date}}</p>
  {{partial object.template}}
{{/each}}
{% endraw %}{% endhighlight %}

<small>
\* Normally it would be incorrect to define a view (the template) in a model under MVC, but Ember's actually
follows MVVM, so it's a bit confusing.  In my judgment, this is the clearest solution.
</small>

