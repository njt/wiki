---
url: https://www.oreilly.com/radar/who-owns-the-code-claude-wrote/
title: "Who Owns the Code Claude Wrote?"
author: Sena Evren
date_fetched: 2026-07-05
date_published: 2026-06-15
source: O'Reilly Radar (originally appeared on Legal Layer newsletter)
topics:
  - agent-coding-workflow
---

## TL;DR

Agentic coding tools like Claude Code, Cursor, and Codex produce code that may be uncopyrightable, employer-owned, or contaminated by invisible open source licenses. Some issues are settled law; others are actively contested.

## The Opening Hook

On March 31, 2026, Anthropic accidentally published 512,000 lines of Claude Code's source code via a missing config file. The codebase was mirrored on GitHub before sunrise, rewritten in Python by another user, and the resulting "claw-code" repository became the fastest ever to reach 100,000 GitHub stars in a single day. DMCA takedowns followed, raising an unresolved question: if Claude Code was "predominantly written by Claude itself," can Anthropic assert copyright over it? The piece argues this incident "compressed every open question about AI-generated code ownership into a single news cycle."

## The Copyright Baseline

Copyright only protects work created by a human. The US Copyright Office has consistently held this, and the DC Circuit upheld it in *Thaler*. The Supreme Court declined to hear the Thaler appeal in March 2026. Two limits: (1) Thaler involved zero human involvement (AI listed as sole author), not the harder case of AI-assisted work; (2) Thaler involved visual art, not code — the logic extends, but no direct precedent exists yet.

## Meaningful Human Authorship

The determinative phrase is "meaningful human authorship," which the Copyright Office has deliberately refused to quantify. Evidence of genuine human creative decisions: choosing the architecture, deciding what to reject, restructuring the output to fit a specific design. Simply specifying an objective is insufficient.

In an agentic workflow (e.g., a one-line prompt to build a rate limiting module), the author's contribution is unresolved. The honest assessment: "probably yes for modules you substantially redirected, probably no for code you accepted verbatim, and unclear for everything in between."

*Allen v. Perlmutter* (currently unresolved): artist Jason Allen challenging denial of registration for work created with "more than 600 detailed prompts and subsequent editing in Photoshop." The Office acknowledged Photoshop edits as human-authored but denied registration for AI-generated elements.

*Zarya of the Dawn* precedent: partial protection — registration granted for human-authored text but denied for Midjourney-generated images. Evren argues developers can apply this principle now: architecture docs, design decisions in commit messages, ADRs, and prompt logs showing deliberate redirection "may be protectable as human-authored expression even if the code they produced is not."

## Employer Ownership (Work-for-Hire)

Even if code is copyrightable, it likely belongs to the employer. The work-for-hire doctrine means code created by an employee within their employment scope "is owned by the employer, who is treated as the legal author," regardless of whether it was hand-written or AI-generated.

Broad IP clauses covering "any work product created using company equipment or resources" or "company-licensed tools" are a specific danger. Example: a senior developer in San Francisco whose company claimed ownership of his personal fitness tracking app, arguing that because Claude had "access to open work files in the IDE, any AI output was a derivative work of company IP."

Advice: "If you are building something on the side, use a personal account, a personal machine, and tools you pay for yourself."

## Open Source Contamination

AI coding tools train on public code including GPL, LGPL, and other copyleft-licensed code. When an AI reproduces "substantial verbatim portion of GPL-licensed code from its training data" and a developer ships it commercially without releasing source, a copyleft violation may result. "I did not know" is not a defense.

The chardet community dispute (early 2026): a developer used Claude to rewrite the Python library and rereleased it under MIT, arguing it was a "clean room" implementation free of the original LGPL license. "The chardet dispute did not resolve cleanly and no court has issued a definitive ruling."

The *Doe v. GitHub* litigation, working through the Ninth Circuit as of April 2026, asks whether Copilot reproduces licensed code without attribution. Regardless of outcome, the litigation has already changed behavior: "GitHub Copilot added duplicate detection filters, and acquisition due diligence now routinely includes an AI codebase license scan."

## Four Recommended Actions

1. **Run a license scan** on AI-assisted code using FOSSA, Snyk Open Source, or Black Duck.
2. **Document human creative contributions as you go.** A commit message like "Restructured Claude's module architecture, rejected initial state management approach, rewrote error handling from scratch" is evidence; "Add rate limiting module" is not.
3. **Read the IP clause in your employment contract** before building anything on the side.
4. **Check which Anthropic plan you are on** before shipping commercially. Consumer plans have narrower indemnification; API/enterprise plans include IP indemnification. Neither covers downstream GPL violations.

## What's Settled vs. What's Not

**Settled:** (1) Works lacking human authorship are uncopyrightable. (2) Work-for-hire applies regardless of how code was generated. (3) Verbatim copying of GPL-licensed code violates the license.

**Emerging consensus, no definitive rulings:** (1) How much human direction is enough for meaningful authorship in agentic workflows. (2) Whether AI output reproducing training data patterns counts as verbatim copying.

**Genuine speculation:** Whether any of this will be litigated at scale in the near term. Most code copyright claims never reach court; the unsettled questions become concrete today in "M&A due diligence and institutional fundraising."

## Disclaimer

The piece is informational and does not constitute legal advice.
