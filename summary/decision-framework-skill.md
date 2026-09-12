---
url: https://github.com/FeroVolar/Decision-Framework-Skill
title: Decision Framework Skill
author: Fero Volar
date_fetched: 2026-07-03
date_published: 2026
topics:
  - claude-code
---

# Decision Framework Skill — Full Analysis

## Source

- Repository: https://github.com/FeroVolar/Decision-Framework-Skill
- Author: Fero Volar
- License: MIT
- Size: 3 files (SKILL.md: 215 lines, README.md: 187 lines, LICENSE: 21 lines)
- Type: Claude Code skill (prompt template, no executable code)
- Inspiration: Jason Fried's "The 37signals Guide to Making Decisions" (38-question framework)

## What It Is

A Claude Code skill that transforms Jason Fried's static 38-question decision-making framework into an interactive, adaptive coaching conversation. Rather than presenting a checklist, it guides users through structured reflection, asking only the questions relevant to the specific decision at hand.

## Architecture

The skill is entirely prompt engineering — a 215-line SKILL.md file with YAML frontmatter defining the skill metadata:

```yaml
name: decision-framework
description: A structured decision-making coach inspired by practical business decision principles.
```

**Phase structure (4 phases):**

1. **Phase 1 — Understand the Decision** (lines 45-55): Establish context — what decision, why now, who's affected, desired outcome. Gates further exploration.

2. **Phase 2 — Decision Exploration** (lines 58-161): The core — 12 topic areas organized as reflection prompts:
   - Purpose (4 questions)
   - Ownership (3 questions)
   - Time Perspective (3 questions)
   - Alternatives (4 questions)
   - Reversibility (3 questions)
   - Intuition vs Evidence (3 questions)
   - Consequences (5 questions)
   - Missing Information (3 questions)
   - Stakeholders (3 questions)
   - Values (3 questions)
   - Effort (4 questions)
   - Learning (4 questions)

3. **Phase 3 — Reflection** (lines 164-174): Summarize the decision, assumptions, risks, missing information, and strongest arguments — remaining neutral.

4. **Phase 4 — Recommendation** (lines 178-198): Provide recommended direction with: why strongest, what assumptions it depends on, what could invalidate it, confidence level (High/Medium/Low), and the smallest useful next step.

**Style section** (lines 201-215): Be calm, concise, avoid corporate language, prefer questions over lectures, challenge assumptions respectfully, never overwhelm.

## Key Techniques

### 1. Adaptive Question Selection (Not a Checklist)

The defining technique. The README explicitly states: "Sometimes that means asking three questions. Sometimes fifteen. Never all of them."

This is implemented via the Core Principles (lines 29-42):
```
Do not dump all questions at once.
Instead:
1. Understand the decision.
2. Ask only the most relevant questions.
3. Skip irrelevant ones.
4. Keep the conversation flowing naturally.
5. Summarize the reasoning before recommending a path.

Questions are thinking tools—not a checklist.
```

The skill trusts the model to calibrate question count to decision complexity. This is the opposite of most conversational agent designs, which default to exhaustiveness.

### 2. Phase Gating with Mandatory Completion

Each phase has explicit completion criteria. The model is told "Only after understanding the context continue" (line 54) and "Only after completing reflection provide" recommendations (line 180). This prevents premature recommendations — a common failure mode where AI assistants jump to solutions.

### 3. Neutrality-by-Structure

The skill separates analysis (Phase 2-3) from recommendation (Phase 4). The reflection phase is explicitly instructed to "Remain neutral" (line 174). This structural separation is more reliable than prompting alone — the model can't accidentally slip into recommendation during exploration because the phases create a sequential dependency.

### 4. Language Auto-Detection

Simple but practical: "If the conversation starts in English, continue in English. Otherwise use the language already used by the user" (lines 20-25). No configuration needed.

### 5. Minimal Viable Intervention

