---
url: https://phronesis.world/papers/ways-of-checking/
title: "Ways of Checking"
author: Rincón, D., with Claude
date_fetched: 2026-07-25
date_published: 2026
---

A catalogue of ten ways automated checks fail silently, drawn from a single audit day on the author's own site. Every defect sat behind a check that had already passed.

The ten failure modes: checks structurally incapable of failing (a subshell variable that can never be set); checks using the wrong dialect (regex vs trailing slashes, JSX vs HTML); checks looking where tooling is easy rather than where consequences happen; measures that track something else (a "coherence" score mostly measuring text length); hand-written test samples that conceal unexpected confounds; experimental designs that erase the effect they seek; silence read as success (swallowed errors, catch blocks with no handler); documents that drift from the data they quote; edits that report success while matching nothing; and verification against the wrong copy of the artifact.

Each category comes with a concrete example and a diagnostic tell. The tells form a practical checklist: break the thing once to prove the check can fail; hand every checker a known positive before trusting negatives; check where the consequence lives; test invariance to transformations the measure should ignore; use evidence that predates the question; design for the distinction, not the effect; require the positive signal, not just the absence of error; verify the artifact, not the alias.

The core insight: re-running the same instrument finds nothing twice. Every genuinely new instrument found a defect class the others structurally could not see, on its first run. "A needle that cannot move measures nothing, whatever it points at."
