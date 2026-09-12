# Building Shippy — Agent Architecture for High-Stakes Domains

How Ai2's Skylight team built a maritime AI agent for 70+ countries by decomposing the agent into soul/skills/config, wrapping a messy API in a deterministic CLI, isolating every user session in its own Kubernetes pod, and evaluating the whole agent against live data instead of static benchmarks.

---

## The Architecture

### Soul, Skills, Config — a named decomposition

Ai2 decomposes Shippy into three separable components, each with a different change cadence:

- **Soul:** The system prompt — persona and behavioral guardrails. Shippy "won't make legal determinations about whether a vessel is breaking the law" and "won't speculate beyond what the data supports." These are explicit prompt-level constraints, not implicit tuning.
- **Skills:** Markdown files describing how to handle specific request types, following the agent-skills spec used by Claude Code and Codex. Four skills (Skylight API query, EEZ/MPA boundary lookup, vessel track interpretation, map link generation) can be combined in a single turn — a query about vessels near a specific MPA draws on three skills simultaneously.
- **Config:** Runtime settings (harness, model, API keys) injected at runtime. Soul + skills are baked into a versioned Docker image; config changes don't require a rebuild.

> "The soul and skills are baked into a versioned Docker image; config changes don't require a rebuild."

This is a clean separation of concerns that mirrors how the field is converging. Skills as markdown files with structured frontmatter match the [[Agent-Native Architectures (Every)]] files-as-interface principle, and the decomposition into prompt/changing-context/runtime-config maps onto the six-component taxonomy in [[Components of a Coding Agent]]. The key insight isn't the categories themselves — it's that each has a different release cadence, and conflating them creates unnecessary rebuild friction.

### Deterministic tools for nondeterministic agents

The article's sharpest architectural insight:

> "Each layer narrows what the next layer can get wrong."

Shippy doesn't talk to the Skylight API directly. Early prototypes that did produced "a steady stream of subtle bugs" — the API has dozens of input types, nested filter objects, pagination cursors, and complex geometry inputs. Instead, the team built a purpose-built CLI (`skylight events search` with typed filter flags) that handles authentication, pagination, and structured output automatically.

Three design choices worth stealing:

1. **Output to local JSON files, not stdout.** This avoids pipe buffer issues and makes output programmatically accessible across multi-step analyses without the agent having to reconstruct state from conversation context.
2. **Extensive `--help` text and detailed error messages.** The CLI is designed for agent consumption first — verbose, unambiguous error messages that the agent can act on without guessing.
3. **Every layer tested independently.** The API has its own test suite, the CLI can be exercised by human or agent, and skills reference CLI commands — not raw API calls.

This is the [[10 Principles for Agent-Native CLIs]] playbook in production, and it's the same pattern Trevin Chow articulates: design for agents first, humans benefit. It's also the [[Layer-First Pattern — Keep Data Out of the LLM Context]] applied to tool design — the CLI is the data layer, and the LLM only touches structured flags and file paths, never raw API responses.

The file-output-to-JSON choice is particularly elegant. It answers the reader's question about how Shippy tracks which output belongs to which call: the agent specifies the output path, and every step reads from a known location rather than trying to thread state through conversation context. It's filesystem-as-working-memory, a pattern that shows up repeatedly in [[StrongDM Factory Techniques]] and the broader dark-factory tradition.

### Mothership: per-session Kubernetes as the isolation primitive

> "Mothership provisions a dedicated Kubernetes deployment for each user session."

This is the most important production detail in the article. Skylight serves "hundreds of government agencies and NGOs across over 70 countries." A fisheries officer in the Philippines and an analyst in Norway share infrastructure but must never share data. The solution: when a conversation opens, pods spin up containing the agent runtime, skills, and CLI. The user's JWT is injected at provision time. Session-scoped files are never shared across sessions. Network access is restricted to only needed services.

