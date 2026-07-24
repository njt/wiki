---
url: https://phronesis.world/papers/ways-of-checking/
title: Ways of Checking
author: Rincón, D., with Claude
date_fetched: 2026-07-25
date_published: 2026
site: phronesis
---

# Ways of Checking · Phronesis

**Author:** Rincón, D., with Claude
**Published:** 2026, on *phronesis*
**Subtitle:** "checking again is not checking"

## Opening Thesis

The piece opens with the assertion that "A passing check is a claim, not a fact." The author recounts a single working day where a paid book was freely accessible, a contact form lost all enquiries, the sitemap rendered blank, and a "coherence" metric mostly tracked text length — all after having "already passed a check." The central thesis: "checking again re-runs the instrument; checking differently tests it."

## The Ledger

The examples come from one audit day (July 18–19, 2026) on this site. The checker whose failures are documented was Claude, acting as the site's engineer. Every serious defect "sat behind a check that had already passed."

### 1. The check that cannot fail

A script printed twelve `BAD` lines yet concluded `PASS` — the failure flag was set inside a subshell that terminated, and the summary read a variable that could never be written to. The check was "incapable of telling the truth." Its cousin is a test suite recorded from current system behavior, which passes on the day it's written and forever after because it asserts whatever the system currently does.

**The tell:** A check that has never failed is unproven. If you cannot deliberately break the thing and see red, the check is "not an instrument; it is a decoration."

### 2. The wrong dialect

A search for `href="/courses"` found nothing because the framework emits a trailing slash. A dollar sign treated literally was read as a regex anchor. A price never appeared as a contiguous string because the renderer injects comment nodes between text runs. An audit expecting `min="0"` encountered JSX writing `min={0}`. In each case, checker and checked spoke different dialects; disagreement produced silence, and "silence read as health."

**The tell:** Before trusting negatives, hand the checker a known positive. If it can't find something you know exists, "its 'nothing found' means nothing."

### 3. Looking where the light is

The sitemap rendered blank most of a day while audits passed. Links were audited in source (all present). The console had no errors because a parse error kills the script before any error handler exists. The HTML fallback held every link, hidden by CSS on desktop. "Each check looked where checking was easy." The failure lived on the rendered page; the only instrument pointed there was a person saying "nothing really visible."

**The tell:** Check where the consequence happens. A source audit is about source; only a rendered check is about the page.

### 4. The measure that measures something else

A "coherence" score — the algebraic connectivity of a sentence graph — was stable, reproducible, mathematically defined, and "mostly measuring length." Truncating identical prose to eight sentences scored 0.39; the whole document scored 0.11 — a fourfold swing on identical prose. An invariance test exposed it: applying a manipulation the measure should ignore (truncation) and watching whether it matters.

**The tell:** Reliability is not validity. Find a transformation the measure should be indifferent to and apply it.

### 5. The sample the checker wrote

Four hand-written sample texts — a tight argument, a rumination, a disconnected list, a technical passage — confirmed the confound the author expected "and concealed the one he hadn't imagined." They varied the suspected thing while holding steady the unsuspected one. Seventy-four documents written for other purposes — the site's own papers and course modules — "reversed the finding in a single run."

**The tell:** A test set authored by someone who knows the hypothesis is the weakest evidence. Use material that existed before the question.

### 6. The design that erases its effect

Three experimental designs failed to find a real hysteresis loop. The first held each measurement so long the memory relaxed — quasi-static, and hysteresis vanishes in the quasi-static limit by definition. The second compared the effect at its best against the control at its worst. The third took the ratio of two numbers that were both noise. The loop appeared on the fourth design (paired, rate-matched, absolute).

**The tell:** Ask what distinguishes the effect from the mimicking artefact and design for that distinction. A design asking only "is there an effect?" will find or erase one by construction.

### 7. Silence read as success

The contact form posted to a non-existent worker. Every submission failed; every failure was swallowed by a catch block; the form looked fine and told no one. A suggestion box had the same disease. Game scores went to undeployed endpoints, each request ending in `catch(()=>{})`. Nothing looked broken because "looking broken had been explicitly handled away." Silence is ambiguous between no failure and no signal, and these systems "had been built to prefer the flattering reading."

**The tell:** Absence of error is not presence of success. Probe end to end and require the positive signal — the 201, the stored record, the reply.

### 8. The document that drifted from its data

A page claimed a steel bench and a pine bench differ by about 9°C at the skin. The figure came from a table of conductivities and densities in the same file. Then the table was edited (better numbers, sound reasons), and every test stayed green because "the code was never wrong. Only the sentence was." A document quoting a figure it doesn't measure is an untested assertion that fails silently.

