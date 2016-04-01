---
layout: post
title: Introducing CooperDB
---
A few weeks ago my friend @jtolds posted the following on Twitter:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Why hasn&#39;t someone named a database &quot;CooperDB&quot; yet? Peak saturation is when all the obvious puns are used and I thought we were closer.</p>&mdash; JT Olds (@jtolds) <a href="https://twitter.com/jtolds/status/709528436688039936">March 14, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I took this as a personal challenge.  It was clearly my destiny to create [CooperDB](https://github.com/mattvryan/cooperdb).

So what is CooperDB?

* Obviously, it is a database.
* Obviously, it is named for [D.B. Cooper](https://en.wikipedia.org/wiki/D._B._Cooper).
  * And let's be clear - I'm not naming it **after** D.B. Cooper nor am I naming it **in honor of** D.B. Cooper, nor out of any disrespect to victims of such a terrifying crime.
  
That doesn't tell us much.  There are lots of databases.  What type of database is this?  It has two key features:

* It has a simple REST-style interface, e.g. POST to create, GET to retrieve.
* You can store anything you want in it, but you will never be able to find it later.

In truth, the last feature is what really sets CooperDB apart from all the others, and which also contributes to its rather simple codebase and architecture.  CooperDB is very performant and scalable.  In fact, scaling CooperDB is super simple - it can be accomplished simply by adding nodes.  Despite it's share-nothing architecture, you can issue requests to any node in the cluster, irrespective of where previous requests went.  You can POST an object to one node, GET the same object from a different node, and always get a consistent answer.

Load balancing for CooperDB is also super easy.  Any HTTP load balancer can work great for creating a CooperDB cluster.  Since CooperDB is completely stateless and each request is fully independent, the load balancer doesn't have to route any specific request to any specific node, so even simple load balancing schemes will work perfectly.

In fact, I will go so far as to say CooperDB is the first ever database to achieve all three elements of CAP.  It is an immediately-consistent, highly-available, and partition-tolerant database.  No other modern database can say that.

I'll do a more involved write-up of CooperDB soon.  In the meantime, you can download the source code on [GitHub](https://github.com/mattvryan/cooperdb).