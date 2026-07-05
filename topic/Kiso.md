# Kiso

Kiso is an Apache 2.0-licensed static site generator that converts **Open Knowledge Format (OKF)** bundles into static websites designed for both human readers and AI agents. Built in Java by oak-invest, it takes a directory of structured Markdown with explicit metadata and produces HTML pages, an `llms.txt` file, and a `sitemap.xml` — with each page retaining a link back to its Markdown source. A GitHub Action enables CI/CD publishing to GitHub Pages.

## Key Quotes

> "Turns Open Knowledge Format (OKF) bundles into static websites for humans and AI agents."

Kiso's pitch is two audiences in one sentence. The "for humans and AI agents" framing is the interesting bit — it's not just a static site generator, it's a bridge between the Markdown source-of-truth that humans edit and the structured output that agents consume.

> "Pages keep enough context to be validated, linked, and rendered consistently."

The metadata-first approach is the architectural bet. Rather than inferring structure from Markdown conventions (the Jekyll/Hugo approach), Kiso demands explicit metadata per the OKF spec. This makes the bundle self-describing and machine-validatable — you can't generate a broken site from a malformed bundle because the tool knows what shape the data should have.

> "Agent friendly — The generated HTML keeps clear links back to the original Markdown files."

The backlink to source is the key design decision that separates Kiso from every other static site generator. Most SSGs treat the Markdown as a build artifact you discard after rendering. Kiso treats it as the durable source that the rendered page must remain connected to — which is exactly the pattern [[Agent-Native Architectures (Every)]] describes as "parity."

## Key Themes

- #tool — CLI + GitHub Action for building static sites from OKF bundles
- #pattern — Markdown as source of truth with backlinks from rendered output
- #pattern — Dual-audience publishing: human-readable HTML + agent-consumable llms.txt/sitemap.xml
- #concept — OKF (Open Knowledge Format): Google Cloud Platform's spec for structured knowledge bundles

## Critical Analysis

Kiso is small (122 commits, 11 stars, v0.1.2) and the OKF ecosystem it targets is even smaller. This isn't a competitor to Hugo or Jekyll — it's a purpose-built tool for a specific format that Google Cloud Platform defined. The bet is that OKF will matter, and Kiso will be the go-to renderer when it does.

The dual-audience design is genuinely smart. Most SSG output is hostile to AI agents — generated HTML with no structural metadata, no machine-readable index, no backlinks to source. Kiso's `llms.txt` + `sitemap.xml` + source backlinks make it trivial for an agent to navigate the rendered site and trace claims back to their Markdown origin. This is the publishing equivalent of [[10 Principles for Agent-Native CLIs]]: design for agents first, and humans benefit from the same structural clarity.

The risk is that OKF never escapes Google's orbit. The spec lives in the `knowledge-catalog` repo under GoogleCloudPlatform, and there's no visible community around it. If OKF stays a Google-internal format with one external renderer, Kiso is a well-built tool for a party nobody showed up to.

That said, the pattern it demonstrates — structured Markdown → dual-audience output with source provenance — is worth stealing regardless of OKF's fate. Any documentation site could adopt this model.

## Related

- [[Agent-Native Architectures (Every)]] — Parity principle: rendered output must stay connected to editable source
- [[10 Principles for Agent-Native CLIs]] — Design for agents first; Kiso applies this to publishing
- [[How AI Coding Agents Actually Use Your Technology]] — The invisible failures when agents can't navigate your output

*Source: [oak-invest.github.io/kiso](https://oak-invest.github.io/kiso/), fetched 2026-07-03*
