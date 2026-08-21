# Socrates Skill

A 73-line Claude Code skill that turns any AI coding agent into a Socratic tutor — never giving direct answers, always guiding users to discover answers themselves through progressive questioning. By Choi Wontak (RoundTable02), MIT-licensed, distributed as a standard Agent Skills package. The skill is a pure prompt artifact with zero code dependencies, defining a behavioral overlay that replaces the agent's default helpful-assistant stance with relentless pedagogical discipline.

---

## Architecture

The entire "application" is a single `SKILL.md` file with three structural layers:

**Trigger layer** — the YAML frontmatter `description` field activates the skill when user input contains `Socrates`, `socratic`, or `소크라테스` (Korean). The overlay loads into the agent's context, replacing default behavior for the duration of the interaction.

**Constraint layer** — the Core Rule ("NEVER give a direct answer") plus five explicitly enumerated anti-patterns (lines 61-66). These are the skill's real engine: they name specific failure modes that otherwise-sophisticated agents reliably fall into, from "stating the answer then asking 'do you understand?'" to "giving up and providing the answer after a few failed attempts."

**Procedure layer** — a five-phase sequential workflow with explicit gates:

| Phase | Action | Key Design Choice |
|-------|--------|-------------------|
| 1. Read | Silently analyze target resource, build internal understanding | Understanding is kept private — never shared directly |
| 2. Assess | Ask opening question to gauge current understanding | Adapts starting point to learner, not domain |
| 3. Guide | Progressive questioning through five escalating types | Clarifying → Probing → Connecting → Counter → Hypothetical |
| 4. Adapt | Four-way response branching based on user answer | Correct (deepen), wrong (expose contradiction), don't-know (simplify), ask-for-answer (redirect) |
| 5. Confirm | Ask user to summarize their understanding | Forces construction, not just recognition |

## Key Techniques

### Negative specification through anti-patterns

The five anti-patterns are the sharpest design move. Rather than saying "be Socratic" and hoping, the skill names the specific failure modes a helpful-default agent will find: performative questions ("do you understand?"), transparent hints, lectures disguised as dialogue, and the persistence problem. The model can detect these named violations more reliably than it can approximate an abstract concept like "Socratic questioning." This is the same design instinct behind [[Thought Refiner Skill]]'s exclusion list — define what the thing *won't* do as precisely as what it *will*.

### Progressive question taxonomy

The five-type escalation (Clarifying → Probing → Connecting → Counter → Hypothetical) maps loosely to Bloom's taxonomy and gives the model a **scaffolded question-design vocabulary**. Most "Socratic mode" prompts produce flat distributions of clarifying questions. This taxonomy provides distinct question types with escalating cognitive demand and an implicit progression order the model can follow without explicit sequencing logic.

### Response branching without explicit logic

The four-way response handbook (correct/wrong/don't-know/asks-directly) implements branching purely through natural language. Two branches require understanding (correct vs. wrong direction), two are syntactic surface matches (I-don't-know, ask-for-answer). This split — judge where judgment is needed, pattern-match where it isn't — is the minimum viable decision tree for an adaptive dialogue system.

### Triple-layered constraint enforcement

The core rule is enforced from three directions: the absolute declaration (hard constraint), the anti-patterns (specific violations named), and the response handbook (what to do when the user pushes back). Each layer catches failures the others miss: the core rule catches when the model strays from Socratic stance, the anti-patterns catch when the model performs compliance while violating spirit, and the response handbook catches the social pressure case (user begging for the answer).

## Design Decisions

**Pure prompt, zero infrastructure.** No API calls, no state, no tools, no subagents. The simplest possible implementation delegates everything to the base model's existing Socratic capability. The skill provides structure, not new capabilities.

**Universality over domain specificity.** The five question types and four response branches are identical whether the user is exploring code, documents, math, or architecture. A domain-specific Socratic tutor would do better within its domain; this skill trades depth for breadth.

**No memory across sessions.** Stateless by design. Each conversation starts fresh with no record of prior learning, struggles, or effective question patterns. Real tutors build on history; this skill optimizes for the single focused interaction.

**The "never answer" constraint IS the product.** The hardest design choice: refuse to give answers even when the user begs. Many users will find this frustrating and abandon the interaction. But the moment you allow "sometimes," the Socratic method collapses into the occasional-question-before-answering pattern that every AI already does. The constraint is what makes the skill different from the default.

## Comparison Notes

**vs. [[Matt Pocock — Grill Me, Then Go AFK]]'s grill-me skill:** Both use Socratic one-question-at-a-time interrogation, but in opposite directions. Grill-me makes the AI interview the human about their plan — the human is the expert, the AI is the interviewer, the goal is alignment. Socrates-skill makes the AI interview the human about a codebase or document — the AI is the expert (having read silently), the human is the learner, the goal is discovery. Grill-me is collaborative alignment; Socrates-skill is pedagogy.

**vs. [[Thought Refiner Skill]]:** Both are minimalist skills that succeed through disciplined constraint rather than exhaustive specification. Both define exclusions as clearly as inclusions. Thought Refiner is one-shot (input → sharp questions); Socrates is multi-turn (ongoing dialogue). The shared instinct is that less specification, well-aimed, beats more.

**vs. [[Decision Framework Skill]]:** Both use adaptive questioning with structured phase gating. Decision Framework has a rich domain taxonomy (12 topic areas from the 37signals framework); Socrates has a universal question-type taxonomy. The shared pattern: sequential phases prevent the most common LLM failure mode — jumping to conclusions.

**vs. [[Lathe]]:** Both invert the typical agent relationship — the LLM teaches, the human learns. Lathe generates static tutorials with faded scaffolding; Socrates runs dynamic interactive questioning. They address different moments in learning: Lathe is self-paced study, Socrates is guided discovery.

**vs. Claude Code's default stance:** The defining tension. Every coding agent defaults to helpfulness — answer questions, solve problems, produce output. Socrates-skill inverts this: helpfulness IS the anti-pattern. The skill's entire design is a harness to keep helpfulness from leaking through.

**vs. [[A Smart Bear Skills]]:** Jason Cohen's Rude Q&A skill is the same "never hand you the answer" stance with the expert/learner roles reversed: here the *human* is the expert and the AI the hostile interrogator. Both skills are masterclasses in negative specification, but for different ends — Socrates refuses to answer in order to teach, Cohen's skills refuse to answer in order to force the founder past their own denial (the "you cannot interrogate yourself" premise). Where Socrates's five anti-patterns name *performative helpfulness*, Rude Q&A's constraint is *won't accept "approximately fine" as a final answer*.

Tags: #tool #project #agents #claude-code #prompt-engineering #pedagogy #socratic-method

---
*Sources: [[raw/socrates-skill]]*
*Last updated: 2026-08-01*
