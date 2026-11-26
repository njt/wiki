# Agentic Algorithm Engineering (AAE)

AAE, from the Algorithm Engineering Group at Heidelberg University, is a Claude Code/Codex skill that gives an autonomous coding agent the discipline of experimental algorithmics: falsifiable hypothesis before every experiment, correctness assertions checked after every change, keep-or-revert against a benchmark, one Git patch per accepted change, and a PDF report per session. The results table is the argument — VieCut 8.9x faster with identical cuts, KaHIP 3.9x faster with unchanged partition quality, byte-identical distributed partitions from ParHIP.

---

## Key quotes

> "A falsifiable hypothesis: 'changing X will improve the metric by roughly Z, because ...'"

Predicting direction *and size* before measuring is the interesting part — it turns the agent's experiments into something a scientist (or a reviewer) can grade after the fact. An agent that must commit to a prediction can be wrong, and wrong predictions become signal.

> "Run the benchmark; keep the change only if it is better and still correct, otherwise revert."

The loop is deliberately dumb. No taste, no judgment call at the merge boundary — a single rule, applied mechanically. This is why it can run unattended: the correctness criterion is agreed with the human at setup, then enforced without the human.

> "Sessions share what they learned: the knowledge base collects optimization techniques and experiment results, and agents read it before they start."

Cross-session memory as infrastructure, not an afterthought — the same instinct behind compounding-workflow systems, here applied to a narrow, measurable domain where lessons can actually be validated.

> Comparison table against AlphaEvolve/OpenEvolve/ShinkaEvolve: "one agent following the Algorithm Engineering cycle ... assertions agreed with you, checked after every change."

The differentiator is *existing codebases and languages* versus evolutionary search over a scored script. AAE optimises real production code under human-agreed invariants; the autoresearch systems evolve artefacts against an evaluator.

## Themes

#concept #tool #pattern

## Analysis

The notable thing about AAE is what it *removes* from the agent, not what it adds. Most agentic-coding rhetoric hands the model more autonomy; AAE hands it less — one keep-or-revert rule, externally checkable correctness, mandatory prediction before measurement — and gets 8.9x on real published algorithms as a result. It is the strongest small demonstration I have seen that the domain with tight, honest benchmarks is exactly where agent autonomy is already safe and productive, and that the honest benchmark is doing nearly all of the work.

The falsifiable hypothesis is more than decoration. It converts the agent run into a falsifiable record: a patch with its predicted effect, its measured effect, and its correctness evidence. That is a reviewable unit in a way most agent PRs are not, and it connects directly to the verification-bottleneck literature — when checking is cheap and structured, the human review step shrinks to reading hypotheses and assertions, not diffs. It also disciplines the model against the classic optimisation failure of silently moving the goalposts: the metric and assertions are frozen at setup.

The honest caveat: algorithm engineering is the easiest possible case. Graph-algorithms codebases have bit-identical oracles and mature benchmark suites. The hard question — which AAE does not answer — is whether the same loop survives in a domain where correctness assertions are expensive to write or the oracle is fuzzy. My guess is it degrades to ordinary agent-coding quality quickly, which is precisely why publishing the loop as a skill for *any* codebase is a bolder claim than the results table admits.

## Relations

- [[Agentic Verification]] — Osmani's thesis that checking is the new bottleneck is strengthened here: AAE is a working instance of a verification loop the agent runs itself, with the oracle agreed up front.
- [[We Should Be More Ambitious (Dan Luu)]] — Luu's claim that ambition is safe only where self-checking verification exists gets a concrete existence proof: verified 8.9x speedups on published algorithms.
- [[Outerloop — Open and Trustworthy Multi-Agent Autoresearch]] — complicates the autoresearch comparison: Outerloop builds multi-agent research infrastructure while AAE deliberately stays single-agent with a dumb keep-or-revert rule, and wins on production codebases.
- [[Claude Code Skills System]] — a concrete instance of the skills architecture carrying a full scientific method: benchmark, metric, assertions, budget, knowledge base, all under SKILL.md discipline.

---
*Sources: [[raw/agenticalgorithmengineering]], [[summary/agenticalgorithmengineering]]*
*Last updated: 2026-11-26*
