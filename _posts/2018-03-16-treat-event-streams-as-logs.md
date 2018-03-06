---
layout: post
title: Treat Event Streams as Logs
published: True
---
There's this thing called the [12-factor app](https://12factor.net/) that's the buzz at work these days.  And in general I think the principles are pretty good.  But I take issue with principle number 11.

Principle 11 says:  "Treat logs as event streams."

I take issue with this principle on two levels.  First, it gets the ordering wrong.  Second, it implies that generating events is maybe optional.

Let's tackle the second one first.  When it says "treat logs as event streams" it is basically saying, "You should regard your logs as a record of events."  What should you do with those events?  Should they be published or broadcasted anywhere?  Should they be externally visible to anyone who can't access the logs directly?  It doesn't bother to say.

For cloud-based systems, especially large-scale or enterprise systems but I'd argue for anyone to whom 12-factor also applies, it should be the case that you are publishing events from your systems.  Those events should be published via some mechanism that allows them to be consumed by any interested external party, assuming appropriate permissions of course.

Writing messages to a log doesn't count as event publishing, and it shouldn't count even if you have something listening for file changes or as a syslog receiver.  Splunk may have made oodles of money convincing the world that monitoring your logs is equivalent to high-quality system monitoring but that is just not the case.  Log messages aren't a good medium for generating monitoring information, and they aren't a good medium for generating event information either.

Unlike log messages, events are well-defined within the context of the source code.  When I'm writing software and I'm going to publish an event from my code, I recognize at that point in time that the event must be easily consumable by other source code.  I'm likely to use some type of schema definition or a class to define my event so that the format is easily consumed by whatever party is on the other end, waiting to be notified about my event.  However, when I write a log message, it tends to be whatever words happen to come into my mind at the time.  I'm not thinking about anyone consuming it, other than a human reading a log file.

Nobody would consider deploying such a system without logging.  Why do we care so much about logging?  Because we need a record of the events so we can diagnose problems.  What else could you do with those events if they were available for consumption?  You could do all kinds of things, like build asynchronous and independent add-on features, or allow your users to customize their experience without impacting your system, or automatically alert operations staff when certain things happen.

But you can't reliably do that if the event stream is just a log.  The reason is that you can't rely on the format of the log messages enough to do any of those things.  The developer might change the wording of the log message ever so slightly and break your integration.

However, if you are publishing events instead, the developer is aware that there are other integrations possible - because you are publishing an event, not writing a simple debug log message.  Care can be taken to consider compatibility with previous message formats to minimize integration failures.

And that's also why I say you should change the ordering.  You should consider publishing events mandatory.  And you should generate logs from the stream of published events.

So I'd restate principle 11 like this:  "Publish event streams, and generate logs from event streams."
