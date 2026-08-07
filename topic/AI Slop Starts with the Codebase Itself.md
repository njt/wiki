# AI Slop Starts with the Codebase Itself

An unnamed author (writing on LinkedIn) argues that AI has inverted the software-rewrite calculus: a codebase's readability by AI — not just by humans — is now a competitive factor. The more your codebase speaks a dialect the models already know from training data, the less time you spend teaching AI before it can build.

---

## The Core Argument

The author observes a pattern across multiple codebases: when the system is clean, consistent, and built on popular, well-documented stacks, AI models generate implementations efficiently with minimal extra prompting. When the codebase carries legacy baggage, proprietary patterns, and inconsistency, the model fumbles — it needs examples, documentation, and hand-holding. The former feels like "using AI to solve the problem"; the latter feels like "trying to get AI to learn your language first."

This reframes the rewrite decision. It's no longer just about technology modernization or developer ergonomics. It's about *AI alignment* — aligning your codebase's patterns with what the models already know.

## Key Quotes

> "My view on software rewrites has changed because of AI."

The essay's thesis, compressed. AI doesn't just make rewrites cheaper to execute — it changes the *return* on a rewrite by adding an AI-productivity dividend that compounds over the life of the codebase.

> "Every minute your AI spends trying to figure out what you meant instead of building what you need is a minute your competitors are shipping."

The competitive framing is sharp and intentional. Prompt engineering matters less than the author once believed; what matters more is the context the codebase itself provides. A codebase that aligns with AI's training distribution is a productivity multiplier. One that doesn't is a tax — and the tax compounds.

> "That time spent teaching AI your proprietary patterns and inconsistent conventions — that's your competitors' advantage."

This inverts the "secret sauce" instinct. The proprietary internal framework that makes your team productive might be the very thing that makes your AI unproductive. The patterns that optimize for human cognition aren't necessarily the patterns that optimize for model cognition.

## Key Themes

- **#concept AI-readability as code quality dimension** — Code quality has always included readability for humans. The author argues it must now include readability for models, which is a different thing: models benefit from patterns they've seen millions of times in training data, not from clever abstractions that require explanation.

- **#pattern The rewrite payoff just got larger** — If a rewrite doesn't just modernize the stack but also aligns the codebase with AI's training distribution, the payoff includes ongoing AI-productivity gains, not just one-time developer-efficiency improvements. This changes the NPV calculation.

- **#concept Codebase as context** — The codebase IS the prompt. The model reads existing code to understand patterns; the quality of that implicit instruction matters at least as much as any explicit instruction you write. A messy codebase is a bad prompt you can't edit.

- **#concept Competitive moat inversion** — Proprietary patterns, once a source of competitive advantage (harder to replicate, tuned to your domain), may now be a competitive *disadvantage* (harder for AI to work with, requiring more human intervention). The moat becomes a tax.

## Critical Analysis

**The argument is sharp but under-specified.** The author is describing a felt experience — "I've noticed a pattern" — not presenting evidence. We don't know which stacks, which models, or what "fumbles" means quantitatively. This is a blog post, not a study, and it reads like one. But the hypothesis is testable and specific, and it aligns with what more rigorous work is finding.

**This pairs tightly with the SonarSource study on [[Code Cleanliness and Coding Agents]].** Trivedi and Schmitt found that cleaner code reduces token consumption 7–8% and file revisitations 34%, even though pass rates don't change. Martin Fowler's [[The Economic Benefit of Refactoring]] provides a third data point: 15 aggressive refactoring steps on an agent-generated Rust monolith reduced input tokens by 83% — an order of magnitude beyond SonarSource. The pattern across all three: structure matters, and the more aggressively you restructure, the larger the savings. The author's claim is stronger: codebase quality doesn't just affect cost, it affects *whether the AI can function at all*. The SonarSource study finds a token tax; this essay claims there's a threshold below which the AI is effectively useless. Fowler's experiment anchors the upper bound. All three might be right — the studies vary cleanliness within a single framework; the essay compares across fundamentally different codebase structures.

**The essay also connects to [[Constraint Decay]].** Dente et al. found that convention-heavy frameworks are a trap for agents — Django and FastAPI performed worse than Express and Flask because implicit behavior creates a minefield. The author's proprietary-patterns argument is the same phenomenon at the organizational level: your internal framework conventions are invisible context the model can't see, and that makes them expensive.

