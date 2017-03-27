---
layout: post
title:  "Make Your Systems Diligent, Not Your People"
date:   2017-03-25
categories: engineering-culture
uuid: 97CF6990-2C36-4537-8EF4-30A3D7FE5EA0

---

Something terrible has happened.  Someone has made a big mistake and customers are very, very upset.

What you do at this moment defines your company culture.  Great companies admit their mistakes and see them as chances to learn.  That’s the easy part.  The tricky part is how you build your culture to learn from its mistakes without adopting the bureaucratic fixes that bring innovation to a halt.  Striking this balance is difficult, and no one seems to talk about it.

So let’s talk about it.

## It All Starts with a Post-Mortem

Great engineering organizations have adopted the medical practice of “post-mortems.”  After a disaster you assess what happened and unearth the root causes of the issue.  The goal is to learn, not to assign blame.  (In medicine the results of a post-mortem are legally protected from being evidence in malpractice suits.)  You announce what happened and assess the underlying causes.  You explain the steps your organization will take to prevent this from happening again.

This all sounds great, but here’s what happens in practice: the report inevitably states that the mistake could have been caught in code review or by automated tests.  The engineer forgot about the latest security standards.  Or the latest guidelines for internationalization.  Or to the requirements of the mobile app.  Or a thousand other things.  The post-mortem concludes that engineers should remember these requirements going forward, and proscribes steps to enforce this diligence.


## The Temptation to Focus on Diligence
The temptation in every post-mortem is to list everything engineers forgot to do and remind them to do it.  After all, if they had remembered to test that integration or to escape that user input, the issue would not have happened, right?

Well, not really.

It’s always tempting to blame issues on a lack of diligence, because superficially this is true.  But engineers are people, and people are human.  Yelling at them feels good, but it is ineffective at preventing future mistakes.

Guilt is a powerful weapon, be careful how you use it.

## Engineers are not Accountants
Accountants are expected to triple-check their work because accounting errors are catastrophic.  Diligence is part of the job definition.

Engineers solve novel problems and are capable of breaking complex issues into simple pieces.  Creative engineers build innovative technology that changes the world.  Creative accounts go to jail for fraud.

Engineers aren’t accountants.  If you’re not careful, you can end up defining “engineering” to really mean “accounting” and soon your team will reflect that definition.  Look at any large company’s lackluster product and you’ll see the direct result of this process.


## Make Your Systems Diligent, Not Your People
Engineers are human.  That’s why you hire them.  Humans are creative and can understand other people’s wants and needs.  Ultimately, an engineer is a human that can translate those wants and needs into computer commands.

Computers, by constrast, are very, very diligent.

Trying to make engineers diligent is to focus on their weakness as humans.  I propose the opposite: focus on their strength as problem-solvers who can break apart problems into clear, unambiguous rules.

In other words, ask your engineers to make your systems diligent.

Building diligent systems means:

- Rather than expecting more and more from your code reviewers, add a lint rule that will catch the issue 100% of the time.

- Rather than reminding your coders to remember to properly escape user input, architect your code so that escaping is the default behavior.

- Rather than nagging your engineers to not forget to test the API and mobile app when they change the web application, add tests that run automatically.

- Rather than guilting people to remember every step, create a clear checklist before deploying new code that ensures you follow the same procedure every time.

If all of that sounds obvious, then I agree.  But few organizations follow these simple principles when designing their tools and processes.  Most post-mortems are filled with remediation steps that boil down to this: don’t forget about X.  And over time this process robs engineering of its creativity and freedom.

Eventually most organizations end up hiring accountants, not engineers.

It doesn’t have to be this way.  In [Part II](/engineering-culture/2017/03/25/make-your-systems-diligent-not-your-people-part-2/) I’ll discuss the tools you can use to build diligent systems, while preserving an agile environment where you learn from your mistakes and get better over time.

