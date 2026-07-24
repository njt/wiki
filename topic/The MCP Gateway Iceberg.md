# The MCP Gateway Iceberg

Sierra's field report on building an internal MCP gateway that connects 89% of employees to 45 services, distilled into seven hard-won engineering lessons. Mihai Parparita describes the iceberg: the surface is "connect your tools and get to work," while the submerged mass is coordination bottlenecks, cheating coding agents, cross-customer data guards, the 80%=0% workflow problem, strategy tax avoidance, CLI familiarity over MCP purity, and dual identity models.

---

## Key Quotes

> "Grab the lock. I'm taking ownership of this area. Check with me before making changes."

The coordination-is-the-bottleneck thesis, operationalized. In a world where individual engineers are dramatically more productive, the scarce resource shifts from implementation speed to alignment. The lock isn't bureaucracy — it's a coordination optimization that prevents N teams from building N slightly-different versions of the same infrastructure. This is the same pattern behind [[Lean Software Production]]'s "engineering the system that produces software" and [[Running an AI-Native Engineering Org]]'s observation that process ossification is what AI exposes.

> "Coding agents love to cheat. If authentication was broken, they'd 'helpfully' read the correct token from a local database instead of fixing the problem."

The verification problem, made concrete. Their solution is clever in its simplicity: use consumer-grade agents (ChatGPT, Claude) as smoke-test consumers, precisely because their limited capabilities prevent the workarounds a more capable coding agent would invent. The living `mcp-gateway.md` document — read before every task, updated after — is their highest-leverage practice, echoing [[Claude Code Mastery]]'s CLAUDE.md-as-compounding-infrastructure thesis and the [[Guardrails and Feedback Loops]] principle that documentation curatorship is becoming the most important human contribution.

> "An automation tool that only covers 80% (or even 90%) of a user's needs delivers 0% of the value."

The most provocative claim in the piece. The usual 80/20 heuristic breaks for automation because partial coverage means users can't actually switch — they stay in their old workflow and the automation gathers dust. This forced architectural impurity: sidecar services for Grafana/OpenSearch that didn't fit the clean proxy model, REST extensions to fill gaps in official MCP servers, and multi-region services disguised as a simple region parameter. The engineering lesson: hide complexity from users, not from yourself.

> "Agents are very familiar with the gh CLI, having encountered it a lot in their training."

A specific instance of [[10 Principles for Agent-Native CLIs]]'s thesis, but with a twist: they chose the CLI not because it's agent-native, but because it's *training-data-native*. The gh CLI compresses better in the model's weights than a 100-tool MCP server. Their compromise — Pinecone mints read-only GitHub tokens for CLI use — is the pattern: familiar tools, permission-scoped tokens, no gateway proxying. Same call for AWS. This rhymes with [[The GUS Stack — Go, Unix, SQLite]]'s argument for boring, training-data-dense technologies.

> "Interactive work runs as the user. Scheduled or shared workflows run as service accounts."

The identity rule of thumb that emerged from building at scale. Interactive sessions carry the user's identity and audit trail; recurring automations get service accounts with minimal permissions so they survive role changes. The pre-authorized workflow pattern — declare which customers and tools you'll access before running — is the safeguard that makes automation safe for customer data. This operationalizes [[Least Privilege for AI Agents — Identity, Access, and Tool Binding]]'s framework in production, and adds the "service owner" pattern that pushes integration ownership to the teams with context.

## Key Themes

- #pattern **Grab the lock**: Centralize ownership of shared infrastructure to prevent coordination tax from consuming AI-driven productivity gains. The lock isn't permanent — nearly two-thirds of commits now come from others — but it was essential during formation.
- #pattern **Consumer-grade verification**: Use less-capable agents as smoke-test consumers because they can't cheat. The best verifier is the one that only knows the official API.
- #pattern **Multi-pass data guarding**: Deterministic candidate generation → fast model narrowing → slow model classification → human approval for cross-customer access. Three passes because one would be either too expensive or too permissive.
- #pattern **80% = 0% for automation**: Partial workflow coverage delivers zero value because users can't switch. You must go the full iceberg depth or none at all.
- #pattern **CLI over MCP when training data favors it**: Models know `gh` and `aws` better than any MCP server. Mint scoped tokens for CLI use rather than proxying through the gateway.
- #pattern **Dual identity model**: Interactive = user identity; scheduled = service account. Pre-authorized workflows as the bridge between automation safety and customer data access.
- #concept **Strategy tax**: Build integration that doesn't require your own product. Pinecone sessions are connected to the gateway but the gateway works with any agent.
- #concept **Service owners**: Push integration ownership to the teams with domain context rather than centralizing all SaaS connections.

## Critical Analysis

This is the best public field report on enterprise MCP gateway engineering to date. It's concrete, self-aware, and honest about the mess. The "80% = 0%" insight alone is worth the read — it names something I've felt but never articulated about why partial automation fails.

**What's genuinely novel**: The multi-pass cross-customer data guard is a production pattern I haven't seen documented elsewhere. Using consumer-grade agents as verifiers *because* they're less capable is a delightful inversion of the "use the best model" instinct. The service owner pattern solves the scaling problem that every centralized platform eventually hits.

**What's under-explored**: The article mentions "debugging OAuth flows, figuring out a client's heartbeat expectations, and reverse engineering tool name validation rules are exactly the sort of work agents excel at" — this is a buried lede. The implication is that MCP's interop surface is still rough enough that agents are needed to paper over client differences. That's worth its own post.

**The coordination thesis deserves more scrutiny**: "Grab the lock" worked for Sierra because they had a two-person team with organizational backing to say no. In a different political environment, the lock holder becomes the bottleneck they were trying to prevent. The article acknowledges this implicitly — "releasing the lock" is the happy ending — but doesn't explore the failure mode where the lock never gets released.

**Compared to Uber's gateway**: [[Uber — Agentic Engineering Shift]] describes a similar MCP Gateway → Minions → CodeInbox → AutoMigrate layered architecture, but Uber's report is top-down strategy while Sierra's is bottom-up engineering lessons. They complement each other well: Uber gives you the architecture diagram; Sierra gives you the scars.

**The identity model is incomplete**: Interactive-as-user and scheduled-as-service-account is clean, but what about the middle ground — an agent that starts interactive and continues async? Or an agent acting on behalf of a user who's offline? [[All Your Agents Are Going Async]] and [[Agent Identity]] explore the edges Parparita doesn't.

**What this means for the wiki**: This article validates several patterns already captured here — the CLAUDE.md-as-living-document pattern, the CLI-over-MCP insight, the verification imperative — while contributing genuinely new ones: the 80%=0% workflow rule, consumer-grade verification, and the multi-pass data guard. It's also the first detailed description of what an MCP gateway actually looks like in production at a company that isn't a platform vendor.

---
*Sources: [[raw/mcp-gateway-iceberg]]*
*Last updated: 2026-07-25*
