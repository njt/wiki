# AI Team Mistakes

Doug Turnbull's field report from a dozen-plus failing AI teams: the mistakes are structural, not superficial, and they rhyme with what search teams learned the hard way in the 2010s. Four lessons — evals before building, retrieval as the product, context as metadata rather than chunks, and multidisciplinary teams — form a practitioner's checklist for not repeating history.

---

## Key Quotes

> "Great search/AI organizations spend ~50% of their investment on understanding the problem, not solving it."

Turnbull's opening thesis and the whole piece in miniature. The half-you-don't-see is evals, not models. This is the same 50%-on-understanding number that the broader eval literature keeps converging on, stated from the search side.

> "I learned to actively NOT trust my instincts."

The epistemic move underneath every other lesson. His Advanced Auto Parts anecdote makes it concrete: he assumed employees searching for a product wanted that product ranked first, when what they actually wanted was "what will I get an incentive for selling?" The scientist's job — evaluate, hypothesize, test, improve — is the antidote to instinct.

> "We need to recognize that AI teams are search teams. One of the easiest findings out there in research: **retrieval dictates AI quality**."

The boldest claim in the piece. If retrieval quality is the dominant lever on answer quality, then treating retrieval as a checkbox to fill after the model is picked is exactly backwards. It's a one-sentence reframing that reorders team priorities.

> "RAG is about helping the implicit judge inside the LLM make better decisions. It's not about arguing how / where to exactly split articles up into paragraphs."

The "context means metadata, not chunks" thesis, compressed. A chunk that carries title, popularity, and publication date lets the LLM weigh recency and trustworthiness; a bare passage stripped of provenance can't. Turnbull names this a third, hidden pillar of retrieval — query understanding and metadata — sitting alongside keyword and embeddings.

> "You need minds that can fit both perspectives into one brain to make minute-to-minute tradeoffs when building. Not data science throwing models over the wall, wait 3 months once built out, only to realize it's the wrong model."

The organizational diagnosis. The wall-throw failure mode is why siloed engineering + data science produces AI teams that ship the wrong thing slowly. The fix is hiring and cultivating people who hold both scalable-systems and hypothesis-testing instincts at once.

## Key Themes

- **#pattern** — Evals first: half the budget on understanding the problem; don't trust PM opinions or your own instincts — measure.
- **#concept** — AI teams are search teams: retrieval dictates AI quality, so retrieval is the product, not an integration checkbox.
- **#concept** — Context = metadata + provenance, not chunks. Structured domain metadata is a third retrieval pillar alongside keyword and embeddings.
- **#pattern** — Multidisciplinary teams: engineering + data science in one brain beats throwing models over the wall.

## Critical Analysis

Turnbull is upfront about his bias — "like the cop that only sees the hard, rough side of life in the streets" — and it's a productive one. The 2010s-search-history framing earns its keep because it gives each lesson a precedent: the eval-first discipline, the sunk-cost trap of over-building retrieval before experimenting, the chunk-vs-metadata distinction. When a search veteran says AI teams are search teams, it's not a hot take, it's pattern recognition from the last cycle of the same failure.

The metadata-over-chunks argument is the strongest and most original section. It extends the "cool story bro" problem from his [[Three Kinds of Agentic Search]] piece — where context-free chunks are invisible to agents trained on web-scale results — into a positive prescription: represent a unit of information *and its provenance* so the LLM's implicit judge can decide whether to trust it. That's a concrete, actionable reframing of RAG that most teams still haven't internalized, and it lands harder than "chunk better."

The eval section is right but comparatively thin. Turnbull points to Hussain and Shankar's course and his own Quepid, but doesn't give the calibration detail that [[Eval-Driven Development (Airbnb)]] or [[The Lifecycle of LLM-as-a-Judge]] do. He's arguing from search-team scar tissue, not a measurement methodology. Fair enough for a blog post, but a team reading this wanting to *act* will still need the eval playbook from elsewhere.

The multidisciplinary point is the most underdeveloped. "Both perspectives in one brain" is a hiring heuristic, not a plan. It gestures at the real organizational problem — that AI work doesn't decompose cleanly into engineering vs. data science — without the mechanism for how to build that hybrid capacity, which is exactly the gap a piece like [[Building Reliable Agentic AI Systems]] fills from the production side.

What's conspicuously absent is cost and safety. For a piece about teams colliding with reality, there's no mention of token economics, guardrails, or the failure modes that dominate [[Guardrails and Feedback Loops]]. That's a scope choice, not an oversight — Turnbull is writing about *why teams fail at the problem*, not *what breaks once it ships* — but it bounds the piece's usefulness.

## Connections

- [[Three Kinds of Agentic Search]] — Same author, same underlying thesis. This piece is the team-level prequel: where that one disentangles *which* retrieval strategy to pick, this one argues retrieval is the thing AI teams under-invest in at all. The "metadata not chunks" section is the direct generalization of the "cool story bro" content-purpose mismatch.
- [[Context Engineering at the Frontier (Linus Lee)]] — Lee's "context engineering IS search engineering" is Turnbull's "AI teams are search teams" from the agent side. Turnbull's metadata-as-provenance is Lee's write-time pre-structuring made concrete for RAG.
- [[Building Reliable Agentic AI Systems]] — Bayer's PRINCE is the production proof of Turnbull's thesis: its 8-step pipeline literally includes a "metadata filter generation" step, and its data-sufficiency reflection loop is eval discipline applied at retrieval time.
- [[Eval-Driven Development (Airbnb)]] — The operational recipe behind Turnbull's "50% on understanding the problem." Airbnb turns the eval-first instinct into a calibration methodology Turnbull only gestures at.

---

*Sources: [[raw/ai-team-mistakes-html]], [[summary/ai-team-mistakes-html]]*
*Last updated: 2026-09-04*
