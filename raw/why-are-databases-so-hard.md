---
url: https://gtowey.blogspot.com/2026/07/why-are-databases-so-hard.html
title: Why are databases so hard?
author: gtowey
date_fetched: 2026-08-01
date_published: 2026-07-30
---

You've probably all experienced it; another outage and the database is the root cause. Why are databases such a frequent cause of problems in most tech stacks? Why can't we seem to solve these problems industry-wide? Are database engineers and database admins just bad at their jobs?

Over my career as a database reliability engineer, I've come to a conclusion which I don't see repeated often:

All practical implementations have to balance the opposing concerns of correctness vs. performance & availability (this is kind of similar to CAP theorem, but not exactly the same). Perfect correctness with no data loss across geographic distances would result in a database which is too slow or too costly to be useful for most applications. And these constraints cannot be overcome because it's the physical bounds of reality which imposes these limits.

Why geographic distances? I'll explain this step-by-step below.

## Step 1: A Single Isolated Database Instance

You start with installing a copy of PostgreSQL or MySQL on a single instance, or using a managed database product from a cloud provider such as AWS's RDS. At this scale the database performs everything you need. Full ACID compliance, fast reads and writes, transactions all work beautifully. We're done, right?

Well only if you ignore that hardware and VM instances are not flawless. Despite the database software being more than adequate, sometimes that underlying hardware or VM infrastructure will fail. The naive response is to just wait until the original instance can be restored and you continue on your merry way. However this could take minutes, or hours, or days. If you have customers paying to use your service, they're not going to be so patient. So now you need High Availability!

## Step 2: High Availability

High Availability means that you want your system to recover to a usable state as quickly as possible. How quick? It depends on the specific implementation. For me, <10 seconds is common. <1 second is the goal. For systems like RDS their default gives you ~2 minutes average recovery time, with options that will get that down to ~30 seconds. Not really HA-enough for my taste, but for most people it's a vast improvement over hours or days of outage!

BUT THERE IS A TRADEOFF -- A COST!

Do you see it yet? It's that improving reliability means having another copy of your database ready to take over when the primary fails. Maintaining that copy takes time. Time is the tradeoff.

Every time you write data to your primary/live instance, that data must be recorded somewhere else where it will be available when another database instance takes over to maintain availability. Well, you could certainly build a system where this isn't true, but imagine your customer's surprise when you flip from one database instance to another and data they thought they had persisted suddenly disappears. Even worse is the problems you would invite when you flip back to the original copy of the database and that data suddenly appears again. This is our "correctness" problem. The correctness problem is this: when we have to maintain multiple copies of our "source of truth" data, how can we make sure they all stay in-sync?

I have some good news and bad news on that front: the good news is that we absolutely can keep all our copies perfectly in-sync and ensure perfect correctness. The bad news is our database system will now be so slow it's probably unusable on a practical level. It works like this: When a request comes to our primary/active database to update/insert/delete data, we can pause the transaction at the time of commit and go transfer that transaction data to our other copies. Only once we have confirmed the data has been durably persisted to our other copies do we finish the commit and return a success to the original transaction's client. This adds time to the client's request. They have to wait for data to be transferred over the network between our databases. For two database instances in the same datacenter, this could be microseconds -- not terrible, maybe not even noticeable. However, that's not the end of the problems we've added. Now what happens if our backup database fails and can no longer accept updates, even when it's not being actively used? To maintain correctness we would have to stop accepting writes to the primary database as well! It's the only way to ensure they always remain perfectly in sync is to treat a failure of one node as a whole-system failure. Wait, we were supposed to be increasing availability. Did we just actually decrease it instead?

Also when the primary database fails and we flip to the secondary we now no longer have a backup copy and we lose HA properties until the other instance is restored. We could just run more backup copies, but now we have more data transfers to keep everything in-sync. We could just say we only need 2 out of N nodes to be in-sync at all times and mark the others as unusable temporarily until they can re-sync. Or is that 3 out of N, so that we have a backup-for-the backup. And our cost to serve a single copy of our data set has gone up to what? 3x? 5x? It's starting to get more expensive now too.

Now I hope you're starting to see the complexity of the problem here. We could go into permutations of redundant architectures until the cows come home, but I'll spare you. Suffice it to say that every single architecture we could examine or invent is going to run into the same fundamental limit -- it takes time to keep copies of our data up-to-date perfectly. And the only way to mitigate the time constraint is to relax the correctness constraint. There is no way around this.

