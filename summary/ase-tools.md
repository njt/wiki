---
url: https://ase.tools/
title: "ase.tools — ASE (Agentic Software Engineering) Skill Manual Pages"
author: Dr. Ralf S. Engelschall
date_fetched: 2026-11-09
date_published: 2026
topics:
  - coding-agents-and-frameworks
  - specifications-as-the-product
---

The site is the manual-page documentation for **ASE (Agentic Software Engineering)**, a large skill catalog by Dr. Ralf S. Engelschall — 147 man-pages, one per skill, written in classic Unix man-page format (NAME / SYNOPSIS / DESCRIPTION / ARGUMENTS / SCENARIOS / EXAMPLES / SEE ALSO). ASE runs as a plugin across multiple agent tools (Anthropic Claude Code CLI, GitHub Copilot CLI, OpenAI Codex CLI), backed by an `ase` CLI, an MCP server, and a `plugin/skills/*/help.md` layout.

The catalog spans several families. **Meta/cognitive skills** (`ase-meta-*`) include Five-Whys root-cause analysis with widened candidate chains and backtracking (`ase-meta-why`), Devil's Advocate (`ase-meta-diaboli`) and its constructive mirror Steelman (`ase-meta-steelman`), collaborative brainstorming with clarify→diverge→converge→shortlist phases (`ase-meta-brainstorm`), weighted multi-criteria decision matrices (`ase-meta-evaluate`), multi-LLM quorum (`ase-meta-quorum`), multi-backend web search, and a compatibility self-test (`ase-meta-compat`) that probes how faithfully the current model executes ASE's four "interpreter primitives" — XML placeholders, control flow, regex matching, arithmetic — and reports a 0–100% rating.

**Engineering skills** cover specs (`ase-spec-lint/edit/activate`), code (`ase-code-*`), repos (diff, changelog, commit message, merge, resolve), sync, and a large **task-plan lifecycle**: `ase-task-edit`, `ase-task-grill` (adversarial interrogation of a plan), `ase-task-research`, `ase-task-estimate`, `ase-task-implement`, `ase-task-integrate` — a spec-driven pipeline where the task plan is the durable artifact, with configurable lifecycles (`solo`, `team`, `enterprise`) and task-id schemes.

Two design choices stand out. First, skills are documented and discovered like a Unix toolchain: `ase-help-skill` resolves a skill from a full name, abbreviation, or purpose description against a catalog; `ase-help-intent` matches free text against the accumulated help and generates ranked concrete commands. Second, **aliases are expanded deterministically by ASE, never by the AI**, and configuration is a layered scope chain (default < user < project < task < session) accessed only via MCP tools, never by shelling out.
