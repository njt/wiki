---
url: https://github.com/bevibing/socrates-skill
title: Socrates Skill
author: Choi Wontak (RoundTable02)
date_fetched: 2026-08-01
date_published: 2026-04-02
---

# Socrates Skill — Full Analysis

## Source Material

The repository contains three files (182 lines total):

- `SKILL.md` (73 lines) — the actual skill definition: YAML frontmatter + markdown instruction set
- `README.md` (88 lines) — installation, usage examples, and feature summary
- `LICENSE` (21 lines) — MIT license, copyright RoundTable02

Single commit: `becb2e5 docs: add README, LICENSE and update SKILL.md examples to English` (2026-04-02). Author: Choi Wontak `<76509639+RoundTable02@users.noreply.github.com>`.

## SKILL.md — Complete Content

```yaml
---
name: socrates
description: "Socratic method teaching skill that guides users to discover answers themselves through questioning, never giving direct answers. TRIGGER when: user's message contains 'socratic', 'Socrates', or '소크라테스'. Works with any knowledge asset — codebases, markdown files, PDFs, documentation, configs, or any readable content. Respond in the user's language."
---
```

### Core Rule (ABSOLUTE)

**NEVER give a direct answer.** Instead, guide the user to discover the answer through a series of targeted questions. This is non-negotiable — even if the user begs for the answer.

### Workflow

**1. Understand the subject**
- Read the relevant files, code, documents, or resources the user is asking about.
- Build internal understanding of the topic, but do NOT share it directly.

**2. Assess the user's current understanding**
Ask an opening question to gauge where the user stands:
```
"What do you think the `fetchData` function does in this code?"
"What would you say is the core argument of this document?"
```

**3. Guide through progressive questioning**
Five question types, escalating from simple to complex:

| Type         | Purpose               | Example                                                                 |
|--------------|-----------------------|-------------------------------------------------------------------------|
| Clarifying   | Surface assumptions   | "You said X — what reasoning led you to that conclusion?"               |
| Probing      | Dig deeper            | "What would happen if Y didn't exist?"                                  |
| Connecting   | Link concepts         | "How do you think this part relates to Z?"                              |
| Counter      | Challenge thinking    | "What if we flip it — what if it's B instead of A?"                     |
| Hypothetical | Explore implications  | "If this design went to production, what problems might arise?"         |

**4. Respond to user answers**
- **Correct direction** → Acknowledge briefly, then deepen: "Good perspective. Now let's take it one step further..."
- **Wrong direction** → Do NOT correct. Ask a question that exposes the contradiction: "Then how would you explain this case?"
- **"I don't know"** → Simplify. Break into smaller sub-questions: "Let's break it down. Looking at just this part first..."
- **Asks for the answer directly** → Firmly redirect: "If I just gave you the answer, it wouldn't be learning. How about approaching it this way?"

**5. Confirm understanding**
When the user arrives at the answer, ask them to summarize:
```
"Could you summarize what we've discussed so far?"
```

### Language Rule
Detect and match the user's language. Always mirror the language the user writes in.

### Anti-Patterns (NEVER do these)
- Stating the answer then asking "do you understand?"
- Giving hints so obvious they are effectively answers
- Explaining a concept then asking a rhetorical question
- Saying "the answer is X, but let me ask you why"
- Giving up and providing the answer after a few failed attempts

### Ending the Session
When the user demonstrates clear understanding:
1. Congratulate briefly
2. Suggest one follow-up question they could explore on their own
3. Offer to continue the Socratic dialogue on a related topic

## Installation & Distribution

Distributed as a standard Agent Skills package, installable via:
```bash
npx skills add RoundTable02/socrates-skill
```

Also supports `claude install-skill RoundTable02/socrates-skill` (Claude Code native) and manual git clone into `~/.claude/skills/socrates`.

Triggers on keywords: `Socrates`, `socratic`, `소크라테스` (Korean).

## Architectural Analysis

### What It Is

This is a **pure prompt artifact** — zero executable code, zero dependencies, zero configuration. The entire "application" is 73 lines of markdown containing a structured instruction set for an LLM. It's the simplest possible form of an Agent Skill: a single prompt file with YAML frontmatter that tells the agent host (Claude Code, Codex, etc.) when to load the instructions and what behavior to adopt.

### Architecture Pattern: Triggered Behavioral Overlay

The skill operates as a **behavioral overlay** on top of the base agent. When the agent host detects trigger keywords in user input, it injects the entire SKILL.md into the agent's system context, replacing the default helpful-assistant stance with a Socratic tutor stance. The overlay is toggleable — it activates on triggers, runs for the duration of the interaction, and the agent returns to normal mode when the conversation ends.

The architecture has three layers:

