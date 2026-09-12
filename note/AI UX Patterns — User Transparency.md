# AI UX Patterns — User Transparency

Kathryn Grayson Nanz's practitioner checklist for the transparency layer of AI UX: four concrete patterns — granular permissions, revocable/forgettable memory, up-front cost estimates, and a visible signal when the AI acts autonomously — united by one thesis, that users won't integrate what they can't see and understand, so transparency is the adoption lever, not a compliance chore.

---

## Key Quotes

> "Most users won't integrate what they can't see and understand."

The thesis in one line, and a clean bridge to [[Experience Design for Agents]]'s claim that adoption is won or lost on experience design, not capability. Where Kemple argues *authority must be visible*, Nanz is the field guide to *what* to make visible: capability scope, data reach, cost, and who's currently driving.

> "Permissions are also not a one-and-done situation. A user might feel comfortable allowing an action to happen once under their direct supervision, but don't want to allow it permanently."

The most useful sentence in the piece, because it rejects the binary consent model. Nanz enumerates the gray zones — one-time vs. permanent, one folder vs. all folders, notify-each-time vs. silent — and this maps directly onto the granularity [[The Agent Access Model]] enforces at runtime (task-scoped credentials) and that [[How We Contain Claude]] discovered is invisible in a 93%-approval prompt wall.

> "If you want to create a system that can 'remember' things, you also need to make sure it can 'forget' as well."

Memory as a symmetric obligation, not a feature. The user must hold final say over what gets remembered, and see a history of every grant so they can revoke it. This is the *user-facing* face of what [[Golem Covenant]] makes a mandatory, tested requirement (return-to-dust revocation) and what [[Agent Memory and Context]] treats as an engineering problem — Nanz is pointing out the ordinary user experiences it as a trust problem.

> "There needs to be a banner, sidebar, outline or some other kind of indication that the user is no longer driving the interaction."

The visual-state idea is the freshest one here, and it's a small thing with a large purpose: not just transparency (the user understands system state) but coordination (the user doesn't unintentionally interrupt a long-running process). It's the concrete UI counterpart to [[Real-Time Multiplayer Interfaces]]'s claim that *interruption is the interface* — you can't interrupt correctly if you can't see the agent's current mode.

---

## Key Themes

#concept **Transparency as the adoption lever.** Nanz's opening move is economic, not ethical: "the more transparent we can make these features, the less hesitation our users will have adopting them." Transparency here is a growth strategy, which is a distinct framing from the safety-first lens of [[Security and Sandboxing]].

#pattern **Granular, revocable permissions.** Permissions aren't boolean. The design space is one-time vs. permanent, scoped vs. global, notified vs. silent — and every action (read vs. send, reference vs. delete, run vs. view) deserves its own sign-off. The ideal flow adds a history ledger the user can inspect and revoke at any time.

#pattern **Remember implies forget.** Data collection is a two-way door. A "remembering" system that can't forget — or that forgets only on the vendor's schedule, not the user's — is a privacy liability in waiting. The user, not the developer, owns the memory.

#pattern **Cost transparency at approval time.** Beyond approving the *steps*, users need to know the *price* — time and money/tokens — before they commit. Rough estimates are enough to let a user revise a request that will exceed their comfort, which doubles as a soft budget control.

#pattern **Visible autonomy.** A persistent visual marker of "the AI is acting now" separates the user's intent from the agent's execution, and prevents the user from interrupting a process they've stepped away from.

---

## Critical Analysis

Nanz is writing a checklist, not a theory, and it shows in both directions. The strengths are the *specific* failures she names: the "notify me each time" preference, the "one folder but not every folder" scope, the user who wants to watch an action before trusting it permanently. These are real requests real users make, and most permission UIs flatten them into a single allow/deny. The "remember implies forget" symmetry is the kind of line that reorganizes a feature backlog.

But the piece is conspicuously innocent about consent fatigue. It assumes a user who *wants* to make a considered decision at every prompt, and that's exactly the assumption [[How We Contain Claude]] falsifies with a single number: a 93% approval rate isn't a boundary, it's consent theater. Nanz's own answer to this is partial — the history ledger and revoke-anywhere controls are good hygiene, but they shift the burden onto the user to notice misuse after the fact, rather than preventing rubber-stamp approval in the first place. [[The Agent Access Model]] is the hard-edged version of the same instinct: move the deliberate decision up to template-design time, because a per-action approval that's always granted is a ritual.

The cost-estimate point is underdeveloped. "Rough estimates generally give users enough information" is doing a lot of work — token cost is notoriously hard to predict for multi-step agent work, and a wrong estimate either scares users off or trains them to ignore it. This is a hard problem the piece waves at rather than engages.

The autonomy-visual-signal idea is the standout, and it's the one that connects outward: it's the UI complement to the interruption economics in [[Real-Time Multiplayer Interfaces]], and a reminder that transparency isn't only about *disclosure* but about *shared state* — the user needs to know who's driving before they can be a collaborator rather than a rubber stamp, which is the argument [[Understand to Participate]] makes from the safety side.

---

## Related Pages

- [[Experience Design for Agents]] — the theory; Nanz is the field guide to making "authority visible"
- [[Intent Is the Interface]] — inverted initiation ("act, then surface for approval") is what the autonomy signal must mark
- [[Real-Time Multiplayer Interfaces]] — interruption as the interface; you can't interrupt correctly without visible agent state
- [[The Agent Access Model]] — runtime granularity and revocation as the enforcement counterpart to Nanz's UX
- [[How We Contain Claude]] — the 93% approval rate that complicates any permission-prompt design
- [[Golem Covenant]] — tested, default-deny revocation as the moral-engineering pole of "remember implies forget"

---
*Sources: [[raw/ai-ux-patterns-user-transparency]], [[summary/ai-ux-patterns-user-transparency]]*
*Last updated: 2026-08-26*
