# CI Speedup — Linear's CI Post Remixed for Agents

StarSling takes Linear's popular CI-optimization write-up (15% faster suite, 50% less runner time per test, while tests grew 4x) and converts each of its 15 optimizations into a copyable agent prompt, backed by a `ci-speedup` skill with 70+ catalogued optimizations. The post is itself a demonstration of a new genre: engineering write-ups rewritten for machine readers.

---

## What happened at Linear

Four buckets of wins, all measured:

- **Infrastructure and tooling**: third-party runners 34% faster like-for-like (tsc dropped 52%); the native `tsgo` compiler cut weekly median tsc time 73%; custom lint rules rewritten as AST-only static analysis dropped type information from ESLint entirely — 68% faster API lint, 55% faster full-repo lint — and made the later Oxlint port easy.
- **Critical-path gating**: small change-detection jobs gate everything, so capping fetch depth took the slowest gate from 94s to 20s, and removing checkout from jobs that never needed a working tree took 27s to 7s. A custom checkout composite action with retry, backoff, `GIT_HTTP_LOW_SPEED_LIMIT`, and a persistent git mirror on a sticky disk fixed hangs caused by third-party runners' direct IP link to GitHub.
- **Repeated setup**: Postgres client baked into a CI base image; filtered `pnpm install` for just the API package (44–73s → 16–18s); deleting the node_modules cache because restoring it took ~28s versus ~7.5s to rebuild; consolidating seven tiny jobs into two, saving ~87,000 runner-minutes/month (11.8% of total CI usage).
- **Test execution**: moving from four to eight Vitest shards made the critical job 19% faster and cheaper — but only because setup had already shrunk to ~40s; at 110–140s per shard, eight shards would have spent more on setup than tests. The single biggest win (~17% of monthly savings) was an opt-in `isolate: false` Vitest project sharing module state across safe files.

## The genre is the point

This is not primarily a CI article. It is an article-shaped prompt bundle: "Just send this post to your agent and they'll work through all 15 of Linear's optimizations." The remixer explicitly discloses a conflict of interest (StarSling sells the runners its skill will suggest benchmarking), which is at least honest. The remix format — numbered, patterned, each optimization a self-contained task with a prompt — is a real signal about how practitioners now consume engineering knowledge: as training material for their agents, not themselves.

## Key quotes

> "Because agents now write the majority of our tests, we updated our respective agent skills to account for this performance opt-in as well, so generated tests follow the same constraints by default."

The sharpest line in the piece, and it's buried at the end of the riskiest optimization. The correctness risk of `isolate: false` isn't managed by review — it's managed by pushing the constraint upstream into the skills that generate the tests. That is governance migrating from review into generation, the same move the [[Code Standards That Survive the Next Agent]] thesis predicts.

> "Further sharding only pays off when the fixed cost per shard is low... At 110–140 seconds per shard, eight shards would have spent 15–19 minutes of runner time on setup alone, more than the tests themselves."

A genuinely transferable systems lesson: parallelism is bottlenecked by per-unit fixed cost, so the setup optimizations are what made the parallelism optimization *possible*. The optimizations compound; you can't take the last one first.

> "We were writing cache markers as part of the final check before merging, which meant a pull request could sit in the merge queue even after its tests had passed."

The subtlest class of bug in CI systems: work that gates nothing still sits on the critical path. Forty-two seconds saved by moving a write to a non-gating job.

## Themes

#pattern #tool #concept

## Analysis

The honest way to read this: it's vendor content wearing a costume, but the costume fits. The underlying Linear numbers are real and detailed (94s→20s gates, 87k runner-minutes, per-shard timings), and packaging them as agent prompts is a legitimate — arguably superior — remix, because the optimizations are exactly the kind of bounded, verifiable, locally-checkable tasks agents are good at: change a fetch depth, move a cache write, split a test file. The catch is that the correctness-risky one (`isolate: false`) is precisely the kind an agent will apply confidently and a human must audit. Linear's own answer — encode the constraint in the skills that generate tests — is the more durable pattern than any prompt in the bundle.

The "never finished" framing is also worth taking seriously: with 2,000 tests added weekly, the contribution isn't the 15 fixes but the recurring diagnostic loop of matching current CI data against a catalog of known patterns.

## Related pages

- **[[Enterprise-Grade CI at Solo Founder Scale]]** — complicates this: where Linear optimizes CI at monorepo scale with a fleet, the solo-founder version asks whether enterprise CI is worth running at all; both treat the CI pipeline as the load-bearing artifact.
- **[[CI Forge (ciforge)]]** — the same instinct (bundle CI knowledge into one agent-usable tool) applied to quality scanning rather than speed; ciforge's "false negatives over blocking" philosophy is a mirror-image tradeoff to `isolate: false`'s speed-over-isolation bet.
- **[[Loop Engineering Optimizing Code Generation and Verification]]** — strengthens it: Linear's recurring skill-vs-bottleneck loop is loop engineering applied to build infrastructure, and the "run it again whenever CI slows down" cadence is exactly Gibbs' learning loop with CI runs as the signal.
- **[[Agent Skills Library (dzhng)]]** — the packaging pattern: skills as the distribution medium for distilled engineering practice, this time sourced from another team's blog post rather than one's own experience.

---
*Sources: [[raw/linear-ci]], [[summary/linear-ci]]*
*Last updated: 2026-10-14*
