# Satellite-Grade Traceability in Grist (Dotphoton Webinar)

Bruno Sanguinetti, CTO of Swiss startup Dotphoton, explains in a Grist webinar how his tiny company delivered ESA-grade requirements traceability for a satellite image compressor using Grist, GitHub, LaTeX/Typst, and Docker — achieving a documentation-to-execution ratio of 2:1–3:1 against the 5:1 he saw at his previous Ariane-rocket testing company. His central claim is that Grist is effectively the only tool where this workflow works: Doors is clunky and ruinously priced for a small team, Word produces unreproducible documents, and Notion's exports aren't WYSIWYG.

---

## Key Quotes

> "Someone points to the moon and the idiot looks at the finger. Meaning that you don't want your quality assurance system to become what you focus on. You want to focus on the quality, not the quality assurance."

The sharpest insight in the talk, and one that generalizes far beyond aerospace: formal process is a means that metastasizes into the end. The engineers buried in requirement IDs are the ones who miss the centimeters-vs-inches bug a fresh outsider would spot instantly. The remedy isn't less process but *low-friction* process — deleting a Grist field in two seconds, generating a PDF in ten milliseconds — so the ceremony never becomes the job.

> "And it can't be that way because if it's not fun, no one does it."

A quiet ergonomic argument: traceability fails as a human-compliance problem first, a tooling problem second. Tools that make compliance painful guarantee non-compliance.

> "Sad truth is that we don't know how to make software that doesn't have bugs."

ESA doesn't expect bug-free software; it expects bugs to be found, declared, and traced so a conscious go/no-go decision can be made and fail-safes designed. This is a more honest framing than most commercial QA rhetoric — and notably, it's an *accountability* posture rather than a perfection one, close to what agent-era reviews keep rediscovering.

> "Grist for me is the opposite of SAP."

The business-model contrast: SAP sells consultants; Grist lets you do it yourself, with a commercial backer available when needed. Buying the commercial version wasn't about features but procurement — the tool itself had to have traceable development for space-project approval.

> "If someone sends you the same file, but it was originally done in the French version of Word, it might have something in it that you can't find... it's going to be one more page."

The most vivid failure case: Word's renderer is not a function of the document, so page references — the entire backbone of cross-document traceability — silently rot. The requirement that killed Word/Access was not features but *reproducibility*, which is exactly why versioned text formats (LaTeX, Typst) and a SQLite-backed database win.

## Key Themes

- **#pattern — The repo is the release.** Grist file, LaTeX/Typst source, and code committed into one repository that *becomes* the release; documents and code can never drift apart. This is configuration management done with boring primitives.
- **#pattern — Picasso-bison schema design.** Build the overbuilt version, then delete fields in seconds until only essential lines remain — enabled by Grist's malleability and the download/backup-everything escape hatch.
- **#concept — V model traceability.** One-to-one matching across layers: acceptance tests → needs, tests → requirements, all consistent across versions over a 2021–2023 project.
- **#tool — Grist, Typst, Docker, GitHub.** Toggle-column + REST trigger replaces n8n; Typst renders PDFs in ~10 ms with functions inside templates; local Gemma LLM handles non-critical CRM chores.

## Analysis

The strongest part of this talk is the diagnosis, not the sales pitch. The Word-pagination horror story and the moon-and-finger warning are genuinely transferable lessons: traceability tooling fails when it isn't reproducible and when it isn't painless. The three-generation arc (v1 Python+LaTeX → v2 over-integrated → v3 simpler with Docker/Typst) is an honest engineering history, including the admission that v2 had "maybe too many features."

The weakest part is the "only software" claim, which is asserted repeatedly but never tested: Doors is dismissed on price and clunkiness, while modern dedicated requirements tools (Jama, Polarion, codebeamer) are never mentioned, and no ESA auditor ever appears on screen receiving a Grist-backed package. The 8:1 compression claim — the actual engineering achievement — is never connected to the traceability system; what's demonstrated is cheaper documentation, not better engineering. The claimed 5:1→2:1 improvement is also cross-company, not controlled. And the change-impact analysis confession is the most interesting admission: they *couldn't* model requirement changes in Grist without it getting "too messy and conceptually difficult," so they went meta with hand-written Markdown files — partly by Claude where code isn't critical, because local LLMs "are not very good" and outside LLMs can't receive critical code. A real labor cost, unquantified, and an honest limit of treating a spreadsheet-database as a requirements management system.

Still, as a field report it's valuable precisely because it shows formal QA (usually the most tool-conservative domain) being satisfied by commodity open-source parts plus one well-chosen hybrid — a workflow reproducible by any small team with regulatory obligations.

## Related Pages

This source deepens [[Grist]] with its most demanding documented production use case: ESA-adjacent traceability where SQLite-as-document-format versionability and sub-second copy-paste weren't conveniences but procurement and reproducibility requirements. It converses with [[The Coming Need for Formal Specification]] from the opposite direction — Congdon argues specs must get *more* formal because agents generate unreviewable code, while Dotphoton shows heavy formal process winning when it's made low-friction; both agree the durable artifact is the specification, not the code. And it supplies concrete prior art for [[SLCR Launches]]' machine-readable requirements format: both treat requirements as queryable, linked data (requirement IDs, verification methods, traced test status) rather than prose documents, though Dotphoton's V-model framing predates the agent-driven variants.

---
*Sources: [[raw/33b7c11c97b2bef00cd06fca032b2445]], [[summary/33b7c11c97b2bef00cd06fca032b2445]]*
*Last updated: 2026-11-03*
