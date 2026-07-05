---
url: https://xcancel.com/i/article/2072432348805669139
title: "Fable open sourced NanoClaw's agent factory. It cost $800."
author: Gavriel Cohen (@Gavriel_Cohen)
site: X/Twitter (via xcancel.com)
date_fetched: 2026-07-05
date_published: 2026-07-01
---

Fable open sourced NanoClaw's agent factory. It cost $800.

Gavriel Cohen @Gavriel_Cohen · Jul 1

Claude Fable 5 came out on June 9. Like everyone else, I spent launch day throwing things at it, and by 2am I was out of ideas. Before going to bed I gave it one last job. There's a private fork of NanoClaw we call our PR Factory: every pull request on the repo creates a Slack thread run by an agent that triages, reviews, and tests the change, with a human approving every sensitive action from an approval card in Slack. The fork was a franken fork, 405 commits behind upstream and stale beyond repair.

I pointed Fable at NanoClaw's customization guidelines and at the fork, told it to run an ultracode workflow to bring the two into agreement, and went to sleep.

I woke up to a session that had run for five hours and cost about $800. The fork was maybe 80% of the way to open-sourceable: mapped, decomposed into skills, and covered by tests. The surprising thing to me was the judgment. Overnight, Fable sorted 8k lines of messy code into piles a staff engineer would recognize: a latent bug in upstream that everyone has, send it to core. A missing hook, propose it to core. This is an architectural gap (three Slack bots crammed into a core that assumes one bot per platform), and here is the analysis for why the fix belongs in core rather than in our fork. This is our private tuning, keep it out of the contribution entirely. I didn't prompt Fable it to make those distinctions, it got them from the guidelines I pointed it to.

The contract: NanoClaw's customization guidelines

To explain why that worked, you need the context I handed it at 2am. NanoClaw recently adopted a set of customization guidelines, three docs (customizing, the skills model, skill guidelines) that I drafted because of a problem every extensible project has: people fork it, change the code, and then every upstream update turns into a merge fight. The more you customize, the worse it gets, until your fork is stale beyond repair.

The guidelines' answer is one idea: every change you make to your fork is a skill. A skill is a self-contained description for how to apply a modification to the projects source code in order to add or modify a capability. It's structured in the format of the skill standard that Anthropic created. It contains its own code, knows how to install itself, and knows how to remove itself.

The rules that make this work:

Mostly add. A good skill adds files, appends a line to a barrel, adds a dependency. The only problematic thing for maintainability over time is reaching into existing code, and each reach-in should be a line or two that imports from the skill's own files.

A test for every integration point. Every functional reach-in gets a test that goes red if the wiring is deleted or drifts. This is not a unit test of the skill's own logic, but a guard on the integration point with core. When upstream moves something, there's a failing test that's basically your upgrade TODO.

Removal ships with the skill. A REMOVE.md that reverses everything apply did.

Your fork is defined by a recipe. One file listing your skills in apply order. That's what lets you rebuild the fork from scratch on fresh upstream, or hand it to someone else.

And the maintainer's side of the deal: when many skills reach into the same spot in core, that's a hotspot, and the maintainer adds a proper hook there so the reach-ins become clean appends.

Our agent factory fork didn't follow any of this. But the guidelines gave the agent a clear, checkable definition of done, and a decision rule for every piece of code it encountered. Bug fixes and hooks go to core. Features stay skills. Every integration point has a test.

The overnight 80%

It mapped the mess. Six reader agents fanned out over the fork delta (about 8,000 lines across 78 files) and produced a capability map: five distinct capabilities, 41 integration points with core, each classified and assigned a test archetype. Also a pile of dead code we'd forgotten about, doc claims that disagreed with the code, and a stale migration log.

It wrote tests. Every integration point got a guard test, and every guard was mutation-verified. The agent literally deleted each reach-in, ran the test, and required it to fail, then restored and required green. About 90 mutations, executed in isolated worktrees. One verification batch mislabeled its indices and silently skipped a guard; the orchestrator caught the mismatch in the matrix and re-ran it by hand.

