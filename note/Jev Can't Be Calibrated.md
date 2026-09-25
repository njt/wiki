# Jev Can't Be Calibrated

Alex Molas argues that Jev, TypeSafe's System One decision model, is useful
without training data but cannot honestly claim "calibrated probabilities":
calibration is a property of a model *and* a data distribution, and a single
model cannot be calibrated for every customer at once. His prescription is to
treat the outputs as ranking scores and recalibrate locally with a cheap Platt
scaling if you need real probabilities.

---

## The argument

Molas's core move is definitional, not empirical. Calibration means
$P(Y=1 \mid \hat{p} = p) = p$ — among all the cases where the model says 70%,
about 70% should be true. But that expectation depends on the population you
draw cases from. Two companies can define "spam" identically and still have
different base rates, and Jev will return the same probability to both. A model
can be perfectly calibrated on TypeSafe's evaluation distribution and wrong on
your production distribution, so a vendor-side calibration guarantee is
almost meaningless in practice.

He then shows it's worse than distribution shift:

> There is some evidence the failure is worse than just a distribution shift.
> … Jev says a fair coin lands heads with probability 0.92. That is worse than
> the drift explained above. The true probability is in the prompt, and the
> model still does not report it.

That coin-flip example is the sharpest point in the piece: distribution shift
is at least a subtle excuse, but failing to read a probability *stated in the
prompt* is a plain reasoning failure. And because a different Jev primitive
(`Noul` vs `Choice`) calibrates differently on the same problem, he concludes
the marketing term "calibrated probabilities" doesn't have a stable meaning
within the product itself.

His positive claim is that Jev is still genuinely useful — a universal
classifier you can throw at any problem without collecting training data, which
is exactly what a fine-tuned BERT can't do. The trade is that being useful
without data is precisely *why* its probabilities can't be calibrated for you.
The fix is cheap: a few hundred labeled examples suffice to fit a Platt scaling
on top of Jev's scores.

## Take

This is a model piece of skeptical product criticism. It accepts Jev's most
defensible claim (zero-data classification) and dismantles its most
marketable one with a two-line mathematical argument plus one embarrassing
example. The observation that calibration is distribution-relative is standard
ML orthodoxy that the launch marketing quietly elided — which is why this
matters beyond Jev: any "calibrated by construction" claim about a model
serving many customers deserves the same scrutiny.

The practical guidance is unusually actionable for a critique: scores are
fine for ranking and thresholds you tune empirically; don't feed the raw
number into expected-cost arithmetic or ensemble combination until you've
measured calibration on *your* data. Platt scaling on a few hundred examples
is a day's work.

## Related pages

- [[System One Models and Jev]] — the launch material this essay audits: Molas
  takes TypeSafe's RLCD training and "calibrated probabilities" pitch at face
  value and shows where it breaks, sharpening rather than replacing the
  launch's claims.
- [[Jev's Architecture Unmasked]] — Hume's black-box reconstruction found
  RLCD-trained calibration with ECE 0.0313 *on his probe distribution*; Molas's
  distribution-relativity argument explains why that number doesn't transfer to
  your data, and the finding that `confidence` is arithmetic, not learned,
  rhymes with the Noul/Choice semantic gap.
- [[You Could Have Built Jev]] — both essays are first-principles takedowns of
  Jev marketing from different angles: sgnt.ai attacks the architecture moat,
  Molas attacks the calibration claim; together they suggest the durable value
  is in the product surface, not the model.
- [[LLM Evals]] — Molas's prescription (measure calibration on your own
  distribution before trusting numbers) is a concrete instance of the
  evals-discipline thesis that vendor benchmarks don't transfer to your
  workload.

---
*Sources: [[raw/jev-cant-be-calibrated-html]], [[summary/jev-cant-be-calibrated-html]]*
*Last updated: 2026-09-25*

#concept #ai-research-and-models #guardrails
