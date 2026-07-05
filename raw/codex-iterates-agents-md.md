---
url: https://www.stet.sh/blog/how-i-used-codex-to-improve-its-own-agents-md
date_fetched: 2026-07-05
backfilled: true
---

# I had Codex iterate on its own AGENTS.md 8 times and measured each version against real PRs. The best one still regressed on a clean holdout.

I have a confession: I vibe-coded my `AGENTS.md`, and I'm pretty sure it's slop.

I needed to make it better. Naturally, I asked Codex to do it.

The difference: this time, Codex used a benchmark on my repo to measure each change, and optimized `AGENTS.md` against the data, instead of on pure vibes.

First scored result within the hour, on the Claude subscription you already have.

Join the waitlist## Why We Should Take AGENTS.md Seriously

Saying "`AGENTS.md` is important" is, at this point, a cliche. At risk of beating a dead horse, I'll say it again.

Someone adds a rule that sounds smart, senior, and reasonable, commits it, and hopes the agent behaves better. But `AGENTS.md`, `CLAUDE.md`, and shared skills are not normal docs. They are part of the runtime behavior of your coding system.

**The shift is to start treating  AGENTS.md like a tunable part of the harness:** holding everything else the same, how does agent behavior differ when I change 

`AGENTS.md`? That's what I measured.## The Results

After eight candidate runs, one version looked useful on a five-task training slice. It fixed the task the baseline missed, improved footprint risk, and moved several craft scores up.

Then I ran it on a clean ten-task holdout. The candidate regressed. Not catastrophically, but enough that blindly shipping would have been wrong. Footprint widened, tokens climbed, tool calls climbed, and code-review correctness fell, all while tests held even.

*Caveat: one repo (mine), n=10 on the holdout. This is directional, not statistically significant.*

*For this post, "equivalent" means the patch matched the intent of the merged human PR; "code-review pass" means an AI reviewer judged it acceptable; craft/discipline is a 0-4 maintainability/style rubric; footprint risk is how much extra code the agent touched relative to the human patch.*

The pattern is the agent doing more work for mixed outcomes - better on local craft (clearer names, coherent implementations), worse on boundary judgment (scope, minimality, robustness). Tokens and tool calls confirm it: the candidate was spending more to get there, not less. "Better instructions make the agent cheaper" did not hold on the holdout.

## Methodology

The setup was Codex with `gpt-5.5`, medium reasoning, on real historical Stet tasks (dogfooding). Stet scored tests, strict publishability, equivalence, code review, footprint, total input/output tokens, duration, and craft/discipline rubrics like simplicity, coherence, robustness, instruction adherence, scope discipline, and diff minimality. The grader was `gpt-5.4`.

8 iterations on an n=5 sample set, and a n=10 task holdout.

**I know sample size is small - the goal of this was to get directional analysis, and prove the methodology**

Codex was set with a simple `/goal`: iterate `AGENTS.md` to improve performance on the benchmark.

## Process

The first round of iteration showed something I wish more people internalized: **plausible instructions are not necessarily good interventions.**

Codex first tried a broad router rule: identify the work type, state a hypothesis before editing, read the right docs, and treat scope as part of correctness. It sounded good but exposed a failure mode: the agent could interpret "small scope" as permission to miss named obligations.

The next candidate added an "obligation ledger". Before editing, the agent had to identify the named behavior, compatibility constraints, docs, tests, and non-goals. Before reporting back, it had to mark each as met, missed, or not checked.

Here is the actual diff shape.

That obligation-ledger candidate was the first useful signal. Code review improved by `+0.75`, correctness by `+0.60`, maintainability by `+1.00`, simplicity by `+0.64`, coherence by `+0.60`, and scope discipline by `+0.36`. Tests stayed flat at 5/5. But footprint risk got slightly worse, and the evidence was still a small same-sample read.

If I were editing by vibes, I might have shipped it. The eval said: useful direction, not a clean win, keep iterating.

Codex then tested the kind of rule that intuitively makes sense: prefer existing helpers, schemas, reporting paths, and public contracts before adding new machinery. It sounded correct - and the eval hated it. Tests still passed, which is exactly why tests alone are not enough for this kind of change, but simplicity, coherence, robustness, clarity, instruction adherence, scope discipline, intentionality, and diff minimality all moved down.

The rule was philosophically right and empirically bad (exactly why measurement is important!).

Codex tried a narrower version: extend the owning surface instead of creating adjacent machinery. That also failed. Review quality, correctness, scope discipline, duration, footprint, and token use all got worse.

So the loop rolled back toward the obligation-ledger idea. The best candidate from that first pass was simply a small process rule that made the task contract harder to forget.

Codex ran three more candidates. The next run was easy to reject: tests and strict publishability fell from 5/5 to 4/5, footprint risk got worse, and simplicity dropped by `-0.64`.

The next candidate was the best one. It made the obligation rule more concrete: identify the obligation, identify the owner of the change, identify the validation path, then edit. On the same five-task slice, it fixed the one task the baseline missed, recovering tests and strict publishability from 4/5 to 5/5. Footprint risk improved from `0.41` to `0.31`. Simplicity improved by `+0.40`, coherence by `+0.44`, diff minimality by `+0.30`, and code review overall by `+0.10`.

That sounds like a win.

