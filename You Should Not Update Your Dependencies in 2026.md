# You Should Not Update Your Dependencies in 2026

Olivier Gambier's case that dependency updates have crossed the line from maintenance to threat vector — and that the only viable response is treating every version bump as an untrusted code contribution reviewed by AI agents in CI.

---

## The Argument

Gambier traces a clear arc: 1990s sysadmin culture (manually vet a dozen known vendors), through the Dependabot era (automate the boring stuff because you trust the registries), to 2026 (registries are compromised, maintainers are burned out, AI is flooding the supply chain with code, and Dependabot is now an attack vector).

The core reframe is sharp: **every dependency update should be treated like a PR from an unknown external contributor.** You wouldn't auto-merge a stranger's code. Why do you auto-merge a `package-lock.json` change that pulls in 47 transitive dependencies you've never read?

The provocateur's title ("you should not update") is bait — the real argument is that the *default* of auto-merging dependency bumps is what's broken. You should update, but only after review.

---

## Key Quotes

> "open-source maintainers are not just free labor, they are also overworked, under-equipped, wildly understaffed"

This isn't a complaint about fairness — it's a threat model claim. When the person who owns the publish button is exhausted and under-resourced, their credentials are the weakest link in your supply chain. The TeamPCP campaign proved this: compromised maintainer accounts, not technical exploits, were the entry point.

> "Dependabot and siblings were a genuine and major progress five to ten years ago. Now they are just harmful."

A tool designed for a trust model that no longer exists is worse than no tool — it creates the illusion of maintenance. The auto-merge of a Dependabot PR looks like diligence but is actually a blind pass-through. This is the same pattern as [[Supply Chain Security for Software Developers]]'s finding that OIDC + legacy token coexistence creates the *appearance* of security without the reality.

> "Dependency updates must be considered as untrusted code contributions."

This one sentence does more work than the rest of the article combined. It reframes the entire practice of dependency management. If this became conventional wisdom, it would restructure how every CI pipeline in the industry works.

> "Going back is not a strategy."

Aimed at the reactionary impulse to fork everything, pin everything, and refuse to update. Gambier's point is that industry velocity makes this a losing strategy — your frozen dependencies will drift from the ecosystem while the attacks evolve around them.

> "The institution of the last standing, already frail, human safety guardrail, the good old code review, has been trampled"

This is the most pessimistic line in the piece. It's not just that automation is dangerous — it's that the human review process that was supposed to catch problems was already failing *before* AI made it impossible. The supply chain grew faster than our capacity to read it.

> "AI is not a magic wand and (at least for now) cannot outperform the best humans."

Gambier is careful here. He's selling an AI product but refuses to oversell the AI. The claim is narrower: AI can do the *mechanical* review work at scale — pattern matching, diff inspection, behavioral comparison — freeing the few humans who are actually qualified to focus on the ambiguous cases. This is honest vendor positioning, which is rarer than it should be.

---

## Key Themes

#concept — **Dependency updates as untrusted contributions**: the central reframe. Every version bump is a PR from a stranger. Review it or don't merge it.

#pattern — **AI-in-CI as supply chain gate**: mechanical review of every dependency change — typo-squatting detection, known-bad-version blocking, behavioral drift analysis, CVE reachability evaluation — all inside the CI pipeline rather than as a separate security scanning step.

#concept — **Version age as heuristic**: packages less than 7 days old get heightened scrutiny; less than 72 hours gets max scrutiny. This echoes the 7-day cooldown in [[Supply Chain Security for Software Developers]] but operationalizes it as a continuous variable rather than a binary gate.

#tool — **Mendral**: the AI DevOps Engineer platform Gambier is building. The article is part manifesto, part product pitch — but the manifesto is strong enough to stand alone.

#person — **Olivier Gambier**: ex-Docker distribution team lead (2014, rebuilt image distribution on content-addressability). Co-founder with Sam Alba and Andrea Luzzardi (both ex-Docker/Dagger). Deep supply chain credibility.

---

## Critical Analysis

**The article gets the diagnosis exactly right and the prescription mostly right.** The diagnosis: we built an industry on trust assumptions that were valid in 2010 and are catastrophic in 2026. The prescription: automated review of every dependency change. Where it gets interesting is what Gambier *doesn't* say.

