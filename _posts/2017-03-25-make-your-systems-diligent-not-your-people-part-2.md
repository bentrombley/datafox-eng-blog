---
layout:       post
uuid:         E2068F60-80A9-4716-B8CD-81B311DEA171
categories:   engineering-culture
tags:         [engineering, culture]
title:        "Make Your Systems Diligent, Not Your People, Part II: How To Build Diligent Systems"
date:         2017-03-25
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2017-03-25T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'


---

Good engineering organizations admit their mistakes and learn from them so they don’t happen again.  Great engineering organizations, however, improve without sacrificing agility.

As I described in my [previous post](/engineering-culture/2017/03/25/make-your-systems-diligent-not-your-people-part-1/), the temptation in any post-mortem discussion is ask engineers to be more diligent.  This is a mistake.  Great organizations make their systems diligent, and allow their engineers to be creative.

Here are your tools for making diligent systems.

## Linters
Linters are the highest-leverage tool in your engineering toolbox.  Any time you find yourself reminding an engineer not to do X or that our style guide is Y, add a lint rule that flags the issue in your code editor and again when you attempt to merge your code.

The clearest use of linters is for enforcing consistent style in your code.  A linter ensures that you will never again argue about tabs vs. spaces.  Every time a disagreement occurs, decide on the official style and codify it as a lint rule.

But linters can solve much deeper coding problems.  For example, in NodeJS you will get an error if two files mutually require each other.


{% highlight javascript %}
// In user.js
Account = require("account");

...

// In account.js
User = require("user");
// User = {}, due to a silent error in the Node require process.
{% endhighlight %}


At DataFox we use avoid this issue by using service locator pattern, and use a linter to remind everyone not to `require` such files directly.

The caveat is that your lint rules have to be correct.  Overly-assertive linters are as dangerous as airbags that go off prematurely, and as annoying as Clippy in Microsoft Word.  For this reason, linters should be added by the developers who they will affect, not their managers or a separate tools team.  This ensures that they will be updated as the code changes.

## Unit Tests are For More than Catching Bugs
Hopefully, you already believe in automated tests for catching bugs.  However, they can also be great ways to diligently check rules that are too difficult to implement in a linter.

For example, we have two files in different projects that store a set of constants that need to stay in sync.  Rather than adopt a complex module system to share code, we wrote a simple integration test that asserts that two files have matching content.

Similarly, we have mocha tests that inspect our data models to assert best practices.  For example, we have a test that fails if you add a denormalized field to a model, so you don’t forget to update the denormalized field.  The test simply checks against a whitelist, but it prevents anyone from forgetting and corrupting data.

I encourage you to think beyond the traditional definitions of a “unit test” or “integration test” to just ask, how can I write automatically check all new code?  Write that test, and don’t worry what it’s called.

## Architect Your Code So The Right Way is the Easier Way
To paraphrase Joel Spolsky’s excellent article, [Making Wrong Code Look Wrong](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/), the mark of a senior engineer is the ability to not only recognize wrong code, but to architect systems so that such code is obviously wrong to others.

For example, he suggests a clear naming structure to differentiate unescaped strings which could cause security vulnerabilities.  Using these dangerous strings in output looks wrong, and is easier to spot.

I suggest taking this example one step further and making the correct behavior the default.  In many templating languages, inputs are escaped by default, so you have to explicitly use unescaped data like this in Handlebars:

{% highlight handlebars %}{% raw %}
  {{{hopefullyNotUserInput}}}
{% endraw %}{% endhighlight %}

This code looks wrong because it uses the {% raw %}`{{{`{% endraw %} notation, which can be flagged by linters and in code review.

Another example of diligent architecture at DataFox is our permissioning system to guard sensitive customer data.  The key is that database access is centralized in a single place where enforce our access permissions.  This is further enforced with lint rules that flags any code that circumvents this convention.

In my experience, engineers follow the path of least resistance; if your frameworks make it so the right way is the easy way, you don’t have to remind anyone to do things the right way.

Of course, this won’t prevent a determined developer from circumventing the checks, but that’s not the point.  Any such egregious acts would be caught in code review, and frankly should be caught in the interview process.

## Checklists
Not all automation has to be done with code.  In fact one of your most powerful tools is a simple checklist.

