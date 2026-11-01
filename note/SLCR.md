# SLCR — Requirements as a Graph, Validators as Proof

SLCR is an open standard plus a ~26k-line Rust CLI (`slcr`) for storing a project's requirements as a graph instead of a document, giving coding agents **slices** (the minimal subgraph needed to implement one requirement) and giving CI **validators** (pinned tool invocations that prove an acceptance criterion). It attacks the two failure modes of spec-driven agentic work head-on: context rot from handing agents whole documents, and compounding drift that nobody can detect cheaply.

---

## Architecture

The repo is a single Rust binary (`src/bin/main.rs`) over a library split into clear layers:

- **`src/spec/`** — the document model. `node.rs` defines sections, requirements (with RFC 2119 modalities and AND/OR/XOR refinement combinators), acceptance criteria, and terms. A hand-rolled `tagged!` macro implements serde tagging with `#[serde(remote = "Self")]` so the `kind` discriminator is *checked* on deserialization and error locations survive — a tagged enum would have buffered the whole map. `vet.rs` (~1.4k lines) is the static well-formedness checker that runs before anything executes.
- **`src/query/`** — materializes the vetted document into an in-memory property graph (`SpecGraph`, backed by petgraph) and runs read-only GQL over it via `slcr-gql`, a forked/pinned graph-query crate. Node labels: `Section`, `Requirement`, `AcceptanceCriterion`, `Term`; edge labels: `CONTAINS`, `REFINES`, `DEPENDS_ON`, `USES_TERM`. `slcr query "MATCH (r:Requirement) RETURN r.id, r.modality"` is a real query surface for both humans and agents.
- **`src/slice.rs`** — the centerpiece. `members()` computes `slice(R)` by iterative one-edge hops over the graph, each hop itself a GQL query against a `$frontier` parameter. It classifies every included node by `Detail` level: `Full` (R and its transitive refinement descendants), `Dependency` (depended-on requirements' bodies plus acceptance criteria), or `Body` (containment ancestors, what R refines). Terms used by anything included come along. Siblings of R and descendants of dependencies are deliberately excluded.
- **`src/validators/` + `src/tools/`** — validator files (`.validators.slcr.toml`) live beside the code and bind a criterion to a tool call. Tools are declared in small TOML descriptors (an argv template, a JSON Schema for `with` params, pass exit codes) and pinned by blake3 hash in the project manifest `SLCR.toml`. There are no built-in tools at all.
- **`src/run/` + `src/cli/check/schedule.rs`** — template substitution, PATH resolution, and a bounded-concurrency scheduler (`-j`, Ctrl-C aware) that runs validators as child processes.
- **`src/satisfaction.rs`** (~1.1k lines) — pure verdict algebra: how validator outcomes combine into per-requirement `Status`.

## Key techniques

- **Hop-by-hop graph traversal instead of path patterns.** `slice.rs` documents why: a quantified pattern like `-[:REFINES]->{1,}` matches once per path, and a DAG can have exponentially many paths between two nodes. Each closure is walked one edge at a time until a step finds nothing new, so each node is visited once. There is literally a test with 40 layers of binary dependency fan-out (2^40 paths) asserting it stays linear.
- **Strong Kleene logic over requirement refinements.** A leaf requirement is the Kleene AND of its validators' verdicts; a leaf with *no* validators is `Unknown`, never vacuously satisfied — a coverage gap, not a pass. Non-leaf requirements combine their refiners per their combinator (AND drops MAY refiners; OR/XOR count them), then AND in their own validators. `Unknown` propagates: a requirement is Unknown when some resolution of the undecided parts satisfies it and another does not. This is a small three-valued logic used exactly where it belongs.
- **Absorption rule.** An unsatisfied requirement that refines at least one requirement and where every refined parent is nevertheless satisfied is demoted from violation to warning — the spec machinery models "this requirement's job was covered by a sibling."
- **Pin-everything supply chain.** Tool descriptors are fetched over HTTPS only, pinned by blake3 in the manifest, verified on install ("refuses a source that serves other bytes"), and the GQL engine itself is a pinned reviewed commit of a fork, not a crates.io release. Validators run with their own process group on Unix (via `rustix`, because the crate forbids `unsafe_code` and std can only kill the direct child) so a timed-out validator's grandchildren die too; `kill_on_drop` covers cancellation.
- **Templates that never re-tokenize.** `run/template.rs` substitutes placeholders into argument segments by concatenation only — a `with` value with spaces or braces stays one argument. Nothing is ever split or joined, which closes the classic shell-injection hole in "run tool with user-ish parameters."
- **Agents edit the source of truth; humans read renders.** `SPEC.md` is generated; `slcr render --expect-no-diff` fails CI when the Markdown goes stale, so the two views cannot silently diverge.
- **Dogfooding as proof.** `SLCR.toml` + `SPEC.slcr.toml` + snapshot-test validators in `tests/cli/` mean `slcr check` in this repo checks `slcr` — the standard's feasibility argument is its own test suite.
- **A Claude skill ships in-repo** (`skills/slcr/SKILL.md`) that teaches agents why slices are intentionally small ("reading the whole spec invites scope creep"), how to learn the CLI from `--help` rather than trusting the doc's version, and the loop: read slice → write failing validator → implement → `slcr check`.

## Design decisions

- **Spec-first as database, not document.** The graph is the artifact; YAML/JSON/TOML are just serializations, and Markdown is a view. This is a bet that requirements get *queried* as often as read.
- **Unknown is a first-class state.** Most checkers would treat "no validator yet" as failure or silence; SLCR makes it a countable coverage gap so adoption can be incremental — you can check a spec where 1 of 2 requirements is proven.
- **Strictness is modality-scaled**: unsatisfied MUST fails the run, SHOULD is a warning, MAY is informational; `--deny-warnings` ratchets up. And the doctrine is explicit: a failing validator means the code *or the validator* is wrong, not that the requirement should be loosened.
- **What it sacrifices**: everything about authoring flow. SLCR has no opinions about how you get to a spec (no brainstorming, planning, or task decomposition — compare Spec Kit's constitution→specify→plan pipeline), and nothing helps you *write* validators for fuzzy requirements; if you can't phrase an acceptance criterion a tool can prove, it stays Unknown forever. The cost of the slice discipline is also the cost of spec hygiene: refinement and dependency edges must be maintained by hand or by agents who understand the graph.

## Comparison notes

- Unlike [[Spec Kit]], which is a *process* harness (markdown templates, constitution, human gates per feature), SLCR is a *data* standard: the spec is a queryable graph and the deliverable is machine-checkable proof, not a filled-in template. They are complementary rather than rivals.
- It sharpens [[AI Agents Need Clear Specs]] from advice into an executable contract: "clear spec" becomes "requirement whose acceptance criteria have validators," and drift becomes an exit code rather than a review opinion.
- It answers [[The Coming Need for Formal Specification]] at a deliberately modest formality level — no model checking, just RFC 2119 modalities plus tools that exit 0 — trading the power of [[Your .md File Is Not a Specification]]'s Dynamic Logic approach for adoptability by ordinary test suites.
- [[LLM-as-a-Verifier]] argues LLM judges are the pragmatic verifier; SLCR insists verification be deterministic and pinned instead, demoting the model to writing the validator, not being one.
- Against [[Agentic Verification]]'s broad survey of agentic checking, SLCR contributes a concrete missing primitive: context *sourcing* from a spec graph, so the verification loop starts from the right slice rather than whatever the agent gathered.

---
*Sources: [[raw/slcr]], [[summary/slcr]]*
*Last updated: 2026-11-02*
