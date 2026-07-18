# Aviator Verify

Aviator's product that replaces traditional code review with intent-based verification: capture what the developer and agent agreed to build, then check every acceptance criterion against running code with attached evidence (screenshots, tool calls, request/response data). It's the most opinionated commercial implementation of the "review behavior, not diffs" thesis that [[The End of Code Review]] and [[Agentic Code Review]] argue for — and the first to make compliance audit trails a first-class feature rather than an afterthought.

---

## Key Quotes

> "Replace code reviews with verified intent"

The product's tagline is also its thesis. The shift is from "does this code look right?" to "does this code do what we said it should do?" This reframes review from a code-reading exercise to a behavior-verification exercise, and it's the right reframe — but the whole product depends on whether "intent" can be captured accurately and cheaply enough that teams actually do it.

> "AI can generate in minutes what takes hours to review"

The bottleneck math that drives the business case: 5-minute generation vs. 45-minute review = 91% slowdown. This is marketing math (real review times vary enormously by change size and complexity), but the structural claim is correct — human review capacity is fixed while AI generation throughput accelerates. Something has to give, and what gives first is review quality. [[Agentic Code Review]]'s 861% churn and 441% longer review times are the independent confirmation.

> "Invariants — past team review comments encoded as reusable checks so recurring flagged patterns never come back"

The genuinely novel feature. Every team has patterns they catch in review over and over — "did you check for N+1 queries?", "is the error message user-facing?", "does this endpoint have rate limiting?" Aviator encodes these as automated checks that run on every PR. This turns review knowledge from tacit expertise that lives in senior engineers' heads into executable infrastructure. It's the same insight as [[Guardrails and Feedback Loops]]'s "linters beat prompts" principle, applied at the team-process layer rather than the code-quality layer.

## Key Themes

- #tool **Intent-first verification** — The contract is what the developer and agent agreed to build, not the code itself. Reviewers approve against intent, not against the diff. This inverts the traditional review model: the spec is the primary artifact, and the code is evidence that the spec was implemented.
- #pattern **Three-layer verification** — Scenarios (end-to-end exercises with screenshots and request/response capture), Invariants (encoded past review comments as reusable checks), and Code-scan (AST and structural analysis). An LLM fallback with confidence labeling handles custom criteria. This layered approach means deterministic checks run first and cheap, with LLM judgment reserved for what can't be automated.
- #concept **Evidence over opinion** — Each criterion gets a verdict with attached evidence: screenshots, matched invariants, code-scan results. Reviewers don't have to trust the agent or run the code themselves — they inspect the evidence. This is the operational version of [[The End of Code Review]]'s "structured review reports" vision.
- #pattern **MCP-native workflow** — Developers work in Claude, Cursor, or Copilot. The agent captures intent, generates acceptance criteria, and submits to Aviator in a single MCP tool call. No context switch, no separate tool. This is the right architectural choice — meet developers where they already work rather than demanding they come to you.
- #concept **Compliance as product feature** — Immutable audit records link spec approval, implementation, verification, and deployment with timestamps and segregated duties. One-click export for SOC 2, ISO 27001, or custom frameworks. This is a genuinely smart positioning move: compliance isn't an add-on, it's the natural output of the verification process itself.

## Critical Analysis

**The product is a thesis, not a proven solution.** Aviator Verify is what [[The End of Code Review]]'s "agent-in-the-loop verification" looks like as a product. But the page offers no independent benchmarks, no comparison data against traditional review, and no evidence that intent capture survives contact with real development workflows. The "23 Verified today" counter on the page is a nice touch but tells you nothing about false positive/negative rates, reviewer experience, or whether teams actually ship faster. This is a vision document with a pricing page.

**The invariants feature is the most important idea here and it's not Aviator-specific.** Encoding past review comments as automated checks is something every team should be doing regardless of whether they buy this product. It's the team-scale version of [[Guardrails and Feedback Loops]]: turn your review scar tissue into automated enforcement. You could implement a crude version today with a markdown file of checklist items and a Claude Code hook that checks them. The product value is in making this systematic and reducing the friction to zero, but the idea itself is bigger than any one tool.

**The intent-capture problem is the whole game.** The pitch is elegant: capture what you and your agent agreed to build, then verify it. But "what you agreed to build" is often fuzzy, emergent, and discovered during implementation — not written down beforehand. If intent capture becomes another form of ceremonial documentation that developers fill in after the fact to make the tool happy, it's compliance theater wearing an engineering hat. The MCP integration helps (the agent generates criteria from what was actually built), but it also creates a circularity problem: the agent that built the code is also the agent that says what the code should do. Where's the independent perspective? [[AI Agents Need Clear Specs]]'s U-curve applies here: the cost of spec validation sits between "write spec" and "run agent," and it's never zero.

**The "vs. AI code review" comparison is a straw man, but a useful one.** Aviator positions itself against AI code reviewers that "read the diff" and "post comments" without knowing intent. This is a fair critique of first-generation AI review tools, but it undersells what the better ones actually do. [[Orchestrating AI Code Review at Scale]] describes Cloudflare's system with 7 specialized agents, a coordinator judge, and circuit breakers — that's more sophisticated than "reads diff, posts comments." And [[OpenCodeReview]] combines deterministic tree-sitter analysis with LLM confirmation in a hybrid architecture that looks a lot like Aviator's three-layer model. The real distinction isn't AI vs. intent — it's whether the verification runs against running behavior or against static code.

**Compliance as product feature is either genius or a red flag.** Making audit trails a natural output of verification rather than a separate compliance process is smart design. But it also tells you who the buyer is: enterprises with SOC 2 and ISO 27001 requirements, not startups optimizing for speed. The risk is that the compliance tail wags the engineering dog — verification becomes about producing auditable records rather than finding real problems. The page's emphasis on "segregation of duties" and "full traceability" reads like it was written for a CISO, not a tech lead.

**What's genuinely new here:** Invariants as encoded team knowledge. The three-layer routing (deterministic → LLM fallback with confidence labeling). MCP as the submission channel rather than a separate web UI. Evidence-attached verdicts. These are good ideas that point toward what post-review verification should look like, whether or not Aviator is the tool that delivers them.

**What's missing:** Any mention of false positives. Any discussion of what happens when the agent-generated acceptance criteria miss something important. Any data on reviewer experience — does this actually reduce the cognitive load, or does it just shift it from reading code to evaluating evidence? Any pricing. Any independent validation that teams using this ship faster or with fewer defects. The product page is a bet that the thesis is compelling enough on its own. For some teams, it will be.

---

*Sources: [[raw/aviator-verify]]*
*Last updated: 2026-07-18*
