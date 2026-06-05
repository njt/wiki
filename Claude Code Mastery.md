# Claude Code Mastery

Arpan Patel's 27-minute field manual covering the entire Claude Code ecosystem: `.claude` directory anatomy, CLAUDE.md as compounding infrastructure, skills as reusable expertise, custom subagents, plugins, underused commands, MCP integration, and daily workflow optimization — all sourced from Boris Cherny, Cat Wu, and the Anthropic team. The densest single-article tour of Claude Code's power-user surface available as of mid-2026.

---

## Key Quotes

> "Give Claude a way to verify its own work."

The article's thesis in one sentence, attributed to Boris Cherny. Without verification, you're the only feedback loop; with it, Claude iterates until code actually runs. This is pegged at 2-3x quality improvement. The implication: **verification infrastructure matters more than prompt quality.** This echoes the core argument of [[Guardrails and Feedback Loops]] — deterministic enforcement beats instructions.

> "The model performs best if you treat it like an engineer you're delegating to, not a pair programmer."

Cat Wu's rule. The distinction is subtle but sharp: delegation means giving a task with acceptance criteria and walking away; pair-programming means real-time correction. The delegation model scales; pair-programming doesn't. This is the mental model that makes [[Agent Coding Workflow]] work at all.

> "CLAUDE.md is compounding infrastructure — every mistake becomes a rule."

The article's most quotable insight. Boris's practice of telling Claude "Update CLAUDE.md so you don't repeat this" after mistakes means the instruction file appreciates over time. Unlike documentation (which rots), CLAUDE.md gets *better* with use. This is genuinely novel infrastructure thinking. See also [[Writing a Good CLAUDE.md]] and [[How Boris Uses Claude Code]].

> "If you do something more than once a day, turn it into a skill."

The heuristic for skill creation. Combined with the fact that skills load only their frontmatter at session start (~100 tokens), this makes skills the cheapest form of reusable context. Compare with dumping everything into CLAUDE.md — that's linear token cost on every session. Skills are *O(1)* until invoked. This is the architecture insight [[MinMax Skills]] builds on.

> "Setup is the work; execution is verification."

The closing punchline and the article's real thesis. Your job shifts from writing code to creating the conditions where Claude writes good code. Everything else — CLAUDE.md, skills, subagents, MCPs, rules — is just different forms of setup.

---

## Key Themes

#claude-code #skills #subagents #CLAUDE.md #MCP #workflow #devtools #compound-engineering

---

## Critical Analysis

**The density is the value.** This isn't an essay; it's a manual disguised as a blog post. In 27 minutes of reading you get: directory structure, CLAUDE.md philosophy, skill authoring, subagent design, plugin ecosystem, command reference, MCP configuration, daily workflow, and team tips. Most articles cover one of these. The trade-off is that nothing gets deep treatment — it's a map, not a territory. For depth on specific topics, you'd need [[A Guide to Claude Code 2.0]] (skills and subagents) or [[The Claude Code Playbook]] (workflow patterns).

**The "Compounding Engineering" concept is the real contribution.** Boris's insight that CLAUDE.md appreciates with use — every mistake becomes a permanent instruction — is genuinely novel. Traditional documentation rots because it's written once and never updated. CLAUDE.md, maintained this way, is more like a test suite: it captures learned behavior and prevents regression. This deserves its own term and deeper treatment than it gets here.

**There's a tension between minimalism and configurability.** Boris says keep CLAUDE.md short. But the ecosystem described — skills, agents, rules, plugins, MCP servers, hooks, settings — is a combinatorial explosion of configuration surface. The Anthropic team is preaching minimalism while building ever more places to put instructions. This isn't necessarily wrong — progressive disclosure means unused configuration costs nothing — but it creates a discoverability problem. New users won't know which of the 10+ configuration mechanisms to use for a given problem.

**The Obsidian MCP workflow is aspirational but overengineered for solo devs.** Three tiers of memory (hot/warm/cold), hierarchical folder structure, automated session logging via Stop hooks — this is an architecture you'd design for a team of five engineers who need shared context. For a solo developer, a CLAUDE.md and a notes file is probably sufficient. The article presents it as normative rather than conditional.

**The article is an optimist's guide.** Missing topics: cost management (Opus with xhigh effort burns tokens fast), model selection strategy, what to do when Claude gets stuck in a loop, how to debug agent errors, handling failures gracefully. The "combine `/goal` + auto mode + `/focus` and walk away" advice is great until it isn't — and the article doesn't cover what "isn't" looks like.

**`/goal` gets disproportionate attention for what it is.** It's a while loop with a test condition. The real power comes from combining it with hooks (auto-commit on success, notification on failure) and auto mode. Presenting `/goal` as the killer feature without emphasizing the composition with hooks undersells what the team actually built.

**The best tactical advice is buried in the middle.** "Pipe errors with `cat error.log | claude`", "use `@` syntax instead of describing files", "`Ctrl+G` to edit Claude's plan before implementation", "run 3-5 sessions in parallel via worktrees" — these specific behaviors change daily throughput more than any configuration file. The article would benefit from pulling these to the top.

---

## Cross-References

- [[A Guide to Claude Code 2.0]] — the closest sibling: deeper on skills, subagents, hooks, and context engineering
- [[How Boris Uses Claude Code]] — the source material for the CLAUDE.md philosophy
- [[Claude Code Cheat Sheet]] — complementary reference for commands and shortcuts
- [[Writing a Good CLAUDE.md]] — the craft of instruction files
- [[The Claude Code Playbook]] — workflow patterns and team practices
- [[Claude Code is a Beast — Tips from 6 Months of Hardcore Use]] — power-user field notes
- [[Gas Town After 10,000 Hours of Claude Code]] — the extreme end of Claude Code practice
- [[Gas Town's Agent Patterns]] — concrete agent patterns that emerge at scale
- [[Agent Coding Workflow]] — synthesis hub for agentic development practices
- [[Guardrails and Feedback Loops]] — the verification-first philosophy in depth
- [[MinMax Skills]] — skill design minimalism: what to leave out
- [[Elements of Agentic Systems Design]] — the broader agent architecture taxonomy
- [[Spec-Driven Development]] — plan mode as spec-first development
- [[Planning With Files]] — the "plan before code" pattern generalized
- [[Benchmarking AGENTS.md Changes]] — empirical evidence that instruction files matter

---

*Sources: [[raw/claude-code-mastery]]*
*Last updated: 2026-06-05*