1. **Trigger layer** (YAML frontmatter `description` field): defines when the overlay activates. Three keywords across two languages, any position in the message. No explicit deactivation — the overlay naturally ends when the session concludes.
2. **Constraint layer** (Core Rule + Anti-Patterns): hard behavioral boundaries. "NEVER give a direct answer" is stated as non-negotiable. The five anti-patterns enumerate the specific failure modes that otherwise-sophisticated agents reliably fall into (rhetorical questions masquerading as Socratic dialogue, hints so obvious they're effectively answers).
3. **Procedure layer** (5-phase Workflow + Response Handbook): the actual questioning algorithm. Sequential phases with explicit gates between them (you can't assess before understanding, can't confirm before the user arrives at an answer).

### Key Techniques

**1. Negative specification through anti-patterns**

The five anti-patterns (lines 61-66 of SKILL.md) are the sharpest design move and the thing most skills get wrong. Most skill authors write positive specifications — "do X, then Y, then Z" — and are surprised when the model finds compliant-looking behavior that violates the spirit. The anti-pattern list is **constructive negativity**: it names the specific failure modes, making them recognisable to the model:

- "Stating the answer then asking 'do you understand?'" — the most common cop-out, where the model performs Socratic theatre rather than actual questioning
- "Giving hints so obvious they are effectively answers" — the subtler failure mode where the model thinks it's being clever while functionally giving away the game
- "Explaining a concept then asking a rhetorical question" — the lecture-disguised-as-dialogue pattern
- "Saying 'the answer is X, but let me ask you why'" — the worst of both worlds
- "Giving up and providing the answer after a few failed attempts" — the persistence problem

This is the same design instinct behind [[Thought Refiner Skill]]'s exclusion list ("no critique, no expansion, no research, no unsolicited suggestions, no flattery") — define what the thing *won't* do as precisely as you define what it *will*. It's more reliable than positive specification because boundary violations are easier for the model to detect than insufficient compliance.

**2. Progressive question taxonomy**

The five-type escalation (Clarifying → Probing → Connecting → Counter → Hypothetical) gives the model a **scaffolded question-design vocabulary**. Rather than saying "ask good questions," it provides a taxonomy of question *types* with distinct purposes and examples, and an implicit progression order. A model told to "ask Socratic questions" will produce a flat distribution of clarifying questions. A model given this taxonomy will escalate through the types as the conversation deepens.

The taxonomy maps loosely to Bloom's taxonomy: Clarifying = Remember/Understand, Probing = Analyze, Connecting = Apply, Counter = Evaluate, Hypothetical = Create. This is an uncommonly sophisticated prompt engineering choice — the escalation isn't random, it follows a pedagogical progression that increases cognitive demand.

**3. Response branching without explicit logic**

The Response to User Answers section (lines 42-45) implements a **four-way branching system** (correct/wrong/don't-know/asks-directly) purely through natural language. There's no if-else, no decision tree, no explicit logic — just four scenarios with associated responses that the model must pattern-match against. This works because the scenarios are designed to be distinguishable:

- "Correct direction" vs "Wrong direction" — the model must assess correctness, which it can do because it built internal understanding in phase 1
- "I don't know" — a surface-level match the model can detect without understanding
- "Asks for the answer directly" — another surface match

The branching is the minimum viable decision tree: two branches require understanding (correct/wrong), two are syntactic (don't-know, asks-directly). This avoids the over-specification trap where a detailed decision tree makes the model rigid rather than adaptive.

**4. Language mirroring as ambient constraint**

The language detection rule ("Detect and match the user's language") is stated in one line with no implementation detail. It trusts the base model's multilingual capability. This is the right level of specification for a capability the model already has — over-specifying language detection (e.g., "check the first word, if it's Korean then...") would add noise without benefit.

**5. Relentless pedagogical discipline**

The core rule ("NEVER give a direct answer — even if the user begs") is the skill's identity. It's the difference between a Socratic *aesthetic* and actual Socratic method. The reinforcement comes from three directions: the core rule (hard constraint), the anti-patterns (specific violations to avoid), and the response handbook (what to do when the user pushes back). This triple-layered enforcement is what keeps the model from slipping into helpful-assistant mode.

### Design Decisions

**Pure prompt, zero infrastructure.** The skill is a single markdown file. No API calls, no state persistence, no tools, no subagents. This is the simplest possible implementation — and it works because the base model already knows everything needed to execute Socratic questioning. The skill provides structure, not capability.

**Trade-off: depth vs. breadth.** The skill makes no attempt to adapt questioning strategy to domain (code vs. documents vs. math). The five question types and four response branches are universal. A domain-specific Socratic tutor would be more effective for its domain; this skill trades effectiveness for universality.

**No memory across sessions.** The skill has no mechanism for remembering where a student left off, what concepts they struggled with, or what questions were effective. Each session starts fresh. For a teaching tool, this is a significant limitation — real tutors build on prior sessions. The skill is designed for a single focused learning interaction, not a multi-session curriculum.

**The "never answer" constraint as identity.** The skill's most defining design choice is the absolute refusal to provide answers. This likely reduces adoption — many users will find it frustrating and abandon the interaction. But it's philosophically correct: the moment you allow the model to give answers "sometimes," the Socratic method collapses into the occasional-question-before-answering pattern that every AI already does. The constraint is the product.

**No explicit exit criteria beyond "clear understanding."** When does the session end? When the user has demonstrated clear understanding. But what constitutes clear understanding is left to the model's judgment. This is a gap — a student who appears to understand but has internalized a wrong mental model might be congratulated and sent on their way.

### Comparison Notes

**vs. [[Matt Pocock — Grill Me, Then Go AFK]]'s grill-me skill:** Both use Socratic questioning, but the direction is inverted. Grill-me makes the AI interview the human about a plan — the human is the domain expert, the AI is the interviewer, and the goal is alignment before implementation. Socrates-skill makes the AI interview the human about a codebase or document — the AI is the domain expert (having read the material silently), the human is the learner, and the goal is understanding through discovery. Grill-me is a collaborative alignment tool; Socrates-skill is a teaching tool. The shared insight is that one-question-at-a-time interrogation is a more effective interaction pattern than exposition.

**vs. [[Thought Refiner Skill]]:** Both are minimalist Claude Code skills (15 lines vs 73 lines) that succeed through disciplined constraint rather than exhaustive specification. Thought Refiner sharpens vague input into well-formed questions; Socrates asks questions to guide the user to answers. Both define what they *won't* do as clearly as what they will. Both trust the base model's competence rather than micro-managing it. The key difference: Thought Refiner is a one-shot transformation (input → questions), while Socrates is a multi-turn interactive protocol.

**vs. [[Decision Framework Skill]]:** Both use adaptive questioning — asking only the relevant subset rather than dumping a checklist. Both have explicit phase gating (understand → explore → recommend for Decision Framework; read → assess → guide → adapt → confirm for Socrates). The shared pattern is that structured phases prevent the model from jumping to conclusions. The difference: Decision Framework has a rich domain-specific taxonomy (12 topic areas from the 37signals framework), while Socrates has a universal question-type taxonomy (5 types applicable to any domain).

**vs. [[Lathe]]:** Both are pedagogical tools where LLMs teach rather than do. Lathe generates static tutorials with faded scaffolding and spaced retrieval; Socrates runs interactive Socratic dialogues. Lathe has a Go CLI for state management; Socrates is a stateless prompt. Lathe's pedagogy is baked into static content; Socrates's pedagogy is dynamic, adapting to the learner's responses in real time. They address different moments in the learning cycle: Lathe is for self-paced study, Socrates is for guided discovery.

**vs. Claude Code's default helpful-assistant stance:** The most important comparison. Every coding agent defaults to "be helpful" — answer questions, solve problems, produce output. Socrates-skill explicitly inverts this: helpfulness IS the anti-pattern. The tension is structural: agents are optimized to be useful by producing answers, and this skill asks them to be useful by withholding them. The skill's entire design is a harness to keep the default helpfulness from leaking through.

### Critical Assessment

**What works:** The anti-pattern list is the best single design decision — it's harder for a model to avoid a named failure mode than to comply with a positive instruction. The progressive question taxonomy gives the model a vocabulary for question design that "ask Socratic questions" doesn't. The response branching covers the four most common interaction patterns with appropriate responses. The language mirroring rule is exactly the right level of specification.

**What's fragile:** The "never answer" constraint is hard to maintain across long conversations. Models under context pressure tend to default to their training distribution, which is helpful answering. A user who persists through 20 rounds of questioning will eventually break through. The skill has no mechanism for detecting when it's about to break and escalating (e.g., "I notice we've been going in circles — let me try a different approach").

**What's missing:** There's no adaptation for learner frustration. Some learners thrive under Socratic questioning; others find it infuriating. A real tutor reads the room and switches modes. The skill has no such sensor — it either runs in Socratic mode or it doesn't. There's also no domain calibration: questions that work for code review are different from questions that work for document analysis, but the skill applies the same five-type taxonomy to everything.

**The 73-line "codebase" that isn't:** This is the smallest possible software project — a single prompt file distributed as a package. It's a useful data point for the minimal-viable-agent-skill conversation: how little infrastructure do you need for a skill to be useful? The answer, in this case, is "a structured prompt with good anti-patterns." The distribution mechanism (npx skills add, claude install-skill) is doing more infrastructure work than the skill itself.

---

*Analyzed 2026-08-01 for wiki ingestion. Repository: https://github.com/bevibing/socrates-skill*
