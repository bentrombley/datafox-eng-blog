---
layout:       post
uuid:         d97a4b3b-3607-4002-86ca-727ce96477e0
categories:   Planning
tags:         null
title:        "Quarterly Planning"
date:         2017-01-26
author:       
  name:       Cyrus Stoller
  twitter:    cyrusstoller
  github:     cyrusstoller
feature_img:  null
sitemap:
  lastmod:    2017-01-26T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

Balancing the need to be responsive to immediate customer feedback with the need for longer-term strategic product development is hard. While we're still iterating on this process, we wanted to share how we approach quarterly planning. Our goal is to incorporate ideas that will help our engineering, data, customer success, sales, and marketing marketing teams meet their objectives.

To give everyone a chance to participate without stopping company operations, we start our quarterly planning process a month before the end of the quarter.

## Why Quarterly Planning?

Sometimes people ask why we've opted to do quarterly planning instead of sprints (a la agile development). For us, it's about the type of engineering culture we want. We want to give engineers autonomy and the space to think creatively. Often because of the need for user stories to be specific, engineers become so focused on meeting their requirements that they err on the side of easily estimatable tasks instead of ones that might be more difficult / ambitious but better for the long-term health of the company.

We try to minimize the number of meetings. If we worked in two week sprints, we'd be spending a much larger percentage of our time planning instead of working. As a team we feel we get good enough visibility into each other's work from our quick standup meeting before lunch. Afterall, we're still a small enough team that all of the engineers sit in the same room. Likely, we'll have to revisit this in the future, but it's working well for now.

## Phases

- **Phase 0 (pre-work):** Survey our customers to help us catalog their top requests, both in terms of frequency and severity. Reflect on closed lost deals. And, look over objectives that we've tabled in previous quarters. I'll be writing another blog post about this process soon. Stay tuned.
- **Phase 1:** Brainstorming sessions.
- **Phase 2:** Uncovering themes.
- **Phase 3:** Getting feedback on the themes that are under consideration.
- **Phase 4:** Choosing metrics and goals for the themes.
- **Phase 5:** Presenting the plan to the company.

At the end of each section I'll share the status of one of our recent ideas - adding lists of contacts and lists of signals to DataFox.

## Brainstorming

As a product manager, I setup 1.5 hour brainstorming sessions of 6 people each (excluding me) that include a mix of people from different parts of the company. Schedules permitting, I try to assign everyone to a session. The goal is for everyone to have a better understanding of what it would take to make our offering more valuable to our customers and to inspire ideas for how we can get better. When possible I try to make sure that people are *not* in the same sessions as their managers, so everyone feels free to speak plainly.

Since we're trying to optimize the number of ideas generated, I ask everyone to do their best to leave objections at the door; we can figure out what's realistic later. To make it so that no one gets too attached to any one idea, I ask that at the beginning of our brainstorming sessions people write down at least 15 ideas on post-it notes in 10 minutes. No one should be worried about filtering. It's hard to come up with 15 ideas. Towards the end, some of the ideas may feel silly, but they help to inspire ideas that are highly valuable.

To make sure that we all stay focused listening to each other, I ask that we refrain from using technology during the sessions. At first, this was hard, since people wanted to look through our application to think of ideas. But, we're most interested in the suggestions that are top of mind, not the ones that people can find when they go through the product with a fine tooth comb. We've found that the things that are the most painful are pretty easy to remember.

Everyone should feel like they have a forum where they're being explicitly asked for their ideas. That's not to say that they will necessarily get implemented, but they will have a opportunity to present ideas that are more than incremental improvements at least once per quarter.

We try as much as possible to disassociate ideas from the person who came up with it, so that we're not concerned with who will get credit (even subliminally) during this phase. When it's time to evaluate which ideas are feasible, we rely heavily on everyone's particular expertise. For example, engineers may weigh in on how difficult it will be to implement. But we wait to have those discussion later, so that we can get all of the ideas on the table, even if we decide not to pursue them.

In many ways the types of ideas that people bring up are as important as the ideas themselves because they highlight the areas where we should focus in the upcoming quarter. Before I ask people to share the ideas that they came up with, I make sure that everyone has more post-it notes ready to write down ideas that they think of as they hear other people's ideas. This free association is often where we find some of our best ideas and where we find which ideas resonate the best.

*Status:* At this point in the process, people had shared that they wanted to be able to make lists of leads and that they wanted help figuring out how to make better use of DataFox signal data.