The 4-line style guide embodies the philosophy:
```
Be calm.
Be concise.
Avoid unnecessary corporate language.
Prefer questions over lectures.
```

This is remarkably short for a coaching skill — most would over-specify tone. The restraint trusts the base model's conversational ability and only constrains what matters (calm, concise, non-corporate, Socratic).

## Design Decisions

### Why a prompt template, not code?

The entire skill is 215 lines of markdown. No API calls, no external tools, no state management. This is the simplest possible implementation — it delegates everything to the base model's reasoning capability. The trade-off: no persistence between sessions, no access to external data, no multi-turn state beyond what's in the context window. For a coaching conversation, this is sufficient. For a system that needs to track decisions over time, it would be inadequate.

### Completeness vs. Engagement

The explicit decision to never ask all 38 questions trades completeness for conversation quality. A human coach wouldn't work through a checklist either — they'd pick what matters. The skill models this intuition directly.

### Fixed Phase Order vs. Dynamic Routing

The phases are strictly sequential (Understand → Explore → Reflect → Recommend), with no branching or skip logic. For a coaching conversation, this is appropriate — you can't meaningfully recommend before understanding. But it means the skill can't handle users who want to jump straight to a recommendation (it will redirect them to context first).

### No External Knowledge Integration

Unlike skills that call external APIs or databases, this skill works entirely within the conversation. It can't look up market data, check precedents, or verify claims. This is a deliberate simplicity trade-off — the skill is about improving thinking quality, not providing information.

## How It Differs from Other Claude Code Skills

### vs. [[Load-Bearing Assumptions]]

Load-Bearing Assumptions is a multi-agent workflow (finder → strategist → parallel validators) with human-in-the-loop checkpoints. It's designed for verification, not coaching. Decision Framework is a single-agent conversational skill — simpler architecture, different domain. Both share the pattern of structured phase gating.

### vs. [[Thought Refiner Skill]]

Both are Claude Code skills that trust model judgment on output density. Thought Refiner is even more minimal (15 lines) and is about sharpening vague input into questions. Decision Framework is more structured (4 phases, 12 topic areas) and is about coaching through a specific decision. Both share the instinct that less specification is more — constrain the output, not the reasoning.

### vs. Generic "Act as a Coach" Prompts

The key difference: Decision Framework provides a specific taxonomy of question types (12 categories), not just a coaching stance. This gives the model concrete angles to explore rather than generic "ask questions." The taxonomy is derived from a specific practitioner's framework (37signals), not invented by the prompt author — it's domain knowledge encoded as prompt structure.

### vs. [[Agent Identity]]

Agent Identity argues that meaningful agent interaction requires identity and continuity — a stance the agent holds because it has ground to stand on. Decision Framework operates within a single session with no persistence — it's a coaching stance that exists only for the conversation. This is a fundamentally different model of value: transient quality improvement vs. persistent relationship.

## Critical Assessment

**Strengths:**
- The adaptive questioning pattern is genuinely innovative — most agent skills over-specify, this one under-specifies deliberately
- Phase gating prevents the most common conversational failure mode (premature solutions)
- The 37signals framework provides substantive domain knowledge that makes the prompt useful, not just a vibes-based "be a coach"
- Extreme simplicity (215 lines, no dependencies) makes it easy to understand, modify, and deploy

**Weaknesses:**
- No persistence between sessions — can't track decisions over time or learn from past conversations
- No external knowledge integration — can't verify claims or look up information
- The fixed phase order prevents users from jumping to what they want
- The quality depends entirely on the base model's coaching ability — there's no validation or guard against bad advice
- Only 12 of the original 38 questions are organized into the taxonomy — about 2/3 of the source material is unused

**The most interesting pattern:** The skill's value proposition isn't adding capability — it's adding restraint. Most AI tools promise to do more, faster. This one promises to slow down, ask better questions, and not jump to conclusions. In a landscape of acceleration tools, a deceleration tool is a genuinely novel category.
