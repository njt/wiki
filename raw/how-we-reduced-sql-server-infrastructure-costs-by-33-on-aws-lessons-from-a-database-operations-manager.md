---
url: https://www.sqlservercentral.com/articles/how-we-reduced-sql-server-infrastructure-costs-by-33-on-aws-lessons-from-a-database-operations-manager
date_fetched: 2026-09-20
---

## The Starting Point

Last year, our executive leadership team sent a clear message to every department — find ways to cut costs, and find them fast.

As the Database Operations Manager, I knew the database infrastructure bill was one of the first places I needed to look. Not because someone told me to specifically, but because I had seen the numbers before and had a feeling we were paying for more than we actually needed. We were running 27 SQL Server instances on AWS at the time: 15 production and 12 non-production, covering dev, test, and QA. Our monthly bill sat at $60,000. That number did not happen overnight. It grew gradually as the environment expanded, and nobody stopped to ask whether each dollar still made sense.

I built a full inventory from scratch. Every instance, its edition, its snapshot history, its usage pattern. Then I had a call with our AWS Technical Account Manager to pressure-test what I was seeing. They confirmed it. The biggest opportunity was sitting right there in non-production.

Three changes later, we were at $40,000 a month — $20,000 saved every month, $240,000 a year, with no disruption to production. Here is what we did.

## What the Inventory Showed

Before touching anything, I needed a clear picture of what we had. Most teams rush this step, and that is usually why cloud costs get away from them.

All 15 production instances ran SQL Server Enterprise Edition. That made sense — our production workloads need Enterprise features. No issue there. Non-production told a different story. All 12 dev, test, and QA instances also ran Enterprise Edition. Every single one. Teams often build non-production environments by copying production configurations without questioning whether it is necessary. Enterprise in non-production feels safe. It also means paying AWS hourly rates for features developers never touch.

The snapshot situation also surprised me. Going through our inventory, I found old snapshots from projects that teams had finished and closed months earlier. Nobody deleted them. They sat there quietly adding to the bill every day.

And every non-production instance ran 24 hours a day, 7 days a week. Our dev and QA teams work standard business hours. Those instances burned money all night and all weekend with nobody connected to them.

Three problems, all fixable.

## The Three Changes We Made

Here I give details on the three decisions we made. We changed non-production editions, deleted snapshots, and scheduled non-production instances.

### Decision 1 — Edition Standardization in Non-Production ($12,000/month saved — 60%)

SQL Server Developer Edition is free and delivers full feature parity with Enterprise Edition. The only restriction is that you cannot use it in production. For dev, test, and QA that restriction is irrelevant, which means every non-production instance running Enterprise Edition pays for something it does not need.

All 12 of our non-production instances needed to move to Developer Edition.

We followed the same process for each one. We spun up a new EC2 instance with Developer Edition installed, took an AWS snapshot of the existing Enterprise Edition instance, restored it to the new instance, validated that everything came across cleanly, updated all connection strings, tested, and once everything checked out we deleted the old Enterprise instance.

One thing worth flagging before anyone attempts this: audit your non-production workloads for Enterprise-only feature dependencies first. Online index rebuilds, advanced compression, Always On — none of these work on Developer Standard Edition (these features do work in Enterprise Developer Edition). We checked each instance and cleared dependencies before making any switch. That added time upfront but we hit no surprises during the migration.

After completing all 12 instances the edition change alone saved us $12,000 a month.

### Decision 2 — Deleting Stale Snapshots ($6,000/month saved — 30%)

This change needed no architecture decisions and no downtime. It just needed someone to actually look. The inventory surfaced AWS snapshots from past migrations, testing cycles, and version upgrades that teams never cleaned up after finishing the work. AWS charges for snapshot storage by the gigabyte whether you use them again or not.

I reviewed every snapshot across all 27 instances, matched each one to its source project, identified which ones belonged to closed work or had newer copies replacing them, confirmed with the relevant teams they were safe to remove, and deleted them.

That cleanup saved $6,000 a month. No technical optimization involved — just paying attention to what had accumulated in storage.

We now run a quarterly snapshot audit as standard practice. The first one is always the most expensive because nobody reviewed the buildup for the longest time. After that it becomes routine maintenance.

### Decision 3 — Scheduling Non-Production Instance Hours ($2,000/month saved — 10%)

Non-production instances do not need to run all the time. If your developers and QA team work business hours, the instances should too.

We set up automated start and stop schedules on all 12 non-production instances. They start before the team arrives and stop after they leave. Weekends stay off unless a specific deployment needs them up temporarily.

Before enabling scheduling we audited all SQL Server Agent jobs and maintenance tasks that ran outside business hours on those instances. We rescheduled overnight jobs to business hours or flagged them for manual handling on the rare occasions they needed to run. Skipping this check matters as enabling a stop schedule without knowing what jobs run overnight will kill a process mid-execution and cause problems that take longer to fix than the saving is worth.

The scheduling change added $2,000 a month with no impact on how any team worked day to day.

## Results

Two weeks from starting the inventory to completing all three changes. Our monthly costs dropped from $60,000 to $40,000 — a 33% reduction, $240,000 saved annually. All 15 production instances ran through the entire project without a single configuration change. No downtime, no incidents.

The dev and QA teams noticed nothing different. Developer Edition gave them the same environment they were used to. The instances started when they arrived and stopped when they left. From their side nothing changed. From the finance team's side $240,000 came back into the budget every year.

Our AWS TAM confirmed the final numbers matched their projection. That mattered — it showed this reduction was permanent, not a one-month anomaly.

## What I Would Tell Other DBAs Before Starting

Three things drove the difference between a successful cost reduction and what I see most teams do — either nothing at all, or making changes without understanding what they are touching.

### Build the full picture before doing anything

The inventory audit is where this exercise works or falls apart. I did not touch anything until I understood exactly what we had — every instance, every edition, every snapshot, every usage pattern. That audit surfaced the snapshot problem, which I never would have found if I had gone straight after the edition question. Cost opportunities in database infrastructure usually hide in the corners nobody has looked at recently. Build the full picture first, then decide where to cut.

### Use your AWS TAM before things go wrong, not after

Most teams call their TAM when something breaks. That is the wrong time. I engaged ours before making a single change. That conversation validated the approach, confirmed the edition opportunity, and helped me understand the snapshot cost mechanics in a way that changed how I went about the cleanup. If you have a TAM relationship, schedule a cost review call before you start any optimization work.

### Stay current with what SQL Server and AWS actually offer

The Developer Edition opportunity is not new — Microsoft has offered it for years. But many organizations still pay Enterprise rates in non-production simply because nobody stopped to ask whether they needed to. Edition structures, instance types, and pricing tiers change. What did not make sense when your environment was originally built might be the right answer now. A quarterly check of the latest SQL Server release notes and a TAM conversation costs nothing and regularly surfaces opportunities.

## Wrapping Up

Database infrastructure costs rarely spiral from one bad decision. They drift upward because nobody revisits decisions that made sense at the time but no longer do.

Enterprise Edition in environments that do not need it. Snapshots from projects long closed. Instances running when nobody is using them. None of these are complicated problems. They are things that got missed.

The approach that worked for us: build the inventory first, get your TAM involved early, standardize non-production to Developer Edition, audit and delete stale snapshots on a schedule, and match instance hours to actual working hours.

We went from $60,000 to $40,000 a month in two weeks. The opportunities are probably sitting in your environment right now.
