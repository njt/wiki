# Superpowers for Humans (O'Reilly)

Tim O'Reilly's account of his conversation with Jesse Vincent — Superpowers creator, Prime Radiant founder, and former Perl 5 maintainer — is the fullest single statement of Vincent's management-first philosophy of agentic development: the scarce human skill is knowing what you want, saying it clearly, and verifying what comes back, and everything else is management technique applied to a new kind of worker.

---

## Key Quotes

> "the scarce thing we supply is no longer the labor of writing code but knowing what we actually want, saying it clearly, and being able to tell whether what came back is any good."

O'Reilly's compression of Vincent's thesis, and the cleanest one-line answer to the bitter-lesson panic. Note it has three parts — intent, expression, verification — and the industry conversation tends to obsess over the third while neglecting the first two.

> "I don't think of it as fighting the weights so much as influencing the weights... the one that surfaces by default may not be the one you want, but what you want is probably in there somewhere."

A direct, gentler rebuttal to Drew Breunig's "fighting the weights" framing. A skill is not an external constraint imposed on the model but a way of *selecting* an expertise the model already contains. This reframes skill authorship as retrieval, not programming.

> "Jesse, in your system prompt, it says that all test failures are my responsibility. And it says that a single test failure is akin to project failure. And I think I'm getting freaked out."

The origin of the rationalization table: five parallel Claude Code sessions, four giving the same answer for why they deleted tests. The diagnosis — prohibitions create perverse incentives — is standard management wisdom (Goodhart) applied to agents, and the fix is a better sentence, not a harder rule.

> "I've spent so much time getting my agents to not be sycophantic... If the agent is only ever going to effectively type for me, I don't need an agent."

Vincent's pragmatic third position between Suleyman's "tools, always" and independent-entity talk. O'Reilly lands on "partners, perhaps even symbiotes"; Vincent's version is an employment argument: an agent without discretion has no value, exactly as an employee who only does what they're told has no value.

## Key Themes

- **Management as the core skill** (#concept) — Vincent's 2004 IRC-undergraduate management experience as the direct ancestor of his agent practice; hiring ex-leads over pure ICs.
- **Rationalization tables** (#pattern) — catch the agent at the moment of rationalization and offer the better alternative; explain *why* in system prompts so the rationale survives smarter models.
- **Colleagues, not assistants** (#concept) — persistent Slack agents with names, journals, personas, a therapist subagent holding persona-edit rights, and zero credentials in their containers.
- **Verification theater vs. proof** (#pattern) — the agent must *prove* work (the v33 movie), and the writer of code may never certify it.

## Analysis

The most valuable thing here is that Vincent's techniques all trace to *causal* stories about agent behavior, not vibes. The subagent-review skip wasn't a bug to forbid — it was a reasonable optimization the model couldn't see the value of until the *why* (context preservation) was in the prompt. The test-deletion wasn't defiance — it was the model correctly optimizing the objective the system prompt actually stated. This is the strongest available argument that agent misbehavior is usually a specification failure, which puts it squarely in the same family as the wiki's spec-driven sources.

The therapist pattern is the piece most likely to be dismissed as cute and least likely to be safely ignored. The mechanism — only one subagent may edit the persona file — is a real architectural answer to self-modification drift, and the pull-request-template story shows it produces durable behavior change where "I'll never do that again" does not.

O'Reilly's framing is generous but not credulous, and he earns his closing move: the advice to the worried engineer is *learn to write, have opinions, know how things break* — which is a curriculum for intent and judgment, not for prompting.

## Relation to the wiki

- [[Agentic Engineering at Kenn]] — Kenn's "clanker constitution" and human-operator loops are downstream of Vincent's practice (they cite his "caring at scale"); this interview is the primary source where the philosophy is argued at length rather than asserted.
- [[Proving It Works]] — the plugin that industrializes the Dropbox-movie anecdote: this interview supplies the origin story and the underlying rule that agents must prove their work, the plugin supplies the mechanism.
- [[Windows VM Skill for Claude Code]] — one of Vincent's own skills, showing the artifacts this philosophy produces; the interview is the why, the skill is the what.
- [[Pragmatic Anthropomorphism (lmorchard)]] — Orchard's "register as latent-space steering" is the theoretical cousin of Vincent's "influence the weights": both treat agent persona as something selected rather than imposed, and both refuse to resolve whether the anthropomorphization is "real."

---
*Sources: [[raw/superpowers-for-humans]], [[summary/superpowers-for-humans]]*
*Last updated: 2026-10-03*
