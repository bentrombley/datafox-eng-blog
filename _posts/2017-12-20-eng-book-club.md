---
layout:       post
uuid:         D96E9914-5A02-407C-A652-22844C36AF67
categories:   engineering-culture
tags:         [engineering culture, constant-learning]
title:        Engineering Book Club
date:         2017-12-20
author:
  name:       Ben Trombley
  twitter:    bentrombley
sitemap:
  lastmod:    2017-12-20T12:00:00
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

I really enjoy writing about our [core values](https://blog.datafox.com/startup-culture-values/) at DataFox and how we’ve made them an active part of our culture.  Today, I want to talk about Constant Learning.

One of the greatest parts of being an engineer is the variety of the work.  Because the technology changes rapidly and the problems are always evolving, you get the chance to learn new things almost by default.  However, I still believe we can do more to learn as engineers.  It’s a shame that after college or grad school most engineers no longer dedicate time to reading engineering books.  And yes, I do mean, books.  Blogs, code tutorials and Stack Overflow are great, but they aren’t a substitute for the depth of books.  After all, would you trust a doctor that said they learned everything from WebMD?

That’s why we’ve started an engineering book club at DataFox to read and discuss technical works.  We started with a personal favorite of mine, [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) by Bob Martin.  I still find it to be the best explanation of how to write clear, modular code.  I also love his focus on simple rules and hands-on examples.  The lessons were extremely applicable--I still see people mention the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) in code reviews months later.

We next tackled [Solr in Action](https://www.amazon.com/Solr-Action-Trey-Grainger/dp/1617291021) by Trey Grainger and Timothy Potter.  Solr is one of our core technologies and increasingly important to understand as we scale.  It’s a dense book but we tackled it together and learned about the internal workings of the lucene index and ways to scale both read and write volume.  Just as importantly, we spread the knowledge from one expert to everyone on the team.

### Making the Book Club Work

In other book clubs I’ve joined, there is are a lot of good intentions and little follow through.  People genuinely want to be in a book club, but they struggle to find time to read the books.  Even though the club is voluntary, it still feels onerous and its members get little value out of the exercise.

To get around this trap, we tackle 1-2 chapters a week and pick a designated person to lead the discussion.  They read the section and create a quick Powerpoint outline to cover the major topics.  This helps anyone who missed the reading, and by the end we have a nice summary of the key points for new hires to digest.

We are also selective about the portions that matter. In _Clean Code_, for example, Martin is very focused on Java code, which can be tough to translate to our NodeJS micro-services.  Similarly, many portions of _Solr in Action_ were either too basic or too advanced to be relevant, so we omitted them.

My favorite point is the discussion.  We begin with simple prompts: do you agree with the author?  Would we actually use this?  Then we have a frank discussion involving junior and senior engineers alike.  I love when the book answers a question we've been struggling to resolve.  But I also love when we disagree with the author.  Even a legend like Bob Martin can be wrong, at least when it comes to our code and our team.  It is in these discussions where we define our engineering principles and culture.

### In Conclusion
Our next book is the tome [Code Complete 2](link), an intimidating volume full of best-practices from Microsoft.  No doubt, we won’t read all 900 pages, but I’m also confident that we’ll learn a lot and have great discussions.

If this sounds interesting to you, [consider applying](https://www.datafox.com/company/careers/).