It found a design problem. Our multi-bot support reached into twelve core files, including rewriting a state-adapter class. I asked why it was so extensive; it felt like it should be simpler. The analysis that came back decomposed the twelve files into roughly 45% intrinsic (bot identity turns out to be a coordinate in six independent keyspaces of the core), 30% retrofit tax, 15% separable policy, and 10% our own avoidable choices. It also evaluated the "just register each bot as its own channel type" design I proposed, and showed exactly where it breaks: user identity is keyed on channel type, so the same human becomes three users and every role check silently fails for the second bot. The conclusion was that multi-bot identity belongs in core. That became #2733: a native instance dimension, backward compatible, with an FK-safe migration and exact-key dispatch so a named bot's traffic can never silently reroute through a sibling's identity.

It shipped a PR train. Following the guidelines' logic, the fork broke down into small PRs: three bug fixes (including a genuine latent upstream bug where the command-gate denial path opened the outbound DB read-only, so denials were never delivered), three hook/registry PRs that turn common reach-ins into appends, and the instance substrate. Each PR independently implemented, tested, and mutation-checked. I merged seven and rejected one, a container idle-timeout feature, because it was a lot of core surface for an optimization. Fable closed it, stripped the dependency out of everything downstream, and moved on.

The last 20%

The overnight worflow got the fork aligned and the core train proposed. The remaining 20% took a few more workflows over the next two days, to go from "our fork is clean" to "strangers can run this."

It packaged the factory as a recipe. The final PR, #2742, contains no source changes at all. It adds a recipes/pr-factory folder: a meta skill that applies five component skills, each with its own code, apply steps, removal steps, and guard tests. Executing the skill edits core.

And it proved the recipe works the only way that counts. A harness agent took a fresh clone of upstream, was handed the recipe, and was forbidden from looking at any reference implementation. It applied the whole thing unaided: every version probe passed, every patch anchor matched, the test suite grew 458 -> 547 across six green checkpoints. Then it applied everything a second time and confirmed the tree was byte-identical, then removed it all and confirmed the tree came back byte-identical to pre-apply. Apply, idempotency, removal: the three claims a skill makes, all checked mechanically.

What the humans did

The human contribution across all of it: pick which pieces went to core versus the bundle, keep sentry out, keep our tuned review skills private, choose where recipes live, reject one PR, order two features ripped out instead of fixed, declare single-repo scope, and say "yes, push." Around a dozen calls. Every one of them was a decision: taste, scope, risk appetite. None of them was labor.

The labor was something over a hundred sub-agent runs across a dozen orchestrated workflows: mapping, implementing, mutation-testing, adversarially reviewing its own findings (about a third of raw review findings got refuted by verifier agents before reaching us).

The bill

The $800 in the title is the overnight session: five unattended hours that took a 405-commits-stale fork 80% of the way to publishable. The follow-up workflows added their share on top. For the total we got: seven merged core contributions, one architectural upgrade (channel instances) that every NanoClaw multi-bot setup now gets for free, one discovered-and-fixed upstream bug, and the whole factory published as a one-command install. The human version of this (upstreaming a hairy internal fork) is weeks of an experienced engineer's time, so IMO $800 is cheap.

The coda

On June 12, the same day Fable pushed the final revision of the release PR, the US government applied export controls to Fable 5 and Mythos 5, and Anthropic suspended access for everyone while it worked through compliance. The model that open sourced our factory went dark hours after finishing the job, and the PR sat there, open and mergeable.

Access came back this morning. This post is going out a few hours after Fable's return. You can draw your own conclusions about how this post was written.

The factory is in nanocoai/nanoclaw#2742. Point Claude at .claude/skills/recipes/pr-factory/SKILL.md and it'll build you one.
