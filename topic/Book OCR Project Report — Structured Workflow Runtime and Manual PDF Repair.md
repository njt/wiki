# Book OCR Project Report — Structured Workflow Runtime and Manual PDF Repair

A 202-page scanned technical book becomes the validation workload for a durable workflow runtime, evolving from freeform Markdown OCR into a production system where every page has raw model evidence, structured JSON, rendered Markdown, validation metadata, and a reproducible PDF. The report is a masterclass in model-output engineering: where to draw the boundary between AI generation and deterministic code, and how to build systems that can explain *how* they produced their output.

---

## Precis

Manuel (go-go-golems) documents the full arc of a Book OCR project: a generic Go workflow runtime validated against a real workload, the diagnosis of neighboring-page context bleed in freeform OCR, the move to target-page-only structured JSON, workflow-backed parallel execution across 202 pages, figure extraction, PDF rendering, and a day-long manual repair loop that found and fixed five distinct failure classes. The project's most important result isn't the OCR'd PDF — it's the surrounding system that can explain how that PDF was produced and can repair selected pages without discarding the rest of the run.

## Key Quotes

> The project is not only an OCR pipeline. It is an engineering exercise in turning model calls into durable, inspectable, recoverable production work.

This is the thesis. The specific domain (OCR) is almost incidental — the report is really about what happens when you take model calls seriously as production infrastructure.

> Markdown is too broad as a model output contract. It mixes recognition, interpretation, layout, and rendering. A structured JSON boundary lets Go own deterministic rendering and validation.

The central architectural decision. Instead of asking the model to write final Markdown, it returns `StructuredPageOCR` JSON with typed blocks (heading, paragraph, list, table, code, figure, footnote, page_footer, blank). Go parses, repairs limited schema drift, validates, and renders deterministically.

> The freeform run used `--context-window 1`, and neighboring context meant neighboring page PNG images. The model was instructed to treat the first image as the target page and neighboring images as context only. It still copied adjacent visual content into the target page output.

The neighboring-page context bleed problem that forced the architectural correction. A concrete example: pages 12 and 13 — page 12 is prose referencing Figure 1-1, page 13 contains the actual diagram, and the freeform output produced a false page 12 figure crop. **Page provenance is a hard requirement.**

> A benchmark can show that a model behaved correctly on a selected set of cases. It does not make neighboring image context safe as the primary production transcription path.

On why the VLM separation benchmark (which showed no forbidden-caption bleed) didn't change production policy. Diagnostic calls may test context separation; production calls must not use neighboring page images.

> The parser had to accept recurring variants without losing provenance or inventing content. The repairs are intentionally limited. They make known structural variants parseable, but they do not manufacture new OCR text.

The schema drift repair policy: accept common shape drift, repair structure when text is already present, never silently invent content. Observed variants included `page_number` as string, `diagram_text` as string instead of array, list items as strings, figure captions as heading blocks.

> The page JSON was reasonable. The page rendered Markdown was reasonable. The final embedded Markdown was wrong because a later post-processing heuristic was too aggressive.

On the spreadsheet-figure duplication bug — the second-order failure that only appeared in the final artifact. The quality figure embedding pass saw captions and synthesized missing figure markers, producing image links for content already rendered as tables.

> Targeted repair is not just a matter of changing status flags. It must preserve dependency semantics.

The workflow race condition discovered during manual repair: setting downstream assemble/validate ops to `ready` at the same time as page ops allowed assembly to run before rerun pages finished. Fix: set downstream ops to `pending`, let the scheduler release them when dependencies succeed.

> The important review discipline is to inspect the right stage. If a page is wrong in `04-structured.json`, the model/prompt/page call needs repair. If it is right in `04-structured.json` but wrong in `05-rendered.md`, the renderer needs repair. If it is right in `05-rendered.md` but wrong in `embedded-figures.md`, the figure embedding pass needs repair. If it is right in Markdown but wrong in PDF, the Pandoc/LaTeX rendering path needs repair.

The artifact-inspection discipline. Each stage has a defined boundary and a defined owner. This is the operational equivalent of "make the easy change hard" — force the diagnosis to identify the right layer before applying a fix.

## Key Themes

- **#concept** Model-output boundary engineering: where to stop asking the AI and start using deterministic code. Structured JSON as the contract, Go as the rendering/validation owner.
- **#concept** Stage-aware artifact inspection: raw response → structured JSON → rendered Markdown → embedded-figures Markdown → PDF. Each stage has a defined owner for debugging.
- **#concept** Targeted repair: the system must support reprocessing selected pages without rerunning the whole book. This is not an optimization — it's a requirement for practical manual review loops.
- **#concept** Workflow state vs. domain projection state: the runtime knows whether an op succeeded; the projection knows *what the work means* in the OCR domain (page number, figure count, table count, warnings).
- **#tool** Geppetto/Pinocchio: vision model client and turn store. Target-page-only image input, persisted turns for replay/debug.
- **#tool** `scraper/pkg/workflow`: Go workflow runtime with durable execution, steps, queues, leases, retries, artifacts, projections, and operator controls.
- **#pattern** Seven engineering rules: target-page-only OCR, structured model boundaries, preserve raw evidence before parsing, separate workflow/projection state, render PDF from workflow, support targeted repair, encode repeated manual findings as validation.
- **#pattern** "Every repeated manual observation should become a deterministic report" — the audit-tooling stage that comes after the architecture is sound.
- **#person** Manuel (go-go-golems/wesen): author of the Book OCR project, scraper workflow runtime, and this report. Works in Go, uses Geppetto/Pinocchio for AI model calls.

