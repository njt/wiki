# Agent Distraction (Google)

Google's quantified case that stale codebase artifacts — obsolete `TODO(bug/...)` comments above all — actively misdirect coding agents, with regression-backed numbers, two failure archetypes, and an agentic cleanup pipeline that has been deleting confirmed-dead TODOs at >95% human-review acceptance.

---

The insight is refreshingly simple: everything a human skims past is, to an agent, an instruction. TODOs, resolved bugs, stale docs are all ingested as context during the planning phase, and the agent has no way to know they're dead. As agents gain tools and skills, "the ability for side-quests scales proportionally." Google's clever move is measurement: the TODO-with-bug-reference is a perfect proxy because the bug tracker is the canonical record — cross-reference the TODO against bug status and the user's prompt, and you can mechanically classify agent behavior as valid work or wasted detour at a scale no human review could reach.

The methodology deserves attention as much as the finding: multi-way fixed-effects regression with developer-clustered standard errors, controlling for developer, codebase complexity, task type, model tier, files read, prompt length, and the step where the reference appeared. Unprompted bug detours are significant at p<0.0001. This is productivity research done the way epidemiology is done, not the usual vibes-and-anecdotes.

## Key quotes

> What is easy for humans to skim past in code, agents see as a call to action.

The thesis in one line. The agent's strength — taking every textual cue seriously — becomes a liability exactly where human readers have learned to filter.

> This indirection is not free: in the best case it only costs tokens, steps and time - in the worst case it distracts and grounds the agent on falsehoods leading to incorrect logic and decisions during implementation.

The cost taxonomy: worst case isn't slowness, it's being *grounded on falsehoods* — stale context actively corrupting implementation decisions.

> Projects that once were on the backburner to cleanup lingering debt but deprioritized due to perceived low value need to be reevaluated with the new agents acting at scale on your codebase.

The economic punchline: the ROI calculation on tech-debt cleanup has changed because the reader of the codebase changed.

## Key themes

#concept — distraction as a measurable tax on agent sessions
#pattern — TODO/bug-tracker cross-referencing as an objective behavioral signal
#tool — the agentic TODO cleaner: corpus scan → historical confirmation → deletion PR

## Analysis

What's most interesting here is the reframing of codebase hygiene. The classic argument for paying down debt is future human readers; Google's data adds a second, more mechanically impatient reader that never learned to skim. This dovetails with the SonarSource finding in [[Code Cleanliness and Coding Agents]] — cleaner code doesn't raise agent pass rates, it lowers cost and wandering — but Google goes further: it's not just messiness, it's *falsity*. A stale TODO isn't noise, it's a false premise the agent will act on.

The "eager agent" archetype is the sharper of the two failure cases. An agent that unpromptedly fixes unrelated code isn't just wasting tokens — it's committing scope creep that a human has to detect and challenge, which is a review burden, not merely a compute cost. That makes distraction a [[Agentic Code Review]] problem too, not just a context problem.

The remediation pipeline is the honest part of the story: they didn't just bulk-delete TODOs matching closed bugs ("load-bearing" TODOs exist), they had agents verify intent across years of bug, file, and commit history before proposing deletion, then let humans review. That's the same "deterministic where possible, judgment where necessary" tiering seen in [[Uber AutoClean — Agentic Tech Debt Cleanup]], which automates the sibling debt species of stale feature flags and reports the same self-reinforcing economics: agents tax themselves on dead code, so agentic cleanup is now infrastructure. Google's >95% acceptance rate on thousands of deletions is the strongest public evidence yet that this class of agent can be trusted with deletion — the operation engineers fear most — when the verification loop is layered enough.

One mild skepticism: the post never states the magnitude of the detour cost, only that it's "substantial" and statistically significant. With hundreds of millions of steps, tiny effects reach p<0.0001; a per-session overhead number would have made this much harder to argue with. Still, the direction is clear and the two trajectories shown are convincing enough qualitatively.

## Related pages

- Strengthens [[Uber AutoClean — Agentic Tech Debt Cleanup]] with an independent, better-measured data point that the same debt species (dead markers in code) taxes agents, and the same cleanup-as-infrastructure conclusion.
- Nuances [[Code Cleanliness and Coding Agents]]: that study found cleanliness changes cost but not correctness; Google's stale-TODO case is the stronger variant where the artifact doesn't just cost tokens but grounds the agent on falsehoods and corrupts decisions.
- Complicates [[Code Standards That Survive the Next Agent]]: standards written for agents must now include *deleting* legacy markers, not just shaping new code — the agent-facing codebase is a moving corpus that decays.
- Extends [[The Wicked Reason Removing Code Beats Better Scheduling]] with an empirical, agent-side justification for deletion-first prioritization.

---
*Sources: [[raw/agent-distraction]], [[summary/agent-distraction]]*
*Last updated: 2026-11-11*
