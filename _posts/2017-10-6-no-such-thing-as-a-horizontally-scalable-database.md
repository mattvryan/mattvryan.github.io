---
layout: post
title: There's No Such Thing as a Horizontally Scalable Database
published: True
---
Along with my good friend Clint Goudie-Nice, I gave a presentation at OpenWest 2017 entitled "Scaling the Unscalable".  It details some of the things we've learned exploring scalability for Adobe Experience Manager.  One of the items discussed in the presentation was the assertion that, when it comes to production systems, there is no such thing as a horizontally-scalable database.  We arrived at this surprising conclusion as we evaluated some of the characteristics of high-scale databases.

To explain this further, I'll walk through the scenarios.  I'm not going to use any specific database to describe the problem - it essentially applies to any clusterable database.  (Obviously, a non-clusterable database is not horizontally scalable, and therefore not relevant to the discussion.)

## Partition Tolerance is Not Negotiable
Before we get very far into this, one point that needs to be made is that partition tolerance is not negotiable.

A partition occurs when one or more nodes in the cluster are not able to communicate with one or more other nodes in the cluster.  This is a usually temporary situation, one that might happen rarely, but you have to assume that partitions will happen because you are smart enough to remember that the number one fallacy of distributed computing is that the network is reliable.  If we want to be responsible, we must assume partitions will happen, so we must therefore be aware of what will happen when a partition occurs.

If a database is designed or configured such that nodes in the cluster can accept database updates when a partition exists, then some mechanism for conflict resolution must be used in the case where conflicting updates happened on both sides of the partition.  In this diagram, Client 1 and Client 2 both make database updates at roughly the same time, which happens to take place when a partition exists.

```
+---------------------------------------------+
|                                             |
|    +-----------+           +-----------+    |
|    | Client 1  |           | Client 2  |    |
|    +-----------+           +-----------+    |
|          |                       |          |
|          V       Partition       V          |
|    +-----------+     |     +-----------+    |
|    | DB Node 1 |<----|---->| DB Node 2 |    |
|    |           |<-+  |     |           |    |
|    +-----------+  |  |     +-----------+    |
|                   |  |           |          |
|                   |  |     +-----------+    |
|                   +--|---->| DB Node 3 |    |
|                      |     |           |    |
|                      |     +-----------+    |
|                                             |
+---------------------------------------------+
```
Normally we could assume that all three database nodes could possibly coordinate the updates from the two clients and prevent conflicting data.  But when the partition exists this is not possible for the coordination to happen.  So because of the partition there's a chance that Client 1 and Client 2 could both successfully update the database with updates that conflict.

When the partition is resolved, the cluster must choose some technique for resolving the conflict.  There is no perfect resolution technique.  Whether it keeps the oldest change and throws away the newest, keeps the newest and throws away the oldest, tries to merge the changes, emails an intern to look at the conflict and manually resolve it, or uses some other technique, there are scenarios which will result in data loss.

Such a system is not partition tolerant.  To be partition tolerant means that partitions, when they occur, do not impact data integrity.

We assume that if a system is going through the trouble of storing the data, the data must therefore be important enough that data loss or corruption is unacceptable.  (Admittedly this may not be true for some systems.)  Since we can safely assume that in at least the overwhelming majority of cases the data actually matters, we cannot risk corruption or loss due to partitions, and therefore partition tolerance is not negotiable.

The way this is handled in a situation like the one above is to require that the database confirm that a write took place on a majority of nodes before replying to the client with a success response.  If that was done, in the example above Client 1 cannot write during the partition, while Client 2 can.  When the partition is resolved, there are no conflicting writes since only one side of the partition was updated.  DB Node 1 can then sync with the rest of the cluster and all is fine.  The cluster is partition tolerant.

## Partition Tolerance Limits Horizontal Scalability
So far we are following this logical progression:
* The data we are storing is important.
* Therefore, we must do everything possible to avoid data loss or corruption.
* In the case of conflicting writes, conflict resolution algorithms must be employed, which are never perfect and therefore will eventually cause data loss or corruption.
* Therefore, we must never allow conflicting writes to occur, in order to avoid data loss or corruption.
* Conflicting writes happen when a partition allows writes without requiring a majority of nodes to confirm the write.
* Therefore, when using a database cluster we must choose a system and/or configuration that will require a majority of nodes (a quorum) to confirm writes.
* Therefore, requiring a quorum to confirm database writes is an essential component of partition tolerance, and is essential to prevent data loss or corruption.

Now it becomes easy to see why partition tolerance limits horizontal scalability.  Let's start by going back to the simple 3-node cluster.

