# Jouzu

Jouzu is Shisa AI's agentic coding harness — a curated, batteries-included repackaging of the [[Pi Coding Agent]] that treats Pi as a pinned dependency rather than a fork, and layers on the specific workflows Shisa uses daily: goals and measured loops, background jobs with batched summaries, role-assigned child agents, browser-backed search, TextGuard content scanning, searchable session history, CJK-safe terminal rendering, voice dictation, and a model palette that remembers per-model preferences. It is alpha software the team runs as its own daily driver.

---

## Key Quotes

> "Jouzu is Shisa AI's agentic coding harness, built on Pi coding agent. It comes batteries included with the tools and workflows we use every day."

The framing matters: not "a fork of Pi" but "a harness built on Pi." Jouzu pins Pi's exact tag/commit, blocks Pi's own self-update, and forwards most arguments unchanged to it. It is a *distribution* of Pi, the way a Linux distro is a distribution of a kernel plus a curated userspace — a positioning distinct from both Pi's minimalism and [[Oh My Pi (omp)]]'s maximalist fork.

> "Non-interactive first runs use `core` and do not manufacture consent."

The Japanese-profile opt-in is a small, telling piece of engineering. Only an explicit affirmative answer selects `ja`; declining or pressing Enter selects the language-neutral `core`, and non-interactive runs never invent consent. This is consent treated as a correctness property, not a checkbox.

> "Searchable session history. Recall earlier decisions and code after context compaction without keeping the whole conversation in the model's active context."

This is compaction plus retrieval as a single primitive: `pi-vcc` compacts the transcript, and `vcc_recall` fetches dropped details back on demand. It directly operationalizes the [[Agent Memory and Context]] insight that the answer to long sessions is not a bigger window but a retrieval path into what fell out of it.

> "TextGuard checks skills and web results before they reach the model. Review withheld content explicitly; scanning is not a guarantee of safety."

The honesty is the feature. Shisa does not claim TextGuard makes agents safe — it claims it withholds error-level findings until a human approves them. The disclaimer "scanning is not a guarantee of safety" is the kind of non-claim that builds trust, and it is repeated throughout the README: Jouzu "does not infer cache compatibility, model equivalence, cost, routing, privacy, retention, region, or certification guarantees."

## Key Themes

#tool #coding-agents #harness #memory #scheduling #i18n

## Critical Analysis

**The third pole of the Pi ecosystem.** [[Pi Coding Agent]] is the minimal core, [[Oh My Pi (omp)]] is the maximalist fork, and Jouzu is the curated distribution. The three are a natural experiment in how to ship an agent: extend via plugins, fork and ship everything, or pin upstream and layer a hand-picked set of daily workflows. Jouzu's bet is that curation beats completeness — it ships fewer tools than omp's 32, but each one (`schedule_prompt`, `bg_task`, goals, child agents) is a workflow the team actually uses, not a surface-area flex.

**Batteries included, but honest about limits.** The recurring pattern is a capability paired with an explicit non-claim. TextGuard scans but isn't a guarantee of safety. The status bar doesn't report quota or cost "until Jouzu has an authoritative source for those facts." Model switching blocks only on a concrete context-window calculation, then declines to infer equivalence, cost, routing, or privacy. In a field where agents routinely overclaim, Jouzu's consistent habit of *stating what it won't assert* is a differentiator disguised as documentation.

**The commercial layer is cleanly fenced.** The `shisa-api` catalog source and Shisa voice ASR are the platform hooks — a free OSS harness that quietly funnels toward Shisa's paid API. But the fencing is unusually clean: catalog refresh never follows redirects (so a bearer token can't leak to another origin), credentials are never written to config, cache, or diagnostics, voice never auto-sends and writes no recording files, and audio is captured on the local machine even over SSH. It is the honest version of the OSS-as-funnel pattern.

**Verification as default posture.** The self-updater packs the current install as a rollback artifact, verifies SHA-512 integrity, installs with lifecycle scripts disabled, then restores the previous version if verification fails. Profiles use hashes and atomic state records; a conflicting plan exits with status 3; catalog refresh validates complete bytes before activation and quarantines structurally-valid-but-suspect changes pending an exact-revision-and-digest accept. Deterministic checks are woven into the update path itself — the tool trusts its own packaging about as much as it trusts model output.

**i18n as a first-class engineering concern.** Most coding agents are ASCII-centric by default. Jouzu tests its prompt frame, session line, and status bar against CJK, full-width spaces, combining marks, and emoji using terminal display columns rather than JavaScript string length, and ships a Japanese profile plus mixed-width compatibility suite. It is a reminder that "phone-readable" and "correct in 40 languages" are the same underlying discipline, applied to a terminal.

**The open question.** Curation is a bet that the team's taste generalizes. Jouzu's feature set is explicitly "what we use every day," and the README is candid that v0.1.x is alpha and changing fast. Whether a hand-picked daily-driver workflow set outlives its maintainers' taste — or whether that taste is precisely the moat — is the unresolved question the Pi ecosystem as a whole keeps circling.

---

*Sources: [[raw/jouzu]], [[summary/jouzu]]*
*Last updated: 2026-09-08*