And we're not even done yet, because our highly available system still only operates in a single physical datacenter. A single disaster which takes out the whole datacenter still means we're hosed. Many companies just call this good enough and accept the risk (after all us-east-1 never goes down, right?) But for others, their customers won't be happy with an extended outage even if you can claim it's not your fault. For true fault-tolerance you need yet another copy of your data in some other physical location, usually far enough so that the same hurricane, or earthquake or power grid outage doesn't affect both locations. This is how we arrive at geographic distribution.

## Step 3: Disaster Recovery & Geographic Databases

The astute will note that this is just an extension of the same problem we have with transit times in our HA setup, now with larger distances involved. This should be easy! How much more time could we possibly have to manage?

Let's take New York to Los Angeles as an example. If you were able to send data at the speed of light, it would take 16 milliseconds! And that's a one-way trip. To let our primary database receive a confirmation that the data was received we need a minimum of 32 ms. And keep in mind this is the theoretical maximum the laws of physics allow for a straight-line path. In practice even if our network was fiber from end-to-end, we still have stops at various routers along the way for processing. A real network request therefore takes more like 66ms per trip, and a 130ms round-trip time.

I have yet to experience any commercial enterprise willing to accept database write latency of 130ms. Using cloud services you might end up paying thousands, or even tens of thousands of dollars per year for a system that can process <100 write transactions per second.

So what do you do? How do you surmount the laws of physics? You don't. You MUST compromise something.

And that's the entire point of this article -- you cannot escape the fundamental laws of physics. You only choose what properties are desirable and know that you will be giving up other things. If you absolutely cannot tolerate data loss, then your system will be incredibly slow (or costly). If you want great performance and efficiency, there will be ways you can lose data.

I talk about the laws of physics because someone might see the 130ms round-trip-time and think that we just need to do some fancy computing to optimize that. Or change how we build networks. But even if we did that we'd get at most ~4x improvement. There is not even a single order of magnitude left between our current performance and the maximum allowed by the laws of physics. We cannot optimize time much more than we already have! No matter how advanced technology of the future becomes, this same problem will still exist until the end of the universe. Even at the speed of light, it takes a significant amount of time to move data.

Most companies use a strategy of creating an HA cluster with strong consistency guarantees within a single datacenter only, and then using an "eventual consistency" approach to shipping data to another geographic location. If the need arises to run their application from a different geographic location, they call it "disaster recovery" and let clients know that recovery could take hours and some data loss is expected. This applies equally if you're using async replication to a warm-standby database or if you're taking backups or snapshots and filling in the gaps between full backups with transaction logs. If your primary datacenter fails while processing user requests, there will always be some window of time where data written to your database at that location won't make it to your backup. It could be seconds, or minutes. No matter what you cannot guarantee consistency with an async update model.

This is why databases are hard -- there's never a perfect solution which will work all the time for all use-cases. And this also ignores the entire other class of database problems which relate to availability which is what happens when someone writes a bad query that DOSes your database. Just scale up your database, or just partition it, right? But as we see in this article, scaling a database is hard because everything takes time. Partitioning is hard because you create new consistency problems. Availability is hard because keeping things in-sync is hard.

## A Postscript

Communicating this is honestly the most challenging part of my job. Initial development of most software projects starts with the single isolated database either from a cloud provider or just on someone's laptop. They develop against a non-distributed database system and a tiny database size and everything works! It's blazing fast, it's perfectly consistent. No durability issues, etc. When engineers take this to production they are soon frustrated by the production database which seems slower and less reliable. They underestimate that the production database has so much more demands and constraints against it.

Yet I will talk with an engineering team one week that stresses how important consistency guarantees are for them. Sure, I can do that. Then the next week I'll talk to another team that demands the fastest performance possible. Now we have a challenge. Then after an incident I'll be yelled at by a manager who says we need to make availability our highest priority because our largest customer is threatening to churn. Then as we approach the end of our fiscal year I'll have other people breathing down my neck saying we need to cut costs. Then this whole cycle repeats.

You can try to fix this by running multiple database systems. The slow-yet-scalable system; the ultra-fast-but-lossy system; the perfectly-consistent-but-tiny metadata store. This is why so many companies run several different types of databases. Redis for ephemeral data, MySQL/PostgreSQL for transactional guarantees, key-value stores for easy scaling of simple data. But now the job is to make sure engineers are choosing the right location for their data. Inevitably not all data will find the right home on the first try and migrating data from one system to another is always a big task which isn't fun.

It all feels a bit like Sisyphus some days, but at least it's job security!
