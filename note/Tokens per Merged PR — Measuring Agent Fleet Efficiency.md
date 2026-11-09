# Tokens per Merged PR — Measuring Agent Fleet Efficiency

Alex Marquardt's ten-week accounting of a Claude Code agent fleet (up to a dozen background agents, 590–870 merged PRs/week, four repos) measured from the transcripts Claude Code already keeps on disk. Tokens per changed line fell 3.3x, tokens per merged PR 5.5x, and throughput rose 4.4x on roughly flat spend — not by making the model cheaper or smarter, but by attacking the two factors the token total is proportional to: number of calls, and context carried per call.

---

The key realisation is that agent token use is dominated by what agents re-read, not what they write. The model keeps nothing between calls, so every call re-sends the whole conversation; cache reads are 94–98% of this fleet's tokens. Marquardt's six changes all reduce calls or context: one task per agent with a hook-enforced 150k ceiling (median context per request fell 218k → 101k in a day), keeping the prompt cache warm (idle expiry was a shocking 31% of agent usage; now 0.7%), turning rules into code, replacing the AI coordinator with a small program, and making the codebase cheap for an agent to navigate.

The method matters as much as the results: measure from `~/.claude/projects/` transcripts, count each request once (his first script over-counted by a third), and divide by **work done, not time** — tokens per hour didn't move when the ceiling landed; merged PRs per hour went 0.57 → 2.06.

## Key quotes

> **"Tokens used are proportional to the number of calls times the average context per call."**

The whole article is this one formula being optimised from both sides. Every improvement is either fewer calls or less context per call; everything else (shorter briefs at 0.6% of use) was noise.

> **"Don't make the model remember; make the system remember."**

The unifying principle, and the reason it rhymes with deterministic-gate work elsewhere: state lives in files and GitHub, rules live in gates and scripts, and only then can an agent safely forget everything between tasks.

> **"An agent is a new developer every time it starts."**

The sharpest justification for the "make the codebase easy" change. Code maps, pointer comments, file caps and one-home-per-concept are human-onboarding practices rediscovered as token economics.

> **"The waste was not irrelevant content; it was relevant content carried across hundreds of calls."**

Counterintuitive and important: only 1.5–3% of carried context was never referenced again. Compaction-and-prune thinking targets the wrong thing — the fix is shorter lives, not better filtering.

## What did not pay

The negative-results table is rare and valuable: shorter briefs never mattered; blind output caps cut text the agent needed half the time; a mechanical pre-review gate found 71% of blocking findings but would save only ~3% of tokens. And a read-only audit by a strong model found the 31% cache lever that the author's own tuning had missed for weeks — measurement by a second model beat intuition.

## Honest deterioration

Output per changed line actually **rose** 1.8x (fixed per-task costs amortised over smaller PRs), rework per PR rose from 0.8 to ~1 fix round (~20% of agent tokens), reverts ticked up, and the fixed first-turn context grew from 9.9k to 14k tokens. Tokens per PR fell; quality cost per PR did not. That is the next frontier, and Marquardt says so plainly.

---

## Opinionated take

This is the most rigorous public token-efficiency dataset for a running agent fleet, and its discipline — reporting ratios rather than absolute counts, separating confounded changes, publishing what failed — puts most vendor cost posts to shame. Two cautions: it is one project run by one person with changes landing simultaneously (which he flags), and several headline gains are regime-specific to a rule-heavy, high-PR-volume repo where a fleet pays off at all. The single-agent reader should take the five "what to do first" steps, not the fleet machinery.

The deepest point is architectural: the biggest wins came from moving judgment out of the model — a program chooses and launches agents, gates refuse instead of instructing, the handoff replaces compaction. Token efficiency and reliability turned out to be the same project.

## Related pages

- [[Managing AI Coding Costs at Scale]] — treats the same problem from the finance/observability side; this source supplies the per-unit-of-work measurement discipline (tokens per merged PR, not per hour) that cost dashboards usually lack.
- [[The Tokens You Can't Wait For]] — identifies waiting as a token cost category; Marquardt quantifies it precisely (31% of agent usage lost to idle cache expiry, fixed by a 270-second wait cap), turning that essay's intuition into a measured lever.
- [[Auditing a 24-7 Multi-Agent Decompilation Farm]] — the other serious transcript-audit of a running fleet; its finding that duplicated subagent reads dominate complements this article's cache-read dominance, and both conclude the harness, not the model, is the cost lever.
- [[Reducing Token Spend with Deterministic Workflows]] — argues the same "rules into code" move in general terms; this source gives it numbers (coordinator tokens per PR down to 7% of baseline) and a reliability dividend the deterministic-workflow case usually omits.

---
*Sources: [[raw/agent-fleet-token-efficiency]], [[summary/agent-fleet-token-efficiency]]*
*Last updated: 2026-11-09*
