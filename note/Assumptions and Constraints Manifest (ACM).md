# Assumptions and Constraints Manifest (ACM)

ACM (github.com/roblarsen/ACM) is a proposed open standard that makes a code change's silent operational assumptions explicit, machine-readable, and CI-enforced at the pull-request boundary. A PR must carry a YAML manifest of typed contracts — idempotency, concurrency safety, horizontal scalability, ordering, data-loss safety, plus four distributed-systems primitives — where every claim is `guaranteed`, `conditional`, `unsupported`, `not_applicable`, or `unknown`, and a small TypeScript CLI rejects manifests that claim guarantees without evidence or declare low risk while critical contracts are unproven. It matters because it attacks the review bottleneck created by AI-generated code at its actual source: not the code, but the unstated assumptions embedded in it.

---

## Architecture

The repo is ~1,800 lines, mostly specification prose; the only executable artifact is `tooling/acm-cli` (TypeScript ESM, commander + chalk + yaml + zod, ~370 source lines).

- **Two-stage validation pipeline** (`src/parser.ts`): `extractACMBlock` handles the delimiter grammar, then `parseAndValidateACM` does schema + semantics + structure.
- **Schema defined three times**: the zod schemas in `src/types.ts` are what the CLI actually enforces; `schemas/acm.schema.json` (JSON Schema Draft 2020-12) and `schemas/acm.schema.yaml` (OpenAPI) mirror them for other toolchains. Three hand-maintained copies of one enum set is an acknowledged drift risk — nothing cross-checks them.
- **Delimiter extraction** (`extractACMBlock`): matches `<!-- ACM-START/END -->` or `<acm></acm>` tags case-insensitively, with a fence-aware scanner (`findDelimitersOutsideFences`) that toggles a boolean on ```` ``` ````/`~~~` lines so delimiter-looking text inside fenced code doesn't count; it re-patches per-line `match.index` values into document offsets by hand (manual `\r\n` arithmetic). It enforces exactly one block ("singular delimiter grammar" as anti-bypass), balanced delimiters, and falls back to treating a file that starts with `---` as a standalone `.acm.md` manifest.
- **Semantic layer** (`validateSemanticCoherence`): the cross-field rules live here, not in the schema — evidence/condition obligations and risk floors (see below).
- **Structure layer**: four mandatory markdown sections (`MANDATORY_SECTIONS` in `types.ts`); the checker builds a case-insensitive regex per `### <header>`, strips HTML comments, and rejects bodies that are empty or literally `*` or `-` — a direct countermeasure against filling the section with a placeholder bullet.
- **Prompt-side enforcement** (`templates/.cursorrules.md`): a system prompt casting the model as an "uncompromising Principal Distributed Systems Architect" that must emit code *and* a manifest ("dual-output generation"), plus an adversarial self-audit (2,000 concurrent requests across 4 pods; 30-second downstream hang; `SIGKILL` between lines).
- **CI gate** (`.github/workflows/acm-governance.yml`): writes the PR body to a temp file, runs `npx acm-cli validate --file`, posts/updates an idempotent governance comment keyed on a `<!-- ACM-GOVERNANCE-AUDIT -->` marker (with `continue-on-error` so the comment always posts), then a separate step fails the check if the exit code was nonzero.

## Key techniques

- **`unknown` as a first-class status.** The five-value contract status includes "not analyzed" alongside the honest answers, and the risk floor treats `unknown` identically to `unsupported` — so *haven't looked* is never free. This is the sharpest idea in the design: it makes silence itself a falsifiable claim.
- **Cross-field coherence rules instead of per-field validation.** `delivery_semantics: at_least_once` + `idempotency: unsupported` → warning about duplicate side-effects; `backpressure_strategy: unbounded_buffer_risk` + `risk_level: low` → hard error. The manifest is checked for *internal consistency*, not just schema shape.
- **Asymmetric strictness.** `guaranteed` without `evidence` is a hard error; `unsupported` without `failure_mode` is only a warning. The linter cares more about preventing false confidence than about forcing doom-saying (the `.cursorrules` prompt fills the gap by demanding failure modes anyway).
- **Evidence as a string with `min(3)`.** The "proof obligation" is enforced as a 3-character minimum — an anti-laziness floor, not a proof system. Nothing can verify the cited lock or test actually exists.
- **Worst-case defaults.** The PR template ships with all five contracts pre-set to `unsupported` and empty `failure_mode` fields, so the author must actively *downgrade* claims to submit — the template's default posture is distrust.
- **Honest examples as tests.** The three examples deliberately document broken systems (the Go Kafka consumer marks `data_loss_safety: unsupported` because `reader.ReadMessage()` auto-commits offsets before the DB write), and `cli.test.ts` asserts the CLI accepts all three — validating that honest failure declarations pass, not that happy manifests do.

