# Reality Check

A Python CLI and knowledge base framework for tracking claims, sources, predictions, and argument chains with structured epistemic rigor. You register claims with evidence levels and credence scores, link them to analyzed sources, map logical dependencies between arguments, and search the whole thing semantically via LanceDB vector embeddings. Ships with plugins for Claude Code, Codex, Amp, and OpenCode — it's designed to be used by AI agents as much as by humans. Apache 2.0, by Leonard Lin.

---

## Key Quotes

> "With so many hot takes, plausible theories, misinformation, and AI-generated content, sometimes, you need a realitycheck."

The motivation distilled: the information environment is so polluted that structured claim tracking is now a personal tool, not just an intelligence analyst's tool.

> "Claims build on each other across domains (AI claims inform economics claims)."

The design philosophy behind a single unified knowledge base — cross-domain synthesis is the point, not a side effect.

## Key Themes

#tool #concept #pattern

- **Structured epistemics** — Eight claim types (Fact, Theory, Hypothesis, Prediction, Assumption, Counterfactual, Speculation, Contradiction), six evidence levels (E1 Strong Empirical through E6 Unsupported), eleven domain codes. This is intelligence analysis tradecraft formalized as a CLI taxonomy.
- **Three-stage source analysis** — Descriptive → Evaluative → Dialectical. Not "read and summarize" but a structured analytical pipeline that forces you to separate observation from judgment from synthesis.
- **Epistemic provenance** — Reasoning trails that document *why* you believe something at a given credence level, not just *what* you believe. This is the audit trail that most knowledge management tools lack entirely.
- **Prediction tracking with falsification** — Claims of type [P] carry falsification criteria. This is the Superforecasting pattern: predictions without falsification criteria are just vibes.
- **Agent-native knowledge management** — Claude Code slash commands, Codex skills, YAML import/export. The tool assumes its primary operator may be an LLM agent, not a human typing at a terminal.

## Critical Analysis

**What's strong:** The taxonomy is the killer feature. Most knowledge management tools treat all information as equivalent — a note is a note. Reality Check forces you to classify claims by type and evidence level, which immediately reveals the epistemic quality of your knowledge base. A wiki full of [H] Hypothesis claims at E4 (Indirect/Weak) evidence looks very different from one full of [F] Fact claims at E1 (Strong Empirical). That visibility alone is valuable. The argument chain mapping — tracking which claims depend on which other claims — means you can trace the blast radius of a falsified assumption.

The three-stage source analysis methodology (descriptive → evaluative → dialectical) is borrowed from serious analytical tradecraft and is the right level of structure. It avoids the "AI summarization" trap where everything gets compressed into a bland paragraph.

**What's missing:** The public example KB (`realitycheck-data`) would be the real test of whether this scales. A taxonomy this detailed risks being more effort to maintain than the insights it produces — the overhead of classifying every claim by type, evidence level, domain code, and credence score could easily exceed the value for a solo analyst. The 458 tests suggest serious engineering, but the question is whether anyone besides the author uses it regularly enough to stress-test the taxonomy in practice.

No mention of how credence scores get updated when new evidence arrives. The static snapshot problem: if your [H] claim at credence 0.6 gets new supporting evidence, does the tool prompt you to re-evaluate, or does the old score silently rot?

**How it connects:** This is the structured, formalized version of what [[Third Gulf War]] does manually — Schuyler's hypothesis tracking with confidence levels and scenario bands is the same intellectual operation, but hand-built for a single domain. Reality Check generalizes the pattern into a reusable tool. It also relates to [[World Monitor]]'s intelligence aggregation, but where World Monitor focuses on *ingestion* (500+ feeds, 65+ sources), Reality Check focuses on *evaluation* (what do we actually believe, and at what confidence?).

The LLM Wiki pattern ([[LLM Wiki]]) that this wiki itself implements is a sibling approach: both maintain structured knowledge bases with AI assistance, but this wiki optimizes for synthesis and cross-referencing while Reality Check optimizes for epistemic rigor and claim tracking. You could imagine them as complementary layers — the wiki for "what does this mean?" and Reality Check for "how confident are we?"

The agent-native design (Claude Code plugins, Codex skills) connects to [[Agent Memory and Context]] — this is a form of structured agent memory where the structure isn't just organizational but epistemic.

---

*Sources: [[raw/realitycheck]]*
*Last updated: 2026-05-14*
