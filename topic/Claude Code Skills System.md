# Claude Code Skills System

Anthropic's skills architecture is Claude Code's primary extension mechanism: a filesystem-native plugin system where directories containing `SKILL.md` become invocable `/commands` with frontmatter-controlled behavior, dynamic context injection, subagent delegation, and a three-tier deployment model spanning personal workflow to enterprise governance.

---

The skills system is the **unifying architecture** behind every `/` command in Claude Code. What started as `.claude/commands/deploy.md` flat files evolved into a directory-based system with supporting files, frontmatter configuration, and the ability to run skills as isolated subagents. The shift from "commands" to "skills" isn't cosmetic — it's the difference between a macro and a module. A command is text substitution; a skill is a deployable, reviewable, version-controlled unit of agent behavior.

## Architecture

The core abstraction is elegant: a skill is a directory. The directory name becomes the command name. `SKILL.md` is the entrypoint. Everything else — scripts, templates, reference docs, example outputs — is optional supporting material that Claude loads on demand. This is **progressive disclosure as architecture**: the description (~100 tokens) loads at session start so Claude knows what's available; the full content loads only when invoked. [[Steering Claude Code]] maps this as the "skills" mechanism in its seven-mechanism taxonomy, with the key property that skill content is *lazy-loaded* unlike CLAUDE.md's always-loaded context.

> "Create a skill when you keep pasting the same instructions, checklist, or multi-step procedure into chat, or when a section of CLAUDE.md has grown into a procedure rather than a fact."

This is the CLAUDE.md-as-facts, skills-as-procedures distinction that [[Steering Claude Code]] and [[Claude Code Mastery]] both advocate. CLAUDE.md is the codebase overview; skills are the runbooks. The operational difference: CLAUDE.md content costs tokens in every session; skill content costs nothing until invoked.

### Three-Tier Deployment

Where a skill lives determines its blast radius and governance model:

| Level | Location | Scope | Governance |
|-------|----------|-------|------------|
| Personal | `~/.claude/skills/` | All your projects | You |
| Project | `.claude/skills/` | Anyone cloning the repo | Code review |
| Enterprise | Managed settings | Entire organization | Central IT |

Enterprise overrides personal, personal overrides project. A skill at any level overrides a bundled skill of the same name. Plugin skills use `plugin-name:skill-name` namespace and cannot conflict. This hierarchy is the same pattern [[Team-Wide Agentic Harness]] argues for: skills as reviewed, version-controlled team infrastructure rather than personal `.claude/` config.

Nested skills (`apps/web/.claude/skills/deploy/`) appear as directory-qualified names (`/apps/web:deploy`) and auto-activate when Claude works on files in that subdirectory. This is the monorepo pattern: each package owns its own skills, and Claude picks the right variant based on file locality.

### Frontmatter as Configuration Surface

The frontmatter reference covers 20+ fields, but the design decisions cluster around three questions:

1. **Who invokes it?** `disable-model-invocation: true` for deploy/commit/send-slack (you invoke, never Claude). `user-invocable: false` for reference material Claude should know but you'd never type (`/legacy-system-context`).

2. **Where does it run?** `context: fork` isolates the skill in a subagent with no conversation history. `agent: Explore` picks the subagent type. `background: false` makes forked skills block until done.

3. **What can it do?** `allowed-tools` pre-approves tools for the invoking turn. `disallowed-tools` denies them. Both clear when you send your next message.

Boolean fields accept `yes`/`no`/`on`/`off`/`1`/`0` since v2.1.218 — a quiet UX improvement that prevents the "true/false only" footgun. Outside Claude Code (Cowork, cloud sessions, routines), only the Agent Skills spec fields are accepted; including Claude Code-only fields causes a hard packaging error.

### Dynamic Context Injection

> The `` !`command` `` syntax runs shell commands before the skill content is sent to Claude. The command output replaces the placeholder, so Claude receives actual data, not the command itself.

