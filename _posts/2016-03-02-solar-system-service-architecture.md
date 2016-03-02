---
layout: post
title: "Solar System" Service Architecture
---
Some time ago I wrote about [when to start microservicing]({% post_url microservices-when-to-start %}) and my friend Evan replied this way on Twitter:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">I suspect that moving from monolith to microservice arch. often leads to a solar system arch. Lots of microservices orbiting a monolith.</p>&mdash; Evan Farrer (@evanfarrer) <a href="https://twitter.com/evanfarrer/status/697530613763428352">February 10, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

Evan is a man of many outstanding skills, sarcasm among the foremost.  So when he said this I wasn't sure if he was making fun of me, or just making a joke.  However, the more I've considered this I think it has a lot of merit and we should probably talk about it more than we do.

Evan is almost certainly right that often when teams begin with a monolith (as most should) and then begin trying to move to a microservice architecture, their migration path will lead them through a phase resembling a solar system - some number of microservice-y things kicking around the periphery of a larger monolithic thing in the center.  These microservices probably come about because:

* After having created the monolith, the team recognizes that eventually it would be wise to move to a more microservice-style architecture, so when something comes along that seems a good candidate for being a microservice, the team decides to try it out.
* After having created the monolith, the team decides to try splitting things off from the monolith as microservices as a part of the move to a full microservices architecture, but they are doing this incrementally instead of all at once.
* Some combination of the above.

In other words, this solar system style architecture is probably being viewed by the team as the temporary representation of their system while it is in a transition from the monolith they had before to the beautiful and flawlessly-designed microservice system they have envisioned.  And because it is viewed as "temporary", perhaps they also view it as "not correct" or otherwise suboptimal.

I don't think that is necessarily the case.  A solar system architecture can certainly represent the best reasonable state for your system, and could continue to be so for a fairly long time.  Indeed, a system in this state represents exactly what you would expect from a team in the middle of a responsible, intelligent migration - and this could be a "correct" and viable state for a long, long time.

If you've started with a monolith, as many teams do and probably should in most cases, the current constraints on your system might be such that making the effort to go full-microservices-mode can't really be justified on the basis of business requirements.  In other words, it's fully possible to have a system that can meet current and near-term projected future demand just fine as a monolith.  Don't feel like your solar system architecture is wrong just because your transition to a microservice architecture isn't complete - because probably *no* system would ever be claimed to be in it's perfect, final state anyway, no matter how microservice-y it is.

If a solar system architecture is meeting your current and reasonable projected near-term future needs, and your team is able to be sufficiently responsive to business requirements, this "temporary" state may be just fine for a good long while.