## Uncover themes

As a facilitator, I try my best to cluster ideas into themes that we can explore more deeply as a group; often the themes are pretty clear. Once we've had a chance to discuss them, I ask everyone to share some more specific ideas about each theme. While the brainstorming sessions go by quickly, they're just the beginning of our month long discussion.

After each session I record all of the ideas and make them accessible to the whole team, so people can see the ideas that came up in other sessions too. I put them into a Github repo, so that we can easily refer back to them in subsequent quarterly planning sessions. In the process of cataloging all of the ideas, I create a short-list of the ones that seem like they're both the highest priority and most feasible. Since I'm cataloging each of them, I feel relatively confident that I've given each of them a fair shake. It's helpful to put the post-it notes on the wall to figure out how they all fit together.

Once I have the short-list of ideas that I think we should consider pursuing, I put them up in a public place, so I can get feedback from the rest of the team. I'm eager to see whether they agree with my assessment about which ideas to table. Last time around, we had moved from roughly 1100 ideas down to 150 ideas clustered into 11 themes - still too many for our team to execute on in a quarter.

*Status:* At this point in the process, it seemed like the functionality to solve these problems could be similar. So, we started to explore the idea of having lists for every primitive in DataFox.

## Feedback on themes

By putting the themes up in a public place, I can stop people for a couple minutes when they're taking a break and ask them what they think about them. These quick one-on-one interactions are helpful in figuring out which things seem like an economical use of our finite manpower. Ideally, we'll have a good mix of short-term priorities, that directly address customer feedback, with longer-term initiatives. Often our customers astutely point out symptoms of complicated issues. But instead of treating each symptom, we try to make the correct diagnosis so we can cure the underlying problem.

At this stage everyone should feel free to suggest new themes or to help refine the themes that we're considering. I want people to be able to provide feedback however they feel most comfortable -  in person, over email, over slack, with gold star stickers, or via post-it notes. While feedback coming from engineers about feasibility matters much more at this point, we want everyone to feel comfortable giving direct and critical / constructive feedback.

In addition to filtering which themes we want to commit to, we also need to prioritize which ideas within each theme matter the most. So, while everything is still fluid, I ask for help coming up with an initial hypothesis that we can validate once we've decided on our metrics for each theme.

*Status:* At this point in the process, I had started to socialize the idea that we should start to generalize our list functionality to more than just companies. While we may want to make this fully generalizable the future, we opted to limit the scope to just contacts and signals to make it tractable within a single quarter.

## Choosing metrics

When we've finally whittled down our ideas down to a few themes, I look at the ideas and try to figure out how we can measure success. By choosing a metric that we can all agree on, it's easy to know whether the features we ship are pushing us in the right direction. If the metrics aren't moving, then it may be time for us to try something else. Metrics keep us accountable. Otherwise, it's easy to congratulate ourselves for putting in effort even if we fall short of our goals.

After I propose some metrics to the team, I ask for feedback. In particular, I want to know whether the people responsible for each of them think they're realistic and whether there could be unintended consequences. Once we've agreed on metrics that feel well aligned with the ideas we're going to execute against, we need to figure out our targets. Ideally our targets feel slightly uncomfortable, but within reach. In other words, we should hit 80% of our targets.

At the end of our quarterly planning process, we've put together suggestions for how to move our metrics, but at the end of the day, engineers choose the specifics.

To make it easy to track our metrics, I put together a dashboard that tracks our metrics in [Chartio](https://chartio.com/). A screenshot of this dashboard is included in our weekly internal product emails and presented at our weekly customer feedback meeting.

*Status:* At this point in the process, we decided that the number of lists users had would be a good proxy for engagement and hence usefulness.

## Presenting the plan

At the beginning of the quarter, I put together a presentation summarizing the themes that we're committing to. I share why they're important to the growth of the company and the metrics that we've agreed to. And at the end, I put together a list of ideas that we've decided to revisit in the following quarter.

By being deliberate about what we want to accomplish, everyone should know how their contributions are moving the company forward. While we do our best to come up with the best plan we can, we recognize the need to be flexible throughout the quarter. We try to strike the right balance between specific features and broad product direction, so we can be responsive to customer requests.

## Conclusion

We're still iterating on this process. But so far it's helped us stay focused when we receive tangential feedback that's important but unrelated to our current goals. It's also helped us give our sales and marketing teams a better idea of when they can expect new features to be rolled out.

If this sounds like the kind of place you'd like to work, we're hiring.