The fix: write sentences as tests — one per claim, named after the sentence it defends. Within seconds the suite failed. The failing assertion had been overclaimed: it said every plant material beat every petroleum one, and polyurethane foam had the best number. The page was weakened from "biomaterials feel warmer" to "one of them matches the best synthetic there is" — "a truer sentence" that exists "because an assertion refused the flattering one."

Two more issues emerged. A rule change made minerals admissible, and "clean surfaces span less than concrete" began comparing concrete against itself — "the words had held still while the category beneath them changed meaning." A superlative ("foam has the lowest effusivity") stopped being true when a new material was added; it carried a note with rewrite instructions.

The sharpest version: assert what the writing must *not* say. When two figures were too close to call a win, the suite gained a check that fails if prose upgrades to a victory while the gap stays small — and would announce if evidence ever justified the stronger claim. A document that cannot drift toward exaggeration is "rarer than a document that is currently accurate."

**The tell:** Find every number that came from somewhere else and ask what would happen if that somewhere else changed. If nothing would happen, the sentence is unguarded. Assertions from the claim catch this; assertions from the behaviour never do.

### 9. The change that never happened

A script replaced a string, printed "updated," and changed nothing — it assumed six spaces of indentation where the file used four, matched zero times, "and reported success anyway." This happened four times in one day. One silent miss left a build gate in place that passed "because it was checking nothing" — a rule matching no input is indistinguishable from a rule finding no fault. Another survived a full deploy.

This sits upstream of the others: they are ways a check goes green while broken; this is a way "the thing was never touched while the tooling said it was." Everything downstream behaves correctly and misleads.

The tell: the edit and the report come from the same process, which cannot distinguish successful replacement from vacuous. A tool that errors on a missed pattern removes the category. Failing that: assert the match count and read the file back rather than the log.

### 10. The copy that was not the one you changed

A mobile fix, a leaderboard, a rewritten index page, and a corrected count each appeared not to have deployed. Each *had* deployed. The apex domain served edge-cached HTML with a week-long lifetime, so verification measured an old copy of the right page — and query-string cache-busting sometimes returned the new copy and sometimes the old, in the same minute.

The reverse is worse and quieter. A change looks live because a cached copy happens to match what was expected. Examples include a Worker whose deployed code no longer matched the repository source, with only one of three routes answering; and a directory of built output standing in for source that had moved on. "The artifact under inspection was real, current-looking, and not the one that had been edited."

The move: verify against the immutable thing the deploy just returned — the specific deployment URL, commit, build output — and treat the friendly alias as "a question about propagation rather than about correctness."

## What Found Things

Against the ten, the moves that worked:

- **A different question.** Re-running any audit found nothing twice. Every *new* instrument found a defect class the others structurally could not see, on its first run.
- **Assertions from the claim, not the behaviour.** A deterministic reasoning engine was given a suite from its documentation. Two real bugs surfaced in the first minute — one being dead fallback code that had reported every unreadable input as the framework's best state.
- **Evidence that predates the question** (from §5).
- **Exercise, don't inspect.** Drive the page, submit the form, buy the product, call the endpoint after the change. "The report of the thing is not the thing."
- **Promote every failure to a standing check.** Each class ended the day as a build gate — inline scripts parse-checked because one apostrophe blanked a page; the build fails if a route loses its handler because one did silently for weeks.
- **Verify the artifact, not the alias.** Check the immutable thing a deploy just handed you. A friendly domain answers a question about propagation, not about correctness.
- **Let the tool refuse.** Prefer an editor that errors on a pattern it cannot find over a script that reports success. Where a bulk change is unavoidable, assert how many times it matched and fail on zero.
- **A person looking.** The blank map was found by a human glance after five green audits. "Keep the glance in the loop; it is the only instrument with no fixed dialect."

## In the Framework's Terms

A check is an instrument; a passing check is a claim of coherence worth exactly the displacement the check could have registered — "a needle that cannot move measures nothing, whatever it points at." Checking again interrogates the same sector. Checking differently rotates the sector, and the failures "were always in the sector nobody had rotated to."

## Limits

One day, one site, one checker — and the checker writing the catalogue is the one that made the errors, biasing it toward errors eventually caught. "The ones that were not caught are, by construction, absent." That is "the strongest reason to expect this list to be incomplete."

## Pocket Version

Seven distilled principles are listed: (1) a never-failed check is unproven — break the thing once; (2) hand every checker a known positive before trusting negatives; (3) check where the consequence lives, not where tooling is comfortable; (4) reliability is not validity — find the transformation that shouldn't matter; (5) test data that knows the hypothesis is barely data; (6) design for the distinction, not the effect; (7) silence is not success — require the positive signal.

The piece closes noting these get worked out in the open at whatever length the problem takes, with an offer to do the same on a reader's problem: "one thing diagnosed and written up plainly, no build."