```
+---------------------------------------------+
|                                             |
|    +-----------+                            |
|    | Client 1  |                            |
|    +-----------+                            |
|          |                 WriteConcern=2   |
|          V                                  |
|    +-----------+           +-----------+    |
|    | DB Node 1 |<--------->| DB Node 2 |    |
|    |           |<--+   +-->|           |    |
|    +-----------+   |   |   +-----------+    |
|                    V   V                    |
|                +-----------+                |
|                | DB Node 3 |                |
|                |           |                |
|                +-----------+                |
|                                             |
+---------------------------------------------+
```
Here we are indicating that the write concern of the database is 2, taken to mean that 2 nodes (a quorum) have to confirm the write in order for it to take place.  When Client 1 issues an update request to DB Node 1, that will be coordinated throughout the cluster.  Once either DB Node 2 or DB Node 3 also confirms the write, a success result is returned to Client 1.

By definition, updates to this cluster will be slower than if the write concern was set to 1.  This is due to the latency incurred to perform the additional coordination between DB Node 1 and the other nodes, and to serialize the write on an additional disk.

Horizontal scalability means we can meet increasing performance demands simply by adding more nodes.  However, if we want to add more nodes but also retain partition tolerance, we have to increase the write concern so it continues to require a majority of nodes.

```
+---------------------------------------------------------------------+
|                                                                     |
|    +-----------+                                                    |
|    | Client 1  |                                                    |
|    +-----------+           WriteConcern=3                           |
|          |                                                          |
|          |    +-------------------------------------+               |
|          V    V                                     V               |
|    +-----------+           +-----------+           +-----------+    |
|    | DB Node 1 |<--------->| DB Node 2 |<--------->| DB Node 3 |    |
|    |           |<--+   +-->|           |<--+   +-->|           |    |
|    +-----------+   |   |   +-----------+   |   |   +-----------+    |
|            ^       |   |                   |   |    ^               |
|            |       |   |  +--------------- | - | ---+               |
|            |       V   V  V                V   V                    |
|            |   +-----------+           +-----------+                |
|            |   | DB Node 4 |<--------->| DB Node 5 |                |
|            |   |           |      +--->|           |                |
|            |   +-----------+      |    +-----------+                |
|            +----------------------+                                 |
|                                                                     |
+---------------------------------------------------------------------+
```
If we are going to add another node to the original three-node cluster, we may as well add two nodes since we will need to write to three nodes now anyway to get a majority.  However, we can see in the (admittedly hard to read) diagram that when we added two nodes to this now five-node cluster, we also added two network connections to each node and a total of seven additional interconnects.  These are probably logical, not physical, but the complexity is still there.

When Client 1 writes to the cluster, now three nodes must be involved in confirming the write before it takes place.  By definition this will be slower than before.  Horizontal scalability is supposed to increase performance with the addition of nodes.  The capacity of this system may be higher than with only three nodes, but it is likely that the performance seen by Client 1 will be no better, and possibly worse.

## But ... What About (Insert Your Favorite Cloud Database Here)?
That's a good question.  Once I don't know the answer to.  But the math doesn't lie.  That's why I didn't specify a particular database in the thought experiment above.  The math is the same for any clustered data store.

I can think of a couple of approaches large-scale cloud databases might use, and that maybe you can use too.

### Live With It
It is possible that they actually are designing the databases exactly as described here, in three-, five-, or seven-node clusters for example.  Maybe if you put the nodes on big enough servers and fast enough interconnects with fast enough disks, you don't really notice the slowdown all that much.

### Use Partition-Sensitive Write Concerns
The only reason you need a write concern of a majority is to avoid data loss or corruption that could happen in the case of a partition.  Maybe these databases are very good at detecting partitions, and when one is detected a higher write concern is automatically required.  For example, in the five-node cluster, you could have a write concern that is dynamic - it is set to 1 when all nodes can talk to each other, and if a partition is detected the write concern immediately adjusts to 3.

You would need a really reliable network (so you aren't getting recording false partitions all the time) between nodes, and you would need to be 100% sure you never write without knowing whether there is a partition or not.  It seems to me like there's a small but non-zero chance of a race condition there.  But IIUC this is similar to what Google does with Spanner.

### Build Redundant Interconnects
Another possibility for making this scale would be to create redundant interconnects between each node.  So that would mean that each node would have at least two possible physical routes to each other node.  Creating a partition in such a cluster is much more involved and would require that multiple things fail at once.  The odds of such a partition getting created are substantially lower.  In fact, the odds may be so low that you are willing to take the gamble that it won't actually happen at the same time that multiple clients are trying to update the same record in the database at the same time on different sides of the partition.

In this case you would have a system that is not technically partition-tolerant, but for all intents and purposes is effectively partition-immune, so maybe that's okay.  You have to also feel comfortable living with a condition that "will probably never happen".

When you've been around as long as I have, you never feel comfortable with a situation that will probably never happen, but if it does things could be really bad.  But it is true that with such a system the odds are probably higher that you'll win the lottery without buying a lottery ticket than the odds of data loss or corruption due to a partition.  If you give it a shot, let me know how it goes.
