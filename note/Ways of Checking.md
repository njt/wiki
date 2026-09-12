# Ways of Checking

A field catalogue of ten verification failure modes, drawn from a single day auditing a live site where every serious defect sat behind a check that had already passed. The central thesis: "checking again re-runs the instrument; checking differently tests it." The piece is a pattern language for verification epistemology disguised as a bug postmortem.

---

> "A passing check is a claim, not a fact."

The opening salvo. Every check is an instrument with a detection surface; a green result means the instrument registered nothing within its sector. It does not mean nothing is wrong. Collapsing "the check passed" into "the thing is correct" is the root error behind all ten failure modes.

> "checking again re-runs the instrument; checking differently tests it."

The piece's thesis, stated once and then demonstrated ten times. Re-running the same audit found nothing twice; every *new* instrument found a defect class the others structurally could not see, on its first run. This is not a coincidence — it's the diagnostic principle: rotate the sector, don't re-scan the same one.

> "a needle that cannot move measures nothing, whatever it points at"

The framing from the theoretical coda. A check that has never failed is unproven. If you cannot deliberately break the thing and see red, the check is "not an instrument; it is a decoration." This is the meta-tell: every failure mode in the catalogue is ultimately a needle that can't move.

## The Ten Failure Modes

Each mode gets a name, a story, and "the tell" — the diagnostic sign that flags it.

### 1. The check that cannot fail
A failure flag set inside a subshell that terminates, so the summary reads a variable that can never be written to. Its cousin: a test suite recorded from current system behavior, passing forever because it asserts whatever the system currently does. **Tell:** a never-failed check is unproven — break the thing deliberately once. #pattern

### 2. The wrong dialect
`href="/courses"` finds nothing because the framework emits a trailing slash. `min="0"` doesn't match JSX's `min={0}`. Checker and checked speak different dialects; disagreement produces silence, and "silence read as health." **Tell:** hand every checker a known positive before trusting its negatives. #pattern

### 3. Looking where the light is
Audits passed on source links while the rendered sitemap was blank. The console had no errors because a parse error killed the script before any error handler existed. "Each check looked where checking was easy." **Tell:** check where the consequence lives — a source audit is about source; only a rendered check is about the page. #pattern

### 4. The measure that measures something else
A "coherence" score — algebraic connectivity of a sentence graph — was stable, reproducible, mathematically defined, and "mostly measuring length." Truncating identical prose produced a fourfold swing. **Tell:** reliability is not validity — find a transformation the measure should be indifferent to, apply it, and watch. #concept

### 5. The sample the checker wrote
Four hand-written sample texts confirmed the confound the author expected "and concealed the one he hadn't imagined." Seventy-four real documents written for other purposes reversed the finding in a single run. **Tell:** a test set authored by someone who knows the hypothesis is the weakest evidence — use material that existed before the question. #pattern

### 6. The design that erases its effect
Three experimental designs failed to find a real hysteresis loop: one erased it by definition (quasi-static measurement), one stacked the comparison, one divided noise by noise. The loop appeared on the fourth design (paired, rate-matched, absolute). **Tell:** design for the distinction between the effect and the mimicking artefact. A design asking only "is there an effect?" will find or erase one by construction. #pattern

### 7. Silence read as success
The contact form posted to a non-existent worker; every failure was swallowed by a catch block. A suggestion box, game scores — all `catch(()=>{})`. These systems "had been built to prefer the flattering reading." **Tell:** absence of error is not presence of success — probe end to end and require the positive signal (the 201, the stored record, the reply). #pattern

### 8. The document that drifted from its data
A page's claims about material temperatures came from a table in the same file. The table was edited (better numbers, sound reasons); every test stayed green because "the code was never wrong. Only the sentence was." The fix: write sentences as tests — one per claim, named after the sentence it defends — and also assert what the writing must *not* say. **Tell:** find every number that came from somewhere else and ask what would happen if that somewhere else changed. Assertions from the claim catch this; assertions from the behaviour never do. #pattern #concept

### 9. The change that never happened
A script replaced a string, printed "updated," and changed nothing — it assumed six spaces of indentation where the file used four. A build gate passed "because it was checking nothing" — a rule matching no input is indistinguishable from a rule finding no fault. **Tell:** the edit and the report come from the same process, which cannot distinguish successful replacement from vacuous. Assert the match count; read the file back rather than the log. #pattern

