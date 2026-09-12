# Swamp Club

An open-source, local-first framework for giving AI agents repeatable, typed workflows. Agents build Zod-validated models, compose them into DAG-based workflows, and produce immutable versioned data on every run. Built-in encrypted vaults for credentials, agent-created extensions for integrations, and structured Markdown/JSON reports after every execution. From System Initiative; over 1.3 million automation events logged.

---

## Key Quotes

> "Adaptive workflows for AI agents."

The tagline says it all: this isn't a workflow tool that happens to support agents. It's built agents-first, with the assumption that the agent is the one constructing and running the workflow.

> Before: Temporary scripts that deteriorate / After: Typed, reusable models

The before/after table is the real pitch. Every "before" item is a failure mode that practitioners of agentic development know intimately — scripts rot, context evaporates between sessions, secrets scatter, logs vanish.

## Key Themes

#tool #agent-architecture #workflow

- **Agents as workflow authors, not just executors** — Swamp Club's key bet is that agents should *build* the workflow, not just run inside one. The INIT/BUILD/COMPOSE sequence has the agent auto-discovering capabilities, generating typed models from docs, then assembling them into DAGs. This is the opposite of [[n8n]]'s model where humans design the workflow graph and agents are nodes within it.

- **Zod schemas as the type system** — Using Zod for model validation means workflows get runtime type checking that TypeScript agents already understand. This is a pragmatic choice: Zod is the de facto schema library in the TS/JS ecosystem, and it means agent-generated models are immediately validatable without a separate compilation step.

- **Immutable, versioned data** — Every execution produces versioned artifacts. This addresses the [[Context Rot]] problem directly: instead of agent outputs disappearing into conversation history, they're preserved as searchable, immutable data. Combined with structured reports, you get an audit trail that's missing from most agent workflows.

- **Encrypted vaults** — Credential management built into the framework, not bolted on. Similar to [[OneCLI]]'s approach to agent credential management, but integrated into the workflow runtime rather than operating as an external proxy. Referenced through expressions, so secrets never appear in workflow definitions.

- **Agent-created extensions** — Extensions aren't plugins installed by humans; they're integrations the agent builds that become permanent system components. This is a bet on agents accumulating capability over time, similar to the self-improving skills loop in [[Hermes]].

- **DAG-based execution** — Workflows are directed acyclic graphs with parallel execution and nested composition. This is the same computational model as CI/CD pipelines, make targets, and [[workgraph]]'s persistent task graphs. Well-understood, well-tooled, and importantly: deterministic in execution order.

## Critical Analysis

Swamp Club is the most coherent answer I've seen to "what happens after an agent finishes a task?" Most agent frameworks focus on the conversation — giving the agent tools, memory, context. Swamp Club focuses on what the agent *produces*: typed models, versioned data, reusable extensions. That's a meaningful architectural distinction.

The System Initiative pedigree matters. Adam Jacob (Chef co-founder) built System Initiative as a real-time infrastructure management platform with a reactive DAG engine. Swamp Club looks like that same engine repurposed for agent workflows — which means the execution model is probably battle-tested, not a prototype.

The before/after framing is honest and specific. "Temporary scripts that deteriorate" is what actually happens when you ask Claude Code to write a one-off automation. The scripts work once, then the next session has no memory of them. Swamp Club's answer — typed models that persist as extensions — is the right shape of solution.

The risk is adoption friction. Installing a framework, learning its model/workflow/vault abstractions, and restructuring how your agents work is a bigger ask than "just use Claude Code with a good CLAUDE.md." The 1.3M events number suggests real traction, but it's unclear whether that's broad adoption or a few power users running heavy automation. The examples (Proxmox inventory, Jellyfin remediation, key rotation) skew toward homelab/infrastructure — which is a natural fit but also a niche.

The comparison to [[n8n]] is instructive. n8n gives humans a visual workflow builder with AI as one node type. Swamp Club gives agents a programmatic workflow builder with humans reviewing YAML. Both are "workflow automation" but they have opposite opinions about who's in charge. For the [[Agent Orchestration]] crowd — people building systems where agents coordinate other agents — Swamp Club's model is more natural. For teams that want human-visible, human-editable automation, n8n wins.

Missing from the landing page: pricing (presumably free/open-source), governance model, how extensions are shared between agents or teams, and whether workflows are portable across different LLM providers. The "from System Initiative" footer and open-source claim suggest a company-backed open-source play, but the business model isn't visible.

---
*Sources: [[summary/swamp-club]]*
*Last updated: 2026-05-14*
