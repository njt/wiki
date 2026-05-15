# Opus 4.5 Changes Everything

Burke Holland's field report of building four complete applications with Claude Opus 4.5 in VS Code — a Windows utility, a screen recorder, a Facebook scheduling app, and an order-tracking system — nearly entirely by prompting. The piece doubles as a manifesto for AI-first software engineering, including the full custom agent prompt he used. But the real payload is emotional: a seasoned developer admitting he can't tell whether he's exhilarated or depressed.

---

## Key Quotes

> "This is the model that we were promised. It's the promise of AI for coding actually delivered."

Burke's prior AI coding experience was "spaghetti code" and agents that "trashed the codebase." Opus 4.5 crossed a threshold where the output was production-quality on the first or second pass.

> "When I say that Opus 4.5 built this almost entirely, I mean it."

Said about the Facebook posting app — the most complex of the four, involving Facebook auth, Firebase, file storage, and scheduled backend posting. Burke had never used Firebase before. Opus provisioned resources via the Firebase CLI, auto-grepped cloud function logs for errors, and tagged him only for plan upgrades.

> "I'm less worried that a human needs to read the code, because I'm genuinely not sure that they do."

The most provocative claim in the piece. Burke argues the "who will maintain this?" objection assumes humans will do the maintaining. If AI wrote it and AI maintains it, human readability becomes optional. This is the logical endpoint of [[Write Only Code]] — not Slop Radius management, but full abdication.

> "Assume all code will be written and maintained by LLMs, not humans."

The first line of Burke's custom agent prompt. This framing changes everything downstream: no comments, no explanatory variable names, no "why" documentation. The prompt's nine coding principles read less like software engineering and more like a constitution for an AI-only codebase.

> "I don't know if I feel exhilarated by what I can now build in a matter of hours, or depressed because the thing I've spent my life learning to do is now trivial for a computer."

The honest note that makes this piece more than another "I built an app with AI" post. This is the emotional core of [[AI Zealotry]]'s "implementation is now mostly free" — but where Rocklin is bullish, Burke is ambivalent.

---

## Key Themes

- **#concept AI-First Coding Principles** — Burke's nine-principle agent prompt (structure, architecture, functions, naming, logging, regenerability, platform use, modifications, quality) is a concrete attempt to define what code looks like when humans are removed from the read loop. It's a companion to [[Self-Distillation]] — not training on AI output, but architecting for AI maintenance.

- **#tool Claude Opus 4.5** — The model Burke credits with the step change. Multiple wiki pages reference Opus 4.5 as the inflection point: [[How Boris Uses Claude Code]] (Boris uses it exclusively), [[Coding Agents and Complexity Budgets]] (Robinson used it for planning), [[mira-OSS]] (Claude-first design targeting Opus 4.5).

- **#pattern Voice-Driven Development** — Burke used built-in voice dictation to talk to Claude in VS Code. No fancy workflow, no planning documents. This is the far end of the spectrum from [[Addy Osmani's Workflow]] or [[TextForge Case Study]] — pure verbal intent-to-execution.

- **#concept The Security Gap** — Burke's 80% confidence in his apps' security ("too damn low") mirrors the findings in [[AI Coding Tools Create More Bugs Than They Fix]], where 40% of vibe-coded apps exposed sensitive data. The security prompt he shares is a start, but it's prompts, not [[Compound Engineering|compounding systems]]. [[Security and Sandboxing]]'s architectural constraints would help, but Burke's workflow has none of them.

- **#person Burke Holland** — Microsoft developer advocate, runs "Burke Knows Words." This piece landed hard because it came from someone inside the developer tools establishment, not an AI evangelist.

---

## Critical Analysis

**Burke is right about the threshold crossing, but wrong about what it means.** Opus 4.5 is genuinely better than what came before — multiple independent reports corroborate this. But Burke extrapolates from four greenfield solo projects to "humans don't need to read code," and that's a category error. The hard problems in software aren't building version 1.0; they're maintaining version 4.7 while 300,000 users depend on it. His apps have zero users and zero maintenance history.

**The "no human reading" argument is the most dangerous idea here, and also the most honest.** Every other field report dances around this. Burke says it plainly. If you accept the premise — AI writes it, AI maintains it — then the entire [[Software Engineering Craft]] section of this wiki becomes nostalgia. But the premise only holds if AI can maintain code it didn't write, across years of changing requirements, without introducing regressions. We have zero evidence of this. [[The Mythical Agent-Month]] applies: agents attack accidental complexity but generate new accidental complexity, and that compounds over time in ways greenfield projects don't reveal.

**His prompt is more interesting than he admits.** Burke says he has "no proof that this prompt makes a difference" and that "Opus 4.5 writes pretty solid code no matter what you prompt it with." But the prompt encodes a genuine software philosophy — flat architecture, linear control flow, feature-based grouping, full-file rewrites over surgical edits. These aren't AI-specific; they're [[Simplicity in the Age of AI-Assisted|simplicity principles]] that help humans too. I suspect the prompt matters more than he credits.

**The security gap is not a footnote; it's the thesis.** Burke's 80% confidence is accompanied by an actual security prompt (a good one) but no structural guarantees. You can't prompt your way to security. [[Feedback Loop is All You Need|Linters beat prompts]]. An app handling Facebook auth, file storage, and scheduled posting with "maybe 80%" security confidence is terrifying. [[Cybersecurity Is Proof of Work Now]] says security scales with token spend — Burke spent those tokens on features, not hardening.

**The emotional ambivalence is the real contribution.** Most AI coding posts are either boosterism or doomerism. Burke's "I don't know if I feel exhilarated or depressed" captures something truer. This tension runs through the entire wiki: [[Radical Accountability]] says taste is all that's left, but taste only matters if someone wants it. [[AI Zealotry]] says senior engineers should embrace this, but embracing it means accelerating your own obsolescence. Burke doesn't resolve this — he just sits in it, and that's more honest than any take.

**The workflow is notably thin on verification.** Compare with [[How Intercom Uses Claude Code]]'s 100+ skills, hooks, and OpenTelemetry, or [[Minions — Stripe's One-Shot Coding Agents]]'s hard two-CI-round limit. Burke's verification is "I ran it and it worked." That works at N=1. It doesn't scale. [[Slowing the Fuck Down]] argues deliberate friction is a feature, not a bug — Burke's frictionless voice-to-deploy pipeline is exhilarating precisely because it has no brakes.

---

*Sources: [[raw/opus-4-5-change-everything]]*
*Last updated: 2026-05-15*
