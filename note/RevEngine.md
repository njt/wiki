# RevEngine

Laude's open-source GitHub Action for AI pull-request review — a minimal harness (read-only repo access, a recheck pass that drops unsupported findings, severity/confidence ranking) that topped GitHub's ReviewBench at 59.6% F1 and, more interestingly, ablated the model-vs-harness question: the model sets the baseline, the harness adds thoroughness, and a cheap Sonnet under a good harness nearly matches Opus.

---

## What the tool is

The origin story is unusually concrete: after Laude open-sourced their agent harness Headlong, outside PRs flooded in, and full manual review of each one — did it do what it claimed, did it advance the project's goals, was it safe — would not scale for a small maintainer team. They tried Claude Code and Codex skills first, then a bespoke orchestration repo, then consolidated into the RevEngine GitHub Action. That progression — skills, then tooling, then product — is itself a nice micro-history of how ad-hoc agent setups harden into infrastructure.

The pipeline: strong model with read-only access reviews the diff with surrounding code plus the project's rules and design docs; a second pass rechecks every finding against context and drops incorrect or unsupported ones; deduplicate; rank by severity and confidence. ~3 minutes and ~$1 per PR on Opus 5.5.

## Key quotes

> "We concluded that a minimal harness and a strong model would get us quite far (a throughline in much of our work) and give us much more flexibility when it came to a repo's particular needs."

The repeating Laude thesis, applied to review rather than coding. It aligns exactly with the empirical finding in [[harnesseval — Harness vs Prompt for AI Code Review]], where the workflow beat model choice across 42 comparisons — but Laude's ablations complicate harnesseval's strong version: here the *strong model* sets the baseline and the harness adds thoroughness on top, not the reverse.

> "A reviewer can be stricter when an agent opens a PR (but watch out for unbounded nit spirals)... coding agents don't care whether they fix two things or ten."

The sharpest observation in the piece, and one almost no commercial review tool has internalised: the economics of strictness flip when the author is a machine. Human-worn-down-by-nits is the binding constraint the whole PR-review industry is built around, and it dissolves asymmetrically. The honest caveat about benchmark tension — tools that suppress low-severity findings to spare humans score worse on ReviewBench — is a rare admission that the benchmark optimises for thoroughness, not triage cost.

> "These documents let an automated reviewer ask whether a change is correct *and* whether it belongs in the codebase."

The fit-vs-own distinction is precisely what makes review of agent-generated code tractable, and it lands in the same argument as [[Coding Agents Love Decision Records]] and the steering-file material: write rules down in the repo, because the agent will read them every time even when a human reviewer wouldn't.

> "Our experiments suggest that a strong model sets baseline performance, while the harness makes reviews more thorough."

Their ablation design is the valuable part: naive single-call baselines vs the full harness across four models. Harness gains scaled *with* model capability (Opus +11-14, Haiku +8, DeepSeek +5), and Sonnet nearly matched Opus under the harness at half the cost — a frontier-model-choice result that reinforces [[Managing AI Coding Costs at Scale]]'s efficiency-frontier logic, with the wrinkle that Opus retained a real lead on high-severity findings (55% vs 48%).

## Opinionated take

The leaderboard headline is the least interesting fact here. A new entrant topping a benchmark by 12 F1 points is the normal rhythm of 2026; what makes this piece worth keeping is that Laude actually ran the ablation and reported it. The naive-baseline methodology (one model call, title+description+diff only, no verification) is exactly the control most review-tool marketing skips, and it cleanly separates "we have Opus" from "our harness does something." It also quietly contradicts harnesseval's headline claim that model choice barely matters: in Laude's data, model choice sets a large baseline delta, and the harness is a multiplicative thoroughness gain that favors strong models. Both can be true — harnesseval held effort constant and used small sets — but the tension is worth flagging rather than papering over.

The honest limitations are also a credit: they admit the real test is whether RevEngine reduces manual review time without destabilising their projects, and they're collecting that data now. Benchmark F1 at $1/3min is not the same thing as a calmer maintainer.

The caveat nobody in this space can escape: precision of 46% means more than half of reported findings are not real issues, and "review is cheap compared with attention" only holds if someone still pays the attention to triage false positives. RevEngine's own second-pass recheck exists because unfiltered model findings aren't trustworthy — which means the tool's economics quietly depend on how expensive the human discard step is.

## Themes

#tool #concept #pattern

## Related pages

- [[harnesseval — Harness vs Prompt for AI Code Review]] — the closest existing page: this source strengthens its harness-over-prompt thesis while complicating its model-doesn't-matter corollary, since Laude's ablation shows model choice setting the baseline and harness gains scaling with capability.
- [[Orchestrating AI Code Review at Scale]] — Cloudflare's production system shares the multi-pass, drop-unsupported-findings architecture and the ~$1/review economics, but at 131K-reviews scale with seven specialists; RevEngine is the single-repo minimal-harness version of the same problem.
- [[The End of Code Review]] — Monperrus's argument that human review can't scale against agent code volume gets a concrete maintainer-flood origin story here, plus the explicit human-on-risky-parts carve-out that "keep a person on the dangerous parts" represents.
- [[Agentic Code Review]] — Osmani's field guide's claim that the bottleneck shifted from writing code to trusting it is the framing RevEngine was built against, and its "reviewer pickiness matched to context" practice is Laude's severity-threshold idea stated as a general principle.

---
*Sources: [[raw/revengine]], [[summary/revengine]]*
*Last updated: 2026-10-21*