It still was not promotion-grade. Instruction adherence dropped by `-0.56`. Scope discipline dropped by `-0.28`. The candidate was better in several ways that matter, but worse in others that also matter.

The token story was useful because it was not obvious from patch quality alone. On that run, the candidate used fewer total input tokens and fewer output tokens than baseline: input tokens fell from `33.9M` to `23.5M`, and output tokens fell from `85.3K` to `60.7K`. The shipping decision still came down to quality tradeoffs, not token totals.

This is an example of

AGENTS.md inversion: an instruction change can help most tasks while hurting a measurable subset. The average can improve while specific task types regress. That makes shared instruction files dangerous to edit by feel, because the failure mode isn't simply "everything gets worse." - it's "enough gets better that you miss the damage.".

This is especially bad on shared codebases, where the tasks

youdo may get better, but the tasks your coworkers do get worse without them noticing anything changed.

After that, Codex tried tightening the rule even more. The next candidate required an exact owner file/function and validation command before editing. Again, it sounded better. Again, it was worse. Tests stayed green, but code review overall dropped by `-0.30`, correctness by `-0.40`, coherence by `-0.38`, and simplicity by `-0.10`. More process was not automatically more discipline. Sometimes it was just more ceremony.

Finally, after enough iteration attempts, Codex ran the iteration 7 candidate against a larger clean holdout. This is where the story gets less satisfying, and more real.

On those ten tasks, the candidate did not collapse. Tests tied at 10/10. Strict publishability tied. Equivalence was directionally favorable: one candidate win, zero losses, nine ties. Code review fail/pass still tied, but the sub-scores split: maintainability improved by `+0.30`, edge-case handling by `+0.10`, overall review by `+0.05`, while correctness fell by `-0.20`.

## Tracing Behavior

The trace analysis showed where the regression came from. The candidate wasn't worse in a noisy way - it was systematically making different choices than the baseline, and those choices mapped directly onto the signal drops.

The new `AGENTS.md` made the agent better at producing a coherent local implementation story. It used clearer names, more explicit status/report fields, more structured logs, and more targeted tests around the behavior it chose to implement. That lines up with the gains in coherence, clarity, and slight simplicity.

The regression was in boundary judgment. On several tasks, the candidate narrowed a broad request to the subcommand it understood, documented behavior more broadly than it implemented, or added a parallel metadata/reporting contract instead of extending the existing one. Those three patterns directly produced the losses in scope discipline, diff minimality, robustness, intentionality, and instruction adherence.

Getting into specific examples:

One task asked for durable operator records across evaluation and replay command flows. The candidate produced a cleaner implementation with better names and tests, but reframed the broader eval/replay request into a narrower rules-specific change. Another task asked for grader-configuration provenance in manifest and planning flows; the candidate expanded into runtime artifact plumbing too. The code was often easier to read, but the solution was sometimes less faithful to the original task.

There was one useful counterexample. On a manifest-resolution task, the candidate really did better: fewer steps, tighter scope, and better craft scores. The new instructions helped when the right boundary was obvious, and hurt when the task required judgment about how wide the boundary should be.

## Where I Landed

**The conclusion is: Codex found a promising instruction change, Stet showed exactly where it helped, then Stet stopped me from claiming it was safe to ship.**

That is the version of self-improving agents I currently trust. Not a model recursively making itself smarter in a void, but instead a bounded loop:

write a hypothesis -> test it on real work -> inspect the failures -> revise the rule -> run a holdout -> validate the claim.

The mental model for this is a production rollout: a change can pass CI, pass e2es, and still break something for a customer in prod. That's why we monitor prod rollouts, and take regressions seriously.

On a shared codebase, the failure doesn't announce itself. The engineer who committed the AGENTS.md change sees improvement. The engineers downstream don't know the instructions changed, and nobody files a bug because the agent still passes tests, still ships patches, still looks fine in review. The regression is in aggregate behavior across a task distribution nobody measured.

The most useful candidate from this loop is still useful. It tells the agent to keep named obligations, ownership, and validation in view before editing. But the next version likely needs a new rule: before expanding docs, adding a new contract, or touching adjacent flows, the agent should prove that breadth is required by the task. That's likely the next thing Codex test in my quest to improve `AGENTS.md`.

## Takeaway

If you maintain a shared `AGENTS.md`, `CLAUDE.md`, or internal agent skill, I would ask:

- What behavior should this rule change?
- Which real tasks should expose that behavior?
- Does it improve behavior, or only vibes?
- What did it make worse?
- Did the holdout agree?

The important part is measuring and iterating. I don't think anyone can claim to know model behavior well enough to one-shot a perfect `AGENTS.md`.

Going forward, the difference between AI-native teams, and teams using AI, is not only usage patterns, but how they measure and shape shared-context changes.

*Disclosure: I am building Stet.sh, the local eval tool I used to run this. The product version is exactly what this post shows - you can ask your coding agent to improve its own setup ( AGENTS.md, skills, harness config, reasoning settings) and Stet measures candidate changes against historical repo tasks. Stet runs entirely locally, using your LLM subscriptions. You can now install it and run a first local trial from https://www.stet.sh/private, or reach out to me directly for paid pilots and CI setup.*

First scored result within the hour, on the Claude subscription you already have.

Join the waitlistHow are people here handling shared `AGENTS.md` / `CLAUDE.md` changes today? Are you measuring before committing, or shipping on vibes?
