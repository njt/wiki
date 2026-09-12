---
url: https://code.claude.com/docs/en/slash-commands
title: Slash Commands (Skills)
author: Anthropic
date_fetched: 2026-08-06
topics:
  - agent-coding-workflow
---

Anthropic's official reference documentation for Claude Code's skill system — the `/`-prefixed commands that replaced the older custom commands mechanism. A skill is a directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions; Claude loads skill content on invocation and can automatically activate skills when their description matches the user's task. Skills live at three levels (personal, project, enterprise), support nested monorepo variants, and can run inline or as forked subagents.

The frontmatter reference covers 20+ fields: `description` (the primary trigger mechanism), `disable-model-invocation` (manual-only skills like `/deploy`), `user-invocable` (hide from the menu while keeping model access), `context: fork` (run in isolated subagent), `agent` (pick the subagent type), `allowed-tools` (per-turn pre-approval), `arguments` (named parameter declarations), and `background` (async vs blocking). Boolean fields accept `yes`/`no`/`on`/`off`/`1`/`0` since v2.1.218.

Skills support dynamic context injection via `` !`command` `` syntax — shell commands that execute before Claude sees the content, with output replacing the placeholder. String substitutions (`${CLAUDE_SKILL_DIR}`, `$ARGUMENTS`, `$0`/`$1` positional args) let skills reference bundled files and pass arguments without hardcoding paths. The `skillOverrides` setting controls visibility from outside the skill file itself, with four states: on, user-invocable-only, name-only, off.

Claude Code ships bundled skills (`/doctor`, `/code-review`, `/batch`, `/debug`, `/loop`, `/claude-api`, `/run`, `/verify`, `/run-skill-generator`) that are always available. The `/run` → `/verify` → `/run-skill-generator` triad forms a launch-and-verify pipeline: infer how to run the app, confirm changes against the running app, and record the recipe as a per-project skill for future sessions.

Skill content persists in context across turns after invocation. Re-invocation of identical content adds only a short note; changed content (different arguments or dynamic output) appends the full text again. Auto-compaction re-attaches the most recently invoked skills within a combined 25,000-token budget, dropping older skills first. Live change detection watches skill directories and picks up edits without restart.

The `skill-creator` plugin automates evaluation: test cases in `evals.json`, isolated subagent runs per case, grading against assertions, benchmark comparison of with-skill vs without-skill pass rates, blind A/B version comparisons, and description-tuning that measures hit rate.
