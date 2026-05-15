# Claude Code is a Beast — Tips from 6 Months of Hardcore Use

A solo developer's field report on six months of heavy Claude Code use: $200/mo Max plan, 300k+ LOC rewritten, and an evolving toolkit of skills auto-activation, dev docs, PM2 pipelines, and specialized subagents. The most practitioner-detailed single-developer Claude Code workflow published to date.

---

## Key Quotes

> "If you've spent 30 minutes watching Claude struggle with something that you could fix in 2 minutes, just fix it yourself."

The discipline to recognize when AI assistance has become AI obstruction. This is the [[Slowing the Fuck Down]] principle applied at the micro level: knowing when to take the keyboard back is a meta-skill that separates productive AI users from frustrated ones.

> "An instruction that lives only in model context is not a constraint."

Not the author's words — this is [[claude-ctrl]]'s thesis — but it's the principle the author rediscovered through painful experience. Skills sat unused until hooks forced them into context. The entire auto-activation system exists because Anthropic's native Skills feature didn't reliably trigger.

> "Planning is king."

Said without irony after six months of daily use. The author treats skipping plan mode as professional negligence — comparing it to building a house without blueprints. This connects to [[Specifications as the Product]]: the plan is the durable artifact; code is the output.

---

## Key Themes

#claude-code #skills #hooks #planning #context-management #workflow #case-study #enforcement

### Skills Auto-Activation: The Core Innovation

The author's most significant contribution: a TypeScript hook system that forces Claude to load relevant skills before acting.

**The problem:** Anthropic's Skills feature exists but Claude doesn't reliably use them. Even exact keywords from skill descriptions, even working on files that should trigger skills — nothing guaranteed activation.

**The solution — two hooks working in tandem:**

- **UserPromptSubmit hook** (before Claude sees your message): Analyzes prompt keywords, intent patterns, and file context. Injects a formatted skill activation reminder. By the time Claude reads "how does the layout system work?", it already sees "use the project-catalog-developer skill."
- **Stop hook** (after Claude responds): Analyzes edited files for risky patterns (try-catch, async, database calls). Shows a gentle, non-blocking self-check reminder for error handling.

**The config:** A `skill-rules.json` mapping keywords, regex intent patterns, and file path/content triggers to skills. Example: the `backend-dev-guidelines` skill triggers on keywords like "controller", "Prisma", "repository" or intent patterns like `(create|add).*?(route|endpoint)`.

This is [[Harness Engineering]]'s feedforward pattern — injecting the right context before the model acts — rather than relying on the model to fetch it. It's also [[claude-ctrl]]'s thesis proven at the individual-developer scale: enforcement infrastructure beats prompt instructions every time.

### CLAUDE.md Minimalism

The author restructured from a bloated monolith (BEST_PRACTICES.md at 1,400+ lines) to:

- **CLAUDE.md (~200 lines):** Project-specific only — quick commands, service config, task workflow, testing auth routes
- **Skills:** All coding patterns and best practices — TypeScript standards, React patterns, backend API patterns, error handling, database patterns, testing guidelines
- **Documentation (850+ markdown files):** System architecture, data flows, API references

The principle: **Skills handle "how to write code"; CLAUDE.md handles "how this specific project works."** This is the same separation of concerns that [[Writing a Good CLAUDE.md]] advocates, implemented at scale.

After restructuring to follow Anthropic's recommendation (main SKILL.md < 500 lines with progressive disclosure via resource files), token efficiency improved 40-60% for most queries.

### Dev Docs System: Context as Filesystem

"Claude is an extremely confident junior dev with extreme amnesia." The solution: persistent markdown files per task.

For every large task, three files in `dev/active/[task-name]/`:
- `plan.md` — the accepted plan
- `context.md` — key files, decisions made, current state
- `tasks.md` — checklist, marked complete immediately

This is [[Planning With Files]] applied to the daily workflow. Context window is RAM; filesystem is disk. When context compacts, `/update-dev-docs` captures the essential state before it's lost. New session: just say "continue."

The author also created a `strategic-plan-architect` subagent that generates all three files from planning mode output. Frustratingly, saying "no" to plan mode kills the agent rather than continuing planning — so they built a custom `/dev-docs` slash command as a workaround.

### The Hook Pipeline: #NoMessLeftBehind

A Stop hook chain that runs after every Claude response:

1. **File Edit Tracker** (PostToolUse): Logs which files were edited and which repo they belong to
2. **Build Checker** (Stop): Runs TypeScript builds on affected repos. < 5 errors → shows them. >= 5 errors → recommends auto-error-resolver agent
3. **Error Handling Reminder** (Stop): Non-blocking self-check for try-catch blocks, Sentry integration, repository pattern usage

**Result claimed:** zero instances of Claude leaving TypeScript errors for the author to find since implementation.

A fourth hook (Prettier auto-formatting) was **removed** after reader feedback showed file modifications trigger `<system-reminder>` diffs consuming significant context tokens — in one case, 160k tokens in 3 rounds from formatting alone. This is a sharp, specific observation about the hidden cost of hook-driven automation.

### PM2 for Backend Debugging

Seven backend microservices running via PM2 with `pnpm pm2:start`. Each service gets its own managed process with real-time logs Claude can read, automatic crash restarts, and CPU/memory monitoring.

**Before:** Author manually copies logs, pastes into chat, Claude analyzes. **After:** Claude runs `pm2 logs email --lines 200`, reads logs, identifies issues, restarts service, monitors — all autonomously.

### Agent Squad

Eleven specialized subagents across three categories:

