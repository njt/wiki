# Chiaro Methodology

An open-source SOC 2 audit methodology designed to be run by an AI agent, with published calibration examples that encode the judgment of a CPA firm as machine-readable training data. The methodology is the product of Y Assurance PLLC and powers the Chiaro platform at app.chiarohq.com.

---

## Architecture

The methodology is a structured data framework, not software. It's Git-repository as API: every file is JSON or Markdown, designed to be readable by both humans and the AI agents that execute the audit.

**Three-layer framework** under `framework/`:

- **`controls.json`** (5,560 lines): 89 controls keyed by stable refs (`GOV-01`, `IAM-03`) that never renumber. Each control carries a description, criteria mappings (to the 61 Trust Services Criteria in `criteria.json`), a gating question, follow-up questions, evidence goals, platform-specific hints (Google Workspace, Rippling, Gusto, Notion), and a `can_be_na` flag with conditions. Controls are organized into categories: Governance & Ethics, Identity & Access Management, Change Management, Data Protection, Business Continuity, Incident Response, People, Endpoint Security, Monitoring, Vulnerability Management, Network Security, Risk Management, Evaluation, and Vendor Management.

- **`test_attributes.json`** (16,790 lines): 369 test attributes, each with pass criteria (in both point-in-time and over-a-period forms), typical evidence, `na_condition` rules, and `calibration_examples` embedded directly in the attribute. Each attribute also has a `collection_hint` with `coverage_checks` listing specific questions, evidence lanes (`document`/`record`/`config`), and corroboration requirements.

- **`calibration-examples.json`** (5,282 lines): 528 worked judgment calls in one flat file — the AI's verdict, the correct verdict, and the reasoning. Each carries `source: "synthetic_taste_v1"` — real judgment patterns rewritten as synthetic scenarios to preserve client confidentiality while encoding the lesson. The distribution: 317 correct over-strict AI, 211 correct over-lenient AI.

**Supporting framework files:**
- `criteria.json` (478 lines): the 61 Trust Services Criteria
- `evidence_map.json` (369 lines): 22 evidence sources mapped to the controls they satisfy

**Method layer** under `method/`:
- `scoping-playbook.json` (2,714 lines): classifies every system a SaaS company uses — from AWS to Claude Code to Intercom — into `in_scope_tool`, `infrastructure_dependency`, or `vendor`. Uses a three-test matrix (`customer_data_flow`, `commitment_risk`, `control_enforcer`) scored Y/N/P per system type. Includes a GITC (General IT Controls) matrix mapping each system type to relevant control IDs and evidence examples. Covers 37 system types with ~280 known vendors and aliases.
- `collection-rules.md` (418 lines): the verbatim instructions the platform server sends to the client's AI agent. Hard rules against fabrication ("you may summarize, never generate"), anti-fabrication gates, screenshot timestamp requirements, contradiction handling with `contradicts` anchoring, and the drill protocol for walking every in-scope system.
- `type2-testing.md` (274 lines): complete-population-by-default methodology with census evaluation, deterministic/read-check split, the deviation rule (isolated vs. systematic), and the seeded sampling fallback algorithm.
- `tools.md` (847 lines): full inventory of 40 tools the client's AI can call, with the exact descriptions the AI reads to decide what to do. The tool descriptions *are* the method — the product is that the client's own AI runs the firm's procedure.
- `hipaa-crosswalk.md` (137 lines): 64-row mapping of 45 CFR 164.308–164.316 to the control library with stated resolutions.

## Key Techniques

### Complete population testing, not sampling

This is the most significant departure from standard audit practice. The argument: audit sampling exists because human hours are expensive. But the evidence in modern companies is produced by machines and can be verified the same way, at machine speed. If the complete population is retrievable and every item can be checked, checking a subset is a choice, not a constraint.

The population completeness is corroborated at three tiers: (1) recorded retrieval always — the exact query plus unedited output, re-runnable; (2) reconciliation against an independent second source wherever one exists (HR roster vs. IdP user list, change log vs. deployment history); (3) where no second source exists, a written management representation alongside the recorded retrieval.

The per-item evaluation is layered by judgment intensity: deterministic checks (dates, status fields, identity comparisons — computed, not judged, exact at any population size) and read checks (prose/documents evaluated by calibrated AI reading). Every candidate deviation is confirmed by the CPA before becoming an exception, and a slice of clean items is re-checked as quality control.

### Seeded sampling as fallback, not default

