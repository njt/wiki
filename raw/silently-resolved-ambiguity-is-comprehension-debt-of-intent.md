---
url: https://blog.bl00cyb.org/2026/08/silently-resolved-ambiguity-is-comprehension-debt-of-intent/
date_fetched: 2026-08-25
---

In our theory, the development lifecycle is compressed, and the decisions that are made at “transition” points are moved to later. Even “transition” here is a bit nebulous, as stages of the lifecycle are overlapping more and more with each other. So instead, the *types* of decisions being made with AI tooling (and the debt they incur) is different. Decisions made with awareness reduce comprehension debt but incur naps; decisions you allow (intentionally or accidently) your agent to make prevent the nap but increase debt. Decide which decisions you should be making, and set your system up to surface them to you alongside relevant data.

I’m writing up an experiment along these assumptions to see if I can try it out at work.

Someone who only vibe codes will get shockingly far, and then discover they’ve made something that cannot be changed while maintaining trajectory. Someone who understands every change will be using spicy autocomplete, and be too slow. Something is in the middle, we posit – intentional decision points backed by data, and not getting involved much past those points unless something seems off the rails.

Externalizing expectations is important in any collaboration, including with AI. We are still tweaking how AI navigates ambiguity and unknowns, with hallucinations continuing to be a problem. That means that when we utilize AI, we have to be intentional about what expectations are set down (“constraints” section from the last post) both to have a reasonable context window, and also to allow flexibility where it’s useful. What remains for reducing comprehension debt is understanding how a decision was made – based on your defined expectations, a verifiable reference, or probability (either usefully “creative” or an inaccurate hallucination).

Agents resolve ambiguity constantly. An issue underdetermines a behavior, the agent picks something statistically probable, and nothing surfaces the fact that a decision was made at all. That’s a debt entry with no signature: comprehension debt of *intent*, senior to comprehension debt of implementation. The follow-up question is a design problem: **how do you know when an ambiguity needed authored intent rather than an agent’s silent guess**. What’s the tripwire that says “stop, this one goes to a human,” without routing *everything* to the human?

What does an outcome tie back to? Something I defined, a reference, or probability and LLM? Provenance is a thing Ethan Zuckerman is starting to take on, and it’s the catch for if surfacing decision points hasn’t been done appropriately. Knowing provenance of a decision would allow a model to be further improved through instruction, reducing comprehension debt because it’s behaving in a manner you guided.

Next up, we’ll get to what I think the “golden egg” of this whole thread – how to build reliable, deterministic tools using stochastic parrots.
