---
layout:       post
uuid:         3bcfe28c-e0ad-493d-bf74-bed878845088
categories:   emberjs
tags:         [javascript, emberjs]
title:        Implementing Query Preloading on Ember 2.2
date:         2018-01-02
author:
  name:       Hunter Fox
  twitter:    hefoxed
sitemap:
  lastmod:    2018-01-02T12:00:00
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

DataFox has been using Ember since the company's beginnings over 4 years ago. When I joined about 6 months ago, we were on 1.13. With my help, our main user-facing application is on 2.2 now! We hope to get to LTS release and Ember CLI in the near future so that we can start using more modern Ember tools and plugins.

About me: I learnt Ember on the job.  I had prior experience in Angular, React, jQuery and similar frameworks.

While we are slowly moving towards modern Ember, we still need to make our Ember 2.2 app work for our needs. I'm sure there's many developers and companies out there in the same situation.

### The Task
<img src="/assets/posts/img/2018-01-02-preloading.png" width="100%" />

At DataFox, our goal is to help our users find their best-fit accounts. At times this involves very complex queries that span large data sets and can take awhile. For example, this screenshot is the INC 5000 list of fastest growing private businesses. There's a listing of companies and then additional tabs containing insights into the results, people at the companies, etc.

My task was to add preloading/prefetching functionality so that when viewing one tab, the associated tabs were already fetched. Keep in mind that all of these pages are search pages, so when the companies tab is filtered, the people and insight pages will update and be ready to view. To add to the complexity, the other tabs (such as “Contact Info”) may have their own filters applied that are preserved when switching between tabs.

### The Prior State

As is the legacy Ember way, each of the List tabs is its own Route and therefore has its own Controller. All the search code was spread out in a combination of Controllers and Mixins that were mixed into those Controllers. That code was mostly similar (from copy-paste) but had diverged as the project grew to have differences that made it hard to maintain -- for example, the some templates would check a `hasSearched` property and others checked if “not isSearching.”


On top of that, the Controllers were using a lot of Observers to trigger the search queries to be sent to the server whenever there was a corresponding user action.

My goal was to remove the observers and make the code consistent again, while generally making it simpler to maintain and understand.
The Solution

Although there were a few different ways to implement pre-loading tabs, there were a few factors that helped me narrow down the options:
- It needed to be easily usable by both Controllers and Components, so I knew I wanted a shareable Service
- My ultimate desire is that developers can easily grab the search results and use them wherever they needed to display them

My final solution was to add computed properties on my new Service that would be triggered whenever we needed to run a new search query. This was typically user-triggered events such as changing the filters or other properties that distinguished the Route. In our example of viewing a List of companies, the computed properties would watch the List id.

In order to to have ajax inside of computed property, I define an Ember Object that I can then update after retrieving the ajax results. For example:

{% highlight javascript %}
App.FilteredSearchService = Ember.Service.extend({

searchResultsCompany: computed('companyFilters.[]', 'listId', function() {
   const searchResultsCompany = Ember.Object({isSearching: true, rows: []});
   this.fetchCompanyResults(get(this, 'companyFilters')).then((data) => {
     setProperties(searchResultsCompany, {
       rows: data,
       isSearching: false,
     );
   // Always wrap the error handler so the console trace includes it.
   }).catch((error) => this.handleError(error));
   return searchResultsCompany;
}),
{% endhighlight %}

Now our Controllers and Components can simply alias the results:

{% highlight javascript %}
App.SearchCompanyListDataController = Ember.Controller.extend( {
  filteredSearchService: inject.service('filteredSearch')
  searchResultsCompany: computed.alias('filteredSearchService.searchResultsCompany'),
{% endhighlight %}

Then use then in the template:

{% highlight handlebars %}{% raw %}
  {{#if searchResultsCompany.isSearching}}
    [loading indicator]
  {{else}}
    {{#if searchResultsCompany.length}}
       [results]
    {{else}}
      Nothing results :'(
    {{/if}}
  {{/if}}
{% endraw %}{% endhighlight %}

The “magic” of this is the search is not done until the computed property is used. To achieve the preload effect, after the ajax request for company I just add a `get(this, 'searchResultsPeople')` after the ajax returns and bam, it fetches and caches the people results also while viewing the company results.

One of the key reasons this works is that once a computed property is computed on Service/Controller, it is kept even if nothing is using it anymore.

### The Problems

I ended up hitting some unexpected complications along the way -- and learnt a lot about computed properties.

#### Ghost Properties on Controllers
One of the major snags I discovered was also the most confusing. On the “Contact Info” controller, we had the following computed property:

{% highlight javascript %}
'searchResultsPeople': computed('companyFilters.[]', 'companyPeopleFilters.[]', function() {
  …
  // make query to People endpoint on our server to retrieve the search result
});
{% endhighlight %}

This means that if either `companyFilters` or `peopleFilters` is updated, `searchResultsPeople` should be recomputed.

Switching back to “Data” tab with all the companies meant that no template or JS should have been using the `searchResultsPeople` property anymore and therefore I assumed it wouldn't be recomputed. However, changing `companyFilters` was actually triggering `searchResultsPeople` to be recomputed, even if it wasn't used in the UI anymore! But why?

I eventually figured out the problem was the computed function property on the controller. Even though these properties weren't being used anymore, they remained “alive” because controllers in Ember are singletons (and singletons never die).. However, `computed.alias('..')` was totally fine and didn't have this same problem.

The solution was to move more functionality into components (which do clean up after themselves) and keep to `computed.alias` properties on the controller. In general, having more code in your components instead of the controllers is a good Ember practice.

#### Route Changes and Services

At various points, I reset the filters in the Route `setupController` function -- for example, when switching between different Lists. However, this was sometimes causing an unexpected re-compute of properties that (I thought) weren't been used anymore.

Until the Route completely changes, computed properties on Services in the last route (both aliased in controllers and components) will still re-compute if their dependent properties change. I ended switching up the filters / resetting to work around this. For instance, I had been using `companyFilters` for both Company Lists and Company Search, so instead I created a new `listCompanyFilters`, `listPeopleFilters`, etc. that I reset between viewing different lists (on the shared route between the different tabs) and kept `companyFilters`, `companyPeopleFilters`, etc. for Company Search.

{% highlight javascript %}
App.companyListRoute = App.AuthRoute.extend({
  ...
  setupController(controller, model) {
    this._super(controller, model);
    const listId = get(model, 'id');
    const previousListId = get(this, 'filterStateService.listId');
    if (!previousListId || previousListId !== listId) {
      // resetFilters will reset all filters that start with list, including companies, contacts, etc.
      get(this, 'filterStateService').resetFilters('list');
      set(this, 'filterStateService.listId', listId');
    }
  },
});
{% endhighlight %}

### Conclusion

So far, it looks to be working well, and my team has really enjoyed the ease of use and maintenance compared to our prior implementation. It also has made our site seem a lot faster, despite the complex queries that we're running. I'm eager to hear how others would implement this logic or if there is a better solution out there in modern-Ember.