- **Quality:** code-architecture-reviewer, build-error-resolver, refactor-planner
- **Testing:** auth-route-tester, auth-route-debugger, frontend-error-fixer
- **Planning:** strategic-plan-architect, plan-reviewer, documentation-architect
- **Specialized:** frontend-ux-designer, web-research-specialist, reactour-walkthrough-designer

Key lesson: "Give them very specific roles and clear instructions on what to return." The author learned this after agents would go off-task and return "I fixed it!" without explaining what was fixed. This aligns with [[Scaling Long-Running Agents]]' finding that flat coordination fails and structured roles work.

### Scripts Attached to Skills

Pattern borrowed from Anthropic's official skill examples: utility scripts referenced directly in skill files. Example from `backend-dev-guidelines`:

```
node scripts/test-auth-route.js http://localhost:3002/api/endpoint
```

The script handles Keycloak refresh tokens, JWT signing, cookie headers, and the authenticated request — complexity Claude would otherwise re-derive each time. Principle: if Claude helped write a useful script, immediately document it in CLAUDE.md or attach it to a skill.

---

## Critical Analysis

**What's genuinely new:** The skills auto-activation via hooks is the most practical solution I've seen to the "skills exist but don't activate" problem. It's hack-adjacent — you're building infrastructure to compensate for a product gap — but it works. Anthropic should take this as a bug report, not a feature request. If skills need hook-based enforcement to trigger reliably, the product is incomplete.

**The cost elephant:** $200/mo on the Max 20x plan. The author is paying professional-tool prices for a single-developer setup. For a solo developer shipping a major rewrite, this is defensible. As a general recommendation, it's steep. The article doesn't discuss whether a lower-tier plan would suffice with the same workflow discipline.

**300k LOC from 100k — is this progress?** A 3-4x codebase expansion during a refactor warrants scrutiny. TypeScript + strict typing accounts for some growth. MUI v7 vs v4 accounts for some. But the article doesn't address whether all 300k lines are load-bearing, or whether some is AI-generated bloat. [[Write Only Code]] and [[Cognitive Debt]] are the relevant frames here: the author can't possibly have reviewed all 300k lines personally.

**The hook pipeline is a CI/CD system living in the wrong place.** Build checking, error detection, formatting — these are pre-commit and CI concerns. Running them in the agent's Stop hook is clever but fragile. A TypeScript build error caught by a hook is caught later than one caught by a language server in the IDE. The real insight is that hooks can enforce discipline Claude won't self-impose — but some of these checks belong one layer down in the stack. [[Feedback Loop is All You Need]] makes this point: linters beat prompts. Hooks beat linters? Maybe. Pre-commit hooks beat both.

**What's missing from the system:**
- **Evals.** With 300k LOC and 11 subagents, there's no mention of [[Demystifying Evals for AI Agents]] or any systematic quality measurement. The build checker catches type errors; what catches architectural drift?
- **Cost tracking.** Per-task, per-agent, per-skill — the author is on a $200/mo plan but doesn't seem to know where the money goes.
- **Failure stories.** The article is a success narrative. With six months of heavy use, there must be patterns that didn't work. Their absence weakens credibility.
- **The team dynamic.** The author mentions being the "AI guru" because colleagues are "roughly a year behind." Solo adoption in a team setting creates a bus-factor problem. What happens when the author is on vacation?

**The strongest portable insight:** Skills for patterns, CLAUDE.md for project specifics, documentation for architecture. This three-way separation is clean, scalable, and doesn't depend on any particular tool or hook infrastructure. You could adopt it tomorrow.

**The second-strongest:** Context-as-filesystem via dev docs. [[Planning With Files]] and [[recursive-mode]] both converge on the same pattern. The author's `/update-dev-docs` command — explicitly capturing context before compaction — is a simple, high-leverage practice that anyone using Claude Code should steal.

---

## Cross-Links

- [[claude-ctrl]] — Same thesis (hooks over prompts), the author independently rediscovered it
- [[Harness Engineering]] — The feedforward/feedback framework this system instantiates
- [[Feedback Loop is All You Need]] — The build checker and error reminder are exactly the "linters beat prompts" thesis in hook form
- [[Writing a Good CLAUDE.md]] — The three-way separation (skills/CLAUDE.md/docs) validates HumanLayer's recommendations
- [[CLAUDE.md (Universal)]] — The author's slimmed CLAUDE.md follows the same token-efficiency principles
- [[How Intercom Uses Claude Code]] — Enterprise-scale version of the same patterns: hooks, skills, enforcement
- [[Planning With Files]] — The dev docs system is this concept applied to daily workflow
- [[Scaling Long-Running Agents]] — Both discovered planner/worker/judge independently
- [[Agent Orchestration]] — The agent squad is a practitioner's multi-agent topology
- [[Don't Fear the Dark Factory]] — Same validation-over-generation philosophy, different implementation
- [[Designing Agentic Loops]] — Willison's meta-skill of convergence design; the author's hook pipeline is one answer
- [[Slowing the Fuck Down]] — "Just fix it yourself" is the micro version of deliberate friction
- [[Write Only Code]] — 300k LOC, one reviewer — the slop radius question
- [[Cognitive Debt]] — 3-4x codebase expansion without addressing comprehension burden
- [[How Hightouch Built Their Long-Running Agent Harness]] — Context management as the real engineering challenge, validated again
- [[MinMax Skills]] — Complementary skills library; the author's skills are the custom counterpart

---
*Sources: [[raw/claude-code-beast-6-months]]*
*Last updated: 2026-05-15*
