---
layout:       post
uuid:         E2068F60-80B9-4716-B8CD-8123E1DEA171
categories:   engineering-culture
tags:         [culture, startup, engineering, DataFox]
title:        "Engineering Culture at DataFox"
date:         2017-04-30
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2017-04-30T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---


DataFox has the best engineering culture I’ve ever worked in.  I see it every time the team makes the tough decision to pass on a brilliant engineer with great technical skills because they would not make the team better.  I see it in every detailed code review left by a senior engineer that has a million priorities.  And I see it when our newest engineers stay up late researching a new technology so they can teach their teammates at a lunch and learn session.

I’d like to take credit, but it’s the team that defines it and lives it every day.  As a cofounder my job is simply to document our culture and share it with every new hire so we can build our culture as we grow.

This is what I tell them:

## You are not “just an engineer” at DataFox

DataFox is a not a place for engineers that want to be told what to do.  There are no detailed product requirement docs or code architects to tell you what to build.  Yes, you will work closely with customers and designers and product managers, but ultimately you decide what you build.

We hire great engineers who are interested in more than writing code.  They want to understand the business and talk to customers first-hand.  We expect to know how the sales month is going and what the next marketing launch is.  You didn’t join DataFox to be just an engineer.


## You own entire problems, not implementations

One of our [company values](https://medium.com/notes-from-the-startup-learning-curve/startup-culture-values-34f9ad7000a5) is ownership, and no one embraces this idea more than engineers.  Our goal is to give you problems and let you figure out the solutions, whether that means a frontend tweak or full-scale infrastructure change.

That is why all engineers can work full-stack.  You own the entire solution end-to-end, whether that means writing frontend CSS or low-level linux administration.  Of course, most people gravitate to frontend- or backend-dominated challenges, and we respect that you might not love to make pixel-perfect UIs.  The important point is that you can fully own each task you take on.

## Everyone does devops, everyone goes on-call
Deploying and maintaining live code is not someone else’s job.  You write better code when you’re the one who has to wake up to fix it.

“DevOps” can be a loaded phrase, but at DataFox it means that you engineers do the work of configuring AWS and debugging issues with linux, redis, etc.  We don’t expect you to know all these things coming in, but we do expect you to be willing to learn.


## Diversity is a real priority on our team

Half the engineering team are women and other underrepresented minorities.  That didn’t happen by chance.  Our interviewing process is not perfect, but we are constantly striving to make it fairer and more inclusive.

We’ve retired the term “culture fit” when we interview because we aren’t looking for people exactly like us that fit our culture.  We are looking for “culture adds” that make us a better place to work.


## We value collaboration over efficiency

Our hiring bar is extremely high for technical skill, but we’ve said no to candidates that do not work well on teams.  At DataFox there is no “headphones rule” where you can’t interrupt other engineers while they have their headphones on and are working.  You should be respectful, but it’s more important that you can ask for help or feedback on an idea than that you go a whole day without being interrupted.

This environment isn’t for everyone, but it’s not meant to be.  We look for people that thrive in a collaborative setting.


## Done is better than perfect
We are a startup.  Our biggest risk is not imperfect code, it’s failing to make customers happy.   We take to heart Mark Zuckerberg’s maxim to “move fast and break things.”

I’ve jokingly dubbed this approach “Disaster-Driven Development”: we don’t invest time to fix problems we don’t have yet.  And when something does go wrong, we do a post-mortem and learn from it.  It’s okay to make mistakes, as long as we don’t repeat them.  That’s how we can move fast and remain agile as we grow.


## You will never be punished for fixing bad code
All companies have tech debt.  Every codebase has its unlit alleys and broken-down streets where engineers fear to tread.  There is no documentation and less testing.  The code looks like a dumping ground, so engineers treat it like one.

At most companies the incentives are clear: you will not be rewarded for fixing this code, but you will be punished if something breaks (as it always does when dealing with fragile, untested code).  The result is that engineers are afraid to touch this code, and this fear leads to worse hacks.

I tell every new engineer that they have cover to break things when they fix legacy code.  Of course, this is not an excuse to recklessly refactor without testing, but we hire good engineers and trust their judgment.


## Be enthusiastic about what you build
Humility is one of our company values, but humility doesn’t mean you can’t be proud of what you’ve built.  Too often I see engineers demo an amazing feature only to focus on its minor flaws.

That doesn’t fly here.  When you demo something, sell it:

  - Your audience includes our sales team whose job is to convince customers to spend tens of thousands of dollars on our product.  Don’t undermine their confidence by denigrating your work.

  - As engineers we see all the bugs.  There’s a tendency to focus on what is wrong and how we could improve.  This is valuable, but left unchecked it devolves into a culture of cynicism that problems always exist and progress is impossible.

  - When in doubt, it’s always okay to brag about other engineers’ work and explain why it is impressive.


## We believe in automation over diligence

We do not nag people about forgetting things.  I believe this so strongly I wrote an entire [blog post about it](http://eng.datafox.com/engineering-culture/2017/03/25/make-your-systems-diligent-not-your-people-part-1/).  In brief: we solve problems by automating them through linters, checklists, testing, etc. not by expecting engineers to act like accountants.


## Tools and processes exist to help you, not to block you

We use linters, tests and code review to help catch errors and give you feedback.  But we build all systems with the ability to override the automatic checks because we trust your judgment.  Of course, it’s a problem if someone abuses these overrides, but we don’t hire reckless engineers, and we can address abuse when it actually occurs.

Tools are powerful, but they have to be built around trust.  For that reason tools are built and updated by the engineers that use them, not by a tools specialist.  You know there is a problem when people ask questions like, “But why would you want to do that?” which show they are too far away from the actual work.


## Data wins arguments, but not all arguments are worth it
If we have a question we can’t answer, we add the analytics and charting so we can answer it next time.  Being a data-driven organization requires much more than a love of analytics; the entire team has to be vigilant about tracking and exposing data you really trust.  It’s a lot of engineering and analytics work to get to meaningful metrics.

That said, not all decisions are worth it.  At our size we can’t A/B test every decision.  That’s okay.  Smaller decisions can be based on anecdotal evidence, as long as we’re honest about it.


## Pick the right tool for the job, just don’t be a hipster

It’s more important that a team has the freedom to pick the tools that work best than that we use a consistent tech stack across the entire organization.  One of the great benefits of being small is that we are free to adopt new open source tools, without the “not invented here” attitude of a big company.

That said, you can’t adopt a tool because it’s cool on Hacker News.  We adopt new technology because it solves problems, not because it gives us street cred.

## Clean code trumps clever code

Understandable code is more important than optimized or clever code.  In fact we actively discourage clever code because it is fun to write but difficult to read, and we spend most of our time reading code, not writing it.

One of my favorite books is Clean Code by Bob Martin, and we recently read it as a team and discussed how it applies to our codebase.  My #1 takeaway is this:

Modularity is the only thing that matters in our code because it is the key to scaling both the product and the engineering team.  Modular code has clear interfaces that make it easy to change implementation, even going so far as to rewrite into another language or a distributed set of microservices.




## Conclusion

When I look at the team and company we’ve created, I’m most proud of the evidence of our culture in action.  For example, tomorrow one of our junior engineer goes on his first on-call rotation only three months into the job.  He has never done devops before but is eager to learn and knows his team is there to support him.

Another great example occurred last week when the sales team was practically buzzing with excitement after an engineer demoed her improvements to our Salesforce integration.  Afterwords our head of sales told me how awesome it is as a salesperson to know you have a great engineering team behind you.

Lastly, as I write this paragraph, I see a Slack alert from our newest hire announcing a new button she added after watching a customer struggle with a tedious workflow.  It was a perfect example of being more than "just an engineer."

If this sounds like your kind of place, consider [joining us](http://www.datafox.com/company/careers/).
