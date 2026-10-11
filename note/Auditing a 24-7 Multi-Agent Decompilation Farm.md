# Auditing a 24/7 Multi-Agent Decompilation Farm

momo5502 turns ~2GB of session logs from a round-the-clock multi-agent MW2 decompilation project into hard numbers: where $85k of pay-per-token-equivalent spend went, why subagents were mostly reading the same files as each other, and why the reviewer's model choice matters more than the worker's.

---

## What the logs said

The setup is a worker fleet plus an "Overseer" that reviews every pushed commit and reports to Discord. Running 24/7 on a subscription cost ~€200; the same tokens at list API prices would be **$85,207** ($70,275 workers, $14,933 overseer), with 89% of reads served from cache because the agents never stop.

Of 1,107 worker agent-hours, large chunks went to building (unavoidable in a fixed codebase), searching, and a long tail of pure waste: CI polling, sleep/wait, and idle time — largely eliminated by replacing `/goal` (agents marked goals done prematurely) with a `/loop` cron reminder every 3 minutes, plus a memory telling agents the harness handles context so they never idle "waiting for it to clear."

## Subagents failed an audit and got fired

Subagents couldn't edit the shared workspace, so their only job was research-and-condense. The audit measured their efficiency by duplication: across 47 runs, **54.7% of data read from large files was pure duplication** — `STATUS.md` was read 28 times by 21 different subagents, with a median of 35 minutes between duplicate reads. Had the parent done the research itself, the file would have stayed in its context. Disabling subagents *raised* commits/day from 243.9 to 311.1. The author's remaining worry is fair and general: parallel subagents can uncontrollably multiply token spend.

## Compaction was too late

Sessions lived at 93–100% of the 1M-token window (median 933k before compacting, 1,346 turns, 3.3h between compactions). Nearly every turn paid near-full-window cache-read cost — on context that was mostly volatile, since an already-decompiled function is irrelevant to later turns. Compacting at 60% should save tokens "yet result in nearly equal performance."

## The reviewer's model matters more than the worker's

Sonnet 5 workers: 6.1 commits/hour, 9.1% build-failure rate. Opus 5: 1.6 commits/hour, 23.9% failures. Normalized to bugs per 100 commits, both workers make the same mistake rate — but a **Sonnet overseer flags more bugs than an Opus overseer**. The author's conclusion: "it's the reviewer's model that matters, not the worker's!" The Overseer pulled its weight regardless: 354 distinct bugs flagged out of 2,177 commit-referencing messages.

## Key quotes

> "54.7% of all the data subagents read from big files was pure duplication. The worst offender, `STATUS.md`, was read 28 times, by 21 different subagents."

Subagent isolation bought parallelism nobody needed and paid for it in duplicated context. This is a measurement most multi-agent setups never make — and here the measurement killed the feature.

> "So, it's the reviewer's model that matters, not the worker's!"

The single most transferable finding. If mistake rates per commit are equal across worker models, the marginal dollar belongs in verification, not generation.

> "Soft instructions don't really help, mechanical blockers, to enforce rules, are more effective."

Consistent with the wiki's recurring theme that deterministic structure beats nudging.

## Take

This is one of the few write-ups that actually *audits* a long-running multi-agent system instead of celebrating it, and the honest caveats are part of the value: commits/day as a productivity proxy is admittedly rough, the Opus-vs-Sonnet comparison may be confounded by task selection, and the author repeatedly says "we'll see." What survives any confound is the waste analysis — polling, sleeping, and duplicated reads are objective losses. Two lessons generalize hard: (1) webhook-and-event beats agent polling everywhere, a distributed-systems instinct applied to agents; (2) earlier compaction of volatile context is nearly free savings. The $85k sticker on a €200 subscription also quietly makes the cost story about cache and subscriptions, not tokens per se.

## Related pages

- [[Oversight Degrades the Overseer]] — complicates this source: here a reviewing overseer demonstrably earns its keep (354 real bugs), which sharpens the question of *when* oversight helps versus when it degrades.
- [[Pi Subagents]] — reinforces with numbers: both find subagent research-and-condense modes burning duplicated context; this source goes further and disables them on evidence.
- [[Using LLM-as-a-Judge Scoring to Measure Your Software Factory]] — strengthens the same move of measuring an agent factory from its own logs rather than judging output by vibes.
- [[Managing AI Coding Costs at Scale]] — nuances the cost picture: the same token stream is $85k at list price and €200 on subscription, so unit-economics claims depend entirely on the pricing regime.

---
*Sources: [[raw/2026-08-22-mw2-decompilation-audit]], [[summary/2026-08-22-mw2-decompilation-audit]]*
*Last updated: 2026-10-11*