## Design decisions and gaps

- **Self-attestation is the load-bearing trade-off.** The CLI verifies a manifest is complete and self-coherent; it cannot verify it is *true*. The scheme's bet is that moving the reviewer's job from reverse-engineering assumptions to auditing declared ones is still a large win — but the "Evidence Verification" checkbox in the PR template is where the real verification silently lives. The README's claim that risk floors "mathematically forbid" low risk is a rule-table lookup, not mathematics.
- **Closed enums buy telemetry and cost expressiveness.** Fixed vocabularies (`exactly_once_claimed` — a wry nod to Kafka semantics — rather than `exactly_once`) make aggregated "drift telemetry" across PRs possible, which the whitepaper pitches as the strategic payoff: count `concurrency_safety: unsupported` per service and you have a technical-debt heatmap.
- **Spec–implementation gap.** Spec §4.3 defines four coherence rules; the CLI implements three. Rule 4 (strict ordering claimed while delivery is at-least-once must document partition keys) exists only on paper. The `.cursorrules` prompt also demands `unbounded_buffer_risk` imply `high`/`critical` risk, while the CLI only rejects `low`.
- **Rendered-PR blind spot.** The manifest lives inside HTML comments (`<!-- ACM-START -->`), which GitHub strips from rendered PR bodies — so in the rendered view reviewers see nothing and must read source or rely on the bot's audit comment. Compliance theater risk: the gate can pass while no human ever reads the manifest.
- **Young-project rough edges.** README says `npx @open-standards/acm-cli` but the published package is `acm-cli`; the workflow's comment text still says "SPEC v1.0"; the heredoc that writes the PR body to disk can be terminated early by a PR body containing a line reading `EOF`.

## Comparison notes

- [[Load-Bearing Assumptions]] does the same job at a different lifecycle point: it surfaces falsifiable claims a *plan* depends on, before code exists; ACM declares runtime boundary assumptions *after* implementation, and adds machine enforcement. Plan-time assumption hunting plus PR-boundary contracts would compose well.
- [[Engineering Standards Enforcement at Cloudflare]] enforces standards by *detecting violations in code* with review agents; ACM enforces by *requiring a declaration* and checking its coherence. Agent-detection versus agent-declaration — complementary layers over the same merge gate.
- [[AI-Written Change Descriptions]] argues AI-generated PR prose is worse than useless because it omits the framing that makes review possible; ACM is the structured counter-argument — a constrained, schema-checked artifact where the format itself is the value, and free prose is exactly what it bans ("handles errors gracefully", "production-ready" are outlawed phrases in the `.cursorrules`).
- [[Silently Resolved Ambiguity Is Comprehension Debt of Intent]] names the debt accrued when an agent resolves an underdetermined decision with no record; ACM is precisely the tripwire that ledger needs, applied to operational rather than product intent — every silent assumption about state, clocks, and failure gets a signature before merge.
- The whitepaper's own lineage claim is fair: this is Design by Contract (Meyer) elevated from the method boundary to the distributed PR boundary, with ADRs kept as the macro complement — see [[Agents and Acquiring Debt]] for the argument that ADRs agents both write and read are the paydown mechanism for comprehension debt. It also instantiates the "per-change accountability contracts" called for in [[Own the Outer Loop]].

---

*Sources: [[raw/acm]], [[summary/acm]]*
*Last updated: 2026-09-13*
