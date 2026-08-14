# Meat (Reading Diff)

Meat is a zero-dependency Go CLI that turns a git diff into a "reading diff" — the same change compressed to what a senior reviewer actually needs to understand: what moved, where data came from, what new behavior appeared. Its premise is that in the agent era, human review shouldn't re-check style, imports, or nil-checks (the compiler and tests already did); it should review concepts, algorithm choices, and architecture. The part that makes it novel is *how* it keeps that trust: the LLM proposes an edit plan in source coordinates, and a deterministic compiler validates and renders the result — so the model can never author a single line of output.

---

## Architecture

Meat is a three-layer pipeline with a clean seam between the probabilistic part and the deterministic part.

**CLI → agent loop → compiler.** `cmd/meat/main.go` reads a unified diff from `git show`, `git diff`, staged/worktree flags, or stdin, then calls the `meat` package. `meat.Abridge` (`meat/meat.go`) runs a bounded agent loop: it numbers the diff, sends a system rubric plus the numbered input, and lets the model call tools across up to 24 turns under a 4-minute wall-clock budget. The model's only real output channel is `submit` — a complete `remove`/`replace`/`fold` plan plus a one-line summary.

**The compiler is the author.** `meat/editplan.go`'s `compileEditPlanMoves` validates every plan against the immutable input and mechanically renders the reading diff by walking the original lines with a hidden/folded state (`planState`). `meat/model.go` defines the seam: a minimal `Model` interface (`Generate(ctx, system, messages, tools)`) with flat `Block`/`Message` structs, so the whole pipeline is provider-agnostic and embeddable (the README names the Shelley harness as an embedder).

**Provider backends.** `meat/openai.go` and `meat/anthropic.go` hand-roll both APIs on the standard library — including SSE stream parsing for the Responses API, exponential-backoff retry on 429/5xx, and replay of OpenAI's encrypted reasoning items as opaque `provider_state` blocks (`meat/model.go:44`). `meat/gateway.go` discovers the exe.dev managed-LLM gateway so no API key is needed on those VMs.

## Key techniques

**Edit-plan-as-contract, not text generation.** The model submits `{remove: [ranges], fold: [ranges], replace: [{line, old, new}]}` in 1-based original coordinates. `replace` must be an *elision projection*: `new` matches `old` with every omitted span represented by `...` or `…`, compiled to a regex where each ellipsis becomes `.+` (`meat/editplan.go:566`). This is what guarantees the model cannot invent identifiers or comments — and it explicitly blocks silent changes like `!allowed → allowed`.

**Compiler-owned import removal.** `meat/imports.go` runs a mandatory pass on the immutable diff that removes import/includes/requires/use declarations across six languages using conservative regexes (`goImportBlockStartRE`, `pythonFromStartRE`, `rustUseStartRE`, …), including imports *inside embedded source strings* (test fixtures) via a backtick/triple-quote scanner. Model edits and compiler edits merge with defined precedence; a fold crossing an import row and a behavioral row is rejected.

**Exact move detection with enforced symmetry.** `meat/moves.go` finds relocated blocks by anchoring on a globally-unique substantive line, then extending with indentation-normalized byte-exact rows at one constant indent offset. Ambiguity (a line appearing 3+ times, or overlapping candidates) is discarded rather than guessed. The compiler then *enforces* that both sides of a detected move get identical keep/remove/fold/replace treatment — so a relocation can never read as a one-sided deletion.

**Python semantic-skeleton validators.** `meat/python.go` plus the suite-placeholder logic in `meat/imports.go` enforce that a folded/deleted Python diff still keeps recognizable boundaries: triple-quote parity within a hunk, delimiter balance, backslash continuations, decorators atomic with their definitions, suite owners never severed from a referenced body. A compiler-owned `...` placeholder preserves a suite skeleton when import removal would otherwise orphan a visible `def`.