This is the bridge between static instructions and live state. A PR review skill runs `` !`gh pr diff` `` and Claude receives the actual diff. A change-summary skill runs `` !`git diff HEAD` `` and Claude sees the working tree. The inline form (`!` at start of line) and fenced-block form (` ```!`) support single and multi-line commands. `disableSkillShellExecution: true` in settings disables this globally for untrusted skills.

Combined with `${CLAUDE_SKILL_DIR}` substitution and `allowed-tools`, skills can bundle and execute scripts without permission prompts — a local execution sandbox that sidesteps the prompt-approval loop entirely.

### Content Lifecycle

Invoked skill content **stays in context for the rest of the session**. This is the double-edged sword: it means guidance persists across turns (good), but also that verbose skills are a recurring token tax (bad). The docs are explicit: "State what to do rather than narrating how or why."

Re-invocation of identical content adds only a note ("skill already loaded") — the dedup prevents context bloat from repeated `/deploy` calls. Changed content (different args, different dynamic output) appends the full text again. Since v2.1.202.

Auto-compaction re-attaches the most recent invocation of each skill within a shared 25,000-token budget, filling from most-recently-used first. Older skills can be dropped entirely after compaction if you've invoked many. The docs' advice: "re-invoke it after compaction to restore the full content." This is the same compaction-survival problem [[Steering Claude Code]] documents for all the instruction-delivery mechanisms, and it's why [[Tuning Claude Code Into a Better Engineering Partner]]'s "restart the session" rule is sometimes the only honest answer.

### Description Budget Economics

Skill descriptions load into context at session start so Claude knows what's available. The listing has a character budget that scales at 1% of the model's context window. When the budget overflows, descriptions are dropped from least-used skills first. The `skillOverrides` setting lets you preemptively set low-priority skills to `"name-only"` (no description, saves budget) or `"off"` (hidden entirely).

This is **context economics as feature design**: the system doesn't just enumerate skills — it actively manages which ones consume the scarce description budget. It's the same insight as [[New Rules of Context Engineering]]'s finding that Anthropic deleted 80% of the system prompt without regression: progressive disclosure beats exhaustive enumeration.

## Key Themes

#claude-code #skills #slash-commands #subagents #context-engineering #configuration #plugin-architecture

## Critical Analysis

**The skills system is the most underrated part of Claude Code's architecture.** While hooks and MCP get the "extensibility" attention, skills are where daily workflow lives. A skill is the cheapest form of reusable context in Claude Code — ~100 tokens of description at session start, zero tokens until invoked. Compare with CLAUDE.md's always-loaded cost, or MCP's connection overhead. For teams adopting Claude Code, the skill system is the **first optimization target**: move procedures out of CLAUDE.md, encode them as skills, and watch session costs drop.

**The "custom commands merged into skills" transition is undersold.** The docs note that `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both create `/deploy`, but the skill version gains supporting files, frontmatter, and subagent delegation. This is the flat-file → directory transition that every build system eventually makes, and it's right that Anthropic made it early. The backward compatibility is a migration path, not a permanent dual standard.

**The subagent integration is where skills become a distributed computing substrate.** `context: fork` + `agent: Explore/Plan/general-purpose` means a skill can be a task definition that dispatches to a specialized agent with its own tool set and context window. Combined with dynamic workflows (which author JavaScript harnesses that spawn and coordinate subagents), skills become the **declarative interface** to a multi-agent orchestration system. This is what [[Dynamic Workflows in Claude Code]] formalizes as composable patterns (fan-out, tournament, adversarial verify), but skills are the simpler, more accessible entry point to the same capability.

**The bundled skills are a product strategy document disguised as features.** `/run`, `/verify`, and `/run-skill-generator` form a self-reinforcing adoption loop: use Claude Code to build your app, verify it works, and record the recipe so next time is smoother. The recipe recording (`/run-skill-generator` writes `.claude/skills/run-<name>/`, `/verify` writes `.claude/skills/verify/SKILL.md`) means the product *improves with use* — each project teaches Claude Code how to work with that project. This is the same compounding-infrastructure thesis as [[Claude Code Mastery]]'s "every mistake becomes a rule," but at the project-build level rather than the coding-convention level.

**What's missing from the docs is failure-mode coverage.** The content lifecycle section tells you skill content persists and gets compacted, but it doesn't tell you what happens when a skill gets dropped mid-task and Claude silently loses critical instructions. The description budget section warns about truncation but doesn't help you debug *which* skills lost their descriptions. The nested-skills section describes the lookup algorithm but doesn't address the discoverability problem: how does a teammate know that `apps/web/.claude/skills/deploy/` exists? These are the real failure modes teams will hit, and the reference docs' silence on them is the gap between "how it works" and "how it fails."

**The skill-creator evaluation plugin is the most important thing most users will never run.** Automated A/B testing of skill effectiveness, blind version comparison, description hit-rate measurement — this is the eval infrastructure that [[Guardrails and Feedback Loops]] argues should surround every agent behavior. That it's packaged as an optional plugin rather than a core feature says something about Anthropic's product priorities: ship the creation surface first, the verification surface later, and let power users discover the gap.

**Skills as team infrastructure is the unresolved frontier.** [[Team-Wide Agentic Harness]] argues skills should be reviewed like code. The docs support this — project skills are committed to version control — but they're silent on the social machinery: who reviews skill changes, against what criteria, at what cadence? A skill that grants `allowed-tools: Bash(git:*)` in a project repo is a trust decision, not just a convenience. As [[Malicious Agent Skills in the Wild]] documents, the attack surface is real: 157 confirmed malicious skills in the wild, 84.2% in natural-language SKILL.md files. The docs could benefit from a "reviewing project skills before trust" section that maps the `allowed-tools` surface to risk categories.

---

A concrete example of the skills pattern in production: [[Flint Chart]] ships two agent skills (`flint-chart-author` and `flint-theme-author`) that package chart-template catalog knowledge and theme-application guidance as invocable skill directories. These demonstrate the CLAUDE.md-as-facts, skills-as-procedures distinction in practice — the skills don't contain chart rendering code, they contain the procedural knowledge for navigating Flint's 70+ semantic types, 25-38 chart templates per backend, and ten visual theme presets.

Portability is a real consequence of the spec: [[I Have ADHD Skill]] ships one `SKILL.md` that runs unchanged in Claude Code, Codex, Cursor, Copilot, Zed, Qwen, and Hermes, with `disable-model-invocation: true` keeping it opt-in in each — the same "invoke, don't auto-apply" contract documented above, expressed once and inherited everywhere.

---
*Sources: [[raw/slash-commands]], [[summary/slash-commands]]*
*Last updated: 2026-08-06*
