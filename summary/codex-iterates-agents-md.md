---
url: https://www.stet.sh/blog/how-i-used-codex-to-improve-its-own-agents-md
title: "I had Codex iterate on its own AGENTS.md 8 times and measured each version against real PRs"
author: Stet (stet.sh)
date_fetched: 2026-05-31
date_published: 2026-05-27
topics:
  - agent-coding-workflow
  - guardrails-and-feedback-loops
---

# I had Codex iterate on its own AGENTS.md 8 times and measured each version against real PRs

Stet tasked Codex with improving its own AGENTS.md, but with a benchmark measuring each change against real PRs rather than relying on intuition. The central argument: AGENTS.md files "are part of the runtime behavior of your coding system" and should be treated as tunable harness components.

## Results Summary

Eight candidate runs evaluated. The most promising version on a 5-task training slice fixed a previously missed task, improved footprint risk, and raised several craft scores. However, on a 10-task holdout, it regressed — footprint widened, tokens climbed, tool calls increased, and code-review correctness dropped, even though tests held steady.

The author notes this is directional, not statistically significant: one repo, n=10 on the holdout.

### Decision Deltas Table

| Signal | Iteration 7 (n=5 training) | Clean Holdout (n=10) |
|---|---|---|
| Tests | +20pp | flat |
| Equivalence | 0 wins, 1 loss | 1 win, 0 losses |
| Review overall | +0.10 | +0.05 |
| Correctness | flat | –0.20 |
| Scope discipline | –0.28 | –0.49 |
| Footprint risk | –0.10 | +0.039 |
| Token use | In –10.4M, Out –24.6K | In +17.8M, Out +23.6K |
| Tool calls (captured) | partial –33.6 avg | +12.8 avg |

The pattern: the candidate agent **did more work for mixed outcomes** — better on local craft (clearer names, coherent implementations), worse on boundary judgment (scope, minimality, robustness). The hypothesis that "better instructions make the agent cheaper" failed the holdout.

## Methodology

- **Setup:** Codex with `gpt-5.5`, medium reasoning, on real historical Stet tasks (dogfooding)
- **Grader:** `gpt-5.4`
- **Metrics:** tests, strict publishability, equivalence, code review, footprint, tokens in/out, duration, and craft/discipline rubrics (simplicity, coherence, robustness, instruction adherence, scope discipline, diff minimality)
- **Process:** 8 iterations on n=5 sample set, then a clean n=10 holdout

## Key Discoveries

### Plausible ≠ good
The first attempt introduced a broad router rule requiring the agent to identify work type, state a hypothesis before editing, read relevant docs, and treat scope as correctness. It sounded reasonable but created a failure mode where interpreting "small scope" as permission to skip named obligations.

### The obligation ledger
The next candidate required the agent to, before editing, identify named behavior, compatibility constraints, docs, tests, and non-goals — then mark each as met, missed, or not checked before reporting back. This was the first useful signal.

> Review improved by +0.75, correctness by +0.60, maintainability by +1.00, simplicity by +0.64, coherence by +0.60, and scope discipline by +0.36. Tests stayed flat at 5/5.

But footprint risk got slightly worse. On vibes alone, the author might have shipped it — but the eval showed it was directional, not a clean win.

### Philosophically right, empirically bad
Codex tested a rule preferring existing helpers, schemas, and public contracts before adding new machinery. Tests still passed, but simplicity, coherence, robustness, clarity, instruction adherence, scope discipline, intentionality, and diff minimality all declined. A narrower version also failed.

### The best candidate
The winning rule: identify the obligation, identify the owner of the change, identify the validation path, then edit. On the 5-task slice, it recovered the missed task, improved footprint risk from 0.41 to 0.31, and improved simplicity, coherence, and diff minimality.

But instruction adherence dropped by –0.56, scope discipline by –0.28. Token usage actually improved.

### AGENTS.md inversion
An instruction change can improve most tasks while degrading a measurable subset. The average can go up while specific task types regress. This makes shared instruction files risky to edit by feel.

> "The failure mode isn't simply 'everything gets worse' — it's 'enough gets better that you miss the damage.'"

### Tighter process backfires
The next candidate required exact owner file/function and a validation command before editing. Tests stayed green, but code review overall dropped by –0.30, correctness by –0.40, coherence by –0.38, and simplicity by –0.10. More process did not mean more discipline.

## Holdout Results

On the clean 10-task holdout, the candidate didn't collapse. Tests tied 10/10. But sub-scores split: maintainability improved +0.30, edge-case handling +0.10, while correctness fell –0.20.

## Tracing the Regression

The regression was systematic: the new AGENTS.md made the agent better at coherent local implementation (clearer names, explicit status/report fields, structured logs, targeted tests). The regression was in boundary judgment:
- Narrowed a broad request to the subcommand it understood
- Documented behavior more broadly than it implemented
- Added parallel metadata/reporting contracts instead of extending existing ones

> "The new instructions helped when the right boundary was obvious, and hurt when the task required judgment about how wide the boundary should be."

## Where the Author Landed

A **bounded loop** for instruction changes:
1. Write hypothesis (candidate AGENTS.md rule)
2. Test on real work (n=5 historical tasks)
3. Inspect failures (quality split, not test failure)
4. Revise rule (obligation, owner, validation)
5. Run holdout (clean n=10 slice)
6. Validate claim (do not ship on mixed evidence)

The author identifies the next rule needed: before expanding docs, adding a new contract, or touching adjacent flows, the agent should prove that breadth is required by the task.

## Five Questions for AGENTS.md Maintainers

1. What behavior should this rule change?
2. Which real tasks should expose that behavior?
3. Does it improve behavior, or only vibes?
4. What did it make worse?
5. Did the holdout agree?

## Disclosure

The author is building **Stet.sh**, the local eval tool used in the post. The product version allows users to ask their coding agent to improve its own setup (AGENTS.md, skills, harness config, reasoning settings) while Stet measures candidate changes against historical repo tasks. It runs entirely locally using the user's own LLM subscriptions.
