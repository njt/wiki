# datapitfalls

datapitfalls is an LLM-powered linter for data work: give it a chart image, a SQL/Python snippet, a plain-English analysis description, or a whole PDF report, and it returns a structured audit of which pitfalls from the *Avoiding Data Pitfalls* taxonomy (Ben Jones, Wiley 2019) the artifact exhibits. What makes it interesting architecturally is that it is a **catalog-grounded detector** — the model is constrained to cite rules from a hand-built taxonomy, with every finding validated against the catalog — and that it ships as three surfaces (CLI, web app, keyless MCP server) from one shared engine.

---

## Architecture

The repo is a TypeScript ESM monorepo-of-sorts, ~2,800 lines of engine code plus a Next.js web app:

- **`src/analyze.ts`** (808 lines) — the core `detectPitfalls()`. Builds a prompt from kind-specific system instructions plus a serialized pitfall catalog block, calls the Anthropic API, and extracts findings from a forced `report_findings` tool call.
- **`src/taxonomy/`** — the crown jewel: one YAML file per pitfall rule (id, domain, severity, description, detection strategy, bad/good examples, remediation, book citation), compiled by `scripts/build-taxonomy.mjs` into `data.ts` (1,208 generated lines). `src/taxonomy/index.ts` has the single `rulesForInput(kind)` router both the engine and MCP server call, so rule selection can't drift between surfaces.
- **`src/mcp/`** — MCP server exposing the taxonomy. `catalog.ts` is pure functions over the taxonomy; `server.ts` wires the MCP SDK. Crucially, the MCP path imports *only* the taxonomy module — never `src/index.ts` — so `@anthropic-ai/sdk`, `mammoth`, and `fflate` are never loaded keylessly.
- **`src/bridges/semiotic.ts`** — zero-dependency bridge turning reports into Semiotic v3 annotation specs; deliberately off the engine's hot path.
- **`src/cli.ts` / `src/file-input.ts` / `src/pptx.ts`** — CLI with `--ci` exit codes, file routing, PPTX slide extraction (text + embedded chart images per slide).
- **`evals/`** — a dev-only harness scoring runs against golden fixtures (`fixtures/*.py` + `*.expected.json`), computing precision/recall/F1 and active/latent calibration, with model and effort sweeps and cost estimation.

## Key techniques

- **Active vs latent findings.** The system prompt forces a two-way classification: an "active" pitfall is evident from the artifact alone (a truncated axis, summing a Total row); a "latent" one is a risky pattern that only bites under an unseen data condition (`dropna()` before an aggregate matters only if nulls exist). Latent findings must carry an explicit `condition` field and be phrased conditionally. This is an honest epistemology for a tool that can see code but never the data it runs on.
- **Catalog-grounded output.** The model may only cite rule ids from the catalog; `extractFindings()` drops any finding whose `rule_id` isn't real and takes name/domain/severity/remediation *authoritatively from the catalog*, not the model. Hallucinated rules are structurally impossible.
- **Forced tool call with graceful fallback.** The report is collected via `tool_choice: {type: 'tool'}`, but newer Claude models reject forced tool choice with a 400 — so there's a regex of models that must be prompted instead, plus a retry-once-without-forcing path, plus one re-ask if an unforced model answers in prose.
- **Prompt caching on the taxonomy.** The catalog block sits in a second system block with `cache_control: ephemeral`; it's identical across requests of the same kind, so repeated audits pay full price only for the small artifact.
- **Chain inputs.** A whole workflow (data prep code → analysis → chart → narrative) can be scanned as ordered stages, with the prompt explicitly hunting for cross-stage pitfalls — a transform that biases a later chart, a metric computed one way and described another — that are invisible when stages are reviewed alone.
- **Experimental presentation variants.** A `variant: 'summary'` option appends an addendum asking for a two-sentence summary, an "avoided pitfalls" list (only with citable countermeasures, never a rule also reported as a finding), and consequence ratings (`changes-takeaway` / `weakens-support` / `polish`). The baseline request stays byte-identical; `evals/compare.mjs` A/B tests variants.
- **Unanchored annotations.** The Semiotic bridge emits annotations with no x/y because datapitfalls sees findings, not pixels — and Semiotic v3 *drops* unresolvable annotations rather than floating them, so the host must anchor them. A deliberately honest seam.

## Design decisions

The central trade-off is **precision over recall**: the system prompts repeat "be conservative, avoid speculation" and require evidence quoting, and the eval harness measures active precision and calibration rather than raw catch-rate. A false accusation in this domain (telling someone their analysis is wrong when it isn't) destroys trust faster than a miss. The dual-license split (MIT engine, CC BY-NC taxonomy) is itself an architecture: the valuable thing is the rule text, and it's kept separable from the code.

The keyless MCP server is the boldest bet: rather than the tool calling the API, it hands the rules to Claude and lets Claude do the checking — no key, no network, read-only hints on every tool. That inverts the usual "agent + API backend" pattern and only works because the taxonomy is rich enough to serve as a checklist plus review method.

Weaknesses: detection quality is wholly hostage to one model family (the eval harness exists partly because prompt/catalog changes are unregressable otherwise), and the "single artifact at a time" model means the connected-chain detection the README calls the real goal is still on the roadmap.

## Comparison notes

- Where [[Agentic Verification]] argues agents should run their own checkable loops, datapitfalls is a check that only a human-scale reviewer can act on — it audits the *analytical* claims, not the code's correctness, a layer most verification-loop writing ignores entirely.
- It shares DNA with [[The Lifecycle of LLM-as-a-Judge]]'s concerns about judge reliability, but answers them structurally: constrained rule ids, catalog-authoritative metadata, and a scored eval harness with precision/recall against golden fixtures rather than vibes.
- Compared to [[harnesseval — Harness vs Prompt for AI Code Review]], which studies whether harness structure or prompting drives review quality, datapitfalls is a data point for the "harness" side: the taxonomy-in-prompt plus forced-tool-call plus post-hoc validation is all harness, and the model is swappable (evals sweep models and effort levels, defaulting to Sonnet 4.6 at half Opus's cost).
- Its keyless MCP server inverts the pattern of API-backed MCP servers like [[MotherDuck Remote MCP Server]]: no auth, no network, no backend — the server is just a well-indexed knowledge base, and the client model is the engine.

---
*Sources: [[raw/datapitfalls]], [[summary/datapitfalls]]*
*Last updated: 2026-10-20*
