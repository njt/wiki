# Ratchets in Software Development

A simple lint-time script that counts instances of deprecated code patterns and errors if the count goes up (preventing proliferation) or down (prompting you to lower the baseline). It's the cheapest possible mechanism for turning a team decision ("we don't use this pattern anymore") into enforceable policy — no parser, no AST, just string matching and hard-coded expected counts.

---

## Key Quotes

> "In our codebase, there are 'patterns' which we used to use all the time, but we decided to stop using them, but removing all of the existing instances at once is too much work."

qntm names the universal codebase problem: every team has deprecated patterns they can't afford to fix wholesale. The ratchet is the tool that stops the bleeding while the wound heals on its own timescale.

> "What this technique does is automate what was previously a manual process of me saying 'don't do this, we've stopped doing this' in code review. Or forgetting to say it. Or missing the changes entirely, due to the newcomer having the audacity to request their review from someone else."

The real value isn't technical — it's social. The ratchet removes the human bottleneck from code review enforcement. You don't need the one senior engineer who remembers the decision to be on every PR. The script remembers.

> "This script is intentionally extremely simple. The expected numbers are hard-coded in the script itself."

Not a bug. The simplicity is the design. A sophisticated tool would invite endless improvement; a dumb script that works can be left alone. qntm explicitly warns against the trap of "divert[ing] a huge amount of pointless time and energy into maintaining and improving a 'simple' tool of this kind."

> "It would be easy to abuse this technique to enforce unnecessarily strict 'standards' on a development team who really ought to be allowed some creative freedom. Sometimes it's okay to say 'No' to adding a new rule."

The ratchet is power, and power needs restraint. Every new rule is a permanent tax on every future change. The meta-skill is knowing which patterns are worth enforcing and which are just taste.

## Key Themes

#tool #pattern #enforcement #linting #technical-debt #determinism

The ratchet is a **tool** implementing a **pattern** for deterministic **enforcement** at **linting** time, targeting **technical-debt** reduction through **determinism** rather than persuasion.

## Critical Analysis

**This is the smallest viable guardrail.** The ratchet is gloriously dumb — it's `grep` with a counter and a hard-coded threshold. No ML, no AST parsing, no config file. The dumbness is the feature. Every line of sophistication you add creates a maintenance burden that competes with the problem you were trying to solve. This is the same insight behind [[claude-ctrl]]'s "an instruction in context is not a constraint" — enforce at the mechanism level, not the persuasion level.

**The real insight is about memory, not enforcement.** The ratchet solves an organizational knowledge problem: "how do we make sure every developer, including the ones who weren't in the meeting, follows a decision made six months ago?" Code review is lossy human memory. The ratchet is lossless mechanical memory. This connects to [[Learn from PRs Skill]] — both automate what was previously a human remembering to enforce a rule. But the ratchet is cheaper: it doesn't need an LLM, just `grep`.

**The ratchet is technically a linter, but the name matters.** Calling it a "ratchet" makes the function legible in a way "custom lint rule" doesn't. A ratchet only moves one direction. The name encodes the contract. This is a [[Prefix Effects]] case study — naming shapes how the tool is understood and used.

**The refusal to share specifics is telling.** qntm says "the specifics of our ratchet script (whose content, no, I will not be sharing) are less important here than the generic technique." The forbidden methods are context-dependent; publishing them would distract from the pattern. Good technical writing discipline — show the mechanism, not your particular grudge against `Array.prototype.concat`.

**The edge-case handling is a masterclass in pragmatism.** String matching means false positives from comments and string literals. The answer: "[shrug] ... I guess we'd just raise the ratchet by 1." This is the engineering equivalent of "don't let perfect be the enemy of good." A parse-tree-based solution would be correct but never get built. A string-match solution is wrong in predictable, correctable ways — exactly what [[Elements of Code]] prescribes.

**The ratchet doesn't solve the removal problem.** It prevents new instances but doesn't actively reduce old ones. qntm acknowledges this and treats it as a separate problem. This is honest scoping — the ratchet is a stop-loss mechanism, not a cleanup tool. The cleanup happens when someone touches that code for other reasons, or never. Accepting that some technical debt will die of old age rather than surgery is realistic.

**This is the ur-pattern behind half the agentic guardrails in this wiki.** [[Feedback Loop is All You Need]], [[claude-ctrl]], [[Pre-Commit Lint Checks]], [[Harness Engineering]] — they're all elaborations on the ratchet principle: count, compare, enforce. The sophistication varies but the core mechanism doesn't. The ratchet is to programmatic enforcement what `diff` is to version control: so simple it's easy to forget it had to be invented.

**The meta-point about tool maintenance is the most important paragraph.** qntm writes: "it would be incredibly easy to divert a huge amount of pointless time and energy into maintaining and improving a 'simple' tool of this kind." This is the trap [[Compound Engineering]] warns about from the other direction — the system that was supposed to improve productivity becomes the thing that consumes it. The ratchet works because nobody is allowed to make it fancy.

---

Related: [[Guardrails and Feedback Loops]], [[Feedback Loop is All You Need]], [[claude-ctrl]], [[Pre-Commit Lint Checks]], [[Harness Engineering]], [[Learn from PRs Skill]], [[Compound Engineering]], [[Awesome Agentic Patterns]], [[Prefix Effects]], [[Elements of Code]]

*Sources: [[summary/ratchet]]*
*Last updated: 2026-05-15*
