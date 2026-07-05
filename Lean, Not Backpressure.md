# Lean, Not Backpressure

kqr argues that "backpressure" is the wrong metaphor for what Lucas Costa is actually describing: lean manufacturing's neglected half — managing the unstable input of people. The sharper insight is that AI coding tools make this undeniable, because you can't blame a robot for quality failures the way you blame a human.

---

## Key Quotes

> "A bad system will beat a good person every time."

Deming's maxim, quoted as the piece's philosophical anchor. kqr uses it to argue that when output quality is poor, the system is the culprit — not the individual worker. This is old wisdom, but it lands differently when the worker is a robot and the manager can't reach for the "lazy" or "incompetent" script.

> If your system "requires humans to not make mistakes, then whose fault is this really?"

A safety-engineering principle kqr adopts as a general design heuristic. The framing is deliberately provocative: it inverts the default assumption that workers should be perfect and asks why the system was designed to be fragile. This is the core of the lean argument — don't demand perfection, design processes that tolerate imperfection.

> "We always had to do that, even with people, but with robots it's painfully obvious."

The punchline. AI tools don't change the truth — they just remove the scapegoat. Managers who blamed developers for bugs now face a tool that can't be shamed, disciplined, or fired, and suddenly process design matters.

## Three Lean Practices

kqr distills three lean manufacturing concepts that apply directly to AI-assisted software development:

**Single-piece flow** — Work on one item at a time so downstream can reject defective output before it accumulates. In agentic development terms: don't let an agent generate 5,000 lines before review. Small batches, fast rejection.

**Autonomation (jidoka)** — Equip machines to detect problems and halt on their own. The agentic equivalent: automated tests, linters, and guardrails that stop bad output before a human even sees it. The machine polices itself.

**Poka-yoke** — Design processes that make correct results inevitable by construction. This is the highest-leverage move: shape the workflow so the agent *can't* produce the wrong thing, rather than catching it after.

## Themes

- `#pattern` — Lean manufacturing as a lens for agent-assisted workflows
- `#concept` — System responsibility vs. individual blame in quality
- `#tool` — Poka-yoke, jidoka, and single-piece flow as transferable practices
- `#comparison` — Backpressure vs. lean: two different diagnoses of the same symptom

## Critical Analysis

**The strength of this piece is its reframe.** "Backpressure" is a plumbing metaphor — it signals "slow down upstream." But Costa's actual prescriptions are about upstream doing *different* work, not *less* work. kqr catches this and swaps in a richer metaphor. That's not pedantry; naming determines what you build. A team chasing "backpressure" builds queues and throttles. A team chasing "lean" builds single-piece flow and mistake-proofing.

**The robot argument is clever but incomplete.** Yes, AI tools expose the blame-the-worker reflex by removing the worker. But the piece skips an uncomfortable corollary: if you can't blame the robot, you also can't *train* the robot through social feedback. A human developer, shown a bug, learns. A coding agent, shown a bug, needs a better prompt or a different harness. The "system responsibility" argument gets *more* demanding with AI, not less — you can't coast on human adaptability.

**The lean-to-software mapping is undertheorized.** kqr gestures at the three practices but doesn't work through what they look like in an agentic workflow. Single-piece flow maps intuitively (small PRs, fast review cycles). Autonomation maps to CI/CD guardrails. But poka-yoke in software is harder — what does "impossible to get wrong" mean when the output is code? The closest analog is type systems and formal verification, and those are expensive. The piece would be stronger with a worked example.

**The Deming quote does a lot of heavy lifting.** Deming's insight is real, but it's also a convenient escape hatch: blame the system, not yourself. The risk is that "system responsibility" becomes a way to avoid individual accountability entirely. The better reading — and the one kqr intends — is that system design *is* the accountability. If you're the one designing the process, you don't get to blame the workers when it fails.

## Connections

- [[Lean Software Production]] — Matt Wynne's three-pillar framework applies lean explicitly to agentic software development. kqr's piece is the upstream philosophical justification; Wynne's is the operational playbook.
- [[Queues Don't Fix Overload]] — Fred Hebert's 2014 argument that queues treat symptoms, not causes. The backpressure argument kqr is pushing back against is essentially a queue-management strategy; Hebert's bottleneck-identification approach aligns with lean's "fix the system" ethos.
- [[Guardrails and Feedback Loops]] — Linters beat prompts. This is jidoka in software: equip the system to detect and halt on defects automatically.
- [[Engineering for Bounded Cognition]] — Software methodology as prosthetic cognition. Designing systems for imperfect humans is the same design challenge as designing systems for imperfect AI.
- [[The PM's Playbook for Shipping AI Features]] — The "we'll harden it later" anti-pattern is exactly what lean says not to do. Quality must be designed into the process from the start.

---
*Source: [[raw/lean-not-backpressure]]*
*Last updated: 2026-07-05*
