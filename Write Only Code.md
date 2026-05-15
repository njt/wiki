# Write Only Code

Joseph Ruscio's argument that AI-generated production code that humans never read is not a bug but an inevitable evolution of software development. The question isn't whether it will happen, but how teams prepare for it. The key concept: **Slop Radius** -- how far unintended behavior can impact before being detected or contained.

---

## Key Quotes

> "Pragmatic teams will not adopt Write-Only Code everywhere at once. They will identify where it is safe to begin and where traditional human review should remain."

> "Understanding and controlling the Slop Radius, how far unintended behavior can impact before being detected or contained, will be a critical skill for teams to develop."

> "The next generation of software engineering excellence will be defined not by how well we review the code we ship, but by how well we design systems that remain correct, resilient, and accountable even when no human ever reads the code."

## Key Themes

#agentic-coding #guardrails #cognitive-debt

The historical parallel is sharp: cloud computing removed the hardware procurement bottleneck, moving it to developer velocity. AI removes the developer velocity bottleneck, moving it to... what? Ruscio says: system design, constraint writing, and trust mechanisms.

Engineers transform from **authors and reviewers** to **systems designers and constraint writers**. The role shifts toward shaping intent over implementation, defining interfaces and failure modes, deciding what requires human oversight. This is a deeper version of what [[The Future of Software Engineering is SRE]] argues -- if you can't review all the code, your operational signals and system design are your only safeguards.

The Slop Radius concept deserves its own vocabulary. It's the blast radius for AI-generated defects. Teams that can measure and bound their Slop Radius can safely adopt write-only code; teams that can't are playing Russian roulette. This connects to the guardrails theme in [[Spec-Driven Development]] and [[Simplicity in the Age of AI-Assisted]] -- the human job is recognizing which areas are safe for write-only and which aren't.

## Critical Analysis

Ruscio names something important that others dance around. Most agentic coding discourse assumes human review remains the backstop. Write-only code says: that backstop will break, plan for it. The Slop Radius framing is genuinely useful.

The weakness: it's more manifesto than manual. How do you actually measure Slop Radius? What does "code reading coverage" look like in practice? The piece opens doors it doesn't walk through. But as a conceptual framework for the next phase of AI-native development, it's ahead of most writing in the space.

---
*Sources: [[raw/write-only-code]]*
*Last updated: 2026-05-14*