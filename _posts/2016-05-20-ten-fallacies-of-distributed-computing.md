---
layout: post
title: Ten Fallacies of Distributed Computing
---
The fallacies of distributed computing are [well known](https://blogs.oracle.com/jag/resource/Fallacies.html).  I didn't come up with them, but I'll summarize them here:

1. The network is reliable.
1. Latency is zero.
1. Bandwidth is infinite.
1. The network is secure.
1. Topology doesn't change.
1. There is one administrator.
1. Transport cost is zero.
1. The network is homogenous.

I have two to add.  

1. Processes never stop running.
1. Time is absolute.

Many other things I consider to be fundamental knowledge of distributed systems, like CAP theorem or the challenge of exactly-once delivery or that having relationships between data has a constraining effect on scalability, essentially derive from the original eight fallacies of distributed computing.  It feels like the two I've added are things we know recognize as essential knowledge not captured by the original eight.

I hope I'm not being presumptuous.

Briefly, their explanations:

####Processes never stop running

Of course we know this is not true, but when we design distributed systems we tend to ignore all the ways processes can stop running:

* A process can stop running due to a system crash caused by a coding error.
* A process can stop running because it is killed by a user, malicious or otherwise.
* A process can stop running because of a hardware failure.
* A process can stop running because of a power failure.
* A process can stop running, temporarily, due to garbage collection pauses in the execution environment.

The last one is especially tricky.  There is a way around it:  Don't use a garbage-collected language.  In other words, write your system in C.  Of course, it may take you an order of magnitude longer to actually deliver your application in this case.

Fortunately, if we can make allowances in our system for the fact that processes are going to stop running, we can include another reason they stop:

* A process can stop running because of planned maintenance.

If your system is sufficiently robust to handle when a process stops running, you gain the added benefit of making planned maintenance easy.

####Time is absolute

Essentially, if you were to attempt to ask any N machines in a distributed system, "What time is it?" you should expect N different responses.  Some of these differences have to do with inefficiencies in the network:

* Even if all requests are sent at the same time, each machine may receive the request at a different time
* Even if each machine uses a shared global clock (e.g. NTP server), their requests for the current time may be processed at different times

But even if we assume a completely efficient system where the above differences are mitigated, we still can't assume each machine in the system will agree on the exact time.  This is true even if we assume that at some prior instant in time all clocks on the machines were synchronized exactly.  Differences in processing load alone can account for getting different answers from two different machines that receive the request at the exact same time.  And there's also the possibility of slight differences in hardware that introduce minor amounts of drift over time.

The reason this matters is simple:  You cannot synchronize computation across two different machines by using time.
