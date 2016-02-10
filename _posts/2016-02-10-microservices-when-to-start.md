---
layout: post
title: Microservices - When to Start
---
Recently I read [Sam Newman](https://twitter.com/samnewman)'s excellent book ["Building Microservices"](http://www.amazon.com/Building-Microservices-Sam-Newman/dp/1491950358/ref=sr_1_1).  I highly recommend it for anyone working in or considering microservice architectures - in other words, anyone who works on back-end software systems (if you aren't doing microservices now, you should at least be aware of the approach and considering them).

One interesting revelation from the book had to do with this idea:  When should I start designing and implementing microservices in my system?  More specifically, should I just start out this way from the beginning?

I was a bit surprised to hear the author assert that starting out with a monolith is his recommended approach.  His explanation is sound:  At the beginning, it is difficult to conceptualize exactly what the system really is.  A key element of a microservice architecture is that you group together the things that change together.  It can be difficult at the beginning of a project to envision which parts of the entire application are going to change together.  By starting with a monolith first, you gain the experience with the system you need to recognize where the logical service boundaries actually exist in this application context.

That seemed right to me.  So I was then a bit surprised later to see this article entitled ["Don't Start With a Monolith When Your Goal is a Microservices Architecture"](http://martinfowler.com/articles/dont-start-monolith.html) by [Stefan Tilkov](https://twitter.com/stilkov).  His assertion seemed to be the complete opposite of Newman's.

In fact I tweeted about it:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Real interesting take on monolith vs microservices: <a href="https://t.co/UHwlmrpBPH">https://t.co/UHwlmrpBPH</a> - an opposing view to <a href="https://twitter.com/samnewman">@samnewman</a> in “Building Microservices”.</p>&mdash; Matt Ryan (@mattvryan) <a href="https://twitter.com/mattvryan/status/697173953421320192">February 9, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

After a bit more contemplation I'm not sure these two are in such a stark disagreement as I initally thought.  Here's how I'd evaluate the situation, based on what I read from both authors:

* If you are writing a new system from scratch, you should *probably* just implement a well-designed monolith first.  You should keep in mind that someday you may have to split this monolith apart, so make sure you prioritize good design principles in your monolith (like logical separation of concerns and bounded contexts).  You probably don't have enough information at this point to know if you are actually going to need a microservices architecture.
* If you are writing a new system from scratch **AND** you already know that it will require a microservices architecture, you should build it that way from the beginning.
* If you are rewriting an existing system, you have a pretty compelling case at this point to go for microservices from the outset.

It all depends on the cost tradeoff.  Designing your system as a microservices architecture will require more time and thought to get it right, more iterations and starts and stops.  You shouldn't make that investment if you aren't sure users will even like your system, or if there will be high enough utilization to justify it.  However, if you are sure you will need it, all that effort up front to go for microservices in the beginning will still be faster than writing a monolith that you have to split apart later - especially if there are users on the monolith when you're trying to break it apart.  So if you know that's what you need why waste time writing the wrong thing?

When it comes to the rewrite, presumably by now you understand the system domain well enough that you can be reasonably effective designing the rewrite with a microservices architecture.  The previous argument mostly goes out the window - the additional expense of designing a microservice architecture is diminished via the increased knowledge you have of what the system really should look like, which tips the balance in the tradeoff toward microservices.

Really, it's no different than anything else we do in software development.  Frequently, some variant of this question is asked:  Should I do this the "Better"" but more expensive way now, or should I do this the "Acceptable" way now and do it the "Better" way later?"  The nuanced but correct answer is captured in these questions:

1. How certain are you that you will need it the "Better"" way later?
2. How much more expensive is the "Better" way as compared to the "Acceptable" way?
3. How much harder will it be to switch to the "Better" way in the future as compared to doing it now?

In the case of microservice architectures, uncertainty about the need of the microservice architecture now means the following answers:

1. You aren't certain.
2. It's quite a bit more expensive (probably).
3. It will be harder.

Fuzzy to be sure, but the assertion (a reasonable one, I think) is that the uncertainty in #1 and #2 offsets the additional difficulty expected in #3.

But if you *know* you need a microservice architecture, the answer to #1 changes.  Now with the uncertainty gone, the balance tips the other direction, and you should just pay your dues now.
