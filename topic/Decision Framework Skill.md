# Decision Framework Skill

A Claude Code skill by Fero Volar that transforms Jason Fried's 37signals decision-making guide — 38 questions used internally as mental models — into an adaptive, conversational coaching experience. Rather than dumping a checklist, it asks only the questions that meaningfully improve the current decision, working through four structured phases (Understand → Explore → Reflect → Recommend) before offering a recommendation. The core insight: the skill's value is not doing more, but doing less — it's a deceleration tool in a landscape of acceleration tools.

---

## Architecture

The entire skill is 215 lines of Markdown in `SKILL.md` — no executable code, no external dependencies, no state management. It's a pure prompt template deployed as a Claude Code skill.

**YAML frontmatter** defines the skill name (`decision-framework`) and a one-line description. The body is an instruction set for Claude organized into four sequential phases:

| Phase | Purpose | Lines |
|-------|---------|-------|
| 1. Understand | What decision, why now, who's affected, desired outcome | 45-55 |
| 2. Explore | 12 topic areas of targeted questions (Purpose through Learning) | 58-161 |
| 3. Reflect | Neutral summary: assumptions, risks, missing information, strongest arguments | 164-174 |
| 4. Recommend | Direction + assumptions + confidence level + smallest next step | 178-198 |

**Phase gating** is explicit: "Only after understanding the context continue" (line 54) and "Only after completing reflection provide" recommendations (line 180). This creates sequential dependency that prevents the model from jumping to solutions prematurely.

**The 12 topic areas** in Phase 2 provide a taxonomy of question types derived from the 37signals framework: Purpose, Ownership, Time Perspective, Alternatives, Reversibility, Intuition vs Evidence, Consequences, Missing Information, Stakeholders, Values, Effort, Learning. Each area has 3-5 specific prompts — a total of roughly 42 prompting angles, though the skill explicitly says to use only the relevant subset.

The **Style section** (lines 201-215) is remarkably brief: calm, concise, avoid corporate language, prefer questions over lectures, challenge assumptions respectfully, never overwhelm.

---

## Key Techniques

### Adaptive Question Selection

The defining technique. The Core Principles (lines 29-42) explicitly reject the checklist approach:

> "Do not dump all questions at once. Instead: understand the decision, ask only the most relevant questions, skip irrelevant ones, keep the conversation flowing naturally."

This trusts the model to calibrate question density to decision complexity — three questions for simple decisions, fifteen for complex ones, never all of them. This is the opposite of most conversational agent designs, which default to exhaustiveness.

### Structural Separation of Analysis and Recommendation

The skill separates exploration (Phase 2-3) from recommendation (Phase 4) with an explicit neutrality instruction: "Remain neutral" during reflection. This structural split is more reliable than prompting alone — the model can't accidentally slip into recommendation mode during analysis because the phases create a hard boundary.

### Domain Knowledge as Prompt Structure

The 12 topic areas aren't generic "coaching questions" — they're derived from a specific practitioner's framework (37signals' internal decision-making guide). This gives the model concrete angles to explore rather than generic "ask good questions." The taxonomy is the prompt's real intellectual property.

### Minimal Viable Constraint

The style guide is four lines. It doesn't over-specify tone, persona, or vocabulary — it trusts the base model's conversational ability and only constrains what matters (calm, concise, non-corporate, Socratic). Compare with coaching skills that spend 50+ lines on voice and persona.

---

## Design Decisions

**Prompt template, not code.** No API calls, no tools, no persistence. The simplest possible implementation delegates everything to the base model. Sufficient for a coaching conversation; inadequate for tracking decisions over time or verifying claims.

**Completeness sacrificed for engagement.** The explicit choice to never ask all questions trades systematic coverage for conversation quality. A human coach wouldn't work through a checklist either.

**Fixed phase order, no branching.** Users who want to jump to a recommendation get redirected to context first. This is philosophically correct (you can't recommend before understanding) but inflexible for return users who want to skip.

**No external knowledge integration.** The skill can't look up data, check precedents, or verify claims. It's about improving thinking quality, not providing information.

---

## Comparison Notes

**vs. [[Load-Bearing Assumptions]]**: Both are Claude Code skills with structured phase gating. Load-Bearing Assumptions is a multi-agent verification workflow (finder → strategist → parallel validators) with human-in-the-loop checkpoints. Decision Framework is a single-agent conversational skill. One verifies assumptions; the other coaches decisions. The shared pattern is that structured phases prevent the most common LLM failure modes (premature action in one case, premature solutions in the other).

**vs. [[Thought Refiner Skill]]**: Both trust model judgment on output density rather than prescribing fixed counts. Thought Refiner (15 lines) is even more minimal — it sharpens vague input into questions. Decision Framework (215 lines) adds structured coaching through a domain-specific taxonomy. Both share the instinct that less specification is more.

**vs. Agent Identity**: Agent Identity argues that meaningful interaction requires identity and continuity — a persistent stance. Decision Framework operates within a single session with no persistence. This is a fundamentally different model of value: transient quality improvement vs. persistent relationship.

**vs. Generic "Act as a Coach" prompts**: The difference is the 12-topic taxonomy. Generic coaching prompts say "ask good questions." Decision Framework provides a specific catalogue of question types from a known practitioner's framework — domain knowledge encoded as instruction structure.

---

## Tags

#skill #claude-code #decision-making #coaching #prompt-engineering #conversation-design #agent-patterns

## Related

- [[Load-Bearing Assumptions]] — Another Claude Code skill with phase-gated workflow
- [[Thought Refiner Skill]] — Minimal skill that also trusts model calibration
- [[Awesome Agentic Patterns]] — Catalogue of agent patterns including skills
- [[Prefix Effects]] — How prompt structure shapes agent behavior
- [[MinMax Skills]] — Another curated skills collection
- [[Simmer Skill]] — Claude Code skill design pattern

---
*Source: [[summary/decision-framework-skill]] — https://github.com/FeroVolar/Decision-Framework-Skill*
*Fetched: 2026-07-03*
