# Cybersecurity Is Proof of Work Now

David Breunig argues that Anthropic's Mythos model -- purpose-built for finding security vulnerabilities -- reveals a fundamental shift in cybersecurity economics. Security is no longer a craft problem solved by clever engineers; it's a compute problem solved by spending more tokens than your attacker. The analogy to cryptocurrency's proof-of-work is precise: defence scales with expenditure, not ingenuity. This reframes everything from open-source strategy to software development workflows.

---

## Key Quotes

> Finding exploits represents "a clearly defined, verifiable search problem" well-suited to computational intensity rather than genuine innovation.

This is the crux. Vulnerability discovery is search, and search is exactly what LLMs-with-compute excel at. The insight isn't that AI can hack -- it's that hacking was always amenable to brute computational force, and we just didn't have enough of it until now.

> Defending systems requires spending more computational resources discovering vulnerabilities than attackers invest in exploiting them.

The asymmetry flips. Traditionally, attackers had the advantage (find one hole vs. defend all holes). Now defenders can afford to search exhaustively -- but only if they're willing to pay. Security becomes a line item denominated in tokens, not headcount.

> Code remains inexpensive unless security requirements apply. Even as inference costs decline, organizations must outspend potential attackers to maintain advantage.

This is the uncomfortable punchline. Vibe coding is cheap right up until you need it to be safe. The cost floor isn't set by inference pricing -- it's set by what a discovered exploit is worth on the market. You're not optimising against compute costs; you're optimising against adversaries.

## Key Themes

#concept #security #economics #open-source #compute

**Security as proof-of-work.** The proof-of-work analogy is elegant because it captures the essential dynamic: there's no shortcut, no clever trick. You either spend the compute or you're vulnerable. This is a direct extension of the compute thesis in [[Zheng Dong Wang's 2025 Letter]] -- compute isn't just driving model capabilities, it's driving the security arms race.

**The three-phase development model.** Breunig proposes: (1) Development -- humans drive features, (2) Review -- AI refactors and documents, (3) Hardening -- autonomous vulnerability scanning until the budget runs out. This maps cleanly onto the maturity levels in [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] (levels 3-5) and the plan/work/review/compound loop in [[Compound Engineering]], but adds a distinct security phase that those frameworks treat as an afterthought.

**Open source as collective defence.** If security is proportional to token spend, widely-used open-source projects can pool community resources that no single proprietary codebase can match. This is a direct counter to Karpathy's "replace dependencies with AI-generated code" argument -- writing your own means defending your own, alone.

**The Mythos evaluation.** The AI Security Institute's test -- "The Last Ones," a 32-step attack simulation requiring ~20 human hours -- is notable for its specificity. Mythos completed it 3/10 times at 100M tokens per attempt. Other frontier models: 0/10. The gap isn't incremental; it's categorical.

## Critical Analysis

**What's strong:** The proof-of-work framing is the best mental model I've seen for AI-era cybersecurity economics. It makes the incentive structure legible in a way that "AI finds bugs faster" doesn't. The three-phase development model is also genuinely useful -- it gives teams a concrete workflow rather than vague advice to "use AI for security."

**What's missing:** Breunig doesn't address the attacker side of the arms race. If defenders can use Mythos-class models for hardening, attackers will eventually get equivalent tools. The proof-of-work analogy actually predicts this: both sides escalate compute, and the defender's advantage is only durable if they consistently outspend. He also doesn't address the open-source funding problem -- who pays for the tokens to harden public infrastructure? The "community resources" argument handwaves past the tragedy of the commons that already plagues open-source maintenance.

**What it means for the wiki:** This piece sits at the intersection of several threads. The [[Security and Sandboxing]] synthesis already covers architectural containment (sandboxes, credentials, prompt injection), but treats security as a design problem. Breunig reframes it as an economics problem -- and the two are complementary. The [[AI Killing B2B SaaS]] page notes that vibe-coded replacements lack security; Breunig explains *why* that gap is structural, not just a matter of maturity. The [[Write Only Code]] concept of Slop Radius gets a new dimension: in a proof-of-work security world, your Slop Radius is directly proportional to your hardening budget.

The most provocative implication: if security is proof-of-work, then insecure software is just software whose owners couldn't afford the compute. That's a political statement as much as a technical one.

## Connections

- [[Security and Sandboxing]] — architectural containment (design side) vs. compute-driven hardening (economics side)
- [[Compound Engineering]] — the three-phase model extends the plan/work/review/compound loop with a distinct hardening stage
- [[Zheng Dong Wang's 2025 Letter]] — compute thesis applied to security, not just model capability
- [[AI Killing B2B SaaS]] — why vibe-coded replacements can't just "add security later"
- [[Write Only Code]] — Slop Radius meets hardening budget
- [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] — the hardening phase lives at levels 4-5
- [[VTcode]] — agent-native sandboxing as one piece of the hardening phase
- [[Trailmark]] — source code as queryable graph for security analysis (a tool for the hardening phase)
- [[Building low-level software with only coding agents]] — 38K lines, 900+ tests, $2,871... but was it hardened?

---
*Sources: [[raw/cybersecurity-is-proof-of-work-now]]*
*Last updated: 2026-05-14*
