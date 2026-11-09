# ASE — Man-Page Discipline for Agent Skills

ASE (Agentic Software Engineering) is Dr. Ralf S. Engelschall's catalog of 147 agent skills, documented as classic Unix man pages and packaged as a plugin that runs on Claude Code, GitHub Copilot CLI, and OpenAI Codex alike, with an `ase` CLI and MCP server underneath. The site's interest is less any single skill than the system's design commitments: man-page-format documentation with `SCENARIOS` sections for intent matching, deterministic (never AI-performed) alias expansion, a layered config scope chain, and a compatibility self-test that treats the LLM itself as the skill interpreter.

---

## What it is

Each skill gets a NAME/SYNOPSIS/DESCRIPTION/EXAMPLES page. The catalog splits into meta/cognitive skills (`ase-meta-why`, `ase-meta-diaboli`, `ase-meta-brainstorm`, `ase-meta-evaluate`…), spec skills (`ase-spec-lint/edit/activate`), code and repo skills, and a task-plan lifecycle (`ase-task-edit` → `ase-task-grill` → `ase-task-research` → `ase-task-implement` → `ase-task-integrate`) that makes the task plan the durable, iteratively-refined artifact.

## Key quotes

> ASE skills are not run by a conventional interpreter — they are interpreted by the *model itself*, which must honor control-flow constructs, substitute XML placeholders, evaluate regex conditions, and perform arithmetic. If a model cannot execute these primitives reliably, *every* ASE skill silently misbehaves.

The most intellectually honest move in the whole catalog: `ase-meta-compat` self-tests whether the model running the skills can actually execute them, reporting a weighted 0–100% rating (XML placeholders weighted 4.00, control flow 3.00, regex 2.00, arithmetic 1.00). Most skill frameworks assume the interpreter works; ASE measures it.

> There are three kinds of aliases, all expanded deterministically by ASE itself, and never by the AI.

Aliasing — full command aliases, option aliases, option defaults — is deliberately kept out of the model's hands. Where most agent tooling lets the LLM interpret intent, ASE narrows the model's discretion at the points where variance is pure risk.

> The skill applies the *Five-Whys* … with `--width` *M* (> 1) … each level surfaces up to *M* candidate sub-causes, descends into the single most significant one (justifying the choice), and retains the rest as *fallbacks*. … a mis-rooted choice is *backtracked* into a fallback.

`ase-meta-why` patches the classic Five-Whys failure mode (early commitment to the wrong branch) with beam search plus backtracking — an old-school technique retrofitted onto an LLM reasoning skill.

## Key themes

#tool #pattern #concept

- **Man pages as the skill interface.** Documentation format doubles as machine-readable intent-matching substrate: `ase-help-intent` matches free text against accumulated `SCENARIOS` and generates ranked concrete commands with real flags.
- **Spec-driven pipeline.** The task lifecycle (grill → research → estimate → implement → integrate) treats the plan as the product, with linting and activation of spec know-how.
- **Determinism over model discretion.** Aliases, config scopes, and id minting (`ase-meta-mint` delegates UUID/SHA-1/nanoid to an MCP tool "derived reproducibly instead of guessing") all push non-creative work out of the model.

## Analysis

ASE is what happens when someone with a Unix toolchain aesthetic builds an agent skill ecosystem instead of a pile of prompt files. The man-page format is not nostalgia: it makes skills greppable, cross-referenced (`SEE ALSO`), intent-matchable, and renderable in a terminal `ase man` TUI. Compared to the typical skill dump, this is documentation as API surface.

The interesting tension is the compat self-test. ASE's skills embed an actual control language — `<if>`, `<while>`, `<for>`, XML placeholders — executed by the model. That's a bet on model reliability that most frameworks refuse to make, and `ase-meta-compat` is the honest confession that the bet sometimes loses. It's both a feature (measure before you rely) and an admission of fragility: every skill's correctness rests on the host model's ability to act as a interpreter, not just a reasoner.

## Related pages

- [[Matt Pocock Skills — Skills for Real Engineers]] — ASE is the maximalist sibling: where Pocock ships 25+ small hackable composable skills, ASE ships 147 with a unified man-page grammar, config layering, and a task lifecycle that owns the whole pipeline; both reject process-owning monolithic frameworks in favor of skill granules, but ASE adds the infrastructure layer (CLI, MCP server, catalog resolution) Pocock's plugin lacks.
- [[Claude Code Skills System]] — Anthropic's official SKILL.md architecture is the substrate ASE builds on; ASE's `help.md` per skill and `ase-help-skill` catalog resolution show what a third-party ecosystem looks like once that substrate exists, including multi-host portability (Copilot CLI, Codex CLI) that the official system doesn't attempt.
- [[SDDW (Spec-Driven Development Workflow)]] — Sermakarevich's 7-step spec pipeline and ASE's task-plan lifecycle converge on the same thesis — the plan is the durable artifact — but ASE operationalizes it as standalone skills with grilling and estimation phases rather than a fixed pipeline plugin, strengthening the case that spec-driven development is becoming a standard skill family across independent implementations.
- [[A Smart Bear Skills]] — Jason Cohen's skills interrogate rather than answer; ASE's `ase-task-grill` and `ase-meta-diaboli` (ten ranked anti-theses, then Hegelian synthesis) are the same constructive-devil's-advocate pattern generalized into a whole skill category, showing the pattern replicating across independent skill authors.

---
*Sources: [[raw/ase-tools]], [[summary/ase-tools]]*
*Last updated: 2026-11-09*
