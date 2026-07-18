---
url: https://blog.sqlauthority.com/2026/07/17/you-just-found-bad-data-in-production-now-what/
title: "You Just Found Bad Data in Production. Now What?"
author: Pinal Dave
date_fetched: 2026-07-18
date_published: 2026-07-17
---

> Most teams have a runbook for server failures. Very few have a plan for when the data itself is wrong. Bad data in production is particularly dangerous because it looks correct: dashboards still render, reports still generate, APIs still return values. Nobody sees the problem until someone questions the output. And by then, decisions have been made on that bad data.

Pinal Dave opens with the observation that data quality incidents are a blind spot in operational readiness. Every team plans for infrastructure failure; almost none plans for data failure. The visible scope of the problem makes it uniquely dangerous — leadership sees wrong numbers and wants answers, not root-cause analyses.

## The Six-Step Playbook

### Step 1: Triage Before Touching Anything

Three diagnostic questions before any action:
- What exactly is wrong? (Be precise — "sales are down" isn't diagnostic)
- How far has the bad data spread? (Single table? Downstream reports? Customer-facing output?)
- Who is already acting on this data?

> A wrong number already sitting in a report someone sent to a customer is a very different incident from one that's only been seen by an internal dashboard.

The severity dimension isn't technical correctness — it's blast radius and trust damage.

### Step 2: Contain the Spread

Stop the problem from getting worse before attempting cleanup. Dave uses a running-tap metaphor: cleaning rows while a bad feed continues to write is futile. Containment buys time for a proper fix.

### Step 3: Find the Source, Not Just the Symptom

> Walk it backward: from the dashboard to the published table, to the transformation that populated it, to the source feed that drove the transformation.

The temptation is to update the visible wrong number and move on. The discipline is tracing the full data lineage to find the originating fault. Fixing the symptom without finding the source guarantees recurrence.

### Step 4: Fix, Then Verify the Fix

> A fix you have not verified is a hopeful edit.

Re-run the original check that caught the problem. Inspect neighboring tables and downstream dependencies. The fix isn't done until you've confirmed it worked and didn't break adjacent systems.

### Step 5: Notify Those Who Trusted the Bad Number

Anyone who made a decision based on incorrect data needs to hear from you before they hear it elsewhere. This is about trust, not process.

> This is the difference between a team people trust, and a team people quietly stop believing.

Don't bury it in a status report. Don't hope nobody noticed. Direct, personal communication preserves the relationship that bad data erodes.

### Step 6: Short, Blameless Review

> Spend 15 minutes asking: What broke? Why did it break? What single control would have caught it sooner?

Keep it blameless:

> The moment it becomes a hunt for who to blame, people stop telling you what really happened.

Document findings on a single page. The goal is one concrete prevention, not an exhaustive report.

## Closing

> I once believed a good team never had data incidents. I don't believe that anymore. Data breaks. Sources change, feeds fail, someone fat-fingers a value. The teams I actually trust are the ones that handle bad data calmly and methodically — and then put controls in place so it doesn't happen the same way twice.

Dave also promotes his Pluralsight course, *Respond to Data Quality Incidents*.