### 10. The copy that was not the one you changed
Edge-cached HTML with a week-long lifetime meant verification measured an old copy of the right page. Worse: a change looks live because a cached copy happens to match what was expected. "The artifact under inspection was real, current-looking, and not the one that had been edited." **Tell:** verify against the immutable thing the deploy just returned — the specific deployment URL, commit, build output — and treat the friendly alias as a question about propagation, not correctness. #pattern

## What Worked

The moves that found things when re-running audits found nothing:

- **A different question.** Every *new* instrument found a defect class the others structurally could not see, on its first run.
- **Assertions from the claim, not the behaviour.** (#8) A deterministic reasoning engine ran against its own documentation; two real bugs surfaced in the first minute.
- **Exercise, don't inspect.** Drive the page, submit the form, buy the product, call the endpoint after the change. "The report of the thing is not the thing."
- **Promote every failure to a standing check.** Each class ended the day as a build gate.
- **Let the tool refuse.** Prefer an editor that errors on a pattern it cannot find over a script that reports success.
- **A person looking.** "Keep the glance in the loop; it is the only instrument with no fixed dialect."

## Pocket Principles

1. A never-failed check is unproven — break the thing once
2. Hand every checker a known positive before trusting negatives
3. Check where the consequence lives, not where tooling is comfortable
4. Reliability is not validity — find the transformation that shouldn't matter
5. Test data that knows the hypothesis is barely data
6. Design for the distinction, not the effect
7. Silence is not success — require the positive signal

## Critical Analysis

This is the best single document I've read on verification epistemology. Its specific strength is that every failure mode comes with a *tell* — a falsifiable diagnostic you can carry into any audit. That transforms it from a war story into a checklist. The document-that-drifted (#8) is the richest failure mode; it generalizes beyond software to any document with derived claims, and the fix — sentences as tests, including negative assertions — is the piece's most transferable contribution.

The piece's framing is also its structural limitation. One day, one site, one checker — and the checker writing the catalogue is the one who made the errors, biasing it toward errors eventually caught. The author acknowledges this directly: "The ones that were not caught are, by construction, absent." That's not a flaw; it's honesty. But it means the catalogue should be read as a *lower bound* on failure modes, not a complete taxonomy.

The implicit argument running beneath all ten stories is that every deployment failure should be treated as a *design failure of the verification system*, not a one-off human error. This is the SRE philosophy applied to checking itself — blame the process, fix the guard, make the failure class impossible to repeat. It's the strongest unstated claim in the piece, and it's correct.

The pocket principles at the end risk becoming dogma if memorized without the stories. The value is not in the numbered list but in the detailed failure narratives, which teach the reader to *smell* a bad check. You can't learn that from the summary — you need to live inside the confusion of each story long enough to feel the moment where a green light lied.

A notable absence: the piece doesn't address the *cost* of checking differently. Rotating the sector on every deploy, writing assertions for every sentence-level claim, requiring positive signals end-to-end — these are expensive. The implied answer is that the cost of the bug you'll eventually ship is higher, but that's asserted rather than argued. A companion piece on the economics of verification depth would be welcome.

**Applied to test suites:** [[Test Validation and the Trustworthiness of Tests]] extends the piece's epistemology into the testing domain directly. Typemock's argument that "passing isn't the same as providing confidence" is the same move as "a passing check is a claim, not a fact." Their runtime analysis — inspecting what tests actually *do* at execution time rather than what their source code claims — is "checking differently" applied to the test suite itself. A test that passes while quietly accessing the network, depending on system time, or duplicating another test's logic is the testing equivalent of silence read as success (#7): the instrument registered nothing, but nothing is not the same as confidence.

**Applied to human review:** [[Reviewing Code Is a Skill]] shows the same "check differently" move operating in a person rather than a pipeline. Its three caught bugs — a file-lock race, a CLI-version incompatibility, an S3 checksum ordering — were each surfaced by "thinking in invariants and little proofs" (a different instrument than the trained eye re-scanning the diff), which is why high-end LLM reviewers that merely re-scanned the same surface missed all three.

---
*Sources: [[raw/ways-of-checking]]*
*Last updated: 2026-07-25*