**The "popular stack" claim is a bet on training-data coverage.** If models are trained on public GitHub repos, then Express and React and Django appear orders of magnitude more often than any internal framework. The effect should be measurable: models should produce higher-quality code for popular frameworks than obscure ones, holding task difficulty constant. The [[Lean Software Scaling Laws]] proposal from Gwern — measuring LLM perplexity over codebases as a proxy for language design quality — is essentially the same idea from the training side rather than the inference side.

**The rewrite calculus is the most actionable claim, and also the most dangerous.** "Rewrite it" has been terrible advice for most of software history. The author is arguing AI changes the math, but he doesn't address the survivorship bias: we hear about the rewrites that work, not the ones that die halfway through and leave the org with two broken codebases. The AI dividend is real but it has to be large enough to overcome the rewrite risk premium, and the author doesn't estimate either.

**What the essay gets right:** proprietary patterns as an AI tax is a genuinely novel framing. Most discussions of "clean code" focus on human readers; adding model readers to the audience changes what "clean" means. The observation that popular, well-documented stacks win because models have seen them before — not because they're inherently better — is a consequential claim if true, and one that would push tech choices toward consolidation rather than diversification.

**What's missing:** any acknowledgment that the AI-productivity gap might close. If models get better at reasoning about unfamiliar codebases (bigger context windows, better in-context learning), the advantage of popular stacks shrinks. If models get better at extracting conventions from examples, proprietary patterns become less of a tax. The author treats the gap as structural when it might be contingent — a snapshot of model capability in mid-2026 that could look different by 2027.

**The most subversive implication:** if AI-readability becomes a real factor in technology choice, it accelerates consolidation toward a small set of "AI-native" stacks. The long tail of frameworks, languages, and patterns that make software diverse becomes a liability. This is a monoculture argument, and the author doesn't engage with the risks of monoculture at all.

**The domain-level extension — training-data deserts:** Mathieu Ropert's [[An Honest Review of AI Programming]] takes this thesis one level up: for some entire *industries*, the training data simply doesn't exist. The last AAA game to be open-sourced was Doom 3 (2004, 22 years ago). Production game engines, custom scripting languages, and proprietary tools have left effectively zero public training data — so LLMs trained on GitHub repos see only hobby projects, game jam entries, and tutorial demos. The "proprietary patterns as AI tax" argument applies not just to your codebase but to your entire field. For game development, the AI isn't struggling with your conventions — it's never seen *any* conventions from production systems.

## Connections

This essay sits at the intersection of several wiki threads. [[Code Cleanliness and Coding Agents]] provides the closest empirical backing — cleaner code is cheaper for agents, even if it doesn't change pass rates. [[Constraint Decay]] shows what happens when codebase conventions work against the model: ~30pp assertion-pass-rate drops. [[You Can Just Say It]] gives us the "intent vs. form" lens — a messy codebase is form without discernible intent, and the AI can't extract what isn't there.

The maintainer-side cost of AI slop is the missing half of this essay's argument. [[Zero-Cost Fallacy of Open Source]] documents exactly what happens when slop lands on the other side of `npm install`: maintainers become unpaid code reviewers buried under plausible-but-wrong LLM-generated PRs, and some close their projects entirely to escape. The codebase that produces slop is one problem; the repository that receives it is the other, and the two are directly connected.

The rewrite economics connect to [[The Cost YAGNI Was Never About]] (Kent Beck on AI making rewrites cheaper but the trap still being real), [[Specifications as the Product]] (if the codebase IS the prompt, the spec is what should survive the rewrite), and [[Why Build vs Buy is the Wrong Question]] (Chris James on DDD subdomain taxonomy as the real decision framework). The competitive-framing echoes [[The Founder's Playbook]] — Anthropic's observation that AI introduces *agentic technical debt* that compounds. A codebase that misaligns with AI isn't just harder to work on; it gets harder at an accelerating rate because every AI-assisted change adds more code the model doesn't understand.

---
*Sources: [[raw/ai-slop-starts-with-the-codebase-itself]]*
*Last updated: 2026-07-11*
