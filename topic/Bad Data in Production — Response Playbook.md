# Bad Data in Production — Response Playbook

Pinal Dave's six-step framework for responding to data quality incidents: triage, contain, trace lineage, fix-and-verify, notify stakeholders, and run a short blameless review. The playbook fills a genuine gap — teams have runbooks for infra failure and code bugs but rarely for data that is silently wrong. The core insight is that bad data is a trust incident as much as a technical one.

---

## The Framework

Dave's six steps, in sequence:

1. **Triage** — What exactly is wrong, how far has it spread, and who is already acting on it? The blast-radius question determines severity. A wrong number in a customer-sent report is a fundamentally different incident from one on an internal dashboard.
2. **Contain** — Stop the source of bad data before cleaning up. The running-tap metaphor: mopping while the feed still writes is performative, not productive.
3. **Trace lineage** — Walk backward from dashboard → published table → transformation → source feed. Fixing the symptom without finding the originating fault guarantees recurrence.
4. **Fix and verify** — A fix you haven't verified is a hopeful edit. Re-run the original check, inspect adjacent tables, confirm nothing downstream broke.
5. **Notify** — Anyone who trusted the bad number hears from you directly, before they hear it elsewhere. This is the difference between a team people trust and one they quietly stop believing.
6. **Review, blameless** — 15 minutes. What broke? Why? What single control would have caught it sooner? The moment it becomes a hunt for who to blame, people stop telling you what really happened.

## Key Quotes

> "A wrong number already sitting in a report someone sent to a customer is a very different incident from one that's only been seen by an internal dashboard."

Commentary: Dave's severity model is trust-distance, not data-inaccuracy. The technical fix might be identical in both cases, but the operational response — who you call, how fast, what you say — is completely different. Most incident severity frameworks would miss this distinction because they classify by system impact, not trust impact.

> "A fix you have not verified is a hopeful edit."

Commentary: This is the shortest, sharpest formulation of the verification problem. It applies far beyond data incidents — every AI-generated code change that ships untested is a hopeful edit. The discipline Dave describes (re-run the check, inspect neighbors) is the same discipline coding agents need but structurally lack.

> "The moment it becomes a hunt for who to blame, people stop telling you what really happened."

Commentary: The blameless postmortem truism, restated with precision. Dave doesn't argue blamelessness as moral virtue — he argues it as information hygiene. Blame is not cruel; it's stupid. It destroys the signal you need to prevent recurrence.

## Key Themes

- **#pattern** — The six-step playbook itself is a transferable pattern. It's not specific to SQL Server, or to any particular data stack. Triage → Contain → Trace → Fix → Notify → Review works for any data pipeline incident.
- **#concept** — Trust as the hidden severity dimension. Bad data incidents are uniquely damaging because they corrupt decisions rather than availability. An outage is visible and binary; bad data is invisible and distributed.
- **#concept** — Data lineage as incident response tool, not just governance checkbox. Dave's "walk it backward" is the operational use case for lineage that governance tools promise but rarely deliver under pressure.
- **#tool** — Blameless review as information-extraction mechanism. The goal isn't absolution; the goal is truth. Blame shuts down the flow of information you need to prevent recurrence.

## Critical Analysis

**What the playbook gets right:** The containment step (step 2) is the non-obvious contribution. Most engineers, faced with bad data, jump straight to fixing it — UPDATE the wrong rows, patch the transformation, move on. Dave correctly identifies this as the data equivalent of restarting a server without checking what caused the crash. Containment before cleanup is a systems-thinking instinct that most data teams haven't internalized.

**What's missing:** The playbook assumes you *can* trace lineage. In most real organizations, the answer to "where did this number come from?" is "a spreadsheet someone's manager's manager built in 2019." Dave's framework is correct but assumes infrastructure that doesn't exist in the organizations that need this playbook most. The hidden prerequisite — invest in data lineage before you need it — is the real prescription.

**The silent assumption:** All six steps assume a single person or small team owns the response end-to-end. In a large organization, step 1 (triage) might be analytics, step 2 (contain) might be data engineering, step 3 (trace) might require a different team entirely, and step 5 (notify) might be comms. The coordination cost of the playbook across organizational boundaries is the unexamined variable. Dave's framework works best in teams that already have clear data ownership — which is itself the harder problem.

**The Pluralsight course coda:** Dave ends the article with a promotion for his Pluralsight course on the same topic. This is worth flagging not as a criticism but as a pattern: the blog post is the abstract; the course is the paper. The article is useful at its length precisely because a six-step checklist doesn't need a full course to be actionable.

**The comparison to SRE:** This is effectively an SRE incident response framework adapted for data quality. The structure maps cleanly to Google's incident management practices (detect → triage → mitigate → resolve → postmortem), but Dave's innovation is recognizing that data incidents need a distinct playbook because the failure mode is different — silent corruption rather than visible outage. The SRE parallel also reveals what's missing: there's no equivalent of an SLO for data quality in this framework, no threshold at which "wrong" triggers the playbook. Without a data quality SLO, every wrong number is a judgment call.

---

## Connections

- [[Databases and Data]] — The hub page; this playbook addresses the "data quality" problem it names as a persistent hard problem
- [[Queues Don't Fix Overload]] — Same structural instinct: identify the bottleneck before applying a fix. Hebert's "red arrow" is Dave's "source, not the symptom"
- [[Software Engineering Craft]] — Incident response as craft; the fundamentals don't change even as tooling evolves
- [[Long Live Systems of Record]] — The systems-of-record question is upstream of Dave's playbook: you can't trace lineage if you don't know where the truth lives
- [[The Log is the Agent]] — Event sourcing as the infrastructure that would make Dave's step 3 (trace lineage) trivial rather than heroic
- [[Event Sourcing — Set-and-Remove Bi-Temporal Events]] — Bi-temporal event streams as the technical substrate for answering "what did we think was true, and when did we think it?"
- [[Guardrails and Feedback Loops]] — The single control Dave asks for in step 6 is a guardrail; the blameless review is a feedback loop

---
*Sources: [[raw/bad-data-production-response-playbook]]*
*Last updated: 2026-07-18*
