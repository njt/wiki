# deeper-research

A retrieval-first deep-research pipeline shipped as a shared skill for Claude Code and Codex (~18K lines of Python in `scripts/`, plus a 1,180-line `SKILL.md`). It produces a fact-checked, fully-cited "Research Bible" (Markdown + HTML + BibTeX + `claims.jsonl`) by refusing to let an LLM reason over its own memory: Round 1 fetches a real evidence corpus, a hard gate refuses thin material, and every later round is bound to what was actually retrieved. It matters as the most complete public example of *mechanical* grounding gates — checks that "can't be sweet-talked" — layered between LLM stages.

---

## Architecture

Six rounds, each with a dedicated helper script, all state on disk under a per-run directory (`Deeper_Research/<run-slug>/` with `Process/round1..5/`, `Sources/`, `sections/`):

- **Round 0** — `scripts/scope.py`: rule-based + optional LLM domain classification, injecting source priorities per field (PubMed, NBER, arXiv…).
- **Round 1** — `scripts/slice_search.py`: Exa search slices + a free academic anchor (OpenAlex-first, Semantic Scholar fallback), writing JSONL briefs + a manifest. `dispatch.py` is deliberately a slices-only guide, not a runner.
- **Gate** — `scripts/evidence_gate.py` (exit 0/22): must pass before synthesis. Crucially it **recomputes** corpus metrics from the `slice_*.jsonl` files rather than trusting the derived manifest — a hand-edited manifest cannot wave a thin corpus through.
- **Round 2/2.5** — synthesis over six exact question-bucket headers, then root-cause/consequence/gap deepening (`deepen_questions.py`) via Exa deep-reasoning, each answer keeping its retrieved evidence.
- **Round 3** — section planners + reconciler → slot-batched isolated integration agents; `dedup_bib.py` merges the bibliography DOI-normalized + fuzzy-title.
- **Round 4** — `verify_citations.py` (OpenAlex/Crossref resolution, three-state SSRF-hardened link probe), `classify_sources.py` tier audit, `lit_search.py` missing-canonical-works check, `lint_background.py`, `sweep_numbers.py`, then a refute-mode adversary on an *independent provider family* (OpenAI ↔ xAI excluded for common-corpus risk), then a fix pass.

The whole thing is orchestrated either by the host agent walking `SKILL.md`, or headless: `scripts/batch_research.py` fans out one isolated session per question with an adaptive concurrency controller (start at 2 workers, +1 per healthy 60s window, cap 8; on a 429/capacity error halve the target and pause 120s — never kill active work, never auto-retry evidence/budget failures).

## Key Techniques

- **The number-provenance sweep** (`sweep_numbers.py` + shared `numkeys.py`) is the cleverest piece: extract every numeric claim from prose with a Decimal-exact (never float — "1.07 * 1e9 in float leaves artifacts"), boundary-guarded, unit-aware matcher that normalizes `175,000` == `175k` == `175 thousand` and tries scale-suffixed alternates; then check the normalized key exists *anywhere* in retrieved evidence (Exa text, highlights, KB docs, deepening evidence). It's explicitly an **existence check, not source binding** — cheap and mechanical, with attribution left to the adversary.
- **Citation-strip surgery in `strip_nonclaims`**: markdown link labels survive (only URLs dropped), bracketed spans are stripped only when citation-ish or metadata-shaped, APA parentheticals and section references removed — so visible prose numbers stay checkable.
- **Crash-safe money ledger** (`scripts/ledger.py`): pre-charge worst-case before each metered call, reconcile actual after; a crash leaves a *phantom commitment* counted at full worst-case, deliberately eroding headroom — "over-counting spend is the safe failure." `allow_nan=False` on persist because a NaN would silently defeat the cap; NaN/ Infinity rejected at every load boundary.
- **Fail-closed exit-code grammar**: 21 = cap breach, 22 = thin corpus, 30/31/32 = coverage-audit failures. Exit 0 always means *verified*, nonzero always means *do not synthesize*.
- **Containment for batch Claude runs**: `--allowedTools` alone doesn't path-scope writes, so batch runs require an external wrapper that mounts the skill root read-only and makes only the run directory writable.
- **Grounding gates from a real incident** (2026-09): a deepening call fabricated figures (2.2%, US$175k, 930) with a fake "FAO Co-Chairs background document, 2018" citation that sailed past the citation verifier (wrong shape = invisible), the lint (unfenced), and the adversary (the qualitative point was right). The fix class: retain evidence with each answer and pre-mark unsupported quantities `[UNVERIFIED]`, flag unparseable cite shapes (`--fail-on kb-unknown,unparseable`), and let the fabricated figures themselves become the fixture tests.

## Design Decisions

The founding trade is stated in the README's lineage table: v1's five-model triangulation catches fabrications that disagree across providers but **ratifies** ones the models agree on (five models past their cutoff all "remember" a clause never enacted). v2 pays one retrieval pass to make agreement irrelevant — a fetched source is the evidence, not a vote. Within that, the dominant trade-off is *mechanical over LLM*: every gate that can be deterministic is deterministic, because "LLM review had already missed it twice — only mechanical checks can't be sweet-talked." The cost is brittleness (regex grammars for citations, number matching that misses paraphrase) accepted in exchange for gates that cannot be argued with. Secondary trades: fail-safe over fail-open everywhere (missing evidence marks everything), existence checks over full provenance binding (cheap tripwire first, adversary does attribution), and graceful degradation only where it's provably safe (semantic search exits 0 with a notice; citation chase never degrades the gate).

## Comparison Notes

- vs [[Local Deep Research]]: LDR is a plugin-architected monolith whose quality comes from breadth (25 search engines, LangGraph agent, egress DLP); deeper-research is narrower and harder-edged — one retrieval substrate (Exa + scholarly APIs) but mechanical grounding gates LDR has no equivalent of. LDR filters sources; deeper-research refuses to write at all if the corpus is thin.
- vs [[DeerFlow]]: DeerFlow is a general super-agent platform where research is one graph among many; deeper-research is a single-purpose pipeline where the graph *is* the product, and orchestration (adaptive batch fan-out) serves the gates rather than the reverse.
- vs [[A Serious AI Product]]: Glyph demanded verification as first-class UI and found none; deeper-research is the open-source existence proof of the harness side of his spec — citations resolved mechanically, numbers swept against evidence, disputes tagged `[disputed:]` rather than averaged. It just has no UI, only artifacts.
- vs [[Building Reliable Agentic AI Systems]]: Bayer's three-reflection-loop taxonomy is LLM-judged quality; deeper-research's round 4 shows the complementary pattern — deterministic tripwires between reflections, so each reflection starts from machine-verified material.
- Its own lineage contrast (v1 five-model triangulation → v2 evidence-first) is a clean case study for [[Sherlock Agent Eval]]'s finding that topology beats model size: same idea, different topology, radically different hallucination behavior.

#tool #project #agents #research #verification

---
*Sources: [[raw/deeper-research]], [[summary/deeper-research]]*
*Last updated: 2026-10-05*
