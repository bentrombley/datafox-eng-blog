---
layout:       post
uuid:         0F4503EB-D8E7-41DC-A5B1-A7EAD30A8F89
categories:   code-style
tags:         [code style]
title:        Names Matter
date:         2017-10-07
author:
  name:       Ben Trombley
  twitter:    bentrombley
feature_img:  null
sitemap:
  lastmod:    2017-10-07T18:00:00
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

> There are only two hard things in Computer Science: cache invalidation and naming things.<br>
> -- Phil Karlton


In a recent code review I kept asking the developer to pick clearer names for their variables and classes.  It felt nitpicky at first, but the more I thought about the more I felt it was a critical point.  Good names are the core of good programming.  Here’s why.

## Names Matter
Humans are very good at language.  We've been using it for 100,000 years.  We are so wired for language it takes years of training in meditation to teach your brain to stop talking.

By contrast, humans have been programming computers for a few decades.  Machines understand assembly commands like `move bx, 1`.  It takes years of practice for a developer to understand those commands, and even then it takes mental effort.

And that’s a key point: names matter because you write code for other developers, not the computer.  Even on a solo project, your future self needs to understand what you wrote.

If you need proof, take this lovely awk command:

    awk 'BEGIN{OFS=" | "} $1~/3433$/{
        split($1,a,".")
        split($2,b,".")
        print a[5], b[1]"."b[2]"."b[3]"."b[4]
      }'

It takes effort to parse this statement (and a visit to StackOverflow).  Over time it gets easier, but you pay a heavy cost every time you encounter that line.  If you're like me, you just avoid awk entirely; it’s not worth the productivity lost on anyone reading my code.

To paraphrase Bob Martin in [Clean Code](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?ie=UTF8&qid=1506884839&sr=8-1&keywords=clean+code): well-written code is easy to follow like good writing is easy to follow.  The names clearly map to their meaning.

Unclear writing is easy to spot.  It can even be hilarious like the (real) newspaper headline: “Miners Refuse to Work after Death.”  Unclear names in your code is just as ambiguous.


## How to Pick Good Names
The easiest way to understand good names is to start with what makes names bad.  Here is a collection of issues I’ve seen:

### Truly Bad Names
> Lisa:  "A rose by any other name would still smell as sweet."<br>
> Bart:  "Not if they were called 'Stink Blossoms'."<br>
> --The Simpsons


Let's start with the obviously poor choices

      // x only means the x-axis.
      x = 0

      // you wouldn't actually use "foo" in code, right?
      foo = 'bar'

      // what exactly is a helper and how do they help??
      function userHelper() { … }

      // good old Java, what is a "manager"?
      class AbstractFactoryManager


All of these examples are extremely ambiguous and have no meaning to the reader.  You might as well be writing assembly.


### Joke Names
```
double your_pleasure;
double your_fun;
```

```perl
sub liminal_messages {}
sub ordinate_flunky {}
sub par_perl_subroutine {}
```
([Source](http://www.perlmonks.org/?node_id=156816]))

Joke variable names are like those skits between rap songs.  They're funny once and then they're annoying.  Don't do it.

### Codenames
Codenames are fun for projects.  Calling your project “Street Fighter” or “Batman” is fun.  But don’t put it in code where it will love on long past the secret project.  “Batman” doesn’t mean anything to your reader, and you’ll be stuck explaining it to every new engineer.

### Acronyms
Common acronyms are fine.  But random abbreviations and acronyms can be extremely confusion.

I remember deep in the application framework at a previous job was a mysterious variable named `frob`.  No one could quite remember how it got there or what it did, but we had a vague sense that it was critical to authentication.  We treated it like a strange underworld creature to be avoided.

Naming a class `R2R0` or `TUD` or `RUNO` (all real acronyms I’ve used…) feels convenient when you’re typing, but you will pay the price with every new person that has to look up those confusing acronyms.

### Internal Jargon
When you use a term often enough you can forget that is not actually English.  At Box we shortened “collaborate” to “collab” so often people would regularly say things like “please collab that folder with me.”

But it gets worse.  At DataFox I thought it was clever to name parts of our code after species of foxes.  While fun, this means that every employee now has to learn that “alopex” refers to our news ingestion engine, while “otocyon” is our data matching tool.

Learn from my mistake and just name the tool after what it does.

### Names should match what you show users
Ideally, your code should line up with what you actually call your features.  That sounds obvious, but often the codebase diverges from the product as the features evolve.

In some cases, it is simply not worth the effort to change the name “watchlist” to “list” in the code.  That’s okay.  “Watchlist” is the technical term for a list in your code.

However, I’ve talked to engineers where the core code is still named after long-dead features.  This creates a lot of mental overhead for the engineers because they constantly need to translate from one mental model (the requirements) to another (the legacy code).

### "New" Names

I'm a big believer in renaming code as “old” or “deprecated” when refactoring to tell everyone that it should no longer be used.  That provides clear meaning to the reader, and eventually you will remove them (right?).

But I am _not_ a fan of labeling the new code as "new".  When I see `newGetUsers()` or a file named `new-user-template.hbs` I have no idea how new this code is and what it means.

Worse still, even when the old code is removed, you're left with your `NewUser` class.  Then comes the next refactor and the `NewNewUser` class.  I'm definitely guilty of this one, and created a `NewNewAction` class, breaking all the above rules at once!

### "And" Names
If you name something “createFileAndUpload()” you should feel good that you have picked a clear name.  And then you should refactor the method because it is clearly doing two things (it violates the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

### Missed Opportunities
One of the best pieces of advice for writers is to use fewer adjectives and replace them with interesting verbs.   "She walked slowly and painfully," is boring.  "She limped," is better.

Many variable names are like weak verbs, boring and meaningless.

    params = {rating: 'high', limit: 10, sort: 'rating'}

`params` means very little.  It's not bad, it just misses a chance to tell your reader what it actually means.  Calling it `selectMovieQuery` is clearer.  Yes, it's more typing, but that's okay.  Your code editor has auto-complete.  The seconds you spend typing are repaid by the reduced time your reader spends understanding the code.

### Tedious Naming Conventions

Names should make sense, period.  It’s okay to use technical names like `array` or common abbreviations like `obj` because they have clear meaning.  The thing to avoid are naming rules that add no meaning.

For example, at my first coding job we added a prefix to all class names to namespace them and used Hungarian notation… but our IDE already solved both problems.

Naming rules are an attempt to condense the complexities of language down to a set of rules.  This is overly simplistic.  It’s like trying to condense novel-writing into a set of rules.  The result is a very bad novel.


## So What Makes a Good Name?

As Phil Karlton noted, naming is hard.  Picking a clear name requires that you can clearly state what this code does in one word.  If your code doesn’t do one thing, this will prove impossible.

That’s a good thing.  Focusing on names requires that you focus on clear code.  Great writers agonize over the words they use.  Hold yourself to the same standard.

Clear names are obvious.  They have meaning.  Or rather, they have exactly one meaning.  If you have a class called `UnsafeString` it’s clear that you should escape it before displaying to the user.  If you have a file called `database_thread_pool.py`, you don’t have to ask what it does.  Perfect clarity should be your standard.

Just remember: names matter.  Language is ultimately a way of representing the world in your head.  Your brain maps names to things and even abstract concepts.  Don't make that mapping hard.

