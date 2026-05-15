# Prime Radiant (Company)

Jesse Vincent's AI company, incorporated late 2025. Ships open-source agent tools ([[Serf]], [[Clearance]], [[engineering-notebook]]) while building an undisclosed AI product. The public tools are infrastructure spillover from the private product work -- released because they're useful and because they need to exist anyway.

---

## Origin

Announced January 13, 2026 on Vincent's blog. Incorporated just before Christmas 2025. First employee joined the week of the announcement. The company site at [primeradiant.com](https://primeradiant.com) was deliberately sparse at launch -- "not quite ready to talk about exactly which AI stuff we're at work on."

The name is almost certainly an Asimov reference (the Prime Radiant is the device containing the Seldon Plan in *Foundation*). For a company building AI tools and planning infrastructure, the resonance is on-brand: a plan that unfolds predictably, guided by probabilistic reasoning about the future.

## What's Public

Three open-source tools released under the Prime Radiant banner, all Apache 2.0:

- **[[Serf]]** -- Non-interactive coding agent. Give it a task, it works until done. Go-based, multi-provider.
- **[[Clearance]]** -- Native macOS Markdown viewer/editor with first-class YAML frontmatter support. Swift, local-first.
- **[[engineering-notebook]]** -- CLI tool that ingests Claude Code and Codex sessions and serves a browsable engineering journal.

The product thesis across these three is coherent: Serf generates output (Markdown specs, code, logs), Clearance reads it, and engineering-notebook records how it got made. Together they form a pipeline where agents write and humans review.

## The Hiring Problem

> "I need to figure out how to hire folks in this crazy modern world where any job posting will attract an unending torrent of agentic job applications."

This is the article's sharpest observation and it's only going to get worse. When every job posting triggers a thousand LLM-generated applications, the hiring pipeline inverts: the signal isn't in who applies but in how you filter. Vincent doesn't describe his solution, but the implication is that traditional resume-screening is dead. The companies that figure this out first get access to talent everyone else is drowning in noise trying to find.

## What Vincent Brings

Jesse Vincent (@obra) is one of the most prolific builders in the agent-tooling space. Before Prime Radiant he created [[Superpowers]] (Claude Code workflow system), packnplay (container sandboxing), episodic-memory, external-subagents, and cage (security sandboxing). He's a central figure in the [[Awesome Vibez]] community alongside Dan Shapiro, Wes McKinney, and Harper Reed.

His pattern is instructive: release tools that scratch your own itch, keep them open-source, let the community validate and extend them. Prime Radiant formalizes what he was already doing as an individual -- the company structure presumably exists because the product ambitions exceed what one person can build.

## Critical Analysis

The public-outputs-as-spillover model is smart. Building agent infrastructure forces you to solve real problems whether your product succeeds or not. If the unnamed AI product works, the tooling pipeline is proven. If it doesn't, the tools are independently valuable. This hedges against the most common startup failure mode (building something nobody wants) by ensuring the byproducts have standalone utility.

The risk is the opposite: the public tools are *so* useful that they become the thing, and the actual product never materializes. Open-source agent tooling is a crowded space and maintaining three separate tools while building a stealth product is a lot for what was, as of this post, a two-person company.

The hiring observation deserves more attention than it gets. "Torrent of agentic job applications" is a phrase that dates this moment -- January 2026, when the problem had become acute enough for a founder to name it publicly but before anyone had shipped a standard solution. The hiring-platform disruption that follows from this problem (verifiable skill demonstration replacing resume screening, AI-filtering-AI arms races, credentialing systems that can't be gamed by agents) is one of the under-explored second-order effects of LLM availability.

---

*Sources: [[raw/i-started-a-company]]*
*Last updated: 2026-05-15*
