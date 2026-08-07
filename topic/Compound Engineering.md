# Compound Engineering

Kieran Klaassen's AI-native engineering philosophy from Every.to. The core loop: Plan, Work, Review, Compound, Repeat. The fourth step -- Compound -- is what separates this from traditional engineering with AI assistance. Each unit of work should make subsequent work easier, not harder. Every runs five products with primarily single-person engineering teams using this system.

---

## Key Quotes

> "When you feel as if you can't trust the output, don't compensate by switching to manually reviewing the code. Add a system that makes that step trustworthy, such as creating a review agent that flags issues."

> "First attempts have a 95% garbage rate. Second attempts are still 50%. This isn't failure -- it's the process."

> "The code was never really yours. It belongs to the team, the product, and the users. Letting go of code as self-expression is liberating."

> "The developer who reviews 10 AI implementations understands more patterns than the one who hand-typed two."

> "A system that produces code is more valuable than any individual piece of code."

## Key Themes

#concept #compound-engineering #systems #trust #agentic-coding #plugins

The article is structured around a four-step loop (Plan, Work, Review, Compound) and then builds outward from there into beliefs, adoption stages, and best practices. What makes it more than just another "use AI to code" post is the specificity: Every ships a Claude Code plugin with 26 specialized agents, 23 workflow commands, and 13 skills. The `/workflows:review` command alone spawns 14 parallel review agents (security-sentinel, performance-oracle, architecture-strategist, etc.), each returning prioritized findings.

The **50/50 rule** is the most operationally bold claim: allocate half your engineering time to improving the system rather than building features. The justification is that system improvements compound (an hour creating a review agent saves 10 hours of review over a year) while feature work doesn't. This is the constructive counterpart to [[Cognitive Debt]]'s diagnosis. If the problem is that velocity exceeds comprehension, the solution isn't to slow down -- it's to build infrastructure that makes speed safe.

The **five stages of AI adoption** provide a useful ladder: from manual development (Stage 0) through chat-based assistance (Stage 1) and agentic tools with line-by-line review (Stage 2, where most developers plateau) to plan-first PR-only review (Stage 3, where compound engineering begins), idea-to-PR (Stage 4), and parallel cloud execution (Stage 5, "commanding a fleet").

The **three questions** for reviewing AI output are immediately practical: "What was the hardest decision you made here?", "What alternatives did you reject, and why?", and "What are you least confident about?"

This connects to [[Pre-Commit Lint Checks]] (lint as compound infrastructure), [[claude-code-config (Trail of Bits)]] (hooks as compound enforcement), and [[Building 200+ Integrations with OpenCode]] (where Nango's verification infrastructure was the compound investment that made 200 integrations possible). It's the opposite of [[acceleration-flow]]'s trap -- instead of chasing the dopamine of speed, invest in the infrastructure that makes speed safe.

## Critical Analysis

Now that I can read the full article, the picture is more nuanced than the annotation suggested. The compound engineering philosophy is correct at its core -- building systems that learn from each cycle is better than one-shot feature delivery. The specificity of the plugin (26 agents, 14 parallel reviewers) makes it more than an aspiration.

But there are tensions. The "beliefs to let go" section pushes hard on "code is not self-expression" and "more typing does not equal more learning" -- both true in aggregate, but the framing risks devaluing the deep craft knowledge that comes from wrestling with implementation details. The developer who reviews 10 AI implementations may understand more *patterns* than the one who hand-typed two, but there's a qualitative difference in understanding that comes from working through failure modes firsthand.

The five-stage adoption ladder is useful but somewhat self-serving -- Stage 5 ("parallel cloud execution") conveniently maps to the use case that requires the most tooling (i.e., the paid plugin). The "skip permissions" section, with the `alias cc='claude --dangerously-skip-permissions'` recommendation, is practical but creates a false sense of security: "git is your safety net" only works if you catch problems before they hit main, and not all problems are visible in diffs.

The 50/50 rule (half your time on system improvement) is the strongest and most testable claim. If it holds, compound engineering is a genuine methodology. If it doesn't -- if the compound returns are less than advertised -- it's just over-engineering wrapped in productivity language. The article would benefit from concrete metrics: how much time does Every actually spend on compound work, and what's the measured payback?

Still, the core insight is sound: the bottleneck has moved from typing to thinking, and the most valuable engineering artifact is no longer the code but the system that produces it.

Dan Guido's [[AI as an Enterprise Operating System]] is Compound Engineering scaled to the organizational level: his core thesis — "I want our security expertise to compound as code" — is the Plan/Work/Review/Compound loop applied to an entire company's knowledge. Trail of Bits operationalizes it through bimonthly hackathons with artifact harvesting, a shared configuration repo where "scar tissue becomes infrastructure," and three-tier skills repositories (internal, public, curated) that make every engagement's lessons reusable. The config repo accepting company-wide PRs after each hackathon is the most literal implementation of Klaassen's 50/50 rule in the wild.

---
*Sources: [[summary/compound-engineering]]*
*Last updated: 2026-05-14*