In his book, [The Checklist Manifesto](https://www.amazon.com/Checklist-Manifesto-How-Things-Right/dp/0312430000) Atul Gawande shows that adhering to simple checklists during surgical operations saved far more lives than advanced drugs or cutting-edge tools.  The reason is that checklists both catch mistakes and give subordinates a way to overcome the social stigma of correcting their superiors.  Gawande describes how before the checklists, nurses would rarely correct physicians, even when they saw an error.  Adopting checklists removed this social stigma, to the patients’ great benefit.

Don’t underestimate the power of checklists to ensure that procedures are followed.  For example, you can create runbooks for your on-call team to diagnose and triage issues.  Ultimately, this means that even your newest team members can take on-call and catch mistakes they would otherwise hesitate to report.

To be clear: a thorough and reliable run-book won’t happen overnight.  The key is that every time a new issue emerges and the veterans are called in, you write up the procedures in the runbook.

## Create Alerts, not Dashboards
Everyone loves dashboards.  Giant screens full of data make great visuals in movies and impress your CEO.  It’s easy to think that you wouldn’t have problems if there were simply enough TVs on the wall displaying critical graphs.

This is not what dashboards are for.  Dashboards are for diving into issues, not monitoring the health of the system.  Instead of relying on people to watch screens, invest in automated alerts.

At DataFox, we use [Monit](https://mmonit.com/monit) for low-level alerts like high system CPU usage and memory.  [Pingdom](http://www.pingdom.com) monitors our site uptime and we use [Scalyr](https://www.scalyr.com) to alert on errors and incorrect behavior in our logs.

To highlight two examples, we have alerts on any table scans in our database and connection errors to redis, because both have caused outages before and were difficult to diagnose.  Of course, when one of these alarms goes off, the on-call engineer consults the relevant dashboards for database health and slow queries.  But this is different than expecting the on-call person to constantly monitor the dashboards and notice any anomalies.  Let computers do that for you.

## Responsible Post-Mortems
As I mentioned in the [previous post](/engineering-culture/2017/03/25/make-your-systems-diligent-not-your-people-part-1/), post-mortems are critical to any great company.  After something goes wrong, a good post-mortem dives past superficial problems to uncover the root causes of the issue -- they ask 5 levels of why?

The key to a great post-mortem, however, is in what it recommends to prevent future problems.  It’s easy to become so focused on the problems at hand that you lose the broader context of the business.  Every solution should be balanced against its cost.

For example, it's easy to conclude in a post-mortems that you should ban self-approved code reviews.  After all why should you be self-approving your code or logging onto that machine?  But it's not that simple.  Self-approved code reviews might be fine after pair-programming or when merging pull requests or pushing an emergency fix.  You can still choose to ban self-approved code reviews, but it should be made with a full understanding of the cost to developer productivity.

Too many post-mortems are based around the assumption that _something_ has to be done.  The result are requirements like 100% test coverage or manager sign-off that do little to prevent issues and much to wreck the engineering culture.  Over time this conservative bias erodes engineering productivity and drives away your best engineers -- the #1 reason I see great engineers leave large companies is that they are tired of dealing with the hassle and eventually join a team where they can move quickly again.

They may not say it in their exit interviews, but irresponsible post-mortems are driving away your best talent.


## Code Review
Code review is an incredibly powerful tool to prevent issues by teaching engineers, not by catching bugs.

Inevitably, after a bad bug is deployed, someone asks why it wasn’t caught in code review. This question is based on two wrong assumptions.

First, you’re trying to solve a human error by having more humans look at it.  It’s the wrong tool for the job.

Second, this sacrifices the power of code review as a teaching instrument.  After graduation, code review is your ongoing education.  It’s the one place you get feedback from your peers and seniors.  More importantly, it’s feedback on real-world examples that you care about, not just academic exercises.

As a code reviewer it’s great when you catch a mistake, but it’s very low leverage, like an editor editor catching a typo.  The real benefit is when you can share context like “this code is about to be deprecated,” or “Sam is also touching this code for her feature, so please coordinate with her.”  Great code reviewers will tie in best practices and explain how to refactor code to be much clearer.  Done well, code review is the way engineers learn to become great at their job.

Openness like this requires trust.  If you start treating code review as a way to catch bugs you risk destroying that trust.  For the author, code review feels like running the gauntlet.  If it’s easier to simply push all code into one large code review, I’ll do that.  If it’s easier to pick lenient reviewers, I’ll do that.  At the end of the day I’m measured on the code I’ve deployed, not the code I’ve had reviewed, so I’ll game the system.

For the reviewers, they know that any mistake they miss will be held against them.  My incentive is to err on the side of caution, even if that means blocking others.  Or perhaps I try to just avoid being assigned altogether.  At most companies I’m measured by the features I build, so spending time doing thorough code reviews only hurts me.

This is a tragic way to corrupt one of the most powerful tools in your arsenal for training great engineers.


## Building Diligent Systems
Engineering is hard.  Your team has to balance many concerns.  You will make mistakes.  You can’t change that fact.

What you can change is how your team responds to those mistakes.  The temptation will be to call out the people who made the mistakes.  But the cost will be a culture that slowly becomes more bureaucratic, and eventually drives away your best, most innovative engineers.

The alternative is to build diligence into your systems.  Whether you adopt linters, unit testing, or some combination of the suggestions above, the important point is that you solve issues in your systems, not your people.