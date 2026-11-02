# The Economic Paradox of AI-Generated Software (Shaalan)

Yassien Shaalan argues that AI coding agents have collapsed the cost of *creating* software while leaving untouched the cost of *operating* it — so the build-versus-buy decision has tipped toward building for the wrong reason, and the resulting flood of independently maintained applications creates externalities, rebound consumption, and a migration of the real bottleneck to verification and engineering judgement.

---

## The Argument

Opening with Jeff Atwood's "the best code is no code at all," the essay walks through standard economics applied to vibe coding: transaction costs, externalities, rebound effects, production bottlenecks, real options, and economies of scale. A retailer can now have an AI agent build the order-management app it used to subscribe to — but the commercial platform it replaces was distributing security, maintenance and infrastructure costs across thousands of customers, and a custom rebuild concentrates all of that on one owner.

## Key Quotes

> "AI may have changed the economics of building software without changing the economics of operating it, which means a cheaper implementation does not necessarily produce a cheaper solution."

The thesis in one sentence. The visible cost is generation; the accumulating cost is running it for years.

> "The individual decision to build can be rational while the collective outcome becomes inefficient, particularly when the benefits are local and the costs are distributed across the organisation."

Classic externality framing, with the illustrative arithmetic: 1,000 independently deployed apps at $15/month plus two maintenance hours per quarter approaches $1M/year at $100/hr engineering cost. He is careful to flag these as hypothetical order-of-magnitude figures, not measured averages — an honesty most essays in this genre lack.

> "We may make code generation dramatically cheaper while increasing the total demand for engineering judgement, because every additional system introduces behaviour that somebody must verify and eventually support."

The bottleneck-migration claim, backed by DORA's finding that AI amplifies existing engineering capability rather than improving the delivery system wholesale. He also notes that using a second model to review the first establishes nothing — agreement between models that share assumptions is not correctness.

> "When code becomes abundant, the real advantage may lie not in building more, but in knowing what to build, what to reuse and what doesn't need to exist."

The closing move from economics to taste. Not every problem deserves its own application — but he is careful not to argue for forced centralisation: even a personal tool should justify the resources required to keep it useful.

## Key Themes

#concept #economics #externalities #verification

## Critical Take

This is the most economically literate entry in the "AI generates too much software" genre. Where most essays in this space hand-wave about "slop," Shaalan actually names the mechanism: the externality. The dashboard builder who doesn't pay for infrastructure or security reviews is making a locally rational decision whose costs are distributed across the org — which means no amount of individual taste or better prompting fixes it. Only governance does.

The essay's weakness is the opposite of most: it is almost *too* scrupulous. Every illustrative number is hedged ("order-of-magnitude examples, not measured averages"), every counter-consideration acknowledged (vendor lock-in can justify rebuilds; sharing infrastructure has its own trade-offs). This makes it correct but toothless in the way careful Medium essays often are. The one concrete proposal — the contrast between the org that lets every department build independently and the one that prioritises reusable capabilities before deployment — is stated rather than operationalised. What does that second organisation's review gate actually look like?

The strongest unexplored implication is for the build-versus-buy market itself: if generation is near-free, the sustainable SaaS moat is precisely the *operating* economics — reliability, security, compliance — that generated software cannot reproduce. That connects directly to the argument that SaaS shifts to selling SLAs.

## Related Pages

- [[Why Build vs Buy is the Wrong Question]] — complicates this source: Shaalan treats the build/buy flip as a pricing error, while that note argues the question itself dissolves when agents make both paths continuous rather than binary.
- [[The Cost YAGNI Was Never About]] — strengthens the thesis from a different angle: Beck's options-pricing framing shows cheap generation amplifies the trap of building, exactly the rebound effect Shaalan describes.
- [[Coding Is Not Solved — Alex Ewerlöf]] — the companion diagnosis: creation is cheap, accountability is the real cost, and ownership (Knowledge + Mandate + Accountability) is what a generated app lacks.
- [[AI in the Firm — Bottlenecks in Software Production]] — the same bottleneck-migration argument at organisational scale, which Shaalan's verification-demand claim reinforces.

---
*Sources: [[raw/the-economic-paradox-of-ai-generated-software-93239b25bf1c]], [[summary/the-economic-paradox-of-ai-generated-software-93239b25bf1c]]*
*Last updated: 2026-11-02*
