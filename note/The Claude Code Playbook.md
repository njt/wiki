# The Claude Code Playbook

Marcelo Bairros's five-tip guide to Claude Code productivity, published on White Prompt's blog. A beginner-to-intermediate listicle covering MCP servers, CLAUDE.md configuration, planning mode, the Max subscription plan, and IDE integration. Light on depth but useful as a snapshot of what the "getting started" advice looks like from a consultancy perspective in mid-2025.

---

## Key Quotes

> "These aren't just tips — they're the difference between using Claude Code and mastering Claude Code."

The confidence-to-depth ratio here is sky-high. Five tips, none explored beyond a paragraph. But the framing reveals something real: most Claude Code users aren't doing any of this.

> "Use IDE diagnostics to find and fix errors; Reference prompt-engineering-playbook.md for all prompt improvements"

This is his example CLAUDE.md content. Compare it to the [[CLAUDE.md (Universal)]] six-rule approach or the [[claude-code-config (Trail of Bits)]] security-first defaults -- this is surface-level, but it's where most people start.

## Key Themes

#tool #claude-code #mcp #workflow #beginner

### MCP as Force Multiplier

Bairros's top tip is that most users aren't connecting enough MCP servers. He names Context7 (documentation retrieval), Sequential Thinking (problem decomposition), a Postgres connector, and Test Master AI (project phasing). This aligns with the broader wiki's view that [[Building Agents for Production Systems with MCP]] is the standard integration pattern, but the specific servers he recommends are consumer-grade convenience tools rather than production infrastructure. The interesting absence: no mention of security implications of connecting MCP servers, which [[Security and Sandboxing]] would flag as a concern.

### CLAUDE.md as Entry Point

The `/init` command generates a CLAUDE.md file. Bairros treats this as table stakes. The wiki has extensive coverage of this pattern: [[CLAUDE.md (Universal)]] for the minimal approach, [[How Boris Uses Claude Code]] for how the creator uses it (team-maintained, checked into git), and [[Feedback Loop is All You Need]] for why CLAUDE.md alone is insufficient ("your CLAUDE.md is a suggestion; your linter isn't"). Bairros doesn't reach this deeper insight -- he presents the config file as the destination rather than the starting point.

### Plan Mode Before Execution

"Plan first, code second" maps directly to [[Compound Engineering]]'s Plan-Work-Review-Compound loop and [[Addy Osmani's Workflow]]'s spec.md approach. This is the single most repeated piece of advice across the wiki's agentic development sources. Bairros presents it as a tip; the deeper sources present it as a prerequisite for non-trivial work.

### The Economics Argument

The Max plan at $100/month vs. API costs potentially reaching $6,000/month is a genuine consideration. The wiki hasn't covered Claude Code pricing directly, but [[How to Buy Cheap Claude Tokens in China]] documents the grey market that emerges when costs are high enough. The economic framing is pragmatic: if it saves two hours a month, it pays for itself.

### IDE Diagnostics as Feedback Loop

The IDE extension tip -- letting Claude Code read type errors and syntax issues in real-time -- is actually the most substantive advice in the piece, though Bairros barely develops it. This is a concrete instance of [[Feedback Loop is All You Need]]'s core thesis: mechanical feedback (IDE diagnostics) beats instructions (telling Claude to check its work). The self-correction loop from IDE diagnostics is a lightweight version of the lint-driven guardrails that [[Harness Engineering]] categorizes as "computational feedback."

## Critical Analysis

This is competent introductory content that inadvertently reveals the gap between getting-started advice and the deeper practices documented elsewhere in this wiki. Every one of Bairros's five tips is the shallow end of a pool that goes much deeper:

- "Use MCPs" becomes "build production integration layers" in [[Building Agents for Production Systems with MCP]]
- "Run /init" becomes "your config is a suggestion; your linter is a constraint" in [[Feedback Loop is All You Need]]
- "Plan first" becomes the full Plan-Work-Review-Compound loop in [[Compound Engineering]]
- "IDE diagnostics" becomes the full harness engineering taxonomy in [[Harness Engineering]]

What's missing entirely: verification as a discipline (which [[How Boris Uses Claude Code]] calls "probably the most important thing"), any mention of guardrails or linting, any discussion of what to do when Claude gets things wrong, and any acknowledgment that the Max plan's "unlimited" access still has rate limits that affect parallel workflows.

The piece is most useful as a marker of where mainstream Claude Code advice sits in mid-2025: still focused on setup and configuration rather than the feedback loops and compound systems that separate productive users from people who just have expensive subscriptions. It lives at Level 1-2 on the [[Five Levels from Spicy Autocomplete to the Dark Software Factory]] scale.

Anthropic's own [[The AI-Native SDLC Playbook]] is the other end of the spectrum — the same "plan first, CLAUDE.md, feedback loop" advice rebuilt into a full six-stage lifecycle where every stage commits an artifact (`intent.md` → `spec.md` → `plan.md`) and governance is enforced by hooks and evals. This listicle is the getting-started tier; the official playbook is the enterprise answer.

---
*Sources: [[summary/the-claude-code-playbook]]*
*Last updated: 2026-05-14*
