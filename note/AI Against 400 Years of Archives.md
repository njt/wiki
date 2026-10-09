# AI Against 400 Years of Archives

Jesse Waites extends Benjamin Breen's dodo discovery into a full agentic research pipeline over the Dutch East India Company archive — and surfaces real historical finds: an unrecorded 1812 meteorite fall in Maharashtra, three Javan rhinos lost en route to the King of Kandy, and three eruptions absent from the Smithsonian's volcano list. The article matters to this wiki less for the history than for its architecture: a tiered delegation chain (cheap judge model first, big model only for survivors) wrapped in validation discipline (positive controls before null results, every claim checked against a scan of the original page).

---

## What Happened

Waites, a software engineer with a home GPU lab, read Breen's account of finding a new 1615 dodo eyewitness record in the GLOBALISE transcriptions and decided to industrialise the approach. A "deep research" assistant proposed thirteen candidate historical mysteries, ranked by whether the data was online, whether AI could help, and — crucially — whether an answer could be checked against an original document. Then Waites and Claude Code designed the pipeline together.

The scale claim is the hook: reading the 4.35 million pages of the Company archive at two minutes a page, eight hours a day, would take about 70 years. The pipeline did it in a twelve-hour overnight run, converting the corpus into 5.7 million passage embeddings — searchable by meaning, not keywords, which matters because 17th-century Dutch spells "rhinoceros" about fifteen different ways.

## Key Quotes

> Having it read 59,000 mentions of elephants cost me about three dollars.

The System One filtering idea — a tiny, fast decision model (Jev) answering narrow questions like "Is this a real animal? Is it wild? Where is it?" — was Waites's own, and he notes Claude Code's Fable model didn't even know what a System One decision model was until he explained it. The human supplied the architectural insight; the agent supplied the plumbing. That is a recurring shape in this wiki's agent-writing.

> Before you trust a search that finds nothing, you have to prove it can find something you already know is there. It's like testing a metal detector by burying your own wristwatch in the front yard.

The positive-control rule is the strongest methodological move in the piece. Before hunting anything new, the pipeline had to rediscover Breen's dodo, Laki 1783, Tambora 1815, and a dozen other known events. Only after the controls passed did null results mean anything — which is why the 1808/09 "Unknown" eruption failure ("the reports just aren't there") is credible rather than merely disappointing.

> The systems could search and read at a scale I couldn't, but deciding where to go next, or when to stop, still took my judgment. This was AI-assisted research, not a completely autonomous investigation.

Waites is explicit about the division of labour: steering, lead-picking, and knowing when to abandon a search stayed human. He also candidly flags that the AI's volcano attributions were often wrong (blaming a Java ash fall on a volcano 1,500 km away), so every candidate had to be re-placed using the letter's own geography.

## Key Themes

- #pattern — tiered delegation: embeddings → cheap judge → mid-tier reader → agent verifier → human against catalogues. Cost falls because only survivors advance.
- #concept — positive controls for retrieval: you cannot interpret a "not found" until you've demonstrated a "found" on known ground truth.
- #tool — home-lab inference (one GPU, 12 hours for 5.7M embeddings) as the enabling substrate; the resulting workflow is being open-sourced as "Antiquity".
- #concept — verification against originals: every claim ends at a scan of the actual handwritten page with an archive reference anyone can read.

## Analysis

This is one of the cleanest demonstrations that agent value comes from pipeline design, not model size. The interesting invention is not any single model but the seat assignment: a near-free judgment model doing triage where a frontier model would be unaffordable, with the frontier model reserved for the few dozen passages that survived. It's the historical-research twin of the routing tables in this wiki's decision-model literature — except applied to 4.35 million pages instead of release-readiness fixtures.

The discipline is what makes it publishable. "AI found X" claims usually die on verification; here every finding is anchored to a National Archives inventory number and a scan, and the finds are framed as candidates pending specialist review. Even the oddities section is quarantined ("treat them as stories, not findings"). That epistemic hygiene — controls, original-source checks, declared human steering, reported failures — is precisely the shape this wiki's guardrails material argues agents need.

One caveat: the article is a personal field report, and its headline finds are still unconfirmed by the catalogue maintainers at time of writing. The confident framing ("once confirmed, this will represent the earliest recorded meteorite fall in Maharashtra") is doing some pre-celebration. But the chain-of-custody framing — an officer's letter, a Bombay reprint, a ship to British-occupied Java, a Dutch library scan, a GPU in an office — is a lovely meditation on archival fragility, and the open-sourcing of the workflow is what turns one lucky dig into a repeatable method.

The deeper point is Waites's closing line: "An archive doesn't give anything up on its own; it only answers the questions someone thinks to ask." Breen searched for dodos and found dodos; Waites searched for meteorites and found meteorites. The question-asking remains the scarce human contribution — the pipeline only industrialises the reading between the question and the answer.

---

*Sources: [[raw/i-pointed-ai-at-400-years-of-archives]], [[summary/i-pointed-ai-at-400-years-of-archives]]*
*Last updated: 2026-10-10*