When sampling is unavoidable (paper records, systems with no export path, slow third-party evidence), selection is deterministic: `seed = SHA-256(engagement id | control ref | SHA-256 of the banked population)`. Selection is stratified over the period: the population is sorted by occurrence date and split into K contiguous runs, one draw per run from `SHA-256(seed | run index)`. The property: selection is fully determined the moment the population is banked. Nobody can re-roll.

### Calibration examples as judgment vectors

The 528 calibration examples are the mechanism for making AI judgment auditable. Rather than writing a rulebook of edge cases, each example encodes: what the evidence showed, what the AI concluded, what the correct conclusion was, and why. The examples live both nested in each attribute (for context during evaluation) and in a flat file (so they can be found and aggregated). This is a pattern worth studying: encode expertise as a dataset of corrected decisions rather than as procedural rules.

### The scoping playbook as taxonomy

The scoping classification system is unusually complete. It maps every system a modern SaaS company touches — from AWS to Claude Code to Promptfoo — into a three-way classification with a reasoning template. The three-test matrix (`customer_data_flow`, `commitment_risk`, `control_enforcer`) is applied systematically rather than ad hoc. The GITC matrix extends this by mapping each system type to the specific controls it bears on.

### Anti-fabrication as structural constraint

The collection rules encode several structural defenses against AI fabrication: (1) real artifacts only — verbal claims never satisfy sample-type attributes; (2) verbatim capture with qualifiers — the messy "yes, but only for X" is the audit signal; (3) mismatched evidence defaults to not-submitting; (4) verify evidence belongs to this client before submitting (company names, people, system identifiers, dates). These are not guidelines — they are hard rules that the server-side gates enforce.

## Design Decisions

**Published methodology, proprietary platform.** The methodology is CC BY 4.0. Anyone can fork it, run their own readiness against it, or challenge a specific call. But the evaluation engine and the exact phrases the server-side gates match on are not published. This is a deliberate split: transparency on *what* is tested and *how* judgments are made; opacity on the *automation* that makes it fast.

**Census over sampling, with honesty about the seam.** The method says complete populations by default, but 79 attribute texts still say "for a sample of." Rather than holding back the method until every text catches up, the seam is published and named: the method governs over the wording. This is trust-through-disclosure: the inconsistency is visible to anyone who reads both files.

**Complete populations are the default, not the only option.** The sampling lane exists and is disclosed per-control in the report. "Complete population" is the default; "sampled, full retrieval impracticable" is the exception, stated in the deliverable. This keeps the lane honest: if every control could comfortably land in sampling, the default has no meaning.

**Deviations: isolated vs. systematic, with published rules.** A census changes what a deviation means — there's no sampling risk, no projection. D deviations out of N occurrences is a fact about the period. The systematic markers (same root cause recurring, concentration, >5% rate, persistence across period, design-level cause) are published so the rule can be challenged. Two asymmetries: the CPA may always conclude harsher than the rule; concluding softer requires a recorded reason.

**Remediation never erases a deviation.** A termination caught late and fixed in March is still a deviation in the report, disclosed alongside management's response. This is a hard rule with no exceptions.

**Judgment encoded as examples, not rules.** Rather than trying to write a rulebook that covers every edge case, the methodology encodes judgment as a dataset of corrected decisions. Each calibration example says what the AI got wrong and why. This is a fundamentally different approach to encoding expertise — it's supervised learning for human-supervised AI, where the training data is public and challengeable.

## Comparison Notes

Unlike [[Aviator Verify]], which captures developer intent and verifies it against running code, Chiaro captures *audit criteria* and verifies *evidence* against them. Both share the pattern of structured verification replacing human review, but Aviator Verify operates inside the development loop while Chiaro operates at the compliance boundary.

The calibration example pattern is a more structured version of what [[SimpleEnglish]] does with its 53 aerospace rules — both encode correctness as a finite set of falsifiable constraints, but Chiaro's examples encode *judgment* (the correct call in an ambiguous situation) rather than *form* (the correct way to express something).

Chiaro's agent architecture — a client's own AI running the firm's procedure under hard rules — inverts the [[Cloudflare Security Audit Skill]] pattern. Cloudflare runs its own agents against the target; Chiaro runs the *target's* agent against the target, with server-side gates preventing fabrication. The trust model is different: Cloudflare trusts its own agents; Chiaro trusts its procedural constraints more than any agent.

The published-deviation-rule pattern shares DNA with [[VulnHunter]]'s Hunt → Fix → Verify pipeline — both have a falsification step before the conclusion reaches a human. Chiaro's "every candidate deviation confirmed by the CPA before it becomes an exception" and VulnHunter's adversarial falsification engine serve the same function in different domains.

---
*Sources: [[raw/methodology]], [[summary/methodology]]*
*Last updated: 2026-08-06*
