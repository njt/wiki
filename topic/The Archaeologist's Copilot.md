# The Archaeologist's Copilot

Nik Malykhin's field report on modernizing a 20-year-old Java 1.5 codebase using AI as a force multiplier — not a tour guide. The core insight: AI defaults to optimism, and optimism in legacy restoration is fatal. Success comes from shifting from "Tourist Prompts" ("how do I run this?") to "Archaeologist Prompts" (forensic code audit, containment-first strategy, AI-compiler feedback loops), treating the AI as a tireless junior who handles tedious translation layers while the human owns strategy, architecture judgment, and the decision to stop.

---

> "AI defaults to optimism. [...] We needed to stop acting like Tourists and start acting like Archaeologists."

This is the essay's thesis distilled into a sentence. The "Tourist Prompt" elicits confident, plausible, and wrong answers — hallucinated dependencies, assumed standard layouts, example code that hides thread-safety bugs. The "Archaeologist Prompt" assigns the AI a persona (Senior Legacy Systems Architect), forbids summarization, and orders a structured forensic audit across four dimensions: era, architecture, data flow, and safety.

> "Without changing a single byte of historical source code."

Malykhin's containment-first approach — wrap the legacy system in Docker, use Docker Compose to bend reality (network aliases, mounted paths) to match hardcoded test assumptions — is the article's most practical contribution. It inverts the usual modernization instinct (change things until they work) and instead establishes a verifiable baseline first. Only then does modernization begin, making every subsequent failure attributable to the modernization step, not pre-existing rot.

> "The build turned red. This was a massive narrative victory."

The discovery that tests were lying — swallowing exceptions and exiting with code 0 — and the decision to strip try-catch blocks so failures crash properly is a masterclass in test hardening. An honest red build is more valuable than a lying green one. This applies to any legacy system, AI-assisted or not.

> "Momentum is oxygen."

Malykhin aborts a TestContainers migration that was spiraling into a Big Bang refactor, accepting the "External Sidecar" pattern instead. This is the essay's best piece of practical judgment: knowing when to stop a modernization that's becoming its own project. Ground-level pragmatism over over-engineered perfection.

---

## Key Themes

- **#pattern** — Tourist vs. Archaeologist prompt: the distinction between asking an AI "how do I run this?" and directing it to perform a forensic code audit. One produces plausible lies; the other produces evidence-grounded analysis.
- **#concept** — Containment-first modernization: wrap legacy code in Docker to establish a verifiable baseline before making any changes. Every failure after the baseline is attributable to modernization, not pre-existing rot.
- **#pattern** — AI-compiler feedback loop: feed exact compiler warnings to the AI with targeted refactoring prompts, iterate until zero warnings. The compiler is the deterministic verifier; the AI is the translation engine.
- **#concept** — Lying tests: legacy test suites that pass by swallowing exceptions and exiting with code 0. An honest red build beats a lying green one.
- **#tool** — Docker as time machine: using Docker network aliases and volume mounts to recreate 2008-era hardcoded paths and hostnames without changing source code.

---

## Critical Analysis

Malykhin's essay is a clinic in AI-assisted legacy modernization, and its core contribution is taxonomic rather than technical: naming the "Tourist Prompt" vs. "Archaeologist Prompt" distinction. This is a genuinely useful diagnostic — one that generalizes well beyond Java legacy code. Whenever an AI-generated answer feels too clean, ask whether you're being a tourist.

The containment-first strategy is the right instinct, but the article underplays how fragile Docker-based time capsules are. A Docker image pinned to 2008 dependencies is a ticking clock; OS package mirrors go offline, base images are deprecated, and the archaeology problem shifts from the codebase to the infrastructure. This isn't a flaw in the approach — it's the right first step — but the article's cheerful "we restored a 20-year-old application!" ending elides the half-life of Docker-based preservation.

The decision to target Java 8 (because it's the last version that compiles Java 1.5 and runs natively on Apple Silicon) is a perfect example of the kind of constraint-juggling that AI can't do. The AI would happily recommend Java 17 or Java 21; it takes a human to recognize that the Venn diagram of "compiles Java 1.5" and "runs on ARM64" has exactly one occupant. This is the essay's strongest implicit argument for human agency: AI accelerates within constrained spaces; humans define the constraints.

The TestContainers abort is the most honest section and the one most modernization essays would omit. "Momentum is oxygen" should be a mantra for anyone doing AI-assisted refactoring. The AI doesn't know when to stop; you do.

What's missing: Malykhin doesn't address the question of whether modernization was actually worth it. The codebase ran; it had been running for 20 years. The essay assumes modernization is self-evidently valuable, but for a system with no active users or maintenance burden, the archaeology might have been the point. A cost-benefit reflection would strengthen an already strong piece.

---

*Sources: [[raw/archaeologist-copilot]]*
*Last updated: 2026-07-18*
