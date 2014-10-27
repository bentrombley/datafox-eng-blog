---
layout: post
title:  "Polymorphism in Ember Data"
date:   2014-10-26
categories: emberjs
---

One of the core powers of DataFox is our ability to show you a customized feed
of recent events affecting companies you care about.  For example, this morning
I can see that a company in our space, [Lytics](https://datafox.co/lytics), just
raised $7m in funding so I should check them out.  And my friends who work at
[Apartment List](https://datafox.co/apartmentlist) probably should be aware that
[Redfin](https://datafox.co/redfin) just bought [Walk Score](https://datafox.co/walkscore).

<img src="/img/events-feed-10-26-14.png" style="width: 100%" />
<small>My events feed on 10/26/14</small>

When it comes to displaying an event in our app we create a new Ember data model:

    var Event = DS.Model.extend({
      title: DS.attr('string'),
      date: DS.attr('date'),
      ... ??? ...
    });

and my template would look like:

    {{#each company.events}}
      <h1>{{title}}</h1>
      <p>Date: {{formatDate date}}</p>
      ...
    {{/each}}

But what if I want each event to have different display data and behavior (which they do)?
This is a classic case of polymorphism: we need a base `Event` class for the common behavior
and lots of sublcasses like `CompanyFundingEvent` and `CompanyAcquisitionEvent` that define
the specific data and actions of that event type.  Ember doesn't explain how to handle this
common case, so what do we do?

Fortunately, a friend in the Ember core team pointed us to an undocumented `polymorphic` parameter
in Ember data that does exactly what we need:

    var Event = DS.Model.extend({
      title: DS.attr('string'),
      date: DS.attr('date'),
      object: DS.belongsTo('eventData', {polymorphic: true});
      ...
    });

The corresponding JSON response looks like:

    {
      events: [
        {
          id: 12345,
          date: "2014-10-26T06:01:28.700Z",
          object: {
            id: 456789,
            type: CompanyAcquisitionEvent
          }
        },
        ...
      ]

      company_acquisition_events: [
        {
          id: 456789
          acquirer: ...
          ...
        }
      ]
    }

Ember data automatically wires up the `id` and `type` fields so the `event.object` points directly to
the corresponding `CompanyAcquisition` model.  And we can update our generic event template to include
specific rendering for each event by adding a `template` variable to each model we define*:


    {{#each company.events}}
      <h1>{{title}}</h1>
      <p>Date: {{formatDate date}}</p>
      {{partial object.template}}
    {{/each}}

<small>
{% raw %}*{% endraw %} normally it would be incorrect to define a view (the template) in a model under MVC, but Ember's actually
follows MVVM, so it's a bit confusing.  In my judgment, this is the clearest solution.
</small>

