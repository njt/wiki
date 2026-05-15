# Better Error Messages

Wix's "Errorgate 2021": the company audited 7,643 error-related content pieces and overhauled their error messaging across the platform. The five rules: say what happened, say why, provide reassurance, give a way out, help them fix it.

---

## Key Quotes

> "Generic errors are the result of bad development and product. We must all care about it together." -- CEO Avishai Abrahami

> "Write it like you're talking to a friend"

> The team discovered they were "more like that friend who loves to gossip, but doesn't pick up the phone when life gets hard."

## Key Themes

#error-handling #api-design

The four problems bad error messages share: inappropriate tone (undermining trust), technical jargon (confusing users), blame-shifting (shaming users or looking unprofessional), and generic messaging (when specific solutions exist).

Good error messages need: clear explanation of what happened and why, reassurance about what's unaffected, empathetic language without over-apologizing, actionable solutions, and clear paths to resolution.

The implementation story is as important as the principles. Wix mapped errors in code, prioritized by frequency and user impact, and tracked fixes across PMs, developers, and writers using Monday.com boards. Error messaging is a cross-functional concern -- not something you can delegate to the content team after the fact.

The "gossip friend" metaphor is sharp: most products are chatty about features but go silent when things break, which is exactly when users need help most.

Connects to [[dotnet Slopwatch]] (catching AI-generated code that hides errors behind empty catch blocks) and the broader software craft theme of treating error handling as a first-class product concern.

## Critical Analysis

The principles are timeless and universally applicable -- they work for CLI error messages, API responses, and agent failure states, not just web UIs. The scale of the effort (7,643 items!) demonstrates that error messaging debt accumulates silently in every product. The cross-functional approach (PMs + developers + writers) is essential but politically difficult -- who owns error messages? The article's implicit answer (everyone, with writers empowered to challenge generic solutions) is right but requires organizational support. What's missing is measurement: did better error messages reduce support tickets? Improve retention? The qualitative case is strong, but quantitative evidence would make this more persuasive to skeptical leadership.

---
*Sources: [[raw/better-error-messages]]*
*Last updated: 2026-05-14*
