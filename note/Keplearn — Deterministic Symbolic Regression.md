# Keplearn — Deterministic Symbolic Regression

Keplearn ([owls-on-wires/keplearn](https://github.com/owls-on-wires/keplearn)) is a ~7,000-line Rust engine that recovers closed-form equations from tabular data — Kepler's inverse-square law from columns of masses and radii, in milliseconds. Its pitch is the anti-LLM of ML tools: no stochastic search, no GPU, no dependencies, one static binary, byte-for-byte deterministic output. It matters as a demonstration that for some "learn a model from data" problems, a clever classical algorithm beats the fashionable approach outright.

---

## Architecture

Single Rust crate, library plus thin CLI (`src/bin/keplearn.rs`). The pipeline reads a TSV/CSV from stdin (`driver.rs:ingest`, delimiter inferred from the header), shuffles rows with a fixed-seed LCG so the 25% holdout split is reproducible, and hands the sampled data to the search driver.

- **`search.rs`** — the control loop. Expands frontier nodes into child reductions and pushes them back; also owns a trace facility (JSONL events to a file via a `OnceLock<Mutex<BufWriter>>`) and atomic counters for search/harvest evaluation counts.
- **`frontier.rs`** — the priority queue. Not one heap but several lanes keyed by `TunnelKey`/`SweepKey`/`StructKey`, each ordering by a tuple like (plane, priority, depth, complexity, seq). Five "planes" (defined in `config.rs`: normal, near-exact promotion, Mat pins, root) let verified-promising children jump the queue.
- **`model.rs`** — the core types: `Red` (a reduction step: `Div`, `Shell`, `Affc`, `Mat`), `Node` (target column + chain of reductions + virtual columns + priority), `Hit` (a terminal fit), and `invert_chain`, which replays the reduction chain backwards to build the final expression string.
- **`harvest.rs` / `fit.rs`** — the terminal fits. `harvest_la` runs log-linear least squares on logged columns to get monomial exponents; `harvest_monomial_vcols` runs a matching-pursuit loop that extracts several monomial "virtual columns" from the residual and adds them as derived features for deeper nodes.
- **`generators.rs`** — composite finalist generators: L2 norms over disjoint column partitions (`disjoint_pair_partitions`), rationals, trig with argument search, additive splits.
- **`expr.rs`** — a recursive-descent parser/evaluator over expression strings, used to validate and count nodes in every printed model.
- **`config.rs`** — a single ~400-line table of named tuning constants, including the snap lattices. No magic numbers inline; everything reads this table.

Output: exactly one JSON object on stdout (diagnostics go to stderr), with `results[0].model` the best law, scored `r2` on the supplied data, and `stopped_by: "exact_exit"` when the search proved its answer.

## Key techniques

- **Reduction instead of enumeration.** Rather than searching expression space, Keplearn searches over *chains of invertible operations applied to the target column*: divide out a fitted factor, invert a shell (log, sqrt, arcsin), peel an affine node — then recurse to a fixed depth. Each child is a cheap column transformation, so the search state is just a `Vec<f64>` plus the chain needed to un-invert it later.
- **Cheap probes gate expensive fits.** Every child gets a `quick_probe` R² on a small subsample (`fit.rs`) before it's worth ranking; only nodes that look promising get the full harvest. This is how exact laws resolve in single-digit milliseconds.
- **Constant snapping with verification.** Fitted literals are snapped to nearby rationals and to multiples of π, e, √2 (`math.rs:snap_value`, driven by a declared table in `config.rs`), and monomial exponents are snapped to a rational lattice instead of a hardcoded half-integer grid. Crucially, a snap is *accepted only if holdout and full-data scores don't degrade* — the prettiness heuristic can't silently break a fit.
- **Honest scoring.** The final `r2` is computed by evaluating the printed model string, not carried from the internal fit stage. What you see in the JSON is what the expression actually does.
- **Determinism everywhere.** Fixed holdout-shuffle seed, no threading in the search core, MDL tie-breaking, and a `seq` counter as the last ordering key so the priority heap is totally ordered. Same input → same bytes out.

## Design decisions

The explicit trade-off is exactness against generality: on SRBench's 138 black-box datasets Keplearn's median test R² is near zero, because it targets closed-form laws, not curve fitting. Against that, it hits 64.7% symbolic recovery on noise-free Feynman — beating AIFeynman's 55.8% at a fraction of the compute — with median 1.3 s fits and an exact-exit path that ends half the noise-free suite in under a second. Noise is its enemy: once the data is noisy the exact-exit path can't fire, and symbolic recovery drops to 8.6% at noise 0.1 while fits burn the full 60 s cap. The zero-dependency, statically-linked single binary is a deployment decision, not an accident: it's built to be piped into shell scripts and agent harnesses.

## Comparison notes

This is the sharpest possible counterpoint to [[Why LLMs Fail at Tabular Prediction]]: where that piece argues LLMs are the wrong tool for structured numeric data, Keplearn shows what the *right* tool looks like — 7k lines of deterministic linear algebra that beats frontier-model-adjacent symbolic regression systems on their own benchmark. It is also a concrete instance of [[Deterministic When Possible, Probabilistic When Necessary, Human When Cheaper]]: the whole design spends engineering effort up front so the runtime needs no randomness at all, and unlike [[Sixteen Agents and a Byte-Exact Oracle]], determinism falls out of the algorithm rather than being bolted on with test scaffolding. And compared with [[TabFM (Tabular Foundation Model)]], it's the opposite bet: no learned weights at all, just search over algebraic structure — winning big on exact-law problems and losing completely on black-box ones.

#tool #project #algorithms #data #machine-learning

---
*Sources: [[raw/keplearn]], [[summary/keplearn]]*
*Last updated: 2026-11-02*