This is per-user-session isolation at the Kubernetes level — more heavyweight than the gVisor/OS-sandbox/sealed-VM patterns in [[How We Contain Claude]] but appropriate for a multi-tenant government platform where data isolation failures are career-ending rather than embarrassing. It's the same problem space as [[Building Agents That Don't Break Themselves]] (disposable execution environments) but solved at the orchestration layer rather than the container layer.

Mothership was "built to be general and to host other agents," and it's already spreading to EarthRanger (wildlife conservation) and OlmoEarth (Earth observation). The maritime domain is the first tenant, not the only one — a pattern that recalls [[Cloud Agent Lessons from Cursor]]'s observation that the dev environment IS the product.

## Evaluation: judge the agent, not the model

The team rejected static benchmarks and built an eval pipeline that scores "the whole agent – model, skills, and sandbox together – against live data." The process:

1. Domain experts write scenarios, rubrics, and per-criteria weights
2. A prompt runs through the full sandbox (real session, live data)
3. An LLM judge grades each criterion 0–1 with written reasoning
4. Weighted aggregate is checked against a fixed pass threshold
5. Experts annotate responses as ground truth

This runs on [[Razorback]]'s Harbor framework with a custom plugin that spins up real Shippy sessions. The suite runs in parallel against versioned builds, producing timestamped results and delta reports — so every build gets a before/after comparison.

The latest run surfaced three specific failure modes that a model-only benchmark would have missed:

> - Patrol-planning tasks: Shippy "overstepped into tactical recommendations rather than decision support"
> - Geometry-sensitive queries: boundary simplification caused missed Events
> - One case where the agent "invented a CLI command that didn't exist"

These are the kinds of failures that only show up in integrated testing. The overstepping problem is a soul/guardrail failure — the system prompt said "don't make legal determinations" but didn't cover tactical creep. The geometry bug is a tool-layer problem (boundary simplification in the API, not the model). The invented CLI command is a classic hallucination that a static eval would never catch because it depends on the specific CLI surface area.

This is the [[Guardrails and Feedback Loops]] thesis in practice: eval as the feedback mechanism that tightens the whole system, not just the model. It's also a concrete implementation of the eval pyramid from [[The Agentic Product Standard v2.0]], with domain-expert annotation as the gold layer and LLM-judge scoring as the scaling layer.

## Where this fits

### What's novel

The soul/skills/config decomposition gives a name to a separation that most teams feel intuitively but don't articulate. The CLI-as-deterministic-wrapper pattern is the most transferable technique in the article — it's applicable to any agent that talks to a complex API, and the file-output convention solves a real orchestration problem. Mothership's per-session Kubernetes isolation is the right pattern for multi-tenant government/enterprise deployments, and the Harbor-based whole-agent evaluation pipeline is what eval should look like when the cost of wrong answers is high.

### What's missing

The article doesn't discuss the cost model. Per-session Kubernetes pods aren't cheap, and there's no mention of pod startup latency or cold-start mitigation. The eval pipeline's LLM judge introduces its own failure modes — who judges the judge? The cross-thread memory plan ("persistent facts applied automatically across threads") is sketched but undesigned, and the model-routing plan is gestured at without the tiering architecture. These are the right problems to have, but they're also where the real engineering lives.

### What it means for the field

Shippy is a useful case study because it's a real production agent in a domain where wrong answers have consequences — not another coding benchmark or chatbot demo. The architecture choices (deterministic CLI wrappers, per-session isolation, whole-agent evaluation against live data) aren't maritime-specific; they're the patterns that any high-stakes agent deployment will converge on. The fact that Mothership is already spreading to wildlife conservation and Earth observation suggests the architecture generalizes.

The article is also a quiet rebuttal to the "just wire the agent to the API" school of agent design. Shippy's team tried that first, found it produced "a steady stream of subtle bugs," and built the CLI as a deliberate constraint layer. The pattern isn't "add more intelligence to handle the messy API" — it's "make the API less messy so the intelligence can focus on what it's good at." That's the [[Smart Models Dumb Pipes]] thesis, and Shippy is the production evidence.

---
*Sources: [[raw/shippy-tech-blog]]*
*Last updated: 2026-07-18*
