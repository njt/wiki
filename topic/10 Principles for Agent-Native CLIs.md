# 10 Principles for Agent-Native CLIs

Trevin Chow's definitive framework for building CLIs that agents can actually use — not just tolerate. Expands and replaces his earlier "7 Principles for Agent-Friendly CLIs" with lessons from building his own CLI, studying Cloudflare's Wrangler rebuild, and dogfooding HeyGen's CLI. The central inversion: design for agents first, and humans benefit. The reverse produces CLIs that are "inconsistent, prompt-prone, and stdout-only."

---

## Key Quotes

> "Increasingly, agents are the primary customer of our APIs."

Chow endorses Cloudflare's framing. This isn't speculation — it's observed usage patterns. When agents generate more API traffic than humans, CLI conventions become infrastructure.

> "Manually enforcing consistency through reviews is Swiss cheese."

On why Cloudflare's schema-enforced naming rules matter. Human code review can't catch `info` instead of `get` across thousands of operations. The enforcement has to be mechanical.

> "Design for agents first, and humans benefit. Designing for humans first and bolting on agent support is what produces the inconsistent, prompt-prone, stdout-only CLIs the first five principles exist to correct."

The organizing thesis. Every principle in the post improves the CLI for both audiences — but the design direction matters.

> "Errors are the highest-signal context an agent gets, because they fire exactly when the agent doesn't know what to do next."

Why principle 3 (errors that enumerate valid values) matters more than it seems. An error message is the one moment the agent is guaranteed to be paying attention and receptive to instruction.

> "Token costs are real. A bloated MCP description never gets read by a human, but every agent that loads it pays the toll on every call."

The MCP-specific extension of principle 5. Cloudflare's Code Mode MCP serves ~3,000 operations in under 1,000 tokens — most MCP servers burn that much on a single tool description.

> "Agents don't memorize one CLI at a time. They build a generalized model of what CLIs do, drawn from every CLI they've seen."

Why cross-CLI vocabulary consistency (principle 6) isn't cosmetic. When every CLI invents its own verbs, agents pay a re-learning tax on every tool.

---

## Key Themes

- **#concept** Agent-native design — the inversion of "humans first, agents tolerated" to "agents first, humans benefit"
- **#concept** Schema-driven consistency — mechanical enforcement at the codegen layer, not human code review
- **#pattern** Three-layer introspection — `--help` (human), `agent-context` (structured/versioned), `SKILL.md` (long-form workflow)
- **#pattern** Persistent job ledger — solves the submit-poll-collect arc across disconnected invocations
- **#pattern** Two-way I/O — `--deliver` routes artifacts to agents; `feedback` closes the reporting loop back to maintainers
- **#tool** Cloudflare Wrangler — reference implementation: TypeScript schema generates CLI, SDKs, Terraform, MCP server
- **#tool** HeyGen CLI — exemplar for async-aware execution, artifact delivery, and skill manifests
- **#concept** Token budget as design constraint — both runtime output and MCP tool descriptions must be bounded

---

## Critical Analysis

This is the most practically useful CLI design document I've read in the agent era. Chow has the rare combination of: (1) having actually built agent-facing CLIs, (2) having watched agents use them and break in interesting ways, and (3) being willing to revise his own framework publicly when it proved incomplete.

The two-tier structure is right. Tier 1 (non-interactive, JSON output, teachable errors, safe retries, bounded responses) is table stakes — get these wrong and agents literally cannot use your CLI. Tier 2 (vocabulary consistency, introspection, async-awareness, identity, two-way I/O) is where compounding happens — the CLI gets more useful the more agents use it because each interaction leaves the environment better than it found it.

The strongest insight is one Chow understates: **Tier 2 is impossible to maintain by hand.** Every principle in Tier 2 — consistent verbs across subcommands, versioned introspection that stays in sync with implementation, async detection, profile precedence — is the kind of thing humans are bad at enforcing consistently and codegen is trivially good at. Cloudflare's TypeScript schema isn't a side note; it's the load-bearing architecture that makes all ten principles hold across thousands of operations.

Principle 6 (cross-CLI vocabulary) is the one I'm most convinced about and the one least discussed elsewhere. Agents do build generalized models of CLI behavior from every tool they encounter. `info` instead of `get` doesn't break the agent — it succeeds slowly, burning tokens on `--help` and retries. The cost is distributed, invisible to any single invocation, and enormous in aggregate.

The two-way I/O principle (10) is the most novel. `--deliver` routing artifacts to stdout/file/webhook eliminates the "stdout to temp file then move" dance. But the `feedback` command is the genuinely new idea: a channel for agents to report friction back to maintainers. Most CLI maintainers never learn that a particular error message cost 3 retries because there's no reporting path. A local JSONL by default with optional upstream POST is exactly right.

What's missing: Chow doesn't address auth in the agent context. Agents need to authenticate differently than humans (service accounts, OAuth device flow, MCP auth). The profile system (principle 9) could carry credentials but Chow doesn't go there. Also, the framework inherits the CLI model's assumption that text streams over pipes is the right interop layer — agents may eventually want something richer, like structured event streams with typed schemas.

Still, this is the document I'd hand to any team building a CLI today. The principles are concrete enough to implement, the "what good looks like" sections give clear targets, and the blocker/friction/optimization framework makes prioritization obvious.

---

*Sources: [[summary/ten-principles-for-agent-native-clis]]*
*Also published at: https://trevinsays.com*
*Last updated: 2026-05-15*
