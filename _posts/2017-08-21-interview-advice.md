---
layout:       post
uuid:         05fd8841-a823-4d2d-9c57-6b404ade1d39
categories:   interviewing
tags:         [interviewing, skill]
title:        "Engineering Interviewing Advice I Wish I Had Heard"
date:         2017-08-21
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2017-08-21T06:11:00
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

I recently had the chance to mentor a student at [Hackbright](https://hackbrightacademy.com/), a dev bootcamp dedicated to helping women change careers into software engineering.  As the session came to a close the focus turned to interviewing, so she asked me for advice.

Hackbright does a great job teaching their students how to complete an interview.  They teach them to repeat the question and talking through their answer.  It’s great advice, and I want to add my own lesson from 8 years sitting on the other side of the table.

## Interviewing is a Skill

Here’s the sad truth: most engineering interviews are very limited in their ability to evaluate engineers.  Seriously, judging engineers based on an hour of writing code on a whiteboard is about as fair as judging a chef by asking them to write recipes on a whiteboard.  Worse, interviewers often aren’t trained and recieve no feedback on their interviewing.

Too many people don’t consider interviewing to be a skill, so they don’t invest the time and effort to get better at it.

Interviewers are also biased, though not in the way you might expect.  Of course, there are very real biases against women and minorities, but the one that is rarely discussed is bias based on circumstance.  A [recent study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3084045/) found that Israeli judges are far harsher in their sentencing right before they take a break and are hungry.  The same applies in an interview: if your interviewer is having a bad day they are more likely to grade harshly.  Or if their previous three interviews were poor, they are inclined to be more lenient.

Just remember that not getting an offer does not reflect on your skill as an engineer...  and be aware of this bias when you’re the interviewer.


## Keep It Simple
The single biggest problem I see in interviews is when a candidate overly-complicates their solution.  Start simple and then add the complexity or optimization.

Too often I see candidates jump to the optimal solution only to get stuck in the details of their approach.  I also see them start to refactor before they have actually written a solution.  The result is that they end up with a mess of ideas and incomplete code.

If you start simple and improve you get partial credit for each step.

## Write Small and Leave Space

Whiteboard coding has nothing to do with writing in any code editor, but it’s also a fact of life.  Understand that interviewers associate code that simply looks messy with messy thinking.

My recommendation is to write small and leave lots of space.  For example:

```
def reverse_list(list):
  # … space for new code ...

  start_of_my_code()


```

Leave space above the `for` loop because you will inevitably need to add code.  If your language has curly braces, don’t add the closing brace until you are actually done with the function, or you’ll just need to erase it as your code changes.

## Use Comments to Pseudo-Code Your Solution First

Hackbright recommends writing pseudo-code if possible, and I agree because it guarantees you won’t be judged on syntax mistakes.  But if you have to write code, use quick comments to outline your thinking first:

```
def reverse_list(list):

  # create a new list

  # iterate over original list in reverse and assign to new list

  # return new list

```

This gives your interviewer a chance to see how you are thinking and correct you earlier if you are wrong.

What if you were doing something complex like building a shopping cart?

```
def add_to_shopping_cart(item):

  # load shopping cart from database

  # check item is not already in cart and add if not

  # save cart to database if changed

  # return saved cart

```
In this case your comments make it clear that you have distinct steps, which you should break into separate functions.  So do that.

## Break Your Answer into Smaller Functions

In real code you should write short functions that do one thing.  The same should apply to interviews (see “When in Doubt, Ask the Interviewer What They Care About”).  To continue the last example, if you have clear comments, then go ahead turn them into function calls and explain that you’ll implement them later:

```
def add_to_shopping_cart(item):

  # load shopping cart from database
  cart = load_shopping_cart()

  # check item is not already in cart and add if not
  cart = add_item_to_cart_if_not_already_present(cart, item)

  # save cart to database if changed
  cart = save_cart(cart)

  # return saved cart
  return cart
```

This helps keep your answer very clear and allows you to tackle the problem one step at a time as you implement each function you’ve just defined.  This is how coding should always work.

Of course, this might not be the most optimized solution to some answers, but that’s usually okay if you acknowledge this trade-off.

## Explicitly State Your Trade-offs

As you start with a simple solution, explicitly acknowledge that you making a trade-off by  favoring simplicity over speed (or memory or browser compatibility, etc.).  This tells your interviewer that you understand those concerns, and gives them a chance to tell you what they care about.

For example: I’m going to use an in-memory storage for now, but in real life I assume we would use a real database.
When in Doubt, Ask the Interviewer What They Care about
Some interviewers are concerned only about clever algorithms.  Others care that you understand concurrency.  Still others care about clean code.  Good interviewers will tell you what they are evaluating, but most interviewers will not.

So just ask.

“I’m going to start with the simplest algorithm, is that what you are looking for?”  Or “Is it okay to assume my code is only running on a single machine and I don’t need to worry about threading?”

Let’s say the question is to reverse an array.    Give the simplest solution that works: I’d just call `array.reverse()`.

If they then ask you to implement that function yourself, make the next simplest solution by creating a new array, i.e.:

```
newArray = [];
for i in range(1, len(array)):
  newArray.append(array[len(array) - i])
```

If the interviewer then asks for something faster then dive into the fully-optimized solution that reverses the array in-place.  There are lots of clever ways to write this code, but it’s also very easy to make mistakes and get it wrong:

```
array_len = len(array)
for i in range(0, array_len / 2):
  temp = array[i]
  array[i] = array[array_len - 1 - i]
  array[array_len - 1 - i] = temp
```

Don’t assume that the optimized version is the “best” one.  Remember, in the real world calling `array.reverse()` is almost certainly the best solution.


## Don’t Apologize

I’ve had candidates say things like, “Sorry, I don’t usually code on a whiteboard,” and “I’m not very good at algorithms.”  This tells me as the interviewer that you don’t believe in yourself -- and makes them wonder why they should I believe in you.

It’s totally okay to be nervous and make mistakes.  Interviewers expect this.  Everyone has been a victim of the [imposter syndrome](https://en.wikipedia.org/wiki/Impostor_syndrome).  You can’t stop those feelings, but you can choose to ignore them.  Project confidence, even if you don’t feel it.

## Do Your Homework

One quick way to stand out from other candidates is to research the company and its products.  If you know your interview panel ahead of time, you can look them up on LinkedIn.  If the company is small you should at least know the founders’ backgrounds and the major investors.

It’s also great to read through the company’s github, engineering blog, and job postings to see what technology they use and if they contribute to open-source.  [Builtwith](https://builtwith.com/)’s free tool can usually tell you some interesting things about their technology, like if they use AWS or React.

Well-researched question show that you care.  Asking “How has MongoDB performed as you’ve grown?” is a way better question than “What is your tech stack?”  And “Why did Google Ventures invest in your Series B?” is way better than “Have you raised money?”

## Ask Hard Questions

Remember, interviewing is a two-way street.  Hiring engineers is extremely difficult and they should have to impress you.  Come in with a written list of questions and skip the usual ones like “Tell me about a typical day,” or “What’s the best and worst part about working here?”  Dive right into meaningful questions like:

  - Who are your biggest competitors?  What differentiates your product?
  - Do you measure your [Net Promoter Score](https://en.wikipedia.org/wiki/Net_Promoter) (NPS)?  Is it getting better?  How does this shape your product roadmap?
  - Do you track your test coverage?  Why or why not?
  - Tell me about your last post-mortem.  How did you learn from your mistakes?  How did you communicate the changes to the rest of the company and customers?
  - What specific things does your company do to promote diversity?

There are no right or wrong answers here, but you’ll immediately learn a lot about the company, its culture, and its engineering philosophy.  Press for details.  If they say, we’re really fun, then ask about the last few company events.  Are they frequent?  Are they during work hours?  Do they all revolve around drinking?

Asking hard but fair questions projects confidence and switches the script to make the interviewer sell you on the position (a trick I learned from [sitting with the sales team](http://blog.datafox.com/the-day-i-sat-with-sales/)).

## Believe in Yourself
Interviewing is stressful.  We spend most of our lives trying to avoid situations like this where we will be judged.  But it’s also unavoidable.  Over time it does get easier, I promise.

Don’t forget that it’s extremely hard to hire engineers, so the person on the other side of the table should be just as worried as you are.

It won’t be long before you get an offer and accept it, and then you are the interviewer on the other side of the table.  Try to remember what it was like as interviewee and be transparent about what you’re testing.  Ask for feedback.  Get better.  Make it a core skill.
