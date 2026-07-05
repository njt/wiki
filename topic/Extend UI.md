# Extend UI

Open source React component library for document-heavy applications: PDF, DOCX, XLSX, and CSV viewers with bounding box citations, document splitting, e-signing, and schema builders. 560 GitHub stars. Created by Extend AI. Targets the gap between expensive commercial document SDKs (PSPDFKit/Nutrient) and bare PDF.js wrappers — document components complex enough to be worth paying for, with an open-source core for distribution.

---

## What's In the Box

Seven core components:

- **PDF Viewer** — render and interact with PDFs in-browser
- **DOCX Viewer** — Word document rendering
- **XLSX Viewer** — spreadsheet rendering
- **File Thumbnail** — preview cards for Image, PDF, DOCX, and XLSX files
- **Document Splits** — partition documents into named sections with page ranges (e.g., "Abstract: pages 1–3, Model Architecture: pages 4–7")
- **File System** — file browser interface
- **Schema Builder** — multi-level JSON schema editor with type configuration (String, Number, Enum, Object, Array)

Plus seven composed "layout blocks" that combine primitives into full-page patterns: bounding box citations, PDF dropzone, DOCX editor, e-signature capture.

## Key Quotes

> "React components for PDF, DOCX, XLSX, and CSV viewers, with bounding box citations, file upload, e-signing, and more."

The pitch is exhaustive and specific — it names the formats, the interactions, the use cases. This isn't "building blocks for the future of documents." It's "here are the seven things your document app needs, they work, drop them in."

> "Ready to drop into user-facing flows, agents, or internal tools."

Three audiences in one sentence. "Agents" is the tell — Extend sees coding agents as a distribution channel. If an agent is building a document processing tool, it should reach for Extend UI components. This is [[10 Principles for Agent-Native CLIs]] applied to the frontend: design for agents as a primary consumer.

## Key Themes

- **#tool** Document UI as unsolved problem — PDF.js has been around for over a decade, but document *interaction* (splitting, citing, signing, schema-building) still requires either expensive commercial SDKs or building from scratch. Extend UI is betting that open-source components can capture the middle ground.

- **#pattern** Componentization of complex interactions — bounding box citations and e-signature aren't just rendering problems; they're interaction design problems. Packaging them as drop-in React components makes them accessible to teams that would never build them from scratch.

- **#pattern** The agent-native frontend — positioning the library for "agents" as a user persona is novel. Most UI libraries target human developers; Extend UI explicitly targets AI coding agents as consumers. If agents are going to build document apps, they need document components.

- **#concept** Open-source wedge strategy — the core viewers are open source; the commercial product (Extend) presumably layers on top. Same playbook as every dev-tool company since MongoDB: open-source for adoption, paid for production.

## Critical Analysis

**The gap is real.** Document viewing in web apps is a genuinely unsolved problem. PDF.js gives you rendering but not interaction. Commercial SDKs like PSPDFKit (now Nutrient) give you everything but cost $thousands/year. An open-source middle layer with good defaults is long overdue. Extend UI is the first credible attempt at this that I've seen.

**The component selection reveals product taste.** Anyone building a document app needs viewers first. But the inclusion of Document Splits, Schema Builder, and Bounding Box Citations shows they've actually built document apps before — these are the components you discover you need after your first real deployment, not the ones you imagine you'll need from a blank page.

**The agent positioning is either prescient or premature.** Coding agents building document UIs from components makes sense in theory, but agents today mostly write code, not compose component libraries. However, the [[Agent-Native Architectures (Every)]] principle applies: build for agents and humans benefit. If the components are well-documented and predictable enough for an agent to use, they're well-documented and predictable enough for a human.

**560 stars is modest but honest for a specialist library.** General-purpose UI libraries collect stars from everyone. A document-specific library only attracts people who actually need document components. The star count reflects market size more than quality.

**The commercial question.** Extend AI presumably sells something (the components page links to `extend.ai` which is a commercial document AI product). The library is a funnel: get developers using the free components, then sell them the AI-powered document processing. This is a better strategy than most — the components are genuinely useful even without the paid product.

**What's missing:** CSV viewer is mentioned in the tagline but not listed as a standalone component. No mention of accessibility (screen readers, keyboard navigation for document interactions). No mobile/responsive behavior documented. These gaps matter for real deployment but are typical for an early-stage library.

**The comparison to [[json-render]] is instructive.** Both are React component libraries that want to be the substrate for AI-generated UIs. json-render constrains AI output to a Zod schema of pre-approved components; Extend UI provides the components that an AI would want to use when building document tools. They're complementary: json-render is the harness, Extend UI is the payload.

**The [[Lessons for Reusable Web Components]] test:** De Pietro's five rules — namespace everything, CSS variables as public API, trust platform features, publish over paste, document or it didn't happen. Extend UI is React-only (not web components), which limits reuse outside the React ecosystem. But within that ecosystem, the component API design appears clean.

---

## Cross-Links

- [[json-render]] — complementary: json-render is the AI→UI pipeline, Extend UI provides the document components that pipeline would emit
- [[Lessons for Reusable Web Components]] — De Pietro's rules for publishing components; Extend UI is React-only but the API surface philosophy aligns
- [[10 Principles for Agent-Native CLIs]] — the "design for agents" positioning applied to frontend components
- [[Agent-Native Architectures (Every)]] — files as universal interface; Extend UI's document components are building blocks for agent-built document apps
- [[Kreuzberg]] — polyglot document parsing (97+ formats); Extend UI handles the rendering side of the same document pipeline
- [[Dolphin]] — ByteDance's universal document parsing model; the parsing counterpart to Extend UI's rendering
- [[markitdown]] — Microsoft's document-to-Markdown converter; another tool in the document processing ecosystem
- [[Performative UI]] — the anti-Extend: satirical components that signal AI-ness vs. practical components that solve real document problems
- [[smui]] — shows the shadcn/ui ecosystem Extend UI participates in

---
*Sources: [[summary/extend-ui]]*
*Last updated: 2026-06-11*