**The economic tension is real and unaddressed.** If every dependency update requires CI-level review, the latency of updates becomes a function of your CI queue depth. For a monorepo with hundreds of dependencies and a CI pipeline that already takes 20 minutes, adding sandboxed behavioral analysis per update is a meaningful cost. Gambier handwaves this with "it runs in CI" but CI time is money, and the trade-off between security latency and deploy velocity is the actual decision most teams face.

**The 7-day rule is a heuristic, not a solution.** Both this article and [[Supply Chain Security for Software Developers]] converge on "wait a week before adopting new versions." This works against smash-and-grab attacks but is useless against long-compromised maintainer accounts, slow-burn implants, or targeted attacks that wait out the cooldown. Gambier acknowledges this but the product pitch glosses over the hardest case: the patient adversary.

**The comparison to code review is illuminating but incomplete.** Code review works (when it works) because the reviewer understands the codebase's intent and can spot when a change violates it. A dependency update reviewer needs to understand the dependency's intent *and* the calling code's expectations *and* whether the diff between versions represents legitimate evolution or compromise. That's a harder problem, and AI hasn't solved it — it's doing pattern matching against known-bad, which is valuable but narrower than "review."

**What's missing: the social dimension.** The most dangerous dependency updates aren't random malicious packages — they're legitimate packages whose maintainers were compromised. The AI reviewer can spot a new post-install script, but can it spot a one-line behavioral change buried in a 2,000-line diff? Gambier's architecture says yes (behavioral drift analysis) but the evidence for this working at scale isn't in the article.

**The strongest insight isn't about AI at all.** It's the reframe: dependency updates are untrusted contributions. If you adopt nothing else from this article, adopt that mental model. It doesn't require AI. It requires changing your CI config to stop auto-merging Dependabot PRs and start treating every `package.json` change as a code review event. The AI part is optimization of a process that most teams haven't even started doing manually.

---

## Related Pages

- [[Supply Chain Security for Software Developers]] — The practical companion piece: configuration recipes for the 7-day rule, pinning, and script blocking. Gambier provides the argument; lhl's gist provides the config.
- [[Cybersecurity Is Proof of Work Now]] — The economic framing: security is a compute economics problem. Gambier's AI reviewer is a bet that compute can outspend the attacker.
- [[Security and Sandboxing]] — The broader security landscape: sandboxing, credential management, prompt injection defense.
- [[Harness Engineering]] — Böckeler's framework for feedforward vs. feedback controls. Gambier's AI reviewer is a feedback control: detect problems after the fact rather than prevent them structurally.
- [[Harness Engineering (OpenAI)]] — The OpenAI team shipped 1M lines with zero handwritten code. The same harness engineering patterns apply to dependency review: deterministic gates where possible, AI judgment where necessary.
- [[Compound Engineering]] — When you can't trust the output, add a system. Gambier is proposing a system for the specific case of dependency updates.
- [[Guardrails and Feedback Loops]] — Linters beat prompts. An AI reviewer in CI is a guardrail, not a suggestion.
- [[Feedback Loop is All You Need]] — The self-tightening loop. Every caught compromise feeds back into the detection patterns.
- [[I Don't Want Your PRs Anymore]] — The inversion where the maintainer generates code faster than they can review contributions. Gambier's dependency problem is the same dynamic: the ecosystem generates updates faster than anyone can review them.
- [[You Dont Want Long-Lived Keys]] — The credential hygiene dimension. Most supply chain compromises start with a stolen token.
- [[Smart Models Dumb Pipes]] — AI as judgment machine, not Q&A machine. The AI reviewer is a judgment engine, not a chatbot.
- [[AI Coding Tools Create More Bugs Than They Fix]] — The counterpoint: AI introduces vulnerabilities. Gambier's proposal is AI as defense, but AI as threat is the other side of the same coin.
- [[Scaling Long-Running Agents]] — The CI-based agent architecture Gambier describes has the same coordination challenges.
- [[Radical Accountability]] — Taste is the last differentiator when automation handles everything. The human reviewer's role shrinks to the ambiguous cases that AI flags but can't resolve.

---

*Sources: [[raw/you-should-not-update]]*
*Last updated: 2026-05-31*