## Critical Analysis

**What makes this report exceptional:** It's a rare document that shows the *full* engineering loop — not just the architecture diagrams and the happy path, but the manual PDF review session that found five distinct failure classes, the diagnosis at each artifact stage, the targeted fixes, the workflow race condition, and the second-order post-processing bugs. Most project reports stop at "it works." This one documents what was wrong, how it was found, and how the system was designed to make those findings repairable.

**The insight that generalizes beyond OCR:** The core pattern — model → structured JSON → deterministic render → validation → manual review → targeted repair — is applicable to any system where AI produces content that humans need to trust. The specific domain is OCR, but the engineering pattern is the same one behind [[Harness Engineering]], [[Structural Backpressure Beats Smarter Agents]], and [[Compound Engineering]]. The structured boundary is the moment where probability becomes determinism.

**What's missing:** The report is strong on the *how* but lighter on the *why now*. It doesn't address why this approach wasn't taken from the start — was the freeform path a necessary exploration, or a mistake? The freeform pipeline produced a useful artifact and proved the end-to-end shape, which suggests it was productive waste, but the report doesn't examine that question. Also missing: cost data. How many model calls? What did the full run cost? The report mentions "expensive" but never quantifies.

**The workflow runtime question:** Building a custom workflow runtime is a significant engineering investment. The report makes the case for why it was necessary (long-running, failure-prone, needs retry, needs artifacts, needs projections), but doesn't compare against existing options (Temporal, Prefect, Dagster, even a well-structured Makefile with state files). The runtime is now a reusable asset across projects, which is the classic platform-investment payoff, but the build-vs-buy trade-off is unexplored.

**The seven rules are the real artifact:** More than the PDF or the code, the seven engineering rules at the end of the report are the durable output. They're hard-won, specific, and falsifiable. Rule 3 ("preserve raw evidence before parsing") alone would prevent entire classes of AI-integration failures. Rule 7 ("validation should encode every repeated manual finding") is the operational version of "don't make the same mistake twice."

## Related Pages

- [[Harness Engineering]] — The theoretical framework: feedforward vs. feedback, computational vs. inferential. This project is a worked example of harness engineering applied to document processing.
- [[Harness Engineering (OpenAI)]] — The original field report: 1M lines with zero handwritten code. Same pattern of structured boundaries and deterministic validation.
- [[Structural Backpressure Beats Smarter Agents]] — Behavioral gates vs. structural gates. The structured JSON boundary is a structural gate.
- [[Compound Engineering]] — When you can't trust the output, add a system. This project added a workflow runtime, structured contracts, validation, and audit tooling.
- [[Don't Fear the Dark Factory]] — The dark factory is a validation problem, not a generation problem. The manual PDF repair loop is validation in practice.
- [[The Dark Factory is a DOT File]] — The pipeline DOT file is the valuable artifact; factory code is disposable. The workflow graph here is the DOT file equivalent.
- [[Feedback Loop is All You Need]] — Linters beat prompts. The validation gates and code-fence audit are linters for OCR output.
- [[Guardrails and Feedback Loops]] — Deterministic enforcement, not instructions. Every validation check is a guardrail.
- [[Smart Models Dumb Pipes]] — LLMs as judgment machines, not Q&A machines. The model returns structured JSON; dumb pipes render it.
- [[StrongDM Factory Techniques]] — Filesystem-as-memory, pipeline patterns. The per-page artifact directory is a filesystem-as-memory pattern.
- [[Probabilistic Engineering and the 24-7 Employee]] — The deterministic contract is broken. Structured boundaries are the fix.
- [[If AI Is Doing the Investigation, Version the Investigation]] — Preserve raw evidence before parsing. Rule 3 of this project is the same principle.
- [[Scaling Long-Running Agents]] — Long-running, failure-prone operations. The workflow runtime is a solution to this class of problem.
- [[Specifications as the Product]] — The structured JSON contract is the specification; the rendered Markdown and PDF are disposable outputs.
- [[Correct by Construction]] — Data quality as a whitelist. The structured OCR schema is a whitelist for model output.
- [[Slowing the Fuck Down]] — Deliberate friction. The manual PDF review loop is deliberate friction applied to model output.
- [[Dolphin]] — ByteDance's document parsing model. A different approach to the same problem space.
- [[Kreuzberg]] — Document intelligence in Rust. Another document processing tool in the ecosystem.
- [[What You NEED to Know Before Touching a Video File]] — Another deep-dive technical craft piece that explains *how* rather than just *what*.

---
*Sources: [[summary/book-ocr-project-report-structured-workflow]]*
*Fetched via surf browser automation (JS-rendered Obsidian Publish). WebFetch returned site title only.*
*Last updated: 2026-05-31*
