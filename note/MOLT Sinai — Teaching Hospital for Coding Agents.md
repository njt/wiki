# MOLT Sinai — Teaching Hospital for Coding Agents

Cockroach Labs' five-month field report on MOLT Sinai, a multi-agent coding pipeline built on a teaching-hospital metaphor, is the most detailed published account of a software factory optimized for *quality over throughput* — issues are patients, merging is discharge, and every stage is a GitHub Actions workflow triggered by labels. Its headline result (Db2 database-dialect support in two days for $4,172 vs 9 months/$160K for the equivalent human work in 2024) sits alongside unusually honest failure data: bureaucratic review loops, redundant skill files, and aDb2 delivery that was never human-reviewed.

---

## What it is

- **The trigger:** one GitHub issue (add IBM Db2 as a MOLT migration source) decomposed by a planning agent into 15 sub-issues with a dependency graph; by close, 32 sub-issues, 27 merged PRs, 55 review rejections, 9 escalations (2 to humans), 10K+ lines of test fixtures, $4,172 in tokens.
- **The infrastructure:** GitHub Actions only. Labels in → workflow → agent acts → structured chart note with sentinel → label out. State lives in issue labels plus scratch files. No proprietary orchestrator.
- **The roles:** Fellow (workup + treatment plan), Review Attending (a *different* agent instance prompted to find fault), Discharge Nurse (audits whether review happened, not the code), Charge Nurse (rounds every 30 min for stuck patients), Infection Control (locks down workflows when main breaks), Safety and Research departments, human Chiefs of Medicine.

## Key quotes

> "Agents are relatively cheap to run, but that's little comfort when their code is going to land in customers' hands. We'd much rather have the agents take longer to run (even 10x longer) than land the wrong data in a customer migration."

The reframing that justifies the whole architecture: their metric was "how many of those PRs would we have been embarrassed to have merged ourselves?", not PRs per hour. is the quality-first pole of the factory spectrum that [[Software Factories, Light and Dark]] maps — and it deliberately rejects the throughput-optimized swarm designs of Gas Town and Cursor ([[Agent Swarm Model Economics]]), complicating that page's planner/worker economics with evidence that a rigid role-hierarchy pipeline can beat emergent topology when correctness is the product.

> "An agent may not disable, skip, weaken, or modify a test to make it pass... If a test fails, the test is correct until proven otherwise. All code reviews start by disabling the change and ensuring that all new tests fail."

The strongest anti-gaming rule published by any factory so far, and the inversion of the usual failure mode where agents rationalize tests into compliance. Making a test change its own separately-reviewed commit is a rule worth stealing wholesale.

> "When you're turning over control to agents, you want to ensure that they're doing what they're told, and sometimes they aren't."

The Discharge Nurse's justification — the reviewer is audited by a process, not trusted. the same meta-review move as the auditor agents in [[Orchestrating AI Code Review at Scale]], which automates the reviewer but not yet the audit of the reviewer.

> "Our prompts rotted the way code does: we kept adding rules and never removing them, until the files were too long for anyone to read them end to end. We only noticed because of the token bill."

23% of a 100,000-word skill corpus was removable; the Discharge Nurse loads ~29,000 tokens of instructions before reading a line of the PR. Prompt rot as measurable context debt — and notably, they have no linter for it yet.

> "A Chief-owned decision whose premise the outcome disproved is returned to its owner, not improvised around."

The persona framing produces genuine judgment beyond the skill files: agents refusing to escalate over a docstring fix, deferring authority correctly. The authors' claim — "when you tell an LLM a persona to embody, it does it very very well" — is the same insight as the role-framing skills in this wiki's [[Agent Designer]], but demonstrated at organizational scale.

## What failed

- **Non-converging review loops.** A one-line fix took 11 rework rounds over two days; in one window, three issues consumed $631 of $1,046 spent. The fix: mandatory handoff to an Attending after 4 consecutive rework rounds or 6 substantive rounds. Every factory in this wiki hits this wall; MOLT Sinai is the first to publish a countable circuit-breaker with numbers.
- **Bureaucratic literalism.** A confirmed duplicate couldn't be closed because the plan-review protocol's three outcomes (APPROVE/REJECT/ESCALATE) had no "close as duplicate"; a one-word UI edit went five rounds and reached the Chief. The price of rule-following agents is that the rules become the product.
- **Self-generated work.** Nearly half of 1,299 issues were filed by agents for agents; the Research Department had to be throttled and its unadmitted proposals capped at ten. The pipeline generates its own backlog faster than humans can triage it.
- **It doesn't teach.** A system built by two senior engineers optimizes for senior judgment and gives junior engineers nothing to learn from — they're now building human-plan-review paths where juniors defend their plans against the hospital's agents.

## Critical take

This is the most valuable factory write-up in the wiki because it publishes both sides of the ledger. The Db2 headline (164x faster, 38x cheaper) is an experiment without human review — the authors admit the follow-up work is *checking whether it's even correct* — so the honest comparison is not "two days vs nine months" but "two days plus unbounded verification risk vs nine months of known quality." Where the post genuinely delivers is in the gates: plan-review-before-code (rejected <10% but catching the highest-value bugs), the discharge audit, the precedent log that cut human escalations, and the circuit-breaker for review loops. The hospital metaphor is doing real engineering work — I-PASS handoffs and differential diagnosis are imported structures the models already understand from training data, which is why the roles need so little scaffolding. The 23%-redundant skill corpus is the quiet warning: governance written in prose accretes, and nobody audits the auditors' instructions until the bill arrives.

---

*Sources: [[raw/experiment-running-hospital-code]], [[summary/experiment-running-hospital-code]]*
*Last updated: 2026-10-24*
