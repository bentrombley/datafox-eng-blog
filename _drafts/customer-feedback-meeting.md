---
layout: post
title:  "Our Customer Feedback Meeting"
date:   2015-03-27
categories: customer feedback
---

Startups love to talk about being “customer focused” and “delighting users”, but too often this talk has little to do with what engineers do in their day-to-day work.

We operate differently.

## The Way It Usually Works

Most companies create a role like “product manager” that is in charge of digesting feedback into product specifications.

This is deeply flawed.

To the person in charge of distilling feedback into requirements the job is overwhelming.  They are asked to “advocate for the customer” so engineers build what people actually need.  At the same time they’re asked to respect technical trade-offs and explain to the sales team what we can and cannot build.

The only way this works is if everyone has the utmost respect for this person.

In reality, the engineers feel like the decisions are arbitrary and not really grounded on real feedback.  “Why aren’t we running A/B tests?” they ask.  Sales meanwhile fumes that they can’t close deals when we aren’t building what they ask.

The system is almost designed to antagonize the teams with the unfortunate PM left in the middle to sort it all out.

## The Way It Works at DataFox
Last summer when we received frequent requests to allow searching for companies by mutually-exclusive sectors like “agriculture” or “automotive”.  It was clear we needed to build a solution.

However, the engineers knew from experience that we couldn’t build a classifier because sectors are too subjective.  Take Google: is it a search company or an advertising company?  Is “search” even a sector or is that too specific?  What about all that other stuff like Google Docs and Nest and self-driving cars?

In most organizations, these questions would be confined to engineering as a technical challenge.  Either they find a way to build to spec or the project never happens.

But things go differently in the customer feedback meeting.  With everyone in the room engineers could ask why users were requesting this search and why our current keyword-based searching wasn’t sufficient.  That week when a customer requested sector-based search, our sales team probed deeper on those questions.  What we discovered was that the real concern was that users worried that searching by a keyword like “mobile security” they might miss important companies because they didn’t know to search for similar terms like like “mobile device management”.

This insight was key because similar keywords is a problem we had already solved as part of our similar company algorithm.  The solution we built instead was to suggest keywords as you searched so you could build a comprehensive list of all mobile security companies.  This was a nuanced solution that never would have occurred without engineering and customer-facing roles being in the same room.

## How to Make Customer Feedback Work
An effective customer feedback loop doesn't happen by accident.  We follow these steps:

### Step 1: Record Everything Your Customers Say
Our CEO, Bastiaan, is obsessive about recording everything.  Every meeting, phone call, and thought goes into Evernote so he can find it later.  Even before we started DataFox he started a document with all customer interviews and findings, which at last count is over 300 pages long.

His example has spread to the rest of the team, leading to a culture of note taking and sharing.  Any time someone interacts with a customer or potential customer, they take notes and copy them into a Google Doc.  Feedback isn’t left to a PM, and we make it clear to our sales team that gathering feedback is part of how they’re evaluated.

We include everything whether good, bad, random or even self-contradictory.  The goal is to just collect all of the unfiltered data so we can avoid bias.


### Step 2: Invite _Everyone_ to the Meeting
We believe in avoiding meetings and bureaucracy, but we make an exception for one meeting each week: customer feedback.  Everyone from sales to engineers to data analysts is invited -- _and all of them come_ --
because it is key to doing their job well.

For engineers the meeting is a chance to question salespeople about the details and context of feature requests, bugs, etc.  This in turn gives salespeople insight into what matters for engineers, and encourages better notes in the future. The result is a positive feedback loop where teams feel connected.

### Step 3: Keep the Meeting Short and Focused
12 people attend the meeting now, and this will continue to grow.  But it's still effective because we follow rigid rules:

  3.  Only allow clarifying questions are allowed -- no discussions about solutions!
  2.  The person who recorded the feedback reads it and provides any context.
  2.  Stay brief: you have 60 seconds.
  5.  Assign roles:
    - one person runs the meeting and keeps it moving
    - one person tallies requests
    - one person files bugs

### Step 4: Tally Feedback
We use a simple Google spreadsheet and add a check every time a request comes in.  The system is rough but effective.  It quickly becomes clear that some requests happen at 10x the volume of others.

The key here is that the entire team decides how to translate the user’s request into a vote for a new feature or bug.  This can be very subjective, but that’s okay as long as it’s transparent.  We aren’t relying on a PM or the CEO to filter the feedback so the team trusts the numbers.

### Step 5: Constantly Review the Tallies
When every person on the team knows the top 10 requests, suddenly planning and prioritization is a breeze.  There are no heated discussions or questions about why we’re building X (and why we're not building Y).  Everyone can see the list and has heard customer after customer request the feature.

Of course, not all feedback is equal, and we still decide which features are worth prioritizing and which should be deferred.  However, the customer feedback meeting is the perfect time to explain why we are making this choice.  It's also the perfect time for the team to challenge that decision using the data.  We've often changed our minds after seeing that a “niche” request is actually one of the most common.


## Why It’s Worth the Effort
Being “customer-focused” has become startup gosepl, but it's still important to explain why it’s worth the time and effort to run this meeting.

First and foremost, the meeting keeps the entire team aligned. Debates don’t feel like arbitrary opinions when everyone has seen the data and knows the counts.

For us as engineers, the meeting connects what we're building to real customer needs.  It’s not about debugging an annoying bug in MongoDB, it’s about the customer who is unable to upload a large spreadsheet.

For our sales and support teams on the front lines, the meeting shows them that the engineers and analysts care about their experience and are fixing problems.  They learn which questions to ask customers so engineers can directly address their concerns.

For our customers, having everyone see their feedback and understand their needs means they get the best solutions.

The overall result is a tight feedback cycle where we can iterate faster and delight our customers.  And that is what "customer focused" means to us.








