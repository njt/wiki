# cf — Cloudflare's Agentic CLI

Cloudflare launches `cf`, a successor to Wrangler built agent-first: CLI commands generated from OpenAPI schemas to cover the whole 3,000-operation API, JSON as the default interface, natural-language command discovery, and typed TypeScript configuration — on the explicit bet that agents, now 48% of Wrangler usage, are the primary user of developer tooling.

---

## What it is

The usage data is the story's spine: agents were a quarter of Wrangler use in March 2026 and 48% within six months, using ~2x the distinct commands per day and ~4x as likely to run six or more commands. Agents, not humans, are the heavy CLI user — and Wrangler only exposed ~280 of Cloudflare's thousands of operations.

cf closes that gap by generation rather than hand-crafting: the **Forge** pipeline builds CLI commands from the same OpenAPI schemas that drive Cloudflare's API docs and SDKs. Every Cloudflare product becomes reachable from one tool — deploy a Worker, monitor it, protect it with Access, buy a domain, front it with WAF.

## The design properties that make a CLI Good For Agents

The post is unusually explicit about what agent-first means in practice. Drawing it out:

- **JSON by default, not tables.** Agents were appending `--json` and piping through `jq` because only some Wrangler commands supported it. cf inverts the default: condensed JSON for agents, pretty-printing for humans. The rationale is blunt — "You as the human customer of this CLI are, in reality, one step removed from using it."
- **Discoverability at scale.** 3,000 routes is too many to enumerate in context, so `cf cli search` does natural-language command lookup against a small index of API descriptions and parameters, and the agent is *told about the search command itself* on its first `--help`. Self-describing discovery is treated as a first-class feature, not a nicety.
- **Typed configuration as agent rails.** `cloudflare.config.ts` replaces TOML (no accessible schema) and JSONC (schema agents rarely used). Because agents with LSP plugins — the post names Claude Code and Codex — get type feedback in context, they edit the config accurately even with no prior exposure. The `bindings` helper makes every platform capability auto-completable and self-explanatory in the editor.
- **Forms for the human-in-the-loop residue.** Where an operation would take a "long and unwieldy sequence" of chained parameters (buying a domain), cf renders validated input forms for the human, or the agent can just do it.
- **Context injection via AGENTS.md.** cf can append agent-facing guidance files, part of why the authors claim agents adopting an unfamiliar tool is "actually less confusing" than learning a changed familiar one.
- **Familiar build substrate.** Vite as default dev server/build, with the plugin ecosystem — minimizing the novel surface an agent must learn to just the CLI itself.

## The clean-slate argument

The most interesting strategic claim: replacing a tool agents have absorbed into their training data is *easier* than changing it. Wrangler's years of docs and blog posts are baked into model behavior, so "changing how Wrangler works now goes against learned behavior." A fresh CLI with agent-oriented context injection sidesteps that. It is a notable inversion of the usual network-effects logic for CLIs — learned familiarity, once the moat, becomes the liability.

It also implicitly indicts the old org model: Wrangler's inconsistencies (`d1 info` vs `hyperdrive get` vs `workflows describe`) came from each product team building its own command UX. Schema-generated commands standardize by construction — the same argument as generated SDKs, applied to the shell.

## Analysis

This is the strongest production-grade signal yet that the agent-first CLI pattern is graduating from essay to default. cf independently arrives at most of the checklist in [[10 Principles for Agent-Native CLIs]] — machine-readable output as default, self-describing discovery, structured errors via types — from real usage telemetry rather than principle. The Forge approach also nuances [[Google Workspace CLI]]: both generate a single CLI from an API schema, but Cloudflare controls its own schemas end-to-end and can annotate them for CLI generation, which is why it can claim 3,000 operations rather than a dynamic command surface. It also slots into Cloudflare's broader agent-platform push documented in [[Cloudflare OS]], where the platform itself is being reshaped around agent customers.

The honest caveats are migration debt and the bet itself. 18 months of Wrangler maintenance is a grace period, and the claim that agents handle an unfamiliar tool better than a changed familiar one is plausible but unproven at scale — it leans on the assumption that context injection (AGENTS.md, help-time steering) reliably reaches every harness. And "humans are one step removed" is doing a lot of work; the form-based escape hatch concedes that some flows still want a human directly in the seat.

---

*Sources: [[raw/cloudflare-cf-cli-launch]], [[summary/cloudflare-cf-cli-launch]]*
*Last updated: 2026-09-29*