**Chunking large diffs.** `meat/chunk.go` splits anything over ~400 KB (the single-run budget, bounded because the numbered diff is re-sent every turn) up to 4 MB / 32 chunks, at file→hunk→sub-hunk boundaries. Each chunk is a well-formed diff (metadata replicated, `@@` headers synthesized with exact counts), and whole-diff analyses a fragment can't reproduce — the mandatory import mask and move set — are pre-computed and mirrored into each chunk. Documented cost: a move or Python suite split across chunks can't be enforced.

**Frozen prompt surface hashing.** `meat/rubric.go` hashes the *rendered* prompt surface — system prompt, every user-prompt branch, tool schemas, feedback branches, the nudge — into `RubricHash()`. That hash is part of the cache key (`cmd/meat/cache.go:31`), so re-wording guidance or tightening a compiler invariant silently invalidates every cached result. Validation-error text is deliberately *outside* the surface so wording fixes don't churn caches.

**Retention-pressure refinement.** After submit, if `retentionPressure` sees too many changed rows still visible, the loop nudges the model *once* to re-plan, keeping the first candidate as a fallback (`meat/meat.go:203`). And `meat/elision.go`'s `ElisionLine` computes "kept 12/240 changed lines in 3/7 files" locally by aligning the result back to original lines, never trusting model-reported counts.

## Design decisions

**Trust through immutability.** The model never sees or writes the result diff; the output is always a subsequence (with fixed `...` placeholders) of the original. Every editable thing is source-anchored, and every thing the model *can't* be trusted with (imports, move symmetry, Python structure) is owned by the compiler. This is "linters beat prompts" applied to diff summarization — the model's job is judgment, the compiler's job is enforcement.

**Reject rather than guess.** Ambiguous moves are dropped, unsupported combined diffs error out, uncertain lines are kept, a truncated `max_tokens` response fails loudly instead of being cached (`meat/anthropic.go:152`). Correctness-of-retention is optimized over coverage — a reading diff that hides something real is worse than one that keeps too much.

**Chunking trades locality for feasibility.** Whole-diff reasoning ("this line looks like noise but is explained by another file") degrades per-chunk; cross-chunk moves and Python suites lose atomicity. The splitter prefers file boundaries specifically to make those cases rare, and the costs are documented in the code, not papered over.

**Zero dependencies, stdlib-only.** Hand-rolled HTTP clients, SSE parsing, and regex classification. That's a portability and trust bet (auditable, no supply chain), paid for in code volume — the import classifier alone is ~1000 lines of per-language regex logic.

## Comparison notes

- **vs [[Hunk]]** — Hunk *renders* a diff for reading, keeping the whole changeset visible; meat *transforms* the diff, deleting the noise before any renderer sees it. They're adjacent stages: meat could feed an abridged diff to Hunk's continuous stream.
- **vs [[sem]]** — sem does entity-level diffing via tree-sitter across 31 languages, structurally hashing code entities; meat is line-oriented and delegates the "what matters" judgment to an LLM while constraining it to source anchors. sem is deterministic understanding, meat is constrained-model judgment.
- **vs [[Git Diff Drivers]]** — the driver *produces* the diff (plumbing); meat *consumes and reduces* it. Both sit in the same terminal-native review stack.
- **vs [[AI-Written Change Descriptions]]** — Varda's moratorium targets LLM-written *descriptions* that omit framing; meat attacks the opposite side, compressing the *diff itself* so the change is readable, and keeps its summary to one line. The two together imply the real fix is making the artifact legible, not asking a model to explain it.
- **vs [[Guardrails and Feedback Loops]]** — meat is the cleanest diff-domain instance of the "structural gate beats prompt" thesis: the rubric is a prompt, but the edit-plan compiler is a deterministic gate the model bounces off via `preview_plan` until the artifact validates.

---

*Sources: [[raw/meat]], [[summary/meat]]*
*Tags: #tool #project #code-review #agents #diff #llm #go*
*Last updated: 2026-08-14*
