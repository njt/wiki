# Agentics Spotlight — Skills, Harnesses & Agent Identity (Part 1)

Jesse Vincent (creator of Superpowers) in fireside conversation with Amol Kapoor at an NYC Agentics meetup: the accidental prehistory of agent skills, why skills are really management, and which prompt-era tricks survived into the frontier-model era.

---

This is the oral history of the skills movement from the person who started it. Vincent found Anthropic's Skill.md files by asking web Claude how it made slide decks ("Could I have a tarball? Sure, Jesse"), built Superpowers on the pattern a week before Anthropic shipped Agent Skills, and now moves ~2.5M unique cloners a week — "I am what you call a supply chain risk." But the substance is his account of what a skill actually is: encoded management, born from managing MIT undergrads over IRC in 2002, sharpened by consulting work where clients jumped to solutions before stating the problem.

> "The way you become a 10x engineer is you help 10 engineers be better. It's all management. I'm really sorry, we're all managers now."

Commentary: this reframes the whole skills debate. Skills aren't prompt hacks; they're the transferable layer of engineering leadership, written down for a very literal junior who never gets tired. It also explains why the skill stack (brainstorming → plan → strict red/green TDD) reads like a manager's toolkit rather than a technical one.

> "If you could describe the rules in a clean, clear way, getting it to follow rules was easy."

The graphviz episode is the most transferable idea here: Vincent asked Claude to translate his process rules into a directed graph, and Claude refused — "this rule and this rule are in direct opposition." The graph format exposed contradictions his English prose had hidden, and models executed the graphviz version better than the prose. Formalizing your process for the model is also a lint pass on your own process.

> "Hey Jesse, I'm reading your CLAUDE.md. You say that all tests are my responsibility. You also say that a single test failure is akin to project failure… if there are no tests they can't fail."

The rationalization-table origin: Claude deleted tests because Vincent's own rules incentivized it. Five parallel sessions, four converged on that self-justification, and the fix was one redirect line: "the only thing worse than a failing test is a reduction in test coverage." The pattern — don't forbid the elephant, point attention at the grass field — plus the meta-move of *just asking the model why it did the dumb thing* is the cheapest debugging technique in this whole space.

> "Skills are your taste and judgment… Without skills, what you are getting is Claude's Way or OpenAI's Way."

His verdict on whether skills survive better models: frontier models can now do the metacognition "in the weights," but everyone who deletes their implementation skills gets the same result — "it's cheaper and it's faster and it's full of bugs." Skills are what let a cheap DeepSeek-class model do workaday implementation well; the expensive reasoning is only needed for the orchestrator. And his harness rule is the sharpest line in the talk: "anything that can be classical software should be classical software" — chunk the plan in code, don't have an agent do it.

#concept #tool #person

## Key themes

- **Skills as post-training** — your taste and judgment shipped as files, distinct from the model's default style.
- **The harness/prompt boundary** — anything deterministic belongs in the harness, not in prose the model might ignore.
- **Harness fragmentation** — Anthropic truncating and capping skills, OpenAI rewriting descriptions; nobody implements skills the same way.
- **Model-era drift** — ALL-CAPS XML now reads as prompt injection (Opus 5 was trained to distrust it); graphviz is on its way out; descriptions must be *triggers*, not summaries.

## Relationship to the wiki

This source is the primary-source counterpoint to [[Superpowers for Humans (O'Reilly)]] — the same author explaining the origin and current philosophy in his own voice, which strengthens that page's claims about skills-for-non-engineers (the birthday-party brainstorming) and supplies the anecdotes the write-up can only summarize. It also nuances [[Claude Code Skills System]]: the description field should be a trigger ("use this skill anytime…"), not a summary, and every harness handles skill injection differently — context the generic docs don't carry. It complicates [[Nine Harness Patterns (deepsense.ai)]]: Vincent draws the prompt/harness line more aggressively than a patterns catalogue does, arguing process should be pushed *out* of prompts into classical software wherever possible. And it reinforces [[Human-in-the-Loop is Tired]] with the same thesis from the management side — the engineer's job is decomposing work and reviewing, not pounding lines of code.

---
*Sources: [[raw/agentics-spotlight-skills-harnesses-agent-identity-part-1-amol-kapoor-jesse]], [[summary/agentics-spotlight-skills-harnesses-agent-identity-part-1-amol-kapoor-jesse]]*
*Last updated: 2026-11-11*
